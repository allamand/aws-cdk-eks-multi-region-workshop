---
title: Building Blocks of CDK
weight: 51
---

In this workshop, we create a variety of resources through the CDK.
While we do this, we used App, Construct, and Stack. Let's figure out what exactly each means and how they work together.

## App
![](https://docs.aws.amazon.com/cdk/latest/guide/images/Lifecycle.png)

We use cdk cli to control all resource creation and changes. The target of the command is App.
If we run cdk cli command  in the code repo, this App would go through the verification process as shown in the picture above to create a CloudFormation template.
Based on this template, you are actually interacting with the AWS Cloud.


## Stack

We spent a lot of time in this workshop to write the Stack. You can think of a stack as a deployment unit, which is actually a CloudFormation stack. So there is [the same limit value as this CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html).

One stack can be defined directly within the App's scope, or indirectly by another stack within the App's tree like the NestedStacks of our CDK App. 


## Construct

The resources that we define in this stack looks like: `new eks ...`, and it is how we use Constructs!
Construct is the basic building block of the AWS CDK app, or it can be a single resource such as EC2, or it can be an abstraction of multiple resources. For example, we instantiated `new eks ...` in this workshop just to create EKS cluster resources without worrying about anything else, but in fact the various resources (IAM, VPC, Security Group and etc) are created. This is because `eks` construct abstracted all those other resources in it. In this way, you can create a lot of resources with short code!


---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
