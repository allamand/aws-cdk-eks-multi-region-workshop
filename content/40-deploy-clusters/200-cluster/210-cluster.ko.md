---
title: 기본 클러스터 스택 정의
weight: 210
---

## 패키지 import

이전 단계에서 해당 코드에 필요한 패키지는 모두 설치했습니다.  
아래와 같이 패키지를 코드에 import 하십시오.  

``` typescript
import * as iam from '@aws-cdk/aws-iam';
import * as eks from '@aws-cdk/aws-eks';

export class ClusterStack extends cdk.Stack {
//...
}
```


* `clusterAdmin`은 여러분의 클러스터에 `kubectl` 등의 명령어를 수행할 때 assume 할 IAM Role 입니다.
* `cluster`가 우리가 생성할 EKS 클러스터입니다. [이 가이드](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.Cluster.html)에 따라 여러가지 클러스터 값 설정을 할 수 있는데요, 이 워크샵에서 우리는 다음과 같은 설정을 할 것입니다.
    * `clusterName`: 두 번째 랩에서 사용하기 위해 우리는 이름을 미리 지정했습니다. 이 이름은 한 리전 내에서 고유해야 합니다. 입력하지 않을 경우 CDK가 자동으로 이름을 생성합니다.
    * `masterRole`: kubectl 조작 권한을 가질 수 있게 해주는 Kubernetes RBAC 그룹 `systems:master`에 추가될 IAM 주체를 선언합니다. 우리는 위에서 정의한 `clusterAdmin`을 이용해서 해당 클러스터에 접근하기 위해 이 롤을 입력합니다.
    * `defaultCapacity`: 기본으로 몇 개의 워커노드가 생성될 것인지 지정합니다.



## EKS 클러스터 생성하기

다음과 같이 import된 패키지를 인스턴스화하여 클러스터를 정의합니다.
`super(..)` 구문 아래 다음 코드를 복사하여 붙여주십시오.
```typescript
    const clusterAdmin = new iam.Role(this, 'AdminRole', {
        assumedBy: new iam.AccountRootPrincipal()
        });

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        defaultCapacity: 2
    });

```


완성된 코드는 다음과 같을 것입니다.

``` typescript
import * as iam from '@aws-cdk/aws-iam';
import * as eks from '@aws-cdk/aws-eks';

export class ClusterStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const clusterAdmin = new iam.Role(this, 'AdminRole', {
        assumedBy: new iam.AccountRootPrincipal()
        });

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        defaultCapacity: 2
    });
    //...
```


## 엔트리포인트에 스택 로드하기
그러면 우리가 완성한 이 스택이 실제로 AWS클라우드에서는 어떻게 보일까요?  
`bin/multi-cluster-ts.ts` 파일을 열어 스택을 로드해보겠습니다.
아래와 같이, 스택 생성에 사용할 AWS 계정과 Region 정보가 정의되어 있습니다.
필요하다면 리전을 변경하여 주십시오.

```typescript
const account = app.node.tryGetContext('account') || process.env.CDK_INTEG_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT;
const primaryRegion = {account: account, region: 'ap-northeast-1'};
const secondaryRegion = {account: account, region: 'us-east-1'};
```


아래 코드를 추가하여 스택을 로드한 뒤 저장해주십시오.
```typescript
const primaryCluster = new ClusterStack(app, `ClusterStack-${primaryRegion.region}`, {env: primaryRegion })
```

## 생성할 자원 확인하기
이제 다음 명령어를 사용해 어떤 자원들이 생성될 지 확인해보겠습니다.
{{% notice info %}}
`cdk diff`는 실제 자원을 배포하기 전에 기존 대비 어떤 자원들이 새롭게 생기거나 변경, 삭제될 것인지 보여주는 명령어입니다.
{{% /notice %}}
```
cdk diff
```

{{% notice warning %}}
만약 아래와 같이 결과가 출력되지 않는다면, [이 단계](/content/30-cdk/200-watch.ko.md)를 참고하여 `npm run watch`가 백그라운드에서 수행 중인지 확인하십시오.
{{% /notice %}}

아래와 같은 결과가 출력될 것입니다.
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


## 자원 생성하기
```
cdk deploy
```

이 명령어를 수행하고 나면 아래와 같이 여러분의 승인을 요구할 것입니다.  
IAM 등 보안 관련 자원이 생성될 텐데, 이에 대해 동의하느냐는 말입니다.
콘솔에 출력된 결과물을 살펴보신 뒤 `y`를 입력하십시오.
```bash
...
Do you wish to deploy these changes (y/n)? 
```

약 15분 정도의 시간 뒤에 정상적으로 자원이 생성될 것입니다.  
그동안 콘솔에서 생성 중인 자원의 상황을 확인할 수 있습니다.

## kubeconfig 업데이트하기
자원 생성이 완료되고 나면, 콘솔에 CloudFormation Output으로 ConfigCommand가 출력될 것입니다.
이 명령어를 복사하여 콘솔에서 실행하십시오.  
이를 통해 우리가 방금 생성한 EKS 클러스터에 `kubectl`을 이용하여 자원을 조회/생성/수정/삭제할 수 있습니다.

```
ClusterStack-us-east-1.demogoclusterConfigCommand6DB6D889 = aws eks update-kubeconfig --name demogo --region us-east-1 --role-arn <<YOUR_ROLE_ARN>>
```