---
title: How CDK creates K8S resources
weight: 52
---


![](/images/70-appendix/stacks.png)

We have defined two stacks. One is the stack for the EKS cluster itself, the other is the stack defining the container.
However, as shown in the screenshot above, when we go into the CloudFormation console, we can see that additional stacks which we did not even touch have been deployed. Why is that?

At this point, the AWS CloudFormation resource `AWS::EKS::Cluster` does not have native control over Kubernetes resources. In other words, CloudFormation cannot run kubectl directly. So, if you need to create a resource using a manifest or issue a command directly to a cluster--IAM role mapping and etc--the Amazon EKS construct library uses a custom resource.

This means that NestedStack is deployed as well in the exactly same way CDK did for implicit resources (VPC, Security Group ...). This NestedStack includes AWS Lambda functions, IAM Roles and policies required for them.

It is why we called `ClusterStack` when deploying containers, even though we are defining containers in `ContainerStack`. NestedStacks created in `ClusterStack` work for deployment, not `ContainerStack`. 

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
