---
title: 쿠버네티스 클러스터를 풀스택으로 IaC 한다는 것
weight: 200
---

클라우드에서 인프라를 코드로 정의하는 것(Infrastructure as Code, 이하 IaC)은 선택이 아닌 필수입니다.  
DevOps 엔지니어로서, 여러분이 쿠버네티스 클러스터를 클라우드에 배포할 때 신경써야 하는 코드는 크게 세 가지 종류가 있을 것입니다.  

![](/images/10-intro/layers-of-iac.svg)

#### Infrastructure
AWS 클라우드 상에 하나의 EKS 클러스터를 배포할 때에, 여기에는 여러가지 설정이 동반됩니다.  
VPC 설정, 그 안의 서브넷, 보안그룹, 각종 IAM Role과 정책 등등이 있습니다.  

#### Platform
이렇게 관련 인프라를 코드로 정의하고 나면, 그 위에 클러스터 차원에서 관리해주어야 하는 자원들이 있습니다.  
예를 들어, 네임스페이스를 구분하고 이에 대해 [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)를 지정하거나 오토스케일링, 모니터링, 로깅 설정을 하는 것 등등이 있을 수 있겠죠. 

#### Application
조직에 따라 누가 하는지는 다를 수 있지만, 어플리케이션 레이어에서도 1) 배포 스크립트와 2) k8s manifest 역시 코드로써 같이 관리되어야 합니다.

## 이 워크샵에서는
이 워크샵은, 두 개의 메인 랩을 통해서 CDK를 이용해 1) 인프라와 플랫폼 레이어를 정의하고 2) 어플리케이션 레이어를 Active-Active 클러스터에 모두 배포하는 작업을 수행할 것입니다.