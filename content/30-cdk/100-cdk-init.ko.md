---
title: "cdk init"
weight: 100
pre: "<b>3-1. </b>"
---

## 프로젝트 폴더 생성

새로운 빈 디렉토리를 생성합니다:

```
mkdir cdk-workshop && cd cdk-workshop
```

## cdk init

`cdk init` 명령어를 사용해서, Typescript를 사용하는 새로운 CDK 프로젝트를 생성합니다:

```
cdk init sample-app --language typescript
```

실행 결과는 다음과 같을 겁니다.  

```
Applying project template app for typescript
Initializing a new git repository...
Executing npm install...
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN tst@0.1.0 No repository field.
npm WARN tst@0.1.0 No license field.

# CDK Typescript 프로젝트 살펴보기

이 프로젝트의 내용물을 한 번 살펴보십시오. (`CdkWorkshopStack`)라는 스택 인스턴스를 띄우고 있는 CDK 앱을 확인할 수 있습니다.  
그 스택에는 Amazon SNS 토픽을 구독하는 Amazon SQS 큐 생성 코드가 들어있습니다.

`cdk.json` 파일은 앱을 실행할 수 있는 CDK Toolkit 명령어를 보여줍니다.


## 유용한 명령어

 * `npm run build`   compile typescript to js
 * `npm run watch`   watch for changes and compile
 * `npm run test`    perform the jest unit tests
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk synth`       emits the synthesized CloudFormation template
```

지금 보시는 것처럼, 우리가 바로 이용해볼 수 있는 유용한 명령어들이 출력됩니다.


## 참고

- [AWS CDK Command Line Toolkit (cdk) in the AWS CDK User Guide](https://docs.aws.amazon.com/CDK/latest/userguide/tools.html)
