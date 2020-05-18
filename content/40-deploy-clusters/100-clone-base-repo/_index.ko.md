---
title: 베이스 프로젝트 클론하기
weight: 100
pre: "<b>4-1. </b>"
---

작업을 할 프로젝트를 클론하고 IDE에서 엽니다.

### 베이스 프로젝트 클론
```
git clone https://github.com/yjw113080/aws-cdk-eks-multi-region-skeleton
```

### IDE에서 열기
선호하시는 IDE에서 위 프로젝트 폴더를 열어 주십시오.  
참고로 이 워크샵은 VS Code를 이용하여 작성되었습니다.

### 디렉토리 구조 살펴보기
주요한 디렉토리 구조는 다음과 같습니다.
```bash
├── bin
│   └── multi-cluster-ts.ts
├── cdk.context.json
├── cdk.json
├── jest.config.js
├── lib
│   ├── cicd-stack.ts
│   ├── cluster-stack.ts
│   └── container-stack.ts
├── package-lock.json
├── package.json
├── test
│   └── ..
├── tsconfig.json
├── utils
│   ├── buildimage
│   │   ├── Dockerfile
│   │   └── entrypoint.sh
│   ├── buildspecs.ts
│   └── read-file.ts
├── yaml-ap-northeast-1
│   └── 00_ap_nginx.yaml
├── yaml-common
│   └── 00_namespaces.yaml
└── yaml-us-west-2
    └── 00_us_nginx.yaml
```
* `/bin/multi-cluster-ts`: 이 CDK 앱의 **엔트리포인트**입니다. 즉, CDK 앱이 실행될 때 가장 먼저 보는 지점입니다. 어떤 스택을 실행할 것인지가 등록되는 지점으로, 우리가 생성하는 스택을 실제로 배포하려면 여기에 추가해주어야 합니다. 향후 엔트리포인트 라는 단어가 워크샵에 등장하면, 이 파일을 뜻하는 것이라고 생각하시면 됩니다.
* `/lib`: 이 CDK 앱에서 우리가 작성할 Stack 파일이 위치합니다. `cluster-stack`, `container-stack`을 실습 1과 실습 2에서, `cicd-stack`을 실습 3에서 작성합니다.
* `/utils`: 스택 작성에 필요한 몇 가지 도구들입니다. Codebuild에서 필요한 빌드 베이스 이미지와 빌드 스펙, 컨테이너 생성을 위한 json converter가 있습니다.
* `/yaml-*`: 환경 별로 배포할 k8s manifest들이 위치합니다. 


## 의존성 패키지 설치하기
클론한 레포지토리에서 다음 명령어를 수행하여 의존성 패키지를 설치하십시오.

```
npm i
```

## 터미널 창에서 `npm run watch` 수행
앞서 설명한 것처럼 typescript 에서는 javascript로 트랜스파일 과정을 거쳐야 합니다.  
이 프로젝트 디렉토리에서도 변경분을 자동으로 컴파일하기 위해 [여기](/ko/30-cdk/200-watch/)를 참고하여 `npm run watch`가 백그라운드에서 실행하십시오.



아래 스크린샷은 VScode IDE에서 터미널을 실행하는 예시입니다.

![](/images/20-single-region/npm-run-watch.png)
