---
title: Create EKS cluster
weight: 210
---


## Create EKS cluster

Define a EKS cluster by instantiating the imported package. Please copy and paste the code in the following code block right after the line you defined `const primaryRegion = 'ap-northeast-1';`.

```typescript
    const clusterAdmin = new iam.Role(this, 'AdminRole', {
      assumedBy: new iam.AccountRootPrincipal()
      });

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        version: '1.14',
        defaultCapacity: 2
    });

    cluster.addCapacity('spot-group', {
      instanceType: new ec2.InstanceType('m5.xlarge'),
      spotPrice: cdk.Stack.of(this).region==primaryRegion ? '0.248' : '0.192'
    });


```


* `clusterAdmin` is the IAM role you would assume when executing `kubectl` against your EKS cluster.
* `cluster` is the EKS cluster we would create! You can configure several options with [this guide](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.Cluster.html). In this workshop, we would define the following options:
    * `clusterName`: To make the last lab easier, it statically defined to have a cluster name. This name should be unique in one region. If you do not set this value, CDK automatically assign a unique name.
    * `masterRole`: IAM Principal which would join `systems:masters`, the Kubernetes RBAC group having full control over the cluster. We set this value to include `clusterAdmin` to the RBAC group.
    * `defaultCapacity`: It defines how many worker nodes this cluster have by default. If you do not give any type, CDK would make the instances in Managed Nodegroup.
* `cluster.addCapacity`: added a separate AutoScalingGroup to utilize EC2 Spot instances in addition to the default capacity. (Spot instances are not available in Managed Nodegroup at the point of this workshop is written, May 2020)


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
        version: '1.14',
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


Load the stack by copying and pasting the following line:
```typescript
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
Stack ClusterStack-ap-northeast-1
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
[+] AWS::EC2::NatGateway demogo-cluster/DefaultVpc/PublicSubnet1/NATGateway demogoclusterDefaultVpcPublicSubnet1NATGateway8B5F277B 
[+] AWS::EC2::Subnet demogo-cluster/DefaultVpc/PublicSubnet2/Subnet demogoclusterDefaultVpcPublicSubnet2Subnet39C69507 
[+] AWS::EC2::RouteTable demogo-cluster/DefaultVpc/PublicSubnet2/RouteTable demogoclusterDefaultVpcPublicSubnet2RouteTable20C348DA 
[+] AWS::EC2::SubnetRouteTableAssociation demogo-cluster/DefaultVpc/PublicSubnet2/RouteTableAssociation demogoclusterDefaultVpcPublicSubnet2RouteTableAssociation8151DA4B 
[+] AWS::EC2::Route demogo-cluster/DefaultVpc/PublicSubnet2/DefaultRoute demogoclusterDefaultVpcPublicSubnet2DefaultRoute3A2BEF72 
[+] AWS::EC2::EIP demogo-cluster/DefaultVpc/PublicSubnet2/EIP demogoclusterDefaultVpcPublicSubnet2EIPDD1FE783 
[+] AWS::EC2::NatGateway demogo-cluster/DefaultVpc/PublicSubnet2/NATGateway demogoclusterDefaultVpcPublicSubnet2NATGateway9872618E 
[+] AWS::EC2::Subnet demogo-cluster/DefaultVpc/PublicSubnet3/Subnet demogoclusterDefaultVpcPublicSubnet3Subnet7AC2FC28 
[+] AWS::EC2::RouteTable demogo-cluster/DefaultVpc/PublicSubnet3/RouteTable demogoclusterDefaultVpcPublicSubnet3RouteTableB5078316 
[+] AWS::EC2::SubnetRouteTableAssociation demogo-cluster/DefaultVpc/PublicSubnet3/RouteTableAssociation demogoclusterDefaultVpcPublicSubnet3RouteTableAssociationDFEE60CD 
[+] AWS::EC2::Route demogo-cluster/DefaultVpc/PublicSubnet3/DefaultRoute demogoclusterDefaultVpcPublicSubnet3DefaultRoute2D744EC6 
[+] AWS::EC2::EIP demogo-cluster/DefaultVpc/PublicSubnet3/EIP demogoclusterDefaultVpcPublicSubnet3EIPACAD503B 
[+] AWS::EC2::NatGateway demogo-cluster/DefaultVpc/PublicSubnet3/NATGateway demogoclusterDefaultVpcPublicSubnet3NATGateway8A6E1058 
[+] AWS::EC2::Subnet demogo-cluster/DefaultVpc/PrivateSubnet1/Subnet demogoclusterDefaultVpcPrivateSubnet1Subnet9EBC7F0F 
[+] AWS::EC2::RouteTable demogo-cluster/DefaultVpc/PrivateSubnet1/RouteTable demogoclusterDefaultVpcPrivateSubnet1RouteTableF109BA8D 
[+] AWS::EC2::SubnetRouteTableAssociation demogo-cluster/DefaultVpc/PrivateSubnet1/RouteTableAssociation demogoclusterDefaultVpcPrivateSubnet1RouteTableAssociation098CAA3A 
[+] AWS::EC2::Route demogo-cluster/DefaultVpc/PrivateSubnet1/DefaultRoute demogoclusterDefaultVpcPrivateSubnet1DefaultRoute3000F93D 
[+] AWS::EC2::Subnet demogo-cluster/DefaultVpc/PrivateSubnet2/Subnet demogoclusterDefaultVpcPrivateSubnet2Subnet71BE4D47 
[+] AWS::EC2::RouteTable demogo-cluster/DefaultVpc/PrivateSubnet2/RouteTable demogoclusterDefaultVpcPrivateSubnet2RouteTable0695DE49 
[+] AWS::EC2::SubnetRouteTableAssociation demogo-cluster/DefaultVpc/PrivateSubnet2/RouteTableAssociation demogoclusterDefaultVpcPrivateSubnet2RouteTableAssociationDA611F45 
[+] AWS::EC2::Route demogo-cluster/DefaultVpc/PrivateSubnet2/DefaultRoute demogoclusterDefaultVpcPrivateSubnet2DefaultRouteB3F63E45 
[+] AWS::EC2::Subnet demogo-cluster/DefaultVpc/PrivateSubnet3/Subnet demogoclusterDefaultVpcPrivateSubnet3Subnet7C658410 
[+] AWS::EC2::RouteTable demogo-cluster/DefaultVpc/PrivateSubnet3/RouteTable demogoclusterDefaultVpcPrivateSubnet3RouteTableF5EFFC72 
[+] AWS::EC2::SubnetRouteTableAssociation demogo-cluster/DefaultVpc/PrivateSubnet3/RouteTableAssociation demogoclusterDefaultVpcPrivateSubnet3RouteTableAssociation45D2D250 
[+] AWS::EC2::Route demogo-cluster/DefaultVpc/PrivateSubnet3/DefaultRoute demogoclusterDefaultVpcPrivateSubnet3DefaultRoute029BFE42 
[+] AWS::EC2::InternetGateway demogo-cluster/DefaultVpc/IGW demogoclusterDefaultVpcIGWE387E56C 
[+] AWS::EC2::VPCGatewayAttachment demogo-cluster/DefaultVpc/VPCGW demogoclusterDefaultVpcVPCGW187F7878 
[+] AWS::IAM::Role demogo-cluster/Role demogoclusterRole530AF8F4 
[+] AWS::EC2::SecurityGroup demogo-cluster/ControlPlaneSecurityGroup demogoclusterControlPlaneSecurityGroup2C10657D 
[+] AWS::EC2::SecurityGroupIngress demogo-cluster/ControlPlaneSecurityGroup/from ClusterStackapsoutheast2demogoclusterDefaultCapacityInstanceSecurityGroupE5C0D5D8:443 demogoclusterControlPlaneSecurityGroupfromClusterStackapsoutheast2demogoclusterDefaultCapacityInstanceSecurityGroupE5C0D5D84433834DEB2 
[+] AWS::IAM::Role demogo-cluster/Resource/CreationRole demogoclusterCreationRoleB8F24255 
[+] AWS::IAM::Policy demogo-cluster/Resource/CreationRole/DefaultPolicy demogoclusterCreationRoleDefaultPolicy711D53D3 
[+] Custom::AWSCDK-EKS-Cluster demogo-cluster/Resource/Resource demogoclusterA89898EB 
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/AwsAuth/manifest/Resource demogoclusterAwsAuthmanifestFF93B93C 
[+] AWS::EC2::SecurityGroup demogo-cluster/DefaultCapacity/InstanceSecurityGroup demogoclusterDefaultCapacityInstanceSecurityGroupC4B9CC76 
[+] AWS::EC2::SecurityGroupIngress demogo-cluster/DefaultCapacity/InstanceSecurityGroup/from ClusterStackapsoutheast2demogoclusterDefaultCapacityInstanceSecurityGroupE5C0D5D8:ALL TRAFFIC demogoclusterDefaultCapacityInstanceSecurityGroupfromClusterStackapsoutheast2demogoclusterDefaultCapacityInstanceSecurityGroupE5C0D5D8ALLTRAFFICAB31DDDD 
[+] AWS::EC2::SecurityGroupIngress demogo-cluster/DefaultCapacity/InstanceSecurityGroup/from ClusterStackapsoutheast2demogoclusterControlPlaneSecurityGroupF5A0BAA2:443 demogoclusterDefaultCapacityInstanceSecurityGroupfromClusterStackapsoutheast2demogoclusterControlPlaneSecurityGroupF5A0BAA24434161DB3B 
[+] AWS::EC2::SecurityGroupIngress demogo-cluster/DefaultCapacity/InstanceSecurityGroup/from ClusterStackapsoutheast2demogoclusterControlPlaneSecurityGroupF5A0BAA2:1025-65535 demogoclusterDefaultCapacityInstanceSecurityGroupfromClusterStackapsoutheast2demogoclusterControlPlaneSecurityGroupF5A0BAA21025655350E76CDB2 
[+] AWS::IAM::Role demogo-cluster/DefaultCapacity/InstanceRole demogoclusterDefaultCapacityInstanceRoleD81B758A 
[+] AWS::IAM::InstanceProfile demogo-cluster/DefaultCapacity/InstanceProfile demogoclusterDefaultCapacityInstanceProfile71466EC2 
[+] AWS::AutoScaling::LaunchConfiguration demogo-cluster/DefaultCapacity/LaunchConfig demogoclusterDefaultCapacityLaunchConfig93D71520 
[+] AWS::AutoScaling::AutoScalingGroup demogo-cluster/DefaultCapacity/ASG demogoclusterDefaultCapacityASGCD1D544F 
[+] AWS::CloudFormation::Stack @aws-cdk--aws-eks.ClusterResourceProvider.NestedStack/@aws-cdk--aws-eks.ClusterResourceProvider.NestedStackResource awscdkawseksClusterResourceProviderNestedStackawscdkawseksClusterResourceProviderNestedStackResource9827C454 
[+] AWS::CloudFormation::Stack @aws-cdk--aws-eks.KubectlProvider.NestedStack/@aws-cdk--aws-eks.KubectlProvider.NestedStackResource awscdkawseksKubectlProviderNestedStackawscdkawseksKubectlProviderNestedStackResourceA7AEBA6B

```

여러분은 지금 코드를 대략 20줄 정도 밖에 입력하지 않았는데, 어떻게 이렇게 많은 자원이 생성된 것일까요?  
그 이유는 CDK가 AWS가 권고하는 **베스트 프랙티스를 기본값**으로 갖고 있기 때문에 그렇습니다.  
CDK를 사용하면 CloudFormation이나 Terraform 을 이용하는 것처럼 자원 하나하나를 정의할 수도 있지만,  
**지금처럼 꼭 필요한 부분만 정의하고 나머지 부분은 AWS에게 맡기실 수도** 있습니다.

You have only entered about 20 lines of code now, but how do you create so many resources?
This is because the CDK sets the default values with **best practice** recommended by AWS.
Using CDK, you can define resources one by one like you do with CloudFormation or Terraform, or
**You can define only the parts you really need and leave the rest to AWSå**.

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
We only defined one stack, but why so many stacks in the console?! If you get curiou, please find the reason out [here](/ko/80-appendix/how-cfn-addresource/). {{% /notice %}}



## Update kubeconfig
![](/images/20-deploy-clusters/stack-output.png)

After the resource creation is completed, the ConfigCommand will be displayed as CloudFormation Output in the terminal as shown in the screenshot above.

Copy the `aws eks ...` part after the `=` in the output below and run it from the console.

```
ClusterStack-ap-northeast-1.demogoclusterConfigCommand6DB6D889 = aws eks update-kubeconfig --name demogo --region ap-northeast-1 --role-arn <<ROLE_ARN>>
```

If executed successfully, the following result will be displayed.

```
Updated context arn:aws:eks:ap-northeast-1:<<ACCOUNT_ID>>:cluster/demogo in /Users/jiwony/.kube/config
```


This allows us to list / create / modify / delete resources using `kubectl` on the EKS cluster we just created.