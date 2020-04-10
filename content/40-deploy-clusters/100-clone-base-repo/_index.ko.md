---
title: 베이스 프로젝트 클론하기
weight: 100
---

작업을 할 프로젝트를 클론하고 IDE에서 엽니다.

### 베이스 프로젝트 클론
```
git clone https://github.com/yjw113080/aws-cdk-eks-multi-region-skeleton
```

### IDE에서 열기
선호하시는 IDE에서 위 프로젝트 폴더를 열어 주십시오.
> VSCode를 이용하시는 경우, 프로젝트 디렉토리에서 `code .`를 입력하면 IDE를 열 수 있습니다.

![](images/20-deploy-clusters/ide.png)

### 디렉토리 구조 살펴보기
주요한 디렉토리 구조는 다음과 같습니다.
```bash
├── bin
│   └── multi-cluster-ts.ts
├── cdk.context.json
├── cdk.json
├── jest.config.js
├── lib
│   ├── cicd-for-primary-region-stack.ts
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
└── yaml-us-east-1
    └── 00_us_nginx.yaml
```
* `/bin/multi-cluster-ts`: 이 CDK 앱의 엔트리 포인트입니다.
* `/lib`: 이 CDK 앱에서 우리가 작성할 Stack 파일이 위치합니다. `cluster-stack`, `container-stack`을 실습 1에서, `cicd-for-primary-region-stack`을 실습 2에서 작성합니다.
* `/utils`: 스택 작성에 필요한 몇 가지 도구들입니다. Codebuild에서 필요한 빌드 베이스 이미지와 빌드 스펙, 컨테이너 생성을 위한 json converter가 있습니다.
* `/yaml-*`: 환경 별로 배포할 k8s manifest들이 위치합니다. 


## 의존성 패키지 설치하기
클론한 레포지토리에서 다음 명령어를 수행하여 의존성 패키지를 설치하십시오.

```
npm i
```