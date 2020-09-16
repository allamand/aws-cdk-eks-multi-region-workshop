---
title: Deploy to two regions
weight: 300
pre: "<b>6-3. </b>"
---

Now that we have verified that it works in the first region, shall we deploy it in the second region now?

![](/images/40-deploy-app/pipeline.svg)


As above, let's add the steps to 1) approve the deployment by administrator, and 2) deploy it to the second region.
In this pipeline, deployment is done in the second region, with source and container images from the first region without replicating any resources.
If you have disaster recovery requirements, such as cross-region replication, please refer to [this link](/en/80-appendix/related-bps/ecr-replication/).

One more thing to keep in mind is this scenario assumes that the deployment is verified in the first region before it goes to the second region. If deployment succeeds in the first region but fails in the second region, only the second region is rolled back without the first region being rolled back.

### Export IAM role for CI/CD Pipeline
![](/images/40-deploy-app/2nd-region-pipeline.svg)

As in the previous step in single region deployment, the actual deployment is performed by assuming a privileged role in the region's EKS cluster.
The reason for not using the `for-1st-region` created earlier is to ensure that each role has permission only for clusters in the region. So let's create a Role that can run kubectl commands to the EKS cluster in the second region.
Open **lib/cluster-stack.ts** and add / modify the code below in order.

1. Above `constructor`
    ```typescript
    public readonly secondRegionRole: iam.Role;

    ```

2. Inside `constructor`, the `if` part
    ```typescript
        if (cdk.Stack.of(this).region==primaryRegion) {
            this.firstRegionRole = createDeployRole(this, `for-1st-region`, cluster);
        }
        else {
            this.secondRegionRole = createDeployRole(this, `for-2nd-region`, cluster);
        }
    ```

3. Outside of `class`, interface named `CicdProps`
    ```typescript
    export interface CicdProps extends cdk.StackProps {
        firstRegionCluster: eks.Cluster,
        secondRegionCluster: eks.Cluster,
        firstRegionRole: iam.Role,
        secondRegionRole: iam.Role
    }
    ```

The completed **lib/cluster-stack.ts** should look like this:

```typescript
import * as cdk from '@aws-cdk/core';
import * as iam from '@aws-cdk/aws-iam';
import * as eks from '@aws-cdk/aws-eks';
import * as ec2 from '@aws-cdk/aws-ec2';
import { PhysicalName } from '@aws-cdk/core';

export class ClusterStack extends cdk.Stack {
  public readonly cluster: eks.Cluster;
  public readonly firstRegionRole: iam.Role;
  public readonly secondRegionRole: iam.Role;

  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const primaryRegion = 'ap-northeast-1';
    const clusterAdmin = new iam.Role(this, 'AdminRole', {
      assumedBy: new iam.AccountRootPrincipal()
      });

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
    clusterName: `demogo`,
    mastersRole: clusterAdmin,
    defaultCapacity: 2,
    defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                new ec2.InstanceType('r5.2xlarge') : new ec2.InstanceType('m5.2xlarge')
    });

    cluster.addCapacity('spot-group', {
    instanceType: new ec2.InstanceType('m5.xlarge'),
    spotPrice: cdk.Stack.of(this).region==primaryRegion ? '0.248' : '0.192'
    });

    this.cluster = cluster;

    if (cdk.Stack.of(this).region==primaryRegion) {
      this.firstRegionRole = createDeployRole(this, `for-1st-region`, cluster);
  }
  else {

      this.secondRegionRole = createDeployRole(this, `for-2nd-region`, cluster);
  }

  }
}

function createDeployRole(scope: cdk.Construct, id: string, cluster: eks.Cluster): iam.Role {
  const role = new iam.Role(scope, id, {
    roleName: PhysicalName.GENERATE_IF_NEEDED,
    assumedBy: new iam.AccountRootPrincipal()
  });
  cluster.awsAuth.addMastersRole(role);

  return role;
}
export interface EksProps extends cdk.StackProps {
  cluster: eks.Cluster
}

export interface CicdProps extends cdk.StackProps {
  firstRegionCluster: eks.Cluster,
  secondRegionCluster: eks.Cluster,
  firstRegionRole: iam.Role,
  secondRegionRole: iam.Role
}
```


### Create a CodeBuild project to deploy to the second region

Go to the **lib/cicd-stack.ts** file.
Paste the code below. Please make sure it is defined above the CodePipeline definition.

```typescript
        const deployTo2ndCluster = deployToEKSspec(this, secondaryRegion, props.secondRegionCluster, ecrForMainRegion, props.secondRegionRole);

```
* In the /utils folder, we have pre-defined build specifications for this workshop. If you are curious about the detailed build specifications, please refer to the **/utils/buildspec.ts** file.




### Modify CodePipeline
Add the following stages to the declared CodePipeline construct.
```typescript
            ,{
                stageName: 'ApproveToDeployTo2ndRegion',
                actions: [ new pipelineAction.ManualApprovalAction({
                        actionName: 'ApproveToDeployTo2ndRegion'
                })]
            },
            {
                stageName: 'DeployTo2ndRegionCluster',
                actions: [ new pipelineAction.CodeBuildAction({
                    actionName: 'DeployTo2ndRegionCluster',
                    input: sourceOutput,
                    project: deployTo2ndCluster
                })]
            }
```

* This workshop does not include any additional information that can be consulted for approval. When using it in production, configure it to add the information required by the actual approver.
 
The completed code should look like this.
```typescript
import * as cdk from '@aws-cdk/core';
import codecommit = require('@aws-cdk/aws-codecommit');
import ecr = require('@aws-cdk/aws-ecr');
import codepipeline = require('@aws-cdk/aws-codepipeline');
import pipelineAction = require('@aws-cdk/aws-codepipeline-actions');
import { codeToECRspec, deployToEKSspec } from '../utils/buildspecs';
import { CicdProps } from './cluster-stack';


export class CicdStack extends cdk.Stack {

    constructor(scope: cdk.Construct, id: string, props: CicdProps) {

        super(scope, id, props);

        const primaryRegion = 'ap-northeast-1';
        const secondaryRegion = 'us-west-2';

        const helloPyRepo = new codecommit.Repository(this, 'hello-py-for-demogo', {
            repositoryName: `hello-py-${cdk.Stack.of(this).region}`
        });
        
        new cdk.CfnOutput(this, `codecommit-uri`, {
            exportName: 'CodeCommitURL',
            value: helloPyRepo.repositoryCloneUrlHttp
        });
        const ecrForMainRegion = new ecr.Repository(this, `ecr-for-hello-py`);

        const buildForECR = codeToECRspec(this, ecrForMainRegion.repositoryUri);
        ecrForMainRegion.grantPullPush(buildForECR.role!);
        
        const deployToMainCluster = deployToEKSspec(this, primaryRegion, props.firstRegionCluster, ecrForMainRegion, props.firstRegionRole);
        const deployTo2ndCluster = deployToEKSspec(this, secondaryRegion, props.secondRegionCluster, ecrForMainRegion, props.secondRegionRole);

        const sourceOutput = new codepipeline.Artifact();

        new codepipeline.Pipeline(this, 'multi-region-eks-dep', {
            stages: [ {
                    stageName: 'Source',
                    actions: [ new pipelineAction.CodeCommitSourceAction({
                            actionName: 'CatchSourcefromCode',
                            repository: helloPyRepo,
                            output: sourceOutput,
                        })]
                },{
                    stageName: 'Build',
                    actions: [ new pipelineAction.CodeBuildAction({
                        actionName: 'BuildAndPushtoECR',
                        input: sourceOutput,
                        project: buildForECR
                    })]
                },
                {
                    stageName: 'DeployToMainEKScluster',
                    actions: [ new pipelineAction.CodeBuildAction({
                        actionName: 'DeployToMainEKScluster',
                        input: sourceOutput,
                        project: deployToMainCluster
                    })]
                },{
                    stageName: 'ApproveToDeployTo2ndRegion',
                    actions: [ new pipelineAction.ManualApprovalAction({
                            actionName: 'ApproveToDeployTo2ndRegion'
                    })]
                },
                {
                    stageName: 'DeployTo2ndRegionCluster',
                    actions: [ new pipelineAction.CodeBuildAction({
                        actionName: 'DeployTo2ndRegionCluster',
                        input: sourceOutput,
                        project: deployTo2ndCluster
                    })]
                }
                
                
            ]
        });
        
    }
}
```
### Modify the entry point
Modify the **bin/multi-cluster-ts.ts** file as shown below so that the role to perform deployment of the second region can be injected.

```typescript
new CicdStack(app, `CicdStack`, {env: primaryRegion, 
    firstRegionCluster: primaryCluster.cluster,
    secondRegionCluster: secondaryCluster.cluster,
    firstRegionRole: primaryCluster.firstRegionRole,
    secondRegionRole: secondaryCluster.secondRegionRole});

```

* `secondRegionCluster: secondaryCluster.cluster`, `secondRegionRole: secondaryCluster.secondRegionRole` are added.

### Deploy CI/CD Pipeline

After checking the resources to be created with the `cdk diff` command, deploy the CI / CD pipeline using ` cdk deploy "*" --require-approval never` command.


{{% notice warning %}}

Please note that the above command omits the consent to modify security-related resources such as IAM, in order to simplify the workshop phase.
{{% /notice %}}

You can check the progress in the terminal.

```
  ...
  8/19 | 오후 10:28:52 | UPDATE_IN_PROGRESS   | AWS::CodePipeline::Pipeline | multi-region-eks-dep (repotoecrhellopy560FED9C)
  9/19 | 오후 10:28:53 | UPDATE_COMPLETE      | AWS::CodePipeline::Pipeline | multi-region-eks-dep (repotoecrhellopy560FED9C)
  9/19 | 오후 10:29:00 | UPDATE_COMPLETE_CLEA | AWS::CloudFormation::Stack  | CicdStack
 10/19 | 오후 10:29:01 | UPDATE_COMPLETE      | AWS::CloudFormation::Stack  | CicdStack

 ✅  CicdStack

Outputs:
CicdStack.codecommituri = https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/hello-py-ap-northeast-1
CicdStack.ExportsOutputFnGetAttdeploytoeksapnortheast1Role18912335ArnB28E7CE7 = arn:aws:iam::<<ACCOUNT_ID>>:role/CicdStack-deploytoeksapnortheast1Role189-VIG05L6PETHL
```


### Trigger the modified pipeline again

![](/images/40-deploy-app/release-change.png)
  
You can see the two newly deployed stages in the [CodePipeline Console](https://console.aws.amazon.com/codesuite/codepipeline/home).
Click 'Release Changes' in the screenshot above to reflect the latest code to the second region.

### Approve the release

After the application is successfully deployed to the EKS cluster in the first region by the third stage of the pipeline, you can see that it is in the status of waiting for approval as shown in the screenshot below.
Press the **<Review>** button, then press the **<Apply>** button.

  
![](/images/40-deploy-app/approval-pipeline-en.png)


### Check the deployed resources
1. Let's check the deployed container by the `kubectl` command.

{{% notice warning %}}
Verify that you are working on a cluster in the us-west-2 region via the `kubectl config current-context` command.
{{% /notice %}}

    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    hello-py-576f77b98b-kj7bf           0/1     Pending   0          85s  << Application Pod
    hello-py-576f77b98b-lkqwb           0/1     Pending   0          85s  << Application Pod
    hello-py-576f77b98b-wln9l           0/1     Pending   0          85s  << Application Pod
    metrics-server-6b6bbf4668-vqhjl     1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-5jdkg   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-hns6v   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-kzk8l   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-r7wf7   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-xlxxh   1/1     Running   0          3h58m
    ```

2. Check if the response comes back from `EXTERNAL-IP` of the created service object.
Please note that ELB generation may take about 2 minutes.

    ```
    kubectl describe service hello-py | grep Ingress

    # Result
    LoadBalancer Ingress:     aed0099fad25846a3a469d6abd64926d-847916387.ap-northeast-1.elb.amazonaws.com
    ```

    ```
    curl <LoadBalancer Ingress>

    # Result
    Hello World from us-west-2
    ```
