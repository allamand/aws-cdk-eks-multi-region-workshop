---
title: Cleanup
weight: 70
pre: "<b>7. </b>"
---

긴 시간 워크샵을 수행하시느라 고생 많으셨습니다.  


1. 애플리케이션 생성 시에 배포된 ELB를 삭제합니다.



2. 아래 명령어를 통해 이번 워크샵을 통해 생성한 CDK 스택을 삭제하십시오.

```
cdk destroy "*"
```

두 개 리전의 [CloudFormation 콘솔](console.aws.amazon.com/cloudformation/)에 접근하여 모든 스택이 사라진 것을 확인하십시오.

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
