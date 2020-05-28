---
title: Version control for CDK
weight: 53
---

The CDK App is also code, so it must be versioned like application code.
To do that, we'll store the CDK code in a code repository like Git or CodeCommit.
And based on this, you can assume this repository as Single Source of Truth as you do in [GitOps](https://www.weave.works/technologies/gitops/), manage the code and deploy automatically.

Native support for CI / CD functionality for CDK code is posted on the [Roadmap] (https://github.com/orgs/aws/projects/7#card-28147397) as of May 2020.
Until this feature gets GA, you can write a pipeline that deploy the CDK application into multiple regions in a GitOps fashion. You can find a reference at [this github](https://github.com/yjw113080/aws-cdk-multi-region-cicd). .

![](/images/70-appendix/cdk-pipeline.svg)


