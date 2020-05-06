---
title: CDK 코드를 위한 버전 관리
weight: 53
---

인프라를 정의하는 CDK 코드 역시, 코드이기 때문에 마치 어플리케이션 코드처럼 버전 관리가 되어야 합니다.  
그러기 위해서 Git이나 CodeCommit 같은 코드 레포지토리에 CDK 코드를 저장하게 되겠죠.
그리고 이를 기반으로 [GitOps](https://www.weave.works/technologies/gitops/) 처럼 이 레포지토리를 Single Source of Truth 로 상정하여, 코드를 관리하고 배포 자동화를 구현할 수 있을 것입니다.

CDK 코드에 대한 CI/CD 기능을 네이티브하게 지원하는 것은 2020년 5월 기준 [로드맵](https://github.com/orgs/aws/projects/7#card-28147397)에 올라와 있는 상태입니다.  
그 전까지는, [이 github](https://github.com/yjw113080/aws-cdk-multi-region-cicd)에서 간단한 샘플을 참고하여 여러 리전에 같이 배포하는 GitOps 스타일 파이프라인을 작성하실 수 있습니다.

![](/images/70-appendix/cdk-pipeline.svg)


