---
title: 리전 별로 인프라 설정 변경하기
weight: 300
pre: "<b>5-3. </b>"
---

그런데, 이런 경우가 있을 수 있죠.  
A 리전에는 1번 컨테이너를 추가로 두고, B 리전에는 2번 컨테이너를 두고.  
혹은 클러스터 자체 설정을 달리하고 싶을 수 있습니다. 예를 들어 A 리전 워크노드로는 R5.xlarge 인스턴스를 쓰는데, B 리전은 M5.xlarge를 써야 할 때.  
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

1. EC2 타입 정의에 필요한 패키지를 import 합니다.
```typescript
import * as ec2 from '@aws-cdk/aws-ec2';
```

2. 클래스 선언 내부에 `primaryRegion` 정보를 주입합니다.

1) `bin/multi-cluster-ts.ts`로 이동하여 코드 가장 하단에 아래 코드를 붙여넣습니다.
```typescript
export { primaryRegion }
```

2) `lib/cluster-stack.ts`로 돌아와 아래 코드를 최상단에 붙여 넣어 이 값을 import 합니다.

```typescript
import { primaryRegion } from '../bin/multi-cluster-ts'

```

3. `eks.Cluster` 생성 부분에 다음 코드를 붙여넣습니다.

```typescript
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion.region? 
                                 new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.large')
```

완성된 코드는 다음과 같을 것입니다.
```typescript
      const primaryRegion = 'ap-northeast-1';

      const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        defaultCapacity: 2,
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                 new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.xlarge')
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

작업이 완료되면 아래 스크린샷과 같이 인스턴스 타입이 각 리전에 맞추어 변경된 것을 확인할 수 있습니다.

![](/images/20-deploy-clusters/ap-infra-change.png)
