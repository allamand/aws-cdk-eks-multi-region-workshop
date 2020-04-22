---
title: K8S Manifest 변경 반영 테스트하기
weight: 420
---

그럼 이렇게 EKS 클러스터를 두 개 리전에 쿠버네티스 자원과 함께 배포했는데,  
이 위에 컨테이너 변경은 어떻게 처리될까요?


## 두 리전 모두에 새 manifest 추가하기
`yaml-common` 폴더에 아래와 같이 새 파일을 생성합니다. 파일 이름은 자유롭게 주셔도 좋습니다.
이 manifest를 통해 앞서 생성한 네임스페이스에 ResourceQuota를 부여합니다.

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
  namespace: management
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-ns1
  namespace: management
spec:
  hard:
    storagens2.storageclass.storage.k8s.io/requests.storage: 0
```

아래 명령어를 통해 생성될 자원을 확인해볼까요?
```
cdk diff
```
그러면 아래와 같이 변경이 발생할 것임을 알 수 있습니다.


## 리전 별 자원 변경하기

## 생성될 자원 확인하고 배포하기

## 배포된 컨테이너 변경분 확인하기