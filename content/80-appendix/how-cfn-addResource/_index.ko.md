---
title: CDK가 K8S 자원을 생성하는 메커니즘
weight: 52
---


![](/images/70-appendix/stacks.png)

우리는 스택을 두 가지 정의했습니다. 하나는 EKS 클러스터 자체에 대한 스택, 다른 하나는 컨테이너를 정의하는 스택.  
그런데 위 스크린샷처럼, CloudFormation 콘솔에 들어가보면 우리는 생성한 적 없는 스택들이 추가로 배포된 것을 확인할 수 있습니다. 왜 이럴까요?!

현 시점에서 AWS CloudFormation 자원인 `AWS::EKS::Cluster`는 이런 기능을 네이티브하게 제공하지 못합니다. 그래서 manifest를 적용하거나 IAM role 매핑 같은 작업, 즉 클러스터에 직접 명령을 내려야 하는 작업을 처리하기 위해 Amazon EKS construct library는 커스텀 리소스를 이용합니다.

이 말인 즉슨, 여러분이 명시적으로 정의한 자원이나 그 자원이 필요로 하는 자원들(VPC, Security Group...)이 아닌 다른 것들이 배포됨을 확인할 수 있다는 것입니다. 여기에는 AWS Lambda 함수, 여기에 필요한 IAM Role과 정책이 포함됩니다. 


---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
