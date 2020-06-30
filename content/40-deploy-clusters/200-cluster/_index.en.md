---
title: Create ClusterStack
weight: 300
pre: "<b>4-2. </b>"

---

Let's set up the EKS cluster and related settings for the cloned project.
Open the **lib/cluster-stack.ts** file in the IDE.
It should contain code like this:

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

Inside the `constructor` we are now putting the modules we want to create.
When the stack is instantiated, it is injected with the following: the scope, the id entered by the user, and props of the configuration information to be used by the stack.

* `scope`: This is the context in which the resources are defined. In this workshop, `cdk.App` defined in the `bin/multi-cluster-ts.ts` entry point will be the scope.
* `id`: This is a unique identifier within the `scope`. Think of it as a Namespace within this scope, used to specify the name of the AWS CloudFormation stacks and the names of sub-resources.
* `props`: It allows you to receive environment settings such as AWS account, region information. In addition, it can hand over information injected from other stacks and let us use them within the construct. We will use this `props` value to refer to resources between stacks.


The `super (scope, id, props)` syntax has been set to use whatever is injected when this class is instantiated.

