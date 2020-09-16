---
title: Cleanup
weight: 70
pre: "<b>7. </b>"
---

긴 시간 워크샵을 수행하시느라 고생 많으셨습니다.  


1. ELB 콘솔에서 애플리케이션 생성 시에 배포된 ELB를 삭제합니다.
[ap-northeast-1 리전](https://console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#LoadBalancers:), [us-west-2 리전](https://console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:sort=loadBalancerName) 모두 삭제합니다.

![](/images/70-cleanup/elb-delete.png)


2. [CloudFormation 콘솔](console.aws.amazon.com/cloudformation/)에 접근하여 두 리전 모두의 스택을 직접 삭제하십시오.  
이 때 VPC 자원 간의 의존성을 확인한 뒤 삭제 작업이 진행되어, timeout으로 인해 VPC의 삭제가 실패할 수 있습니다.  
이를 확인하여 모든 자원이 삭제될 수 있도록 하십시오.

{{% notice info %}} 
`cdk destroy "*"` 명령어를 수행해서 스택을 삭제할 수도 있습니다. 한 스택씩 삭제하기 때문에 소요 시간이 길어질 수 있으니 유의하십시오.
{{% /notice %}}

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
