---
title: Deploy to 2nd Region
weight: 200
pre: "<b>5-2. </b>"
---


By the last step in [4-3 stage](/en/40-deploy-clusters/300-container/), An EKS cluster was deployed in one region and Kubernetes resources were deployed as well on top of it.
So how do you deploy this same configuration to the second region?

## Specify the second region to deploy
Let's open the **bin/multi-cluster-ts.ts** file and create a second stack.
For now, only stacks for primary region should be loaded at the entry point as shown below.

```typescript
const app = new cdk.App();

const account = app.node.tryGetContext('account') || process.env.CDK_INTEG_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT;
const primaryRegion = {account: account, region: 'ap-northeast-1'};
const secondaryRegion = {account: account, region: 'us-west-2'};


const primaryCluster = new ClusterStack(app, `ClusterStack-${primaryRegion.region}`, {env: primaryRegion });
new ContainerStack(app, `ContainerStack-${primaryRegion.region}`, {env: primaryRegion, cluster: primaryCluster.cluster });


```

Let's paste the code below to configure it to deploy the same stack to different regions.

```typescript
const secondaryCluster = new ClusterStack(app, `ClusterStack-${secondaryRegion.region}`, {env: secondaryRegion });
new ContainerStack(app, `ContainerStack-${secondaryRegion.region}`, {env: secondaryRegion, cluster: secondaryCluster.cluster });
```

## Deploying the stack
You can see that the stack is newly created in the second region by using the command below.
You can see which resources will be distributed by this stack.

```
cdk diff
```

You can see the same output as the resource created in steps 4-2 and 4-3.

Deploy the resource to the second region using the command below.
```
cdk deploy "*" --require-approval never
```
It takes about 15-20 minutes as it did in the [previous Step](/en/40-deploy-clusters/200-cluster).

{{% notice warning %}}
Please note that the above command omits your permission to modify security-related resources to simplify the workshop phase.
{{% /notice %}}



## Update kubeconfig

After the resource creation is completed, the ConfigCommand will be displayed as CloudFormation Output in the console.
Copy the `aws eks ...` after the `=` in the output below and run it from the console.

```
ClusterStack-us-west-2.demogoclusterConfigCommand6DB6D889 = aws eks update-kubeconfig --name demogo --region us-west-2 --role-arn <<YOUR_ROLE_ARN>>
```

If executed successfully, the following result will be displayed.

```
Updated context arn:aws:eks:us-west-2:<<ACCOUNT_ID>>:cluster/demogo in /Users/jiwony/.kube/config
```


The command below confirms that you can access the EKS cluster in both regions.
From the CLUSTER name, you can see that both the cluster in the Tokyo region we created earlier and the Oregon cluster we just created were successfully registered.

```
kubectl config get-contexts

# Result
CURRENT   NAME                                                     CLUSTER                                                  AUTHINFO                                                 NAMESPACE
          arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo   arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo   arn:aws:eks:ap-northeast-1:ACCOUNT_ID:cluster/demogo
*         arn:aws:eks:us-west-2:ACCOUNT_ID:cluster/demogo        arn:aws:eks:us-west-2:ACCOUNT_ID:cluster/demogo        arn:aws:eks:us-west-2:ACCOUNT_ID:cluster/demogo
```


## Checking resources in the 2nd cluster


Check the currently deployed pod through the `kubectl get pod` command in the current context.
Because it is in the `us-west-2` region, it is different from `yaml-us-west-2/00_us_nginx.yaml` **You can see that 5 nginx pods are distributed** as the contents of the file.

```
NAME                                READY   STATUS    RESTARTS   AGE
metrics-server-6b6bbf4668-xhvtd     1/1     Running   0          12m
nginx-deployment-5754944d6c-6lqmg   1/1     Running   0          12m
nginx-deployment-5754944d6c-6qn8f   1/1     Running   0          12m
nginx-deployment-5754944d6c-d25s6   1/1     Running   0          12m
nginx-deployment-5754944d6c-dnl9l   1/1     Running   0          12m
nginx-deployment-5754944d6c-q2p9c   1/1     Running   0          12m
```


You can change the target cluster for `kubectl` operation with the command below.

```
kubectl config use-context <<cluster-name>>

# Result
Switched to context "<<cluster-name>>".
```