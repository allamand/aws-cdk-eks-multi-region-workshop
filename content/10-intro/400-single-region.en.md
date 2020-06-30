---
title: Lab 1, Create an EKS cluster with CDK
weight: 400
pre: "<b>1-3. </b>"
---

![](/images/20-single-region/intro.svg)

In this lab, we write a construct to provision EKS cluster with re-usable ones that CDK provides. We will also write another construct to provision Kubernetes resources. This enables you managing infrastructure and long-running containers at one view!

If you are not interested in deploying EKS clusters into more than one region, please feel free to stop at Lab 1.