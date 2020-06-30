---
title: Lab 3, Deploy apps in both regions
weight: 60
pre: "<b>6. </b>"
---

EKS clusters are now created in both regions.
So, if you've developed new code from a developer's point of view and want to distribute it in both regions, what do you do?
Let's create a pipeline that spans two regions that makes this possible. <br/><br/>

### Lab Overview
![](/images/40-deploy-app/intro2.svg)
{{% children showhidden="false" %}}


The structure of the pipeline is as follows.
![](/images/40-deploy-app/pipeline.svg)

1. The developer checks the code into CodeCommit.
2. Build the checked-in code using CodeBuild and, if successful, push the image to ECR.
3. If this is successful, create a container resource that reflects the image in the EKS cluster.
4. When it is confirmed that it is operating normally in this cluster, it goes through the approval process.
5. Once approval is complete, the containers are deployed to the second cluster as well.

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
