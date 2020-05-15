---
title: cdk bootstrap
weight: 200
pre: "<b>4-2. </b>"
---

This workshop creates resources over Tokyo (ap-northeast-1) and North Virginia (us-west-2). Only after conduting `cdk bootstrap`, our CDK code would be able to provision actual resources over AWS cloud.


cdk bootstrap is a tool in the AWS CDK command-line interface responsible for populating a given environment (that is, a combination of AWS account and region) with resources required by the CDK to perform deployments into that environment.

When you run cdk bootstrap cdk deploys the CDK toolkit stack into an AWS environment.

The bootstrap command creates a CloudFormation stack in the environment passed on the command line. Currently, the only resource in that stack is An S3 bucket that holds the file assets and the resulting CloudFormation template to deploy.

Run the following commands to bootstrap.

1. `cdk bootstrap ap-northeast-1`
2. `cdk bootstrap us-west-2`