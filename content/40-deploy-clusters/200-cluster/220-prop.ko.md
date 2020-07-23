---
title: Props 내보내기
weight: 220
---

## 다른 스택에서 참조할 Interface 만들기
우리가 생성한 클러스터에 이제 컨테이너를 생성해야 합니다.  
이 스택에 그대로 이어서 컨테이너 생성을 할 수도 있겠지만 유지보수를 위해 컨테이너 생성부는 별도의 스택으로 분리하여 생성해보겠습니다.

앞서 생성한 `cluster-stack.ts`에서 이어서 작업을 해보겠습니다.  
해당 파일 하단에 아래 코드를 붙여넣어 주십시오.

```typescript
export interface EksProps extends cdk.StackProps {
  cluster: eks.Cluster
}

```

다시 위로 올라와 `constructor` 위에 readonly 값을 퍼블릭으로 정의합니다.
```typescript
  public readonly cluster: eks.Cluster;
```

그리고 Props로 export될 클러스터를 지정해줍니다.  
class 선언 부분의 최하단에 아래 코드를 붙여넣어주세요.

```
      this.cluster = cluster;
```

완성된 코드는 아래와 같을 것입니다.
```typescript
import * as cdk from '@aws-cdk/core';
import * as iam from '@aws-cdk/aws-iam';
import * as eks from '@aws-cdk/aws-eks';
import * as ec2 from '@aws-cdk/aws-ec2';
import { PhysicalName } from '@aws-cdk/core';

export class ClusterStack extends cdk.Stack {
  public readonly cluster: eks.Cluster;

  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const primaryRegion = 'ap-northeast-1';
    const clusterAdmin = new iam.Role(this, 'AdminRole', {
      assumedBy: new iam.AccountRootPrincipal()
      });

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        version: eks.KubernetesVersion.V1_16,
        defaultCapacity: 2
    });

    cluster.addCapacity('spot-group', {
      instanceType: new ec2.InstanceType('m5.xlarge'),
      spotPrice: cdk.Stack.of(this).region==primaryRegion ? '0.248' : '0.192'
    });


    this.cluster = cluster;

  }
}

function createDeployRole(scope: cdk.Construct, id: string, cluster: eks.Cluster): iam.Role {
  const role = new iam.Role(scope, id, {
    roleName: PhysicalName.GENERATE_IF_NEEDED,
    assumedBy: new iam.AccountRootPrincipal()
  });
  cluster.awsAuth.addMastersRole(role);

  return role;
}
export interface EksProps extends cdk.StackProps {
  cluster: eks.Cluster
}


```
이제 이 모듈에서 `EksProps`를 export 하여 외부에서도 이용할 수 있게 되었습니다.  
이를 활용해서 컨테이너 스택을 작성해보겠습니다.