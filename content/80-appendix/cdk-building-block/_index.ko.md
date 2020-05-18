---
title: CDK의 빌딩 블록 이해하기
weight: 51
---

이 워크샵에서는 CDK를 통해 다양한 자원을 생성합니다.  
이 때 App, Construct, Stack 이라는 컨셉이 등장하는데요, 각각이 정확히 어떤 의미인지 짚어보고 넘어가려고 합니다.

## App
![](https://docs.aws.amazon.com/cdk/latest/guide/images/Lifecycle.png)

우리는 이 워크샵에서 cdk cli 를 이용해서 모든 자원 생성과 변경을 제어합니다. 이때 대상이 되는 것이 App 이라는 개념입니다.  
우리가 열심히 쓴 코드에 cdk cli 명령어를 주게 되면, 이 App은 위 그림에서처럼 검증 과정을 거쳐 CloudFormation 템플릿을 생성하게 됩니다.  
이 템플릿을 기반으로 실제로 AWS 클라우드와 상호작용 하게 되는 것이죠.



## Stack
우리가 이 워크샵에서 Stack 이라는 것을 정의하고 그 안에서 열심히 작업을 합니다.  
스택은 배포 단위라고 생각하시면 되는데요, CloudFormation 스택이랑 동일한 개념입니다. 그래서 [이 CloudFormation 스택과 동일한 제한값](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)이 있습니다.

한 스택은 앱의 스콥 내에서 직접적으로 정의될 수도 있고 앱의 트리 내에 있는 다른 스택에 의해 간접적으로 정의될 수도 있습니다. 한 스택에서 관여하는 직/간접적으로 생성된 자원은 한 유닛으로 제어됩니다.



## Construct
그렇다면 이 Stack 안에 우리가 `new eks...` 이런 식으로 정의하고 있는 자원들이 바로 Construct 입니다.  
Construct는 AWS CDK 앱의 기본 빌딩 블럭으로, EC2 등의 단일 자원일 수도 있고, 여러 자원을 묶어 추상화해놓은 것일 수도 있습니다. 예를 들어보면, 우리가 이 워크샵에서 `new eks...`를 선언하여 EKS 클러스터 자원만 생성하는 것 같아 보이지만, 사실은 그 EKS 클러스터를 동작시키기 위해 필요한 여러 자원들(IAM 자원, VPC 자원...)을 묶어 놓은 거죠. 그랬기 때문에 여러분이 짧은 코드를 써도 굉장히 많은 자원을 생성할 수 있는 것입니다.




---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
