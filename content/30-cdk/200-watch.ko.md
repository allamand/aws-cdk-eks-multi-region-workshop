---
title: "npm run watch"
weight: 200
pre: "<b>3-2. </b>"
---


## TypeScript 코드 컴파일링

TypeScript 코드는 Javascript로 컴파일되어야 하기 때문에 소스 코드 변경 분을 확인하려면 계속해서 `.js`파일로 컴파일을 해주어야 합니다.

{{% notice info %}}
이후 워크샵 단계를 진행하는 데에 꼭 필요한 부분입니다. watch 터미널 세션을 워크샵 내내 열어두도록 해주세요.
{{% /notice %}}

프로젝트에는 `watch`라는 이름의 npm script 가 이미 설정되어 있어서, 이를 실행하면 매번 수동으로 컴파일 해줄 필요 없이 자동으로 변경분을 `.js` 파일로 컴파일해줍니다.


## 새로운 터미널 윈도우 열기

**새로운** 터미널 세션(혹은 탭)을 여십시오. 이 워크샵을 진행하는 동안, 이 윈도우를 계속해서 백그라운드에 실행 중인 상태로 둘 것입니다.

## 코드 변경분 watch 하기

프로젝트 디렉토리로 이동합니다:

```
cd cdk-workshop
```

그리고 watch 스크립트를 수행합니다:

```
npm run watch
```

그러면 터미널 창의 내용이 지워지고 다음과 같은 결과가 출력될 것입니다:

```
Starting compilation in watch mode...
Found 0 errors. Watching for file changes.
# 중략
```

이 스크립트는 TypeScript Compiler (`tsc`)를 "watch" 모드로 시작해서, 프로젝트 디렉토리를 모니터링 하여 `.ts` 파일의 변경 분을 `.js` 파일로 자동 컴파일해줍니다.

**‼️ 이 워크샵이 진행되는 동안 `watch` 스크립트 수행 터미널 윈도우가 꺼지지 않도록 해주십시오.**

