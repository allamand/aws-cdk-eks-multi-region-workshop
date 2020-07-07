---
title: Create a Pipeline for one region
weight: 200
pre: "<b>6-2. </b>"

---

First, let's deploy the cloned code in the previous step to only one cluster in a single region.
Create a CodePipeline with the following flow.

![](/images/40-deploy-app/single-region-pipeline.svg)


Go to the IDE and follow the steps below to create a class that creates a CI / CD pipeline.

### Export IAM Role for CI/CD Pipeline
CodeBuild, which will be responsible for deploying the application to the EKS cluster, assumes a role to run kubectl commands against the EKS cluster in order to perform the actual deployment. Let’s create this Role. Open lib/cluster-stack.ts and add the below codes in order

1. Above `constructor`
```typescript
  public readonly firstRegionRole: iam.Role;
```

1. inside of `constructor`
```typescript
    if (cdk.Stack.of(this).region==primaryRegion) {
      this.firstRegionRole = createDeployRole(this, `for-1st-region`, cluster);
    }
```

1. outside of `class`, end of the file
```typescript
export interface CicdProps extends cdk.StackProps {
  firstRegionCluster: eks.Cluster,
  firstRegionRole: iam.Role
}
```

* Have you ever wondered what `createDeployRole` function is for in the skeleton state? It creates an IAM role to be assumed by Codebuild project which will be defined in **cicd-stack.ts**. It restricts the role to have a limited access to the specific cluster in one region by examining the region value of the stack.
* The function authorizes this role to enter the master group of the EKS cluster. In production, be sure to assign them to groups with the least privilege.


The completed **lib / cluster-stack.ts** should look like this:
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
        version: '1.16',
        defaultCapacity: 2
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                    new ec2.InstanceType('r5.2xlarge') : new ec2.InstanceType('m5.2xlarge')
    });

    cluster.addCapacity('spot-group', {
        instanceType: new ec2.InstanceType('m5.xlarge'),
        spotPrice: cdk.Stack.of(this).region==primaryRegion ? '0.248' : '0.192'
    });
    
    this.cluster = cluster;

    if (cdk.Stack.of(this).region==primaryRegion)
      this.firstRegionRole = createDeployRole(this, `for-1st-region`, cluster);
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
  firstRegionRole: iam.Role
}
```

### Configuring the CicdStack

The `cicd-stack.ts` file under the `lib` directory should be created as shown below.
Check the file cicd-stack.ts under the lib directory. Please, change primaryRegion and secondaryRegion accordingly.

```typescript
import * as cdk from '@aws-cdk/core';
import codecommit = require('@aws-cdk/aws-codecommit');
import ecr = require('@aws-cdk/aws-ecr');
import codepipeline = require('@aws-cdk/aws-codepipeline');
import pipelineAction = require('@aws-cdk/aws-codepipeline-actions');
import { codeToECRspec, deployToEKSspec } from '../utils/buildspecs';


export class CicdStack extends cdk.Stack {

    constructor(scope: cdk.Construct, id: string, props: CicdProps) {
        super(scope, id, props);

        const primaryRegion = 'ap-northeast-1';
        const secondaryRegion = 'us-west-2';

    }
}
```


We will modify the props created in step 1 to be injected.

1. Import the props which are exported from cluster-stack
    ```typescript
    import { CicdProps } from './cluster-stack';
    ```

2. Inject the props in the constructor
    ```typescript
        constructor(scope: cdk.Construct, id: string, props: CicdProps) {
    ```


The completed code would be like the following code block.

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
    }
}


```

### Create a CodeCommit repository

AWS CodeCommit is a service that securely hosts highly scalable private Git repositories, so you can use repositories privately without having to manage repository hosting servers, storage, etc., just like using git.

Paste the code below into the class `construct` declaration.

```typescript
        const helloPyRepo = new codecommit.Repository(this, 'hello-py-for-demogo', {
            repositoryName: `hello-py-${cdk.Stack.of(this).region}`
        });
        
        new cdk.CfnOutput(this, `codecommit-uri`, {
            exportName: 'CodeCommitURL',
            value: helloPyRepo.repositoryCloneUrlHttp
        });
```

* Create a CodeCommit repository.
* Print out the repository URI as a stack output.

### Create an ECR Repository

Container images created based on this code repository must be stored in a separate Image Registry.
You can use Amazon ECR for it. Amazon Elastic Container Registry (ECR) is a fully managed Docker container registry that enables developers to easily store, manage and deploy Docker container images.

Paste the code below under the line you defined for CodeCommit.

```typescript
        const ecrForMainRegion = new ecr.Repository(this, `ecr-for-hello-py`);
```
* Creates an ECR repository


### CodeBuild for Container image build

When the developer commits the code to the source, it should automatically build a new image that will capture the changes. We will use CodeBuild to accomplish this. AWS CodeBuild is a fully managed build service on the cloud that compiles source code, runs unit tests, and creates artifacts that are ready to deploy.

With a buildspec, you can define your build job.

1) In a typical application, a `buildspec.yml` file can be created and defined in the root path of the code repository.
2) You can also specify when defining CodeBuild project. In this workshop, we will define the buildspec files, which are commonly used across regions directly by CDK, to centrally manage them.


1. Paste the code below to create the ECR Repository.
```typescript
        const buildForECR = codeToECRspec(this, ecrForMainRegion.repositoryUri);
        ecrForMainRegion.grantPullPush(buildForECR.role!);
```


### CodeBuild for deployment to EKS
Let's deploy resources using CodeBuild.
Paste the code below right after the code you wrote.

```typescript
        const deployToMainCluster = deployToEKSspec(this, primaryRegion, props.firstRegionCluster, ecrForMainRegion, props.firstRegionRole);

```

* In the /utils folder, we have pre-defined build specifications for this workshop. If you are curious about the detailed build specifications, please refer to the **/utils/buildspec.ts** file.
  

### Pipeline for single-region deployment
Then, we will create a pipeline by connecting the resources created so far. Paste the code below.

```typescript
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
                }
                
            ]
        });
```

* `sourceOutput` is defined to pass committed code to Pipeline as an artifact.

{{% notice info %}}
In this workshop, we upload and use the build image directly to make this workshop easier. In production, it is recommended to manage the build image in a separate ECR.
{{% /notice %}}


The complete code of **lib/cicd-stack.ts** should look like this:
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
                        }
                        
                    ]
                });
        
    }
}
```

### Load the stack

Paste the code below into the **bin/multi-cluster-ts.ts** file.

```typescript
new CicdStack(app, `CicdStack`, {env: primaryRegion, cluster: primaryCluster.cluster ,
                                    firstRegionRole: primaryCluster.firstRegionRole});


```


### Deploy CI/CD Pipeline

After checking the resources by `cdk diff` command, deploy the CI / CD pipeline by `cdk deploy "*"` command.
At this time, apart from `CicdStack`, the change also occurs in `ClusterStack` to grant permission to resources in this stack.

When the creation is finished you can check the pipeline in the [CodePipeline Console](http://console.aws.amazon.com/codesuite/codepipeline/home). There is no master branch in CodeCommit yet, thus the pipeline will be on a failed status.

![](/images/40-deploy-app/result-pipeline-single-region.png)


### Test your pipeline
Then, shall we deploy the application using the created pipeline?
Please open the [Sample Application](https://github.com/yjw113080/aws-cdk-multi-region-sample-app) IDE.

1. Copy the CodeCommit URI value from the CloudFormation output.
```
CicdStack.codecommituri = https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/hello-py-ap-northeast-1
```

2. Register the codecommit repository in the application project with the following command.
```
git remote add codecommit <<Copied codecommit URI>>
```

3. In the [IAM User](https://console.aws.amazon.com/iam/home?region=us-west-2) console, create the HTTPS Git credentials for AWS CodeCommit of the IAM user your terminal is currently using. If you need help, please see [this link](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html).


![](/images/40-deploy-app/iam-git-credential.png)

If you click Create Credentials, you can download the credentials as shown below. Download these credentials to use in 5th step.

![](/images/40-deploy-app/credential-detail.png)


4. Push the code in the directory to codecommit with the following command:
```
git add .
git commit -am "initial commit"
git push codecommit master
```


5. When prompted for the ID and PW while you run the command above, enter the ID / PW according to the credentials created in step 3.

6. After you push to the repository, you can see in the CodePipeline console that the pipeline is triggered as follows:
![](/images/40-deploy-app/result-pipeline-single-region-working.png)


7. Let's check the deployed container by `kubectl` command.

{{% notice warning %}}
Verify that you are working on a cluster in the primary region with the command `kubectl config current-context`.
If changes are needed, run the command `kubectl config use-context << ap-northeast-1 cluster >>`.
You can check the context name through `kubectl config get-contexts`.
{{% /notice %}}

```
NAME                                READY   STATUS    RESTARTS   AGE
hello-py-9b9bffb64-2f5bl            1/1     Running   0          3m7s << Application Pod
hello-py-9b9bffb64-btvts            1/1     Running   0          3m7s << Application Pod
hello-py-9b9bffb64-xttnw            1/1     Running   0          3m7s << Application Pod
metrics-server-6b6bbf4668-hlgmd     1/1     Running   0          24m
nginx-deployment-5754944d6c-whrmn   1/1     Running   0          24m
nginx-deployment-5754944d6c-wkkkn   1/1     Running   0          24m
```

1. Check if the response comes back from `EXTERNAL-IP` of the created service object.
 

```
kubectl describe service hello-py | grep Ingress

# Result
LoadBalancer Ingress:     aed0099fad25846a3a469d6abd64926d-847916387.ap-northeast-1.elb.amazonaws.com
```

```
curl <LoadBalancer Ingress>

# Result
Hello World from ap-northeast-1
```
