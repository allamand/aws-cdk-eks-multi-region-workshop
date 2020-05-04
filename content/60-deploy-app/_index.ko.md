---
title: 두 리전에 어플리케이션 배포하기
weight: 60
pre: "<b>6. </b>"
---

**이제 두 리전에 EKS클러스터가 모두 생성되었습니다.  
그럼 개발자 입장에서 새로운 코드를 개발했는데 이 코드를 두 리전 모두에 배포하고 싶으면 어떻게 해야 할까요?  
이 작업을 가능하게 해주는, 두 리전에 걸친 파이프라인을 만들어보겠습니다.** <br/><br/>

### 실습 진행 단계 
![](/images/40-deploy-app/intro2.svg)
{{% children showhidden="false" %}}


파이프라인의 구성 내용을 살펴보면 아래와 같습니다.
![](/images/40-deploy-app/pipeline.svg)
1. 개발자가 코드를 CodeCommit에 체크인합니다.
2. CodeBuild를 이용해 체크인된 코드를 빌드하고, 성공하면 ECR에 이미지를 푸시합니다.
3. 이 작업이 성공하면 EKS 클러스터에 해당 이미지를 반영하는 컨테이너 자원을 생성합니다.
4. 이 클러스터에서 정상적으로 동작함이 확인되면 승인 과정을 거칩니다.
5. 승인이 완료되면 두번째 클러스터에 컨테이너가 마찬가지로 배포됩니다.

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
