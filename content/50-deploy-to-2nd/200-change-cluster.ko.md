---
title: 리전 별로 인프라 설정 변경하기
weight: 200
pre: "<b>5-2. </b>"
---

그런데, 이런 경우가 있을 수 있죠.  
A 리전에는 1번 컨테이너를 추가로 두고, B 리전에는 2번 컨테이너를 두고.  
혹은 클러스터 자체 설정을 달리하고 싶을 수 있습니다. 예를 들어 A 리전 워크노드로는 M5.xlarge 인스턴스를 쓰는데, B 리전은 M5.2xlarge를 써야 할 때.  
이렇게 리전에 따라 다른 설정을 주고 싶다면 어떻게 해야 할까요?  
다음 예시를 통해 살펴봅시다.


## 리전에 따라 다른 인스턴스 타입을 갖도록 변경하기
`lib/cluster-stack.ts`에서 클러스터를 정의했던 부분을 살펴봅시다.  
지금 우리가 배포한 EKS 클러스터는 아래와 같이 정의되어 있을 것입니다.  

```typescript

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        defaultCapacity: 2
    });

```

이 부분을 아래와 같이, 스택이 실행되는 리전 값을 참조하여 다른 인스턴스 타입으로 배포되도록 해보죠.
```typescript
      const primaryRegion = 'ap-northest-1';

      const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        defaultCapacity: 2,
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                 new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.large')
      });
```

`cdk.Stack.of(this).region`으로 해당 스택이 실행되는 리전을 조회하여, 이용할 인스턴스 타입을 변경하였습니다.  
일반적인 프로그래밍을 하듯이, 내가 원하는 리전인지 확인하고 이에 따라 다른 값이 들어가도록할 수 있는 것입니다.  
`cdk diff` 명령어를 활용해 변경될 내용을 조회해보면 다음과 같을 것입니다.  

```
Stack ClusterStack-us-east-1
Resources
[~] AWS::AutoScaling::LaunchConfiguration demogo-cluster/DefaultCapacity/LaunchConfig demogoclusterDefaultCapacityLaunchConfig93D71520 replace
 └─ [~] InstanceType (requires replacement)
     ├─ [-] m5.large
     └─ [+] m5.xlarge

Stack ContainerStack-ap-northeast-1
There were no differences
Stack ContainerStack-us-east-1
There were no differences
Stack ClusterStack-ap-northeast-1
Resources
[~] AWS::AutoScaling::LaunchConfiguration demogo-cluster/DefaultCapacity/LaunchConfig demogoclusterDefaultCapacityLaunchConfig93D71520 replace
 └─ [~] InstanceType (requires replacement)
     ├─ [-] m5.large
     └─ [+] r5.xlarge
```

## 자원 생성 확인하기
`cdk deploy "*"`로 해당 스택을 배포해보면, 아래와 같이 인스턴스를 교체하고 AutoScaling을 위한 Launch Configuration을 변경하는 것을 볼 수 있습니다.

```
...
  2/28 | 오후 10:44:50 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::LaunchConfiguration | demogo-cluster/DefaultCapacity/LaunchConfig (demogoclusterDefaultCapacityLaunchConfig93D71520) Resource creation Initiated
  3/28 | 오후 10:44:50 | UPDATE_COMPLETE      | AWS::AutoScaling::LaunchConfiguration | demogo-cluster/DefaultCapacity/LaunchConfig (demogoclusterDefaultCapacityLaunchConfig93D71520)
  3/28 | 오후 10:44:58 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F)
  3/28 | 오후 10:45:03 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Rolling update initiated. Terminating 2 obsolete instance(s) in batches of 1.
  3/28 | 오후 10:45:04 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Terminating instance(s) [i-05a08f1adf2d546ab]; replacing with 1 new instance(s).
 3/28 Currently in progress: demogoclusterDefaultCapacityASGCD1D544F
  3/28 | 오후 10:45:57 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Successfully terminated instance(s) [i-05a08f1adf2d546ab] (Progress 50%).
  3/28 | 오후 10:45:57 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Terminating instance(s) [i-03a198d7d12a8b79b]; replacing with 1 new instance(s).
  ...

```

