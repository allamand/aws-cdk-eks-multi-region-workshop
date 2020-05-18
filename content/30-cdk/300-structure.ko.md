---
title: "프로젝트 구조"
weight: 300
pre: "<b>3-3. </b>"
---

## IDE 열기


이제 여러분이 주로 사용하시는 IDE를 열어서 프로젝트 내용을 확인해봅시다.
> VSCode를 이용하시는 경우, 프로젝트 디렉토리에서 `code .`를 입력하면 IDE를 열 수 있습니다.

## 프로젝트 디렉토리 탐색

IDE가 열리면 다음과 같은 구조를 볼 수 있습니다:

![](/images/15-cdk/structure.png)

* __`lib/cdk-workshop-stack.ts`__ CDK 어플리케이션의 메인 스택이 정의되는 곳입니다.  
  우리가 대부분의 시간을 쏟을 곳입니다.
* `bin/cdk-workshop.ts` CDK 어플리케이션의 엔트리 포인트입니다. `lib/cdk-workshop-stack.ts`에 정의된 스택을 로드합니다.
* `package.json` 은 npm module manifest 입니다. 앱의 이름, 버전, 의존하고 있는 패키지, 앞서 살펴 본 `watch` 같은 빌드 스크립트 등이 포함됩니다. 이 파일은 npm 이 관리하는 파일입니다.
* `cdk.json` toolkit이 어떻게 앱을 실행해야 하는지 알려주는 파일. 우리 프로젝트의 경우  `"npx ts-node bin/cdk-workshop.ts"` 가 들어갈 것입니다.
* `tsconfig.json` 해당 프로젝트의 [typescript
  설정 파일](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
* `.gitignore`, `.npmignore` 는 git과 npm 이 버전 관리를 할 때 / 패키지 매니저로 모듈을 퍼블리시 할 때 어떤 파일을 포함하고 제외할지 정의합니다.
* `node_modules` npm 이 관리하는 폴더로, 코드에서 필요로 하는 의존성 패키지들이 저장됩니다.

## 엔트리포인트

`bin/cdk-workshop.ts`를 빠르게 살펴봅시다:

```js
#!/usr/bin/env node
import * as cdk from '@aws-cdk/core';
import { CdkWorkshopStack } from '../lib/cdk-workshop-stack';

const app = new cdk.App();
new CdkWorkshopStack(app, 'CdkWorkshopStack');
```

이 코드는 `CdkWorkshopStack`을 로드하고 instantiate 합니다.
어떤 스택을 로드할 것인지만 정의되고 나면 이 파일은 더 이상 특별히 볼 일이 없을 것입니다.


## 메인 스택

`lib/cdk-workshop-stack.ts` 파일을 열어보십시오. 우리 어플리케이션의 중요한 부분이 여기에 정의될 것입니다.

```ts
import * as sns from '@aws-cdk/aws-sns';
import * as subs from '@aws-cdk/aws-sns-subscriptions';
import * as sqs from '@aws-cdk/aws-sqs';
import * as cdk from '@aws-cdk/core';

export class CdkWorkshopStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const queue = new sqs.Queue(this, 'CdkWorkshopQueue', {
      visibilityTimeout: cdk.Duration.seconds(300)
    });

    const topic = new sns.Topic(this, 'CdkWorkshopTopic');

    topic.addSubscription(new subs.SqsSubscription(queue));
  }
}
```

보시는 것처럼 우리의 어플리케이션은 샘플 CDK 스택 (`CdkWorkshopStack`)으로 이루어져있습니다.

이 스택에는 다음과 같은 서비스 생성이 포함됩니다:

- SQS Queue (`new sqs.Queue`)
- SNS Topic (`new sns.Topic`)
- SNS 토픽에서 발생하는 모든 메시지를 수신하도록 큐 설정 (`topic.addSubscription`)