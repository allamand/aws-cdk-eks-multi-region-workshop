---
title: Modify K8S Manifest
weight: 400
pre: "<b>5-4. </b>"

---

So I deployed the EKS cluster in two regions with Kubernetes resources,
How will container changes be handled on top of this?


## Adding a new manifest to both regions
Paste the code below at the bottom of the `/yaml-common/00-namespaces.yaml` file.
ResourceQuota is assigned to the namespace created earlier by this manifest.

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


## Check what's going to happen
Shall we check the resources to be created with the command below?
```
cdk diff
```

Then, for the two regions below, you can see that only newly added yaml minutes will be reflected as changes.

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

## Deploy
Let's create the resource using the following command.

```
cdk deploy "*"
```

After the resource deployment is completed, you can see that the new manifest has been applied with the following command.

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