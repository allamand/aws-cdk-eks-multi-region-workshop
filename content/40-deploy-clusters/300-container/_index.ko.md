---
title: ContainerStack 작성하기
weight: 300
---

쿠버네티스 자원을 관리하는 일은 결코 쉽지 않습니다. 클러스터 운영 관리를 위해서는 수많은 오브젝트들이 필요합니다.  
네임스페이스에서 출발해서 모니터링, 로깅에 필요한 컨테이너들, 오토스케일링에 필요한 컨테이너 등등이 있을 것입니다.
이 모든 것을 복수의 클러스터에 대해 관리해야 한다면 어떻게 해야 할까요?

CDK의 두 가지 Construct를 활용해서 주요한 컨테이너 자원을 인프라와 함께 관리할 수 있습니다.

1. [KubernetesResource](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.KubernetesResource.html) 를 활용해 K8S Manifest 를 그대로 전달


2. [HelmChart](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.HelmChart.html) 클래스를 이용해 helm 작업 수행

## 그럼 이 두 가지 방법을 사용해 앞서 생성한 EKS 클러스터에 자원을 생성해볼까요?