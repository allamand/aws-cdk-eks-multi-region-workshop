---
title: Cleanup
weight: 70
pre: "<b>7. </b>"
---

Thanks for coming all this long way! Let's clean up what we made today.


1. In the ELB console, delete the ELB deployed at application creation.
Delete all in [ap-northeast-1](https://console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#LoadBalancers:), [us-west-2](https://console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:sort=loadBalancerName).

![](/images/70-cleanup/elb-delete.png)


2. In [CloudFormation Console](console.aws.amazon.com/cloudformation/), Delete all the stacks in both regions. 
VPC deletion might fail due to the timeout of checking dependencies of the resource. Please make sure all the resources are gone.

{{% notice info %}} 
You can also delete the stack by running the `cdk destory "*"` command. Please note that it may take a long time because it deletes the stacks one by one.
{{% /notice %}}

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
