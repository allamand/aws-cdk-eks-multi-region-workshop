---
title: Create EKS cluster
weight: 210
---


## Create EKS cluster

Define an EKS cluster by instantiating the imported package. Please copy and paste the code in the following code block right after the line you defined `const primaryRegion = 'ap-northeast-1';`.

```typescript
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


```


* `clusterAdmin` is the IAM role you would assume when executing `kubectl` against your EKS cluster.
* `cluster` is the EKS cluster we will create! You can configure several options with [this guide](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.Cluster.html). In this workshop, we will define the following options:
    * `clusterName`: To make the last lab easier, it statically defined to have a cluster name. This name should be unique in one region. If you do not set this value, CDK automatically assigns a unique name.
    * `masterRole`: IAM Principal which would join `systems:masters`, the Kubernetes RBAC group having full control over the cluster. We set this value to include `clusterAdmin` to the RBAC group.
    * `defaultCapacity`: It defines how many worker nodes this cluster have by default. If you do not give any type, CDK will make the instances in Managed Nodegroup.
* `cluster.addCapacity`: added a separate AutoScalingGroup to use EC2 Spot instances in addition to the default capacity. Spot instances are not available in Managed Nodegroup at the point of this workshop is written (May, 2020)


{{% notice info %}} 
FYI, EC2 Spot Instances are not purchased by bidding anymore, but simply with [market price](https://aws.amazon.com/ko/blogs/compute/new-amazon-ec2-spot-pricing/). Therefore when you define Spot instances in your IaC template, you can just put the price of On Demand purchase option without being too strategic about the spot price!
{{% /notice %}}



When you successfully pasted the code, the whole code would be like the following.

``` typescript
import * as iam from '@aws-cdk/aws-iam';
import * as eks from '@aws-cdk/aws-eks';

export class ClusterStack extends cdk.Stack {
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

    //...
```


## Load your stack in the entry point
How the simple code we just created would interact with the actual AWS cloud? Let's figure that out. Open **bin/multi-cluster-ts.ts**, and load the stack we created. 
The skeleton code would contain the information about AWS account and region for the stacks like the following code block. This workshop uses **Tokyo(ap-northeast-1)** and **Oregon(us-west-2)** as our target regions.
You can change those into your favorites, but make sure you also change the whole occurrence of `primaryRegion` and `secondaryRegion`. 

```typescript
const account = app.node.tryGetContext('account') || process.env.CDK_INTEG_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT;
const primaryRegion = {account: account, region: 'ap-northeast-1'};
const secondaryRegion = {account: account, region: 'us-west-2'};
```


Load the stack by copying and pasting after the last line the following code:
```typescript
const primaryCluster = new ClusterStack(app, `ClusterStack-${primaryRegion.region}`, {env: primaryRegion })
```

The completed code will be like the following block.

```typescript
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { ClusterStack } from '../lib/cluster-stack';
import { ContainerStack } from '../lib/container-stack';
import { CicdStack } from '../lib/cicd-stack';

const app = new cdk.App();

const account = app.node.tryGetContext('account') || process.env.CDK_INTEG_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT;
const primaryRegion = {account: account, region: 'ap-northeast-1'};
const secondaryRegion = {account: account, region: 'us-west-2'};

const primaryCluster = new ClusterStack(app, `ClusterStack-${primaryRegion.region}`, {env: primaryRegion })

```

## cdk bootstrap

This workshop uses Tokyo region (ap-northeast-1) and Oregon region (us-west-2) to provision the AWS resources. To successfully do this, we have to `cdk bootstrap` for those two environments.


cdk bootstrap is a tool in the AWS CDK command-line interface responsible for populating a given environment (that is, a combination of AWS account and region) with resources required by the CDK to perform deployments into that environment.

Execute the following commands to cdk bootstrap.

```
ACCOUNT_ID=$(aws sts get-caller-identity|jq -r ".Account")
cdk bootstrap aws://$ACCOUNT_ID/ap-northeast-1
cdk bootstrap aws://$ACCOUNT_ID/us-west-2
```

{{% notice info %}} 
If you have an ssh session and work on another server, we recommend that you update it once so that the session does not expire while you are bootstrapping. 
{{% /notice %}}




## Identifying resources to create

Now let's check which resources will be created. Use the following command.

{{% notice info %}}
`cdk diff` is a command that shows what resources are newly created, changed, or deleted compared to the existing ones before deploying the actual resources.
{{% /notice %}}

```
cdk diff
```

{{% notice warning %}}
If the result is not displayed as below, refer to [this step](/en/40-deploy-clusters/100-clone-base-repo/) and check if `npm run watch` is running in the background.
{{% /notice %}}


You will get the following output.
```
Stack ClusterStack-ap-southeast-2
IAM Statement Changes
┌───┬────────────────────────┬────────┬────────────────────────┬────────────────────────┬───────────┐
│   │ Resource               │ Effect │ Action                 │ Principal              │ Condition │
├───┼────────────────────────┼────────┼────────────────────────┼────────────────────────┼───────────┤
│ + │ ${AdminRole.Arn}       │ Allow  │ sts:AssumeRole         │ AWS:arn:${AWS::Partiti │           │
│   │                        │        │                        │ on}:iam::<<ACCOUNT_ID>>: │           │
│   │                        │        │                        │ root                   │           │
...
...
...
...
Resources
[+] AWS::IAM::Role AdminRole AdminRole38563C57 
[+] AWS::EC2::VPC demogo-cluster/DefaultVpc demogoclusterDefaultVpc0F0EA8D6 
[+] AWS::EC2::Subnet demogo-cluster/DefaultVpc/PublicSubnet1/Subnet demogoclusterDefaultVpcPublicSubnet1Subnet9B5D84CC 
[+] AWS::EC2::RouteTable demogo-cluster/DefaultVpc/PublicSubnet1/RouteTable demogoclusterDefaultVpcPublicSubnet1RouteTableA9719167 
[+] AWS::EC2::SubnetRouteTableAssociation demogo-cluster/DefaultVpc/PublicSubnet1/RouteTableAssociation demogoclusterDefaultVpcPublicSubnet1RouteTableAssociationF6BCC682 
[+] AWS::EC2::Route demogo-cluster/DefaultVpc/PublicSubnet1/DefaultRoute demogoclusterDefaultVpcPublicSubnet1DefaultRoute0A0FDBF1 
[+] AWS::EC2::EIP demogo-cluster/DefaultVpc/PublicSubnet1/EIP demogoclusterDefaultVpcPublicSubnet1EIP42D57092 
[+] AWS::EC2::NatGateway demogo-cluster/DefaultVpc/PublicSubnet1/NATGateway 
```

You have only entered about 20 lines of code now, but how do you create so many resources?
This is because the CDK sets the default values with **best practice** recommended by AWS.
Using CDK, you can define resources one by one like you do with CloudFormation or Terraform, or
**You can define only the parts you really need and leave the rest to AWS**.

## Create Resources
```
cdk deploy
```


After executing this command, you will be asked for your approval as shown below.
It happens when CDK tries to create Security related resources such as IAM roles, policies, and etc.
Look at the output on the console and type `y`.

```bash
...
Do you wish to deploy these changes (y/n)? 
```

The resource will be created normally in about 15 minutes.
In the meantime, you can check the status of the resource being created through the console output as shown below.

![](/images/20-single-region/creation-inprg.png)


When creation is completed, you can see the CloudFormation stack created in [AWS Console](console.aws.amazon.com/cloudformation/) as shown below.

![](/images/70-appendix/stacks.png)


{{% notice info %}} 
We only defined one stack, but why so many stacks in the console?! If you get curious, please find the reason out [here](/en/80-appendix/how-cfn-addresource/). {{% /notice %}}



## Update kubeconfig
![](/images/20-deploy-clusters/stack-output.png)

After the resource creation is completed, the ConfigCommand will be displayed as CloudFormation Output in the terminal as shown in the screenshot above.

Copy the `aws eks ...` part after the `=` in the output below and run it from the console.

```
ClusterStack-ap-northeast-1.demogoclusterConfigCommand6DB6D889 = aws eks update-kubeconfig --name demogo --region ap-northeast-1 --role-arn <<ROLE_ARN>>
```

If executed successfully, the following result will be displayed.

```
Updated context arn:aws:eks:ap-northeast-1:<<ACCOUNT_ID>>:cluster/demogo in /<<YOUR_HOME_DIRECTORY>>/.kube/config
```


This allows us to list / create / modify / delete resources using `kubectl` on the EKS cluster we just created.