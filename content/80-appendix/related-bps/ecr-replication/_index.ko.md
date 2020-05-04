---
title: 리전 간 ECR 복제
weight: 200
---



멀티 리전으로 EKS 클러스터를 운영할 때에는 여러가지 운영 모델이 있을 수 있습니다.  
이 워크샵에서는 컨테이너 이미지 레지스트리 자체를 복제하지는 않습니다. 그렇지만 경우에 따라 이를 리전 간 복제하여 이미지 레지스트리의 DR을 꾀하여 볼 수도 있습니다.  


![Amazon ECR repositories cross-region replication](https://github.com/aws-samples/amazon-ecr-cross-region-replication/raw/master/ecr-replication.png)


이런 니즈가 있으신 분들은 
[ECR의 리전 간 복제가 네이티브하게 지원되기 전](https://github.com/aws/containers-roadmap/issues/140)까지는, [이 링크의 가이드](https://github.com/aws-samples/amazon-ecr-cross-region-replication)를 참조하여 리전 간 복제를 수행할 수 있습니다.



---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
