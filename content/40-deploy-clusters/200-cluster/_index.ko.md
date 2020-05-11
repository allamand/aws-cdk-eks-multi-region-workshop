---
title: ClusterStack 작성하기
weight: 300
pre: "<b>4-3. </b>"

---

클론한 프로젝트에 EKS 클러스터와 관련 설정을 해보겠습니다.  
**lib/cluster-stack.ts** 파일을 IDE에서 여십시오.
다음과 같은 코드가 들어있을 것입니다.


```typescript
import * as cdk from '@aws-cdk/core';
import * as iam from '@aws-cdk/aws-iam';
import * as eks from '@aws-cdk/aws-eks';
import * as ec2 from '@aws-cdk/aws-ec2';
import { PhysicalName } from '@aws-cdk/core';

export class ClusterStack extends cdk.Stack {

  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const primaryRegion = 'ap-northeast-1';

  }
}
}
...
```

`constructor` 안에 이제 우리가 생성하고자 하는 모듈들을 집어 넣게 됩니다.
해당 스택이 인스턴스화 될 때, 해당 시점의 scope과 사용자가 입력하는 id, 그리고 그 스택이 이용할 설정 정보 props를 받아서 이용하게 됩니다.
* `scope`: 우리가 지금 생성하려고 하는 자원들이 어느 스택에 붙을 것인지 정의해주는 것입니다. 우리는 이 cdk App을 루트로 갖도록 생성할 것입니다. 이 단계를 참고해주세요.
* `id`: 이 클래스는 실제로 AWS 클라우드에 배포될 때는 CloudFormation 스택이 되는데, 그 스택의 이름을 뜻합니다.
* `props`: AWS 계정, 리전 정보 및 다른 스택에서 주입해주는 정보들을 받아서 이 클래스 내에서 쓸 수 있도록 해줍니다.

`super(scope, id, props)` 구문을 통해 이 클래스가 인스턴스화 될 때 주입되는 것을 그대로 사용한다는 설정을 했습니다.

## 그럼 이제 실제 EKS 클러스터를 생성해봅시다.