---
title: 두 번째 리전에 배포하기
weight: 400
pre: "<b>4-4. </b>"
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
cdk deploy
```
약 15-20분 정도의 시간이 소요됩니다.

## kubeconfig 업데이트
자원 생성이 완료되고 나면, 콘솔에 CloudFormation Output으로 ConfigCommand가 출력될 것입니다.
이 명령어를 복사하여 콘솔에서 실행하십시오.  
이를 통해 우리가 방금 생성한 EKS 클러스터에 `kubectl`을 이용하여 자원을 조회/생성/수정/삭제할 수 있습니다.

```
ClusterStack-us-east-1.demogoclusterConfigCommand6DB6D889 = aws eks update-kubeconfig --name demogo --region us-east-1 --role-arn <<YOUR_ROLE_ARN>>
```

위 명령어를 실행하고 나면 아래 명령어를 통해 두 개 리전의 EKS 클러스터에 접근할 수 있음을 확인합니다.
```
kubectl config get-contexts

# 결과값
CURRENT   NAME                                                     CLUSTER                                                  AUTHINFO                                                 NAMESPACE
          arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo   arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo   arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo
*         arn:aws:eks:us-east-1:ACCOUNT_ID:cluster/demogo        arn:aws:eks:us-east-1:ACCOUNT_ID:cluster/demogo        arn:aws:eks:us-east-1:ACCOUNT_ID:cluster/demogo
```

아래 명령어를 통해 명령어를 내릴 클러스터를 변경할 수 있습니다.

```
kubectl config use-context <<cluster-name>>

# 결과값
Switched to context "<<cluster-name>>".
```