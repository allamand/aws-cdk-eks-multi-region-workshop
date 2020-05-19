---
title: Doing Full-stack IaC in Kubernetes
weight: 200
pre: "<b>1-1. </b>"
---

Infrastructure as Code (IaC) is not a choice anymore in cloud, it is rather being the de-facto. There are mainly three areas of code in this sense.

![](/images/10-intro/layers-of-iac.svg)

#### Infrastructure
Having EKS cluster requires you to also control complex infrastructure configuration such as VPC setting, security groups, and IAM roles and policies. It gets even crazier when you expand to another region or to another account in AWS cloud.

#### Platform
After you define the infrastructure part, it's not the end! There are bunch of other things that you need to take care on Kubernetes. They are not necessarily related to the application layer directly, but it's more of building the environment. It could be Namespaces, RBAC, ClusterAutoscaler or monitoring tools. 


#### Application

It depends on the organizational priority when it comes to who would cover the deployment of application part to kubernetes. However deployment script and k8s manifest are uncontroversially codes as well, which we would need to take good care.


## Why full-stack IaC matters?

1. Model it all
Modelling your entire infrastructure and application resources provides a single source of truth for all your resources and helps you to **standardize** infrastructure components used across your organization, enabling configuration **compliance** and **faster troubleshooting**.


2. Automate & deploy
It allows you to build and rebuild your infrastructure and applications, without having to perform manual actions or write custom scripts. IaC engine (CDK, Cloudformation, Terraform...) takes care of determining the right operations to perform when managing your stack, orchestrating them in the most efficient way, and rolls back changes automatically if errors are detected. It would lead to less chance of human errors.

3. It's just code
Codifying your infrastructure allows you to treat your infrastructure as just code. You can author it with any code editor, check it into a version control system, and review the files with team members before deploying into production. Also just like you do with your code, when things go wrong, you can just roll back to previous version and make the recovery process a lot easier.



## In this workshop

We covers the three layers of Kubernetes cluster like the following.

1. Infrastructure

    * [Lab 1](/en/40-deploy-clusters/200-cluster/) covers how we define the EKS cluster and necessary IAM, network settings with minimu amount of codes.
    * In [Lab 3](/en/60-deploy-app/200-single-region/), we would define the CI/CD infrastructure to deploy our sample application to the EKS clusters which we provisioned in Lab 1 and Lab 2.


2. Platform

    * We would define the resources that would be classified as platform layer in [Lab 1](/en/40-deploy-clusters/300-container/). In this lab, we would create Namespace and  [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/) in accordance with the best practice to keep the lifecycle of resources similar in your deployment unit.


3. Application

    [In Lab 3](/en/60-deploy-app/100-clone-sample-app/), we would clone a sample application and deploy it into two regions. We do not actually write the application together in this workshop. However it is definitely recommended to take a look into the structure of the app.