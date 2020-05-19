---
title: 실습 1, CDK로 EKS 클러스터 생성
weight: 40
pre: "<b>4. </b>"
---

CDK로 반복 재사용 가능한 Construct를 작성해서 하나의 리전에 EKS 클러스터를 배포하고,
그 위에 컨테이너 자원을 배포해봅니다. <br/><br/>

### 실습 진행 단계

![](/images/20-single-region/intro.svg)

{{% children showhidden="false" %}}

### 주의사항
이 워크샵에서는 다양한 종류의 AWS 클라우드 자원을 생성합니다. 원활한 워크샵 진행을 위해 생성될 자원의 종류 및 개수, 각각의 기본 서비스 리밋을 [이 페이지](/ko/80-appendix/limit)에서 확인해주시기 바랍니다. 생성될 자원이 리밋 범위를 초과하는 상황이라면 자원을 정리하거나 다른 리전에서 워크샵을 수행해주십시오.

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
