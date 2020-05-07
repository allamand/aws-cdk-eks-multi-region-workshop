---
title: 두 번째 리전에 배포하기
weight: 200
pre: "<b>5-2. </b>"
---

4-3 단계까지, 한 리전에 EKS 클러스터를 배포하고 이 위에 쿠버네티스 자원을 배포했습니다.  
그렇다면 이와 동일한 구성을 두 번째 리전에 배포하려면 어떻게 해야 할까요?

## 배포할 두 번째 리전 지정
`bin/multi-cluster-ts.ts` 파일을 열어 두 번째 스택을 만들어보겠습니다.
지금은 아래와 같이 엔트리포인트에 스택이 로드된 상태일 겁니다.
```typescript
const app = new cdk.App();

const account = app.node.tryGetContext('account') || process.env.CDK_INTEG_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT;
const primaryRegion = {account: account, region: 'ap-northeast-1'};
const secondaryRegion = {account: account, region: 'us-east-1'};


const primaryCluster = new ClusterStack(app, `ClusterStack-${primaryRegion.region}`, {env: primaryRegion });
new ContainerStack(app, `ContainerStack-${primaryRegion.region}`, {env: primaryRegion, cluster: primaryCluster.cluster });


```

여기에 아래 코드를 붙여 넣어, 같은 스택을 다른 리전으로 배포하도록 구성해보겠습니다.
```typescript
const secondaryCluster = new ClusterStack(app, `ClusterStack-${secondaryRegion.region}`, {env: secondaryRegion });
new ContainerStack(app, `ContainerStack-${secondaryRegion.region}`, {env: secondaryRegion, cluster: secondaryCluster.cluster });
```

## 스택 배포하기
아래 명령어를 이용하면 두 번째 리전에 스택이 새롭게 생성된 것을 볼 수 있습니다.  
이 스택에 의해 어떤 자원이 배포될 것인지를 확인할 수 있습니다.

```
cdk diff
```

앞서 4-2, 4-3 단계에서 생성한 자원과 동일한 출력값을 확인할 수 있습니다.

아래 명령어를 이용해 자원을 두 번째 리전에 배포하십시오.
```
cdk deploy "*" --require-approval never
```
약 15-20분 정도의 시간이 소요됩니다.  
위 명령어는 워크샵 단계 간소화를 위해 IAM 등 보안 관련 자원 수정에 대한 동의를 생략하도록 했음을 유의하십시오.


## kubeconfig 업데이트
자원 생성이 완료되고 나면, 콘솔에 CloudFormation Output으로 ConfigCommand가 출력될 것입니다.
아래 출력값의 `=` 뒤의 `aws eks ...` 부분을 복사하여 콘솔에서 실행하십시오.  

```
ClusterStack-us-east-1.demogoclusterConfigCommand6DB6D889 = aws eks update-kubeconfig --name demogo --region ap-northeast-1 --role-arn <<YOUR_ROLE_ARN>>
```

정상적으로 수행되면 아래와 같은 결과가 출력될 것입니다.

```
Updated context arn:aws:eks:ap-northeast-1:<<ACCOUNT_ID>>:cluster/demogo in /Users/jiwony/.kube/config
```


아래 명령어를 통해 두 개 리전의 EKS 클러스터에 접근할 수 있음을 확인합니다.
```
kubectl config get-contexts

# 결과값
CURRENT   NAME                                                     CLUSTER                                                  AUTHINFO                                                 NAMESPACE
          arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo   arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo   arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo
*         arn:aws:eks:us-east-1:ACCOUNT_ID:cluster/demogo        arn:aws:eks:us-east-1:ACCOUNT_ID:cluster/demogo        arn:aws:eks:us-east-1:ACCOUNT_ID:cluster/demogo
```


## 클러스터 내 자원 확인
현재 등록된 컨텍스트에서 `kubectl get pod` 명령어를 통해 현재 배포된 pod를 확인합니다.  
`us-east-1` 리전이기 때문에, [도쿄 리전에 배포한 결과](/ko/40-deploy-clusters/300-container/320-resource/#배포하기)와 다르게 `yaml-us-east-1/00_us_nginx.yaml` 파일의 내용처럼 nginx Pod가 5개 배포된 것을 볼 수 있습니다.  

```
NAME                                READY   STATUS    RESTARTS   AGE
metrics-server-6b6bbf4668-xhvtd     1/1     Running   0          12m
nginx-deployment-5754944d6c-6lqmg   1/1     Running   0          12m
nginx-deployment-5754944d6c-6qn8f   1/1     Running   0          12m
nginx-deployment-5754944d6c-d25s6   1/1     Running   0          12m
nginx-deployment-5754944d6c-dnl9l   1/1     Running   0          12m
nginx-deployment-5754944d6c-q2p9c   1/1     Running   0          12m
```


필요한 경우, 아래 명령어를 통해 `kubectl` 조작을 할 대상 클러스터를 변경할 수 있습니다.

```
kubectl config use-context <<cluster-name>>

# 결과값
Switched to context "<<cluster-name>>".
```