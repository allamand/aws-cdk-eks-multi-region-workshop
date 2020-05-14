---
title: cdk bootstrap
weight: 200
pre: "<b>4-2. </b>"
---

이 워크샵은 도쿄 리전(ap-northeast-1)과 버지니아 북부 리전(us-east-1)에 자원을 배포할 것입니다.  
이 두 리전에 대해 `cdk bootstrap` 작업을 수행해주어야, 우리가 워크샵을 통해 작성하는 CDK 코드를 이용해 AWS 자원을 만들 수 있습니다.

cdk bootstrap 은 CDK가 특정 환경(계정, 리전)에 자원 배포를 수행하기 위해 필요한 설정을 하도록 도와주는 AWS CDK cli 입니다. cdk bootstrap 을 수행하면 CDK toolkit을 위한 스택이 AWS 환경에 배포됨을 확인할 수 있습니다. 이를 통해 CloudFormation 템플릿과 기타 asset을 저장하는 S3 bucket이 생성됩니다.

다음 단계 진행 전에 아래 명령어를 반드시 수행하십시오.  


1. `cdk bootstrap aws://<<ACCOUNT_ID>>/ap-northeast-1`
2. `cdk bootstrap aws://<<ACCOUNT_ID>>/us-east-1`

{{% notice info %}} 
ssh 세션을 맺어 다른 서버에서 작업하시는 경우, 세션이 만료되지 않도록 한 차례 갱신한 뒤 수행하실 것을 권고드립니다. 
{{% /notice %}}

{{% notice info %}}
자신의 어카운트 ID는 `aws sts get-caller-identity` 명령어를 실행하면 알 수 있습니다.
{{% /notice %}}
