---
title: Helm Chart 이용하기
weight: 330
---

## Helm Chart?
![](https://helm.sh/img/helm.svg)
* 공식 홈페이지: https://helm.sh/

Helm은 쿠버네티스 어플리케이션을 관리하는 데에 도움을 주는 툴입니다.  
Helm Chart라는 패키징 포맷을 사용하는데, 이 포맷으로 쿠버네티스 어플리케이션을 정의/설치/업그레이드/삭제하실 수 있습니다.
예를 들어 여러분이 서비스하는 웹 서비스 하나에 대한 Deployment, Service가 정의되어야 하고 필요한 Role이 있다고 할 때 이를 한꺼번에 묶어서 패키지처럼 관리하는 것이지요.  
여러분이 만드시는 것 뿐 아니라 다른 사람들이 만든 소프트웨어도 Helm 을 이용해서 쉽게 다운 받아 이용할 수 있습니다. 대표적으로 metrics-server나, prometheus 등이 있지요.
이번 페이지에서는 CDK로 어떻게 Helm Chart를 EKS 클러스터에 배포할 수 있는지 살펴봅니다.

## Helm Chart 배포하기
`lib/container-stack.ts`을 여십시오.  
지금까지 작성한 코드 아래에 다음 코드를 붙여넣습니다.

```typescript

const stable = 'https://kubernetes-charts.storage.googleapis.com/';

cluster.addChart(`metrics-server`, {
      repository: stable,
      chart: 'metrics-server',
      release: 'metrics-server'
    });

```

Helm Chart 를 설치할 때에는 `helm upgrade --install` 명령어를 이용하게 됩니다. 이 말인 즉슨, 같은 릴리즈 네임을 갖고 있는 경우 업데이트를 시도한다는 것입니다.  
Helm Chart는 CloudFormation 자원으로 배포됩니다. 그렇기 때문에 CDK 코드에서 지워지거나, 스택에서 삭제되면 `helm uninstall` 명령어를 통해 클러스터에서 삭제됩니다.  

Helm Chart를 지정하실 때 참고하실 만한 몇 가지 팁을 드립니다.
* `repository`는 URL을 완전하게 작성해주셔야 합니다. 일반적으로 `helm install stable/xx`라고 명령어를 칠 때, `stable`이라는 레포지토리를 등록하는 작업을 사전에 거치시게 됩니다. 그런데 CDK에서는 그 작업 없이 URL을 주는 형태라고 이해하시면 됩니다.
* 릴리즈 이름을 지정하지 않으면 노드 ID의 마지막 53 글자로 임의 지정됩니다. 되도록 사람이 이해할 수 있는 `release` 값을 주시기를 권장합니다.
* 이 외에 Chart 배포에 필요한 [설정값 상세](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.HelmChart.html)는 여기에서 확인하실 수 있습니다.



## 생성될 자원 확인하기
아래 명령어로 생성될 자원을 확인해봅시다.
```
cdk diff
```

아래 결과와 같이 차트가 생성될 것임을 확인할 수 있습니다.

```
Stack ClusterStack-ap-northeast-1
Resources
[+] Custom::AWSCDK-EKS-HelmChart demogo-cluster/chart-metrics-server/Resource demogoclusterchartmetricsserver19E78457

Stack ContainerStack-ap-northeast-1
There were no differences
```


## 배포하기
아래 명령어로 컨테이너를 배포해봅시다.
```
cdk deploy ContainerStack-ap-northeast-1
```

진행과정이 터미널에 출력됩니다.  
다음 명령어를 이용해 아래와 같이 새로운 pod가 Helm Chart를 통해 배포되었음을 확인하세요.

```
kubectl get pod
```


```
NAME                                READY   STATUS    RESTARTS   AGE
metrics-server-6b6bbf4668-vl455     1/1     Running   0          102s << 방금 Chart 선언으로 생성된 Pod
nginx-deployment-5754944d6c-cnxzn   1/1     Running   0          16m
nginx-deployment-5754944d6c-vrrd2   1/1     Running   0          16m
```