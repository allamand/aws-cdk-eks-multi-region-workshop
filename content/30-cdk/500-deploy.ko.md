---
title: "cdk deploy"
weight: 500
pre: "<b>3-5. </b>"
---



CloudFormation을 보았으니 이제 **계정에 실제로 배포**를 해볼까요?

## 환경 Bootstrap

AWS CDK 앱을 환경 (계정/리전)에 배포하기 위해서는 먼저 `bootstrap stack`이라는 것을 설치해야 합니다.  
이 스택에는 툴킷의 운영을 위해 필요한 자원들이 포함됩니다. 예를 들어 CFN 템플릿을 보관하고 배포 프로세스 동안 생성되는 asset 들을 저장하는 S3 버킷 같은 것이 있습니다.

`cdk bootstrap` 명령어를 이용해 한 환경에 대해 부트스트랩 스택을 설치할 수 있습니다.


```
cdk bootstrap
```

Then:

```
 ⏳  Bootstrapping environment 999999999999/us-east-1...
...
```

{{% notice info %}} 
여기에서 Access Denied 에러가 발생하는 경우, AWS CLI가 [제대로 설정](/ko/15-prerequisites/200-account.html)되지 않았거나, 사용 중인 [AWS profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)이 `cloudformation:CreateChangeSet` 작업을 수행할 권한이 없는 경우입니다. {{% /notice %}}

위 명령어가 성공적으로 수행되고 나면 CDK 앱을 배포해봅시다.


## 배포하기

`cdk deploy` 명령어를 이용해 CDK 앱을 배포합니다:

```
cdk deploy
```

다음과 같은 경고가 뜰 것입니다:

```text
This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬────────────────────────────────┬────────┬─────────────────┬────────────────────────────────┬────────────────────────────────┐
│   │ Resource                       │ Effect │ Action          │ Principal                      │ Condition                      │
├───┼────────────────────────────────┼────────┼─────────────────┼────────────────────────────────┼────────────────────────────────┤
│ + │ ${CdkWorkshopQueue.Arn}        │ Allow  │ sqs:SendMessage │ Service:sns.amazonaws.com      │ "ArnEquals": {                 │
│   │                                │        │                 │                                │   "aws:SourceArn": "${CdkWorks │
│   │                                │        │                 │                                │ hopTopic}"                     │
│   │                                │        │                 │                                │ }                              │
└───┴────────────────────────────────┴────────┴─────────────────┴────────────────────────────────┴────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)?
```

이런 경고는, 배포하려는 앱에 보안 리스크가 동반되는 경우 출력됩니다.
토픽에서 큐로 메시지를 보내주어야 하므로 우리는 **y**를 입력하여 스택을 배포하고 자원을 생성합니다.


다음과 같은 결과가 출력될 것입니다. ACCOUNT-ID는 여러분의 계정이 될 것이고, REGION은 앱을 생성한 리전, STACK-ID는 여러분이 생성한 스택의 고유 ID로 출력됩니다:

```
CdkWorkshopStack: deploying...
CdkWorkshopStack: creating CloudFormation changeset...
 0/6 | 1:12:48 PM | CREATE_IN_PROGRESS   | AWS::SQS::Queue        | CdkWorkshopQueue (CdkWorkshopQueue50D9D426)
 0/6 | 1:12:48 PM | CREATE_IN_PROGRESS   | AWS::SNS::Topic        | CdkWorkshopTopic (CdkWorkshopTopicD368A42F)
 0/6 | 1:12:48 PM | CREATE_IN_PROGRESS   | AWS::CDK::Metadata     | CDKMetadata
 0/6 | 1:12:48 PM | CREATE_IN_PROGRESS   | AWS::SQS::Queue        | CdkWorkshopQueue (CdkWorkshopQueue50D9D426) Resource creation Initiated
 0/6 | 1:12:48 PM | CREATE_IN_PROGRESS   | AWS::SNS::Topic        | CdkWorkshopTopic (CdkWorkshopTopicD368A42F) Resource creation Initiated
 1/6 | 1:12:48 PM | CREATE_COMPLETE      | AWS::SQS::Queue        | CdkWorkshopQueue (CdkWorkshopQueue50D9D426)
 1/6 | 1:12:51 PM | CREATE_IN_PROGRESS   | AWS::CDK::Metadata     | CDKMetadata Resource creation Initiated
 2/6 | 1:12:51 PM | CREATE_COMPLETE      | AWS::CDK::Metadata     | CDKMetadata
 3/6 | 1:12:59 PM | CREATE_COMPLETE      | AWS::SNS::Topic        | CdkWorkshopTopic (CdkWorkshopTopicD368A42F)
 3/6 | 1:13:01 PM | CREATE_IN_PROGRESS   | AWS::SQS::QueuePolicy  | CdkWorkshopQueue/Policy (CdkWorkshopQueuePolicyAF2494A5)
 3/6 | 1:13:01 PM | CREATE_IN_PROGRESS   | AWS::SNS::Subscription | CdkWorkshopTopic/CdkWorkshopQueueSubscription (CdkWorkshopTopicCdkWorkshopQueueSubscription88D211C7)
 3/6 | 1:13:02 PM | CREATE_IN_PROGRESS   | AWS::SNS::Subscription | CdkWorkshopTopic/CdkWorkshopQueueSubscription (CdkWorkshopTopicCdkWorkshopQueueSubscription88D211C7) Resource creation Initiated
 3/6 | 1:13:02 PM | CREATE_IN_PROGRESS   | AWS::SQS::QueuePolicy  | CdkWorkshopQueue/Policy (CdkWorkshopQueuePolicyAF2494A5) Resource creation Initiated
 4/6 | 1:13:02 PM | CREATE_COMPLETE      | AWS::SNS::Subscription | CdkWorkshopTopic/CdkWorkshopQueueSubscription (CdkWorkshopTopicCdkWorkshopQueueSubscription88D211C7)
 5/6 | 1:13:03 PM | CREATE_COMPLETE      | AWS::SQS::QueuePolicy  | CdkWorkshopQueue/Policy (CdkWorkshopQueuePolicyAF2494A5)
 6/6 | 1:13:05 PM | CREATE_COMPLETE      | AWS::CloudFormation::Stack | CdkWorkshopStack

 ✅  CdkWorkshopStack

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT-ID:stack/CdkWorkshopStack/STACK-ID
```

## CloudFormation 콘솔

CDK 앱은 AWS CloudFormation을 통해 배포됩니다. CDK 스택은 CloudFormation 스택과 1:1로 매핑됩니다.  
이는 스택을 관리하기 위해 CloudFormation 콘솔을 이용할 수도 있다는 의미입니다.

[AWS CloudFormation
console](https://console.aws.amazon.com/cloudformation/home)을 확인해봅시다.

다음과 같은 화면이 보일 것입니다. 만약 그렇지 않다면 스택을 배포한 리전이 맞는지 확인하십시오:

![](/images/15-cdk/cfn1.png)

`CdkWorkshopStack`를 선택하고 __Resources__ 탭을 클릭하면, 우리가 생성한 자원의 물리적 ID를 확인할 수 있습니다.


![](/images/15-cdk/cfn2.png)

# 그럼 이제 실제 CDK 코드를 작성 해볼까요?!
