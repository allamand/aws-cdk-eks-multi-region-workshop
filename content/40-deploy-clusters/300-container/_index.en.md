---
title: Create ContainerStack 
weight: 300
pre: "<b>4-3. </b>"
---


Managing Kubernetes resources has never been easy. To successfully manage a Kubernetes cluster, there are a number of Kubernetes objects that we have to deal with!

Starting from the namespace, there will be containers for monitoring, logging, containers for autoscaling, and so on.
It’s already crazy enough managing for one cluster, imagine have to manage all of these for multiple clusters?

There are two ways you can manage Kubernetes resources along with your infrastructure in CDK.

1. [KubernetesResource](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.KubernetesResource.html) Object, using Kubernetes manifests.


2. [HelmChart](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-eks.HelmChart.html)

