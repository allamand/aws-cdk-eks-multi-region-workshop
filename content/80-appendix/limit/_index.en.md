---
title: Service Limits to Consider
weight: 54
---


This workshop creates a lot of AWS resources.
For smooth workshop progress, it is necessary to check the basic limit values that must be considered in advance.

Usually the main sources of concern are **EIP** and **VPC**. Please check these carefully.

| Resources to Create                             | Amount to Create | Default Limit (region) |
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