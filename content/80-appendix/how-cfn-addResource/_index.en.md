---
title: How CDK creates K8S resources
weight: 52
---


![](/images/70-appendix/stacks.png)

우리는 스택을 두 가지 정의했습니다. 하나는 EKS 클러스터 자체에 대한 스택, 다른 하나는 컨테이너를 정의하는 스택.  
그런데 위 스크린샷처럼, CloudFormation 콘솔에 들어가보면 우리는 생성한 적 없는 스택들이 추가로 배포된 것을 확인할 수 있습니다. 왜 이럴까요?!

현 시점에서 AWS CloudFormation 자원인 `AWS::EKS::Cluster`는 쿠버네티스 자원을 네이티브하게 컨트롤하지 못합니다. 다시 말해, CloudFormation이 kubectl 을 수행해야 하는 작업은 직접할 수 없다는 것입니다. 그래서 manifest를 이용해 자원을 생성하거나 IAM role 매핑 같은 작업과 같이 클러스터에 직접 명령을 내려야 하는 경우, Amazon EKS construct library는 커스텀 리소스를 이용합니다.

이 말인 즉슨, 여러분이 명시적으로 정의한 자원이나 그 자원이 필요로 하는 자원들(VPC, Security Group...)외에도 NestedStack이 추가로 배포된다는 의미입니다. 여기에는 AWS Lambda 함수, 여기에 필요한 IAM Role과 정책이 포함됩니다. 

이는 우리가 워크샵에서 생성한 `ClusterStack`과 `ContainerStack`의 동작 양상에 대한 답이 됩니다. 
`ContainerStack`을 별도로 생성해서 컨테이너 자원을 정의하기는 하지만, 실제로 배포할 때에는 `ClusterStack` 에서 생성된 NestedStack들이 일을 합니다. 그래서 컨테이너 자원을 배포할 때에도 `ContainerStack`이 아닌 `ClusterStack`을 `cdk deploy`하는 것입니다.

We have defined two stacks. One is the stack for the EKS cluster itself, the other is the stack defining the container.
However, as shown in the screenshot above, when we go into the CloudFormation console, we can see that additional stacks which we did not even touch have been deployed. Why is that?

At this point, the AWS CloudFormation resource `AWS::EKS::Cluster` does not have native control over Kubernetes resources. In other words, CloudFormation cannot run kubectl directly. So, if you need to create a resource using a manifest or issue a command directly to a cluster--IAM role mapping and etc--the Amazon EKS construct library uses a custom resource.

This means that NestedStack is deployed as well in the exactly same way CDK did for implicit resources (VPC, Security Group ...). This NestedStack includes AWS Lambda functions, IAM Roles and policies required for them.

It is why we called `ClusterStack` when deploying containers, even though we are defining containers in `ContainerStack`. NestedStacks created in `ClusterStack` work for deployment, not `ContainerStack`. 

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
