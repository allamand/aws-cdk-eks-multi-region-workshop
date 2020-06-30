---
title: Lab 3, Deploy apps in both regions
weight: 600
pre: "<b>1-5. </b>"
---

![](/images/40-deploy-app/intro2.svg)

In the last lab, we will finally deploy the application into the EKS clusters, even thought they are in two different regions! The deployment process depends of your organisation needs. For instance, you might want to replicate the code and docker image to the secondary region for disaster recovery purposes, or you just want to simply get the app ready at the same time in both clusters. This workshop is similar to the second scenario, where we deploy in one region, check if everything is healthy, and then proceed the deployment into the secondary region.

![](/images/40-deploy-app/pipeline.svg)

We will use the following services
- AWS CodeCommit: a fully-managed source control service that hosts secure Git-based repositories.
- Amazon Elastic Container Registry (ECR): a fully-managed Docker container registry that makes it easy for developers to store, manage, and deploy Docker container images.
- AWS CodeBuild: a fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy.
- AWS CodePipeline: a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. CodePipeline automates the build, test, and deploy phases of your release process every time there is a code change, based on the release model you define.
