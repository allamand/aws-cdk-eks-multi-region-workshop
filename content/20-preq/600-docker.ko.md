---
title: Docker-cli 설치
weight: 600
pre: "<b>2-6. </b>"
---

**실습 3, 두 리전에 어플리케이션 배포하기** 에서, 배포에 사용될 컨테이너 빌드 이미지를 docker 로 말아서 AWS Cloud에 배포하게 됩니다. 이를 위해 docker-cli가 동작해야 합니다.

## Docker-cli 설치
다음 링크를 통해 설치를 진행하십시오.
* https://docs.docker.com/get-docker/


다음 명령어를 통해 설치를 확인하십시오.
```
docker --version

# 결과값 예시
Docker version 19.03.8, build afacb8b

```

## Docker 데몬 실행
다음 링크를 참고하여 Docker 데몬을 실행하십시오.
* Mac: https://docs.docker.com/docker-for-mac/install/#install-and-run-docker-desktop-on-mac
* Windows: https://docs.docker.com/docker-for-windows/install/#start-docker-desktop

다음 명령어를 실행하여 docker 데몬이 실행 중인 상태인지 확인하십시오.

```
docker ps -q
```

* 최초 설치하신 경우, 위 명령어를 수행했을 때 아무것도 출력되지 않습니다.
* 도커 데몬이 실행 중이 아닌 경우, `Error response from daemon: dial unix docker.raw.sock: connect: connection refused` 라고 에러가 발생합니다.