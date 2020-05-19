---
title: 워크샵 수행 전 고려해야 하는 서비스 리밋
weight: 54
---

이 워크샵에서는 많은 AWS 자원을 생성합니다.
원활한 워크샵 진행을 위해 사전에 고려해야 하는 기본 리밋값을 미리 확인이 필요합니다.

주로 문제가 되는 자원은 **EIP**와 **VPC**입니다. 이 부분 유의하여 확인해주십시오.

| 생성되는 자원                             | 생성되는 개수 | Default Limit (region) |
|---------------------------------------|-----------|------------------------|
| AWS::AutoScaling::AutoScalingGroup    | 1         | 200                    |
| AWS::AutoScaling::LaunchConfiguration | 1         | -                      |
| AWS::CDK::Metadata                    | 1         | -                      |
| AWS::CloudFormation::Stack            | 2         | -                      |
| **AWS::EC2::EIP**                         | 3         | **5**                      |
| AWS::EC2::InternetGateway             | 1         | 5                      |
| AWS::EC2::NatGateway                  | 3         | 5 (per AZ)             |
| AWS::EC2::Route                       | 6         | 50 (per routetable)    |
| AWS::EC2::RouteTable                  | 6         | 200 (per VPC)          |
| AWS::EC2::SecurityGroup               | 2         | 2500                   |
| AWS::EC2::SecurityGroupIngress        | 4         | 60 (per SG)            |
| AWS::EC2::Subnet                      | 6         | 200 (per VPC)          |
| AWS::EC2::SubnetRouteTableAssociation | 6         | -                      |
| **AWS::EC2::VPC**                         | 1         | **5**                      |
| AWS::EC2::VPCGatewayAttachment        | 1         | -                      |
| AWS::EKS::Nodegroup                   | 1         | 30 (per cluster)       |
| AWS::IAM::InstanceProfile             | 1         | 1000 (per account)     |
| AWS::IAM::Policy                      | 1         | 1500 (per account)     |
| AWS::IAM::Role                        | 6         | 1000 (per account)     |
| Custom::AWSCDK-EKS-Cluster            | 1         | 100                    |
| Custom::AWSCDK-EKS-HelmChart          | 2         | -                      |
| Custom::AWSCDK-EKS-KubernetesResource | 3         | -                      |