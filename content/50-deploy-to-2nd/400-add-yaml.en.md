---
title: K8S Manifest 수정하기
weight: 400
pre: "<b>5-4. </b>"

---

그럼 이렇게 EKS 클러스터를 두 개 리전에 쿠버네티스 자원과 함께 배포했는데,  
이 위에 컨테이너 변경은 어떻게 처리될까요?


## 두 리전 모두에 새 manifest 추가하기
`/yaml-common/00-namespaces.yaml` 파일 가장 아래에 아래 코드를 붙여넣습니다.  
이 manifest를 통해 앞서 생성한 네임스페이스에 ResourceQuota를 부여합니다.

```
---
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


## 생성될 자원 확인하고 배포하기
아래 명령어를 통해 생성될 자원을 확인해볼까요?
```
cdk diff
```

그러면 아래와 같이 두 리전에 대해, 새롭게 추가된 yaml 분만이 변경분으로 반영될 것임을 알 수 있습니다.
```
Stack ClusterStack-us-west-2
Resources
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-01_resource-quota0/Resource demogoclustermanifest01resourcequota0097ABDB2
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-01_resource-quota1/Resource demogoclustermanifest01resourcequota13FACB412

Stack ContainerStack-ap-northeast-1
There were no differences
Stack ContainerStack-us-west-2
There were no differences
Stack ClusterStack-ap-northeast-1
Resources
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-01_resource-quota0/Resource demogoclustermanifest01resourcequota0097ABDB2
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-01_resource-quota1/Resource demogoclustermanifest01resourcequota13FACB412

```

## 배포된 컨테이너 변경분 확인하기
다음 명령어를 이용해 자원을 생성해봅시다.
```
cdk deploy "*"
```

자원 배포가 모두 완료되고 나면 다음 명령어로 새 manifest가 적용된 것을 확인할 수 있습니다.
```
kubectl describe ns management

# 결과값
Name:         management
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration: {"apiVersion":"v1","kind":"Namespace","metadata":{"annotations":{},"name":"management"}}
Status:       Active

Resource Quotas
 Name:            mem-cpu-demo
 Resource         Used  Hard
 --------         ---   ---
 limits.cpu       0     2
 limits.memory    0     2Gi
 requests.cpu     0     1
 requests.memory  0     1Gi

 Name:                                                    storage-ns1
 Resource                                                 Used  Hard
 --------                                                 ---   ---
 storagens2.storageclass.storage.k8s.io/requests.storage  0     0

No resource limits.
```