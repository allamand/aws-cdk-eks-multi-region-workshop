---
title: Node.js
weight: 300
pre: "<b>2-3. </b>"

---

AWS CDK는 내부적으로 Node.js (>= 10.3.0)를 이용하도록 설계되어 있습니다.  
CDK의 정상 동작을 위해 Node.js를 설치합니다.

* Node.js 를 설치하기 위해서는 [node.js](https://nodejs.org) 사이트를 방문하십시오.

    * __Windows__: 오래된 Node.js 버전이 설치된 경우 관리자로 .msi installation 을 실행해야 할 수 있습니다.

* 이미 Node.js 가 설치되어 있다면, CDK와 함께 사용할 수 있는 버전인지 확인하십시오:

    ```
    node --version
    ```

    이 때 출력된 결과값이 10.3.0 이상 버전이어야 합니다:

    ```
    v10.3.0
    ```
