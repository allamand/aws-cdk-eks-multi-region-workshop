---
title: Cross Region Replication of ECR
weight: 200
---



When operating an EKS cluster in several regions, there would be various operational models.
The container image registry itself is not replicated in this workshop. However, in some cases, you may want to replicate this across regions for Disaster Recovery.

![Amazon ECR repositories cross-region replication](https://github.com/aws-samples/amazon-ecr-cross-region-replication/raw/master/ecr-replication.png)


If you want to replicate ECR, please refer to [this guide](https://github.com/aws-samples/amazon-ecr-cross-region-replication) to accomplish it before it is natively supported by ECR as described in [the roadmap](https://github.com/aws/containers-roadmap/issues/140).




---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
