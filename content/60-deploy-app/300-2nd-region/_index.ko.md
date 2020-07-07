---
title: 두 개 리전에 배포 테스트
weight: 300
pre: "<b>6-3. </b>"
---

첫 번째 리전에서 정상 동작하는 것을 확인했으니, 이제 두 번째 리전에도 배포를 해볼까요?

![](/images/40-deploy-app/pipeline.svg)

위와 같이 1) 첫 번째 리전에 정상 동작 여부 확인한 뒤 2) 수동 승인 과정을 거쳐 3) 두 번째 리전에 배포하도록 단계를 추가해보겠습니다.
이 파이프라인에서는 별도의 복제 과정 없이, 첫 번째 리전에서의 소스와 컨테이너 이미지를 가지고 두 번째 리전에 배포가 이루어지게 됩니다.  
리전 간 복제 등, DR 요소를 갖추고자 하신다면 [이 링크](/ko/80-appendix/related-bps/ecr-replication/)를 참고하십시오.

한 가지 더 유의해야 할 점은, 첫 번째 리전에서 문제가 없을 때까지 검증한 뒤에 두 번째 리전에 배포된다는 시나리오입니다.  
첫 번째 리전에서 배포가 성공했지만 두 번째 리전에서 실패하는 경우에는 첫 번째 리전의 롤백 없이 두 번째 리전만 롤백 됩니다.


### CI/CD Pipeline에서 사용할 IAM Role 내보내기
![](/images/40-deploy-app/2nd-region-pipeline.svg)

앞 단계에서 배포를 수행한 것처럼, 두번째 리전에 배포할 때에도 그 리전 EKS 클러스터에 권한이 있는 Role을 assume 해서 실제 배포를 수행합니다.   
앞에서 생성한 `for-1st-region`을 사용하지 않는 이유는, 각 롤이 해당 리전의 클러스터에만 권한을 갖도록 하기 위함입니다. 그럼 두 번째 리전의 EKS 클러스터에 kubectl 명령을 보낼 수 있는 Role을 생성해봅시다.
**lib/cluster-stack.ts**을 열어 아래 코드를 순서대로 추가/수정합니다.


1. `constructor` 위
    ```typescript
    public readonly secondRegionRole: iam.Role;

    ```

2. `constructor` 내부, if 절 수정
    ```typescript
        if (cdk.Stack.of(this).region==primaryRegion) {
            this.firstRegionRole = createDeployRole(this, `for-1st-region`, cluster);
        }
        else {
            // 스택이 두 번째 리전값을 가지고 있으면, 새로운 롤을 생성하되 그 리전 클러스터에만 접근할 수 있는 권한을 부여합니다.
            this.secondRegionRole = createDeployRole(this, `for-2nd-region`, cluster);
        }
    ```

3. `class` 외부, 최하단 interface 수정
    ```typescript
    export interface CicdProps extends cdk.StackProps {
    firstRegionCluster: eks.Cluster,
    secondRegionCluster: eks.Cluster,
    firstRegionRole: iam.Role,
    secondRegionRole: iam.Role
    }

    ```

완성된 **lib/cluster-stack.ts**는 다음과 같을 것입니다.

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
      clusterName: PhysicalName.GENERATE_IF_NEEDED,
      version: '1.16',
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

### 두 번째 리전에 배포하는 CodeBuild 프로젝트 생성하기

**lib/cicd-stack.ts** 파일로 이동합니다.  
지금까지 작성된 코드에 아래 코드를 붙여넣습니다. 단, CodePipeline 정의 부분보다 위에 정의해야 합니다. 

```typescript
        const deployTo2ndCluster = deployToEKSspec(this, secondaryRegion, props.secondRegionCluster, ecrForMainRegion, props.secondRegionRole);

```
* `/utils/buildspec.ts`에 정의된 빌드 스펙을 가지고 CodeBuild 프로젝트를 생성했습니다. 자세한 내용이 궁금하시면 해당 파일을 열어 확인하십시오.



### CodePipeline 수정
선언된 CodePipeline construct에 아래 스테이지를 추가합니다.
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

* 이 워크샵에서는 승인할 때에 참고할 수 있는 별도의 정보를 포함하지 않았습니다. 프로덕션에서 이용하실 때에는 실제 승인자가 필요로 하는 정보를 추가하도록 구성하십시오. 
 
완성된 코드는 아래와 같을 것입니다.
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
                },
                {
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

### 엔트리포인트 수정하기
두 번째 리전의 배포를 수행할 Role을, 스택 생성시 주입 받을 수 있도록 아래와 같이 **bin/multi-cluster-ts.ts** 파일을 수정합니다.

```typescript
new CicdStack(app, `CicdStack`, {env: primaryRegion, 
    firstRegionCluster: primaryCluster.cluster,
    secondRegionCluster: secondaryCluster.cluster,
    firstRegionRole: primaryCluster.firstRegionRole,
    secondRegionRole: secondaryCluster.secondRegionRole});
```

* `secondRegionRole: secondaryCluster.secondRegionRole`을 추가했습니다.

### CI/CD 파이프라인 배포하기
`cdk diff` 명령어로 생성될 자원을 확인한 뒤 `cdk deploy "*" --require-approval never` 명령어를 통해 CI/CD 파이프라인을 배포합니다.

{{% notice warning %}}
위 명령어는 워크샵 단계 간소화를 위해 IAM 등 보안 관련 자원 수정에 대한 동의를 생략하도록 했음을 유의하십시오.
{{% /notice %}}

터미널에서 진행상황을 확인할 수 있습니다.

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


### 변경 사항 릴리즈를 통해 수정된 파이프라인 다시 트리거하기

![](/images/40-deploy-app/release-change.png)

[CodePipeline 콘솔](https://console.aws.amazon.com/codesuite/codepipeline/?#)에서 새롭게 배포된 두 단계를 확인할 수 있습니다.  
위 스크린샷의 `변경 사항 릴리스`를 클릭하여 현재 시점 최신 코드를 반영하도록 합니다.  

### 릴리즈 승인하기
3단계로 첫 번째 리전의 EKS 클러스터에 정상적으로 어플리케이션이 배포된 뒤에, 아래 스크린샷과 같이 승인을 대기 중인 상태가 됨을 확인할 수 있습니다.  
**<검토>** 버튼을 누른 뒤, **<승인>** 버튼을 누릅니다.

![](/images/40-deploy-app/approval-pipeline.png)


### 배포된 자원 확인하기
1. `kubectl` 명령어를 통해 배포된 컨테이너를 확인해봅시다.
{{% notice warning %}}
`kubectl config current-context` 명령어를 통해 us-west-2 리전의 클러스터에서 작업 중임을 확인하십시오.
{{% /notice %}}

    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    hello-py-576f77b98b-kj7bf           0/1     Pending   0          85s  << 어플리케이션 Pod
    hello-py-576f77b98b-lkqwb           0/1     Pending   0          85s  << 어플리케이션 Pod
    hello-py-576f77b98b-wln9l           0/1     Pending   0          85s  << 어플리케이션 Pod
    metrics-server-6b6bbf4668-vqhjl     1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-5jdkg   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-hns6v   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-kzk8l   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-r7wf7   1/1     Running   0          3h58m
    nginx-deployment-5754944d6c-xlxxh   1/1     Running   0          3h58m
    ```

2. 생성된 서비스 객체의 `EXTERNAL-IP`를 통해서도 정상 응답이 오는지 확인합니다.  
ELB 생성에 2분 정도의 시간이 소요될 수 있으니 참고바랍니다.

    ```
    kubectl describe service hello-py | grep Ingress

    # 결과값
    LoadBalancer Ingress:     aed0099fad25846a3a469d6abd64926d-847916387.ap-northeast-1.elb.amazonaws.com
    ```

    ```
    curl <LoadBalancer Ingress>

    # 결과값
    Hello World from us-west-2
    ```
