---
title: Cross Region Replication of ECR
weight: 200
---



멀티 리전으로 EKS 클러스터를 운영할 때에는 여러가지 운영 모델이 있을 수 있습니다.  
이 워크샵에서는 컨테이너 이미지 레지스트리 자체를 복제하지는 않습니다. 그렇지만 경우에 따라 이를 리전 간 복제하여 이미지 레지스트리의 DR을 꾀하여 볼 수도 있습니다.  

When operating an EKS cluster in several regions, there would be various operational models.
The container image registry itself is not replicated in this workshop. However, in some cases, you may want to replicate this across regions for Disaster Recovery.

![Amazon ECR repositories cross-region replication](https://github.com/aws-samples/amazon-ecr-cross-region-replication/raw/master/ecr-replication.png)


If you want to replicate ECR, please refer to [this guide](https://github.com/aws-samples/amazon-ecr-cross-region-replication) to accomplish it before it is natively supported by ECR as described in [the roadmap](https://github.com/aws/containers-roadmap/issues/140).




---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
