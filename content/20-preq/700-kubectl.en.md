---
title: kubectl 및 aws-iam-authenticator 설치
weight: 700
pre: "<b>2-7. </b>"
---

## kubectl 설치
다음 링크를 통해 자신의 OS에 맞는 방법으로 kubectl 을 설치하십시오.
* https://kubernetes.io/docs/tasks/tools/install-kubectl/ 
* 이 워크샵에서 버전은 크게 중요하지 않지만, 생성할 EKS 클러스터는 1.14 버전을 사용하므로 참고하시기 바랍니다.

다음 명령어를 통해 설치를 확인할 수 있습니다.

```
kubectl version --client

# 결과값 예시
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.1", GitCommit:"d647ddbd755faf07169599a625faf302ffc34458", GitTreeState:"clean", BuildDate:"2019-10-02T23:49:20Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"darwin/amd64"}
```


## aws-iam-authenticator 설치

Amazon EKS를 사용할 때에는, IAM 자격 증명을 이용해 쿠버네티스 클러스터 인증을 할 수 있도록 AWS IAM authenticator 을 함께 사용합니다. AWS IAM authenticator를 설치하고 이를 인증에 사용하도록 kubectl 구성 파일을 수정합니다. 이 과정을 거쳐 위에서 설치한 kubectl 클라이언트가 Amazon EKS와 함께 작동하게 구성할 수 있습니다.

다음 링크에 접속하여 aws-iam-authenticator를 설치합니다.
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-aws-iam-authenticator.html

다음 명령어를 통해 설치를 확인할 수 있습니다.
```
aws-iam-authenticator help
```
