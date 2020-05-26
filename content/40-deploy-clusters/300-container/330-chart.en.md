---
title: Helm Chart
weight: 330
---

## Helm Chart?
![](https://helm.sh/img/helm.svg)
* Official homepage: https://helm.sh/

Helm is a tool to help you manage Kubernetes applications.
It uses a packaging format called Helm Chart, which allows you to define / install / upgrade / delete Kubernetes applications.
For example, when a deployment, a service for a web service that you service must be defined, and there is a required role, bundle it all and manage it like a package.
In addition to creating your own software, you can easily download and use software created by others using Helm. Typical examples are metrics-server and prometheus.

In this chapter, we will look at how you can deploy Helm Chart to EKS cluster with CDK.

## Deploy Helm Chart
Open **lib/container-stack.ts**.
Paste the following code inside `constructor`.

```typescript

    const stable = 'https://kubernetes-charts.storage.googleapis.com/';

    cluster.addChart(`metrics-server`, {
      repository: stable,
      chart: 'metrics-server',
      release: 'metrics-server'
    });

```


When installing Helm Chart, CDK's custom resource uses `helm upgrade --install` command. This means that if you have the same release name, it will try to update.
Helm Chart is distributed as a CloudFormation resource. Therefore, when it is deleted from the CDK code or deleted from the stack, it is deleted from the cluster through the `helm uninstall` command.

A few things to keep in mind when specifying your Helm Chart.
* `repository` must be a full URL. In general, not in CDK, before you issue the command `helm install stable/xx`, you register the repository called `stable` in advance. Similar to this, you should give the context of repository by giving the full path.
* If you don't specify a release name, it is randomized to use the last 53 characters of the node ID. If you do not want this, give the value of 'release' as human understandable.
* In addition, you can check [details] (https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.HelmChart.html) required for chart distribution.

## Identifying resources to be created
Let's check the resources to be created with the command below.

```
cdk diff
```

You can see that the chart will be created as shown below.

```
Stack ClusterStack-ap-northeast-1
Resources
[+] Custom::AWSCDK-EKS-HelmChart demogo-cluster/chart-metrics-server/Resource demogoclusterchartmetricsserver19E78457

Stack ContainerStack-ap-northeast-1
There were no differences
```


## Deploy
Let's deploy the container with the command below.
```
cdk deploy ClusterStack-ap-northeast-1
```
{{% notice info %}}
* If you have created a stack in another region, make sure the correct region name comes after `ClusterStack`.
* Why run `ClusterStack`, not `ContainerStack`? If you are curious, please check out [this](/en/80-appendix/how-cfn-addresource/).
{{% /notice %}}

The progress is printed on the terminal.
Use the following command to confirm that the new pod has been deployed by our definition of Helm Chart as shown below.

```
kubectl get pod
```


```
NAME                                READY   STATUS    RESTARTS   AGE
metrics-server-6b6bbf4668-vl455     1/1     Running   0          102s << the new Chart!
nginx-deployment-5754944d6c-cnxzn   1/1     Running   0          16m
nginx-deployment-5754944d6c-vrrd2   1/1     Running   0          16m
```