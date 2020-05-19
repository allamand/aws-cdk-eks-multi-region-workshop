---
title: Lab 3, Deploy apps into 2 regions
weight: 600
pre: "<b>1-5. </b>"
---

![](/images/40-deploy-app/intro2.svg)

In the lat lab, we would finally deploy the application into the EKS clusters of two regions! How you would do the deployment depends what your organization wants. For instance, you might want to replicate the code and docker image into the secondary region for disaster recovery, or you just want to simply get the app ready at the same time in both clusters. This workshop takes the way to deploy into one region and check if everything is alright, and then deploy it to secondary region.

![](/images/40-deploy-app/pipeline.svg)

We would use the following services
- AWS CodeCommit: a fully-managed source control service that hosts secure Git-based repositories.
- Amazon Elastic Container Registry (ECR): a fully-managed Docker container registry that makes it easy for developers to store, manage, and deploy Docker container images.
- AWS CodeBuild: a fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy.
- AWS CodePipeline: a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define.
