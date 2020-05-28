---
title: Multi-region operation strategy and Active-Active architecture
weight: 100
pre: "<b>5-1. </b>"
---


Before we get into it, let's take a look at what the multi-regional strategy this workshop assumes.

There may be several strategies to utilize another region other than the main region.
The Active-Active configuration distributes all resources and performs actual services in the same way as the main region.
There is also a way to preserve only the main services, and a few kind of replicated data, and deploy other resources only in case of emergency. Or you can just have backups in the secondary region.

## Why do you need an active-active multi-region architecture?


So why do you need an Active-Active configuration that deploys the same level of resources in two regions?

1. To provide optimal service to geographically distant customers.
2. To comply with data sovereignty policies in different countries.
3. To achieve very tight RPO and RTO when a disaster happens.

## Points to consider in the Active-Active architecture

The architectural considerations can be divided into five main areas.
Depending on what you aim for, as describe in previous paragraph, your actual architecture would have different weight in the following areas.

![](/images/10-intro/5pillars.png)

## The premise of this workshop

This workshop focused on **how we would manage things with CDK**. Therefore, there are major architectural elements such as Route 53, which definitely needs to be considered but not covered in this workshop. The workshop contains only the essential parts of showing the Operational Excellence practices with EKS clusters in perspective of IaC.
If you are curious about the other parts of the above five considerations, you can find more information at the link below.


## Reference
* [AWS re:Invent 2019: [REPEAT 2] Architecture patterns for multi-region active-active (ARC213-R2)](https://youtu.be/3K9AzSrCmiQ)