---
title: Change infrastructure settings by region
weight: 300
pre: "<b>5-3. </b>"
---


For some situations when you have, for example, same pods on both regions but different number of replicas due to traffic demand, or you want different types of EC2 instances depending of the region you are running, you need to handle the cluster configuration separately.

Let's look at this example.

## Change to a different instance type by the region
Let's look at the excerpt of our cluster definition in **lib/cluster-stack.ts**.


```typescript

    const clusterAdmin = new iam.Role(this, 'AdminRole', {
      assumedBy: new iam.AccountRootPrincipal()
      });

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        version: '1.16',
        defaultCapacity: 2
    });

    cluster.addCapacity('spot-group', {
      instanceType: new ec2.InstanceType('m5.xlarge'),
      spotPrice: cdk.Stack.of(this).region==primaryRegion ? '0.248' : '0.192'
    });


```

Let’s re-write this part a little bit to have another instance type by referring to the region value of the stack. Paste the following code in the eks.Cluster creation section.
Paste the following code in the eks.Cluster creation section.

```typescript
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                 new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.2xlarge')
```

The completed code should look like this.
```typescript
      const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        version: '1.16',
        defaultCapacity: 2,
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                 new ec2.InstanceType('r5.2xlarge') : new ec2.InstanceType('m5.2xlarge')
      });
```

We changed the instance type by querying the region value with `cdk.Stack.of (this).region`.
As with any other programming, you can create conditional control over the values.
If you check out the change set by `cdk diff` command, it will be as follows.

```
Stack ClusterStack-us-west-2
Resources
[~] AWS::AutoScaling::LaunchConfiguration demogo-cluster/DefaultCapacity/LaunchConfig demogoclusterDefaultCapacityLaunchConfig93D71520 replace
 └─ [~] InstanceType (requires replacement)
     ├─ [-] m5.large
     └─ [+] m5.2xlarge

Stack ContainerStack-ap-northeast-1
There were no differences
Stack ContainerStack-us-west-2
There were no differences
Stack ClusterStack-ap-northeast-1
Resources
[~] AWS::AutoScaling::LaunchConfiguration demogo-cluster/DefaultCapacity/LaunchConfig demogoclusterDefaultCapacityLaunchConfig93D71520 replace
 └─ [~] InstanceType (requires replacement)
     ├─ [-] m5.large
     └─ [+] r5.2xlarge
```

## Verify resource creation

If you deploy the stack with `cdk deploy "*"`, you can see that the instance is replaced and the Launch Configuration for AutoScaling is changed as shown below.
```
...
  2/28 | 오후 10:44:50 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::LaunchConfiguration | demogo-cluster/DefaultCapacity/LaunchConfig (demogoclusterDefaultCapacityLaunchConfig93D71520) Resource creation Initiated
  3/28 | 오후 10:44:50 | UPDATE_COMPLETE      | AWS::AutoScaling::LaunchConfiguration | demogo-cluster/DefaultCapacity/LaunchConfig (demogoclusterDefaultCapacityLaunchConfig93D71520)
  3/28 | 오후 10:44:58 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F)
  3/28 | 오후 10:45:03 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Rolling update initiated. Terminating 2 obsolete instance(s) in batches of 1.
  3/28 | 오후 10:45:04 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Terminating instance(s) [i-05a08f1adf2d546ab]; replacing with 1 new instance(s).
 3/28 Currently in progress: demogoclusterDefaultCapacityASGCD1D544F
  3/28 | 오후 10:45:57 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Successfully terminated instance(s) [i-05a08f1adf2d546ab] (Progress 50%).
  3/28 | 오후 10:45:57 | UPDATE_IN_PROGRESS   | AWS::AutoScaling::AutoScalingGroup    | demogo-cluster/DefaultCapacity/ASG (demogoclusterDefaultCapacityASGCD1D544F) Terminating instance(s) [i-03a198d7d12a8b79b]; replacing with 1 new instance(s).
  ...

```

When you're done, you can see that the instance type has changed for each region as shown in the screenshot below.

![](/images/20-deploy-clusters/ap-infra-change.png)
