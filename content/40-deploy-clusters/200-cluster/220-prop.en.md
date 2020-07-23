---
title: Export Props
weight: 220
---

## Create an interface for another stack
Now we need to create containers in our cluster.
To make the maintenance jobs easier, let's create a separate stack for this. In this case, the other stack needs to refer the cluster we created in `cluster-stack.ts`.

Let's continue working on the `cluster-stack.ts` that we created earlier so that we can export the cluster to other stacks.

Paste the code below at the bottom of the file.

```typescript
export interface EksProps extends cdk.StackProps {
  cluster: eks.Cluster
}

```

Go up again and define the readonly value as public above the `constructor`.

```typescript
  public readonly cluster: eks.Cluster;
```

Then, specify the cluster to be exported as props.
Paste the code below at the bottom of the class declaration.

```
      this.cluster = cluster;
```


The completed code should look like this.
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
Now you can export `EksProps` from this module and use it externally.
Let's use this to create a container stack.