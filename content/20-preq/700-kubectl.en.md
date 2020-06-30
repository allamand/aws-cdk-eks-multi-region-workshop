---
title: kubectl & aws-iam-authenticator
weight: 700
pre: "<b>2-7. </b>"
---

## Install kubectl

Please follow the link to figure out installation guide for your OS.
* https://kubernetes.io/docs/tasks/tools/install-kubectl/ 

You can use the following command to check the installation.

```
kubectl version --client

# Output example
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.1", GitCommit:"d647ddbd755faf07169599a625faf302ffc34458", GitTreeState:"clean", BuildDate:"2019-10-02T23:49:20Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"darwin/amd64"}
```


## Install aws-iam-authenticator

AWS IAM authenticator helps you to use IAM for Amazon EKS. Install AWS IAM authenticator and modify kubectl configuration file to use it.

Please follow the guide from [this link](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-aws-iam-authenticator.html) to install aws-iam-authenticator.


You can use the following command to check the installation.

```
aws-iam-authenticator help
```
