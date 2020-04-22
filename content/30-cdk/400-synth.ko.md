---
title: "cdk synth"
weight: 400
pre: "<b>3-4. </b>"
---


## CDK 앱에서 CloudFormation 템플릿 산출하기


AWS CDK 앱은 코드를 이용해 인프라를 효과적으로 **정의**하도록 도와주는 도구입니다. CDK 앱이 실제로 실행될 때는 AWS CloudFormation template을 스택마다 생성하여 실제 배포를 합니다.

CDK 앱에서 템플릿을 산출하기 위해서는 `cdk synth` 명령어를 사용할 수 있습니다.
샘플 앱에서 추출된 템플릿을 한 번 살펴보죠:

{{% notice info %}} 
**CDK CLI** 는 `cdk.json` 파일이 있는 디렉토리에서만 실행될 수 있습니다. 터미널에서 다른 디렉토리로 이동했다면 다시 해당 디렉토리로 돌아가십시오.
{{% /notice %}}

```
cdk synth
```

다음과 같은 CloudFormation template이 출력될 것입니다:

```yaml
Resources:
  CdkWorkshopQueue50D9D426:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 300
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopQueue/Resource
  CdkWorkshopQueuePolicyAF2494A5:
    Type: AWS::SQS::QueuePolicy
    Properties:
      PolicyDocument:
        Statement:
          - Action: sqs:SendMessage
            Condition:
              ArnEquals:
                aws:SourceArn:
                  Ref: CdkWorkshopTopicD368A42F
            Effect: Allow
            Principal:
              Service: sns.amazonaws.com
            Resource:
              Fn::GetAtt:
                - CdkWorkshopQueue50D9D426
                - Arn
        Version: "2012-10-17"
      Queues:
        - Ref: CdkWorkshopQueue50D9D426
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopQueue/Policy/Resource
  CdkWorkshopQueueCdkWorkshopStackCdkWorkshopTopicD7BE96438B5AD106:
    Type: AWS::SNS::Subscription
    Properties:
      Protocol: sqs
      TopicArn:
        Ref: CdkWorkshopTopicD368A42F
      Endpoint:
        Fn::GetAtt:
          - CdkWorkshopQueue50D9D426
          - Arn
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopQueue/CdkWorkshopStackCdkWorkshopTopicD7BE9643/Resource
  CdkWorkshopTopicD368A42F:
    Type: AWS::SNS::Topic
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopTopic/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.21.1,@aws-cdk/aws-cloudwatch=1.21.1,@aws-cdk/aws-iam=1.21.1,@aws-cdk/aws-kms=1.21.1,@aws-cdk/aws-sns=1.21.1,@aws-cdk/aws-sns-subscriptions=1.21.1,@aws-cdk/aws-sqs=1.21.1,@aws-cdk/core=1.21.1,@aws-cdk/cx-api=1.21.1,@aws-cdk/region-info=1.21.1,jsii-runtime=node.js/v13.6.0
    Condition: CDKMetadataAvailable
Conditions:
  CDKMetadataAvailable:
    Fn::Or:
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-2
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-southeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-southeast-2
          - Fn::Equals:
              - Ref: AWS::Region
              - ca-central-1
          - Fn::Equals:
              - Ref: AWS::Region
              - cn-north-1
          - Fn::Equals:
              - Ref: AWS::Region
              - cn-northwest-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-central-1
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-north-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-2
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-3
          - Fn::Equals:
              - Ref: AWS::Region
              - me-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - sa-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-east-2
          - Fn::Equals:
              - Ref: AWS::Region
              - us-west-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-west-2
```

보시는 것처럼 이 템플릿은 아래와 같은 네 가지 자원을 생성합니다:

- **AWS::SQS::Queue** - SQS 큐
- **AWS::SNS::Topic** - SNS 토픽
- **AWS::SNS::Subscription** - 큐와 토픽 사이의 subscription 정의
- **AWS::SQS::QueuePolicy** - 토픽에서 큐로 메시지를 보낼 수 있는 IAM 정책

{{% notice info %}} 
**AWS::CDK::Metadata** 는 CDK 툴킷에 의해 모든 스택에 자동으로 생성되는 자원입니다.
CDK 팀이 보안 이슈 파악 및 분석 등을 위해 이용됩니다. 더 자세한 내용은 [Version Reporting](https://docs.aws.amazon.com/cdk/latest/guide/tools.html) 를 참조하십시오.
본 워크샵의 다음 단계에서는 메타데이터 자원을 `diff` 조회에서 제외할 것입니다.
{{% /notice %}}

