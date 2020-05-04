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


### 1. 파이프라인에서 Assume할 Role 생성하기
![](/images/40-deploy-app/2nd-region-pipeline.svg)

우리가 두 번째 리전에 배포할 때 실질적으로 사용하게 될 CodeBuild는,  
그 리전에 있는 EKS 클러스터에 `kubectl` 명령을 보낼 수 있는 Role을 assume해서 애플리케이션 배포를 수행합니다.  
이 Role을 생성해봅시다.

`lib/cluster-stack.ts` 파일로 이동하여 아래 코드를 붙여넣으십시오.  

1. `class` 내부
```typescript
public readonly roleFor2ndRegionDeployment: iam.Role;
```

2. `construct` 내부
```typescript
const roleForCodebuild = new iam.Role(this, 'for-2nd-region', {
  roleName: PhysicalName.GENERATE_IF_NEEDED,
  assumedBy: new iam.AccountRootPrincipal()
});

if (cdk.Stack.of(this).region!=primaryRegion) 
  this.cluster.awsAuth.addMastersRole(roleForCodebuild)

this.roleFor2ndRegionDeployment = roleForCodebuild;
```

* 리전, 계정 간 공유되는 자원은 `PhysicalName.GENERATE_IF_NEEDED`을 명시적으로 선언해주어야 합니다. 클래스 윗 부분에 `import { PhysicalName } from '@aws-cdk/core';` 를 추가해주세요.
* secondaryRegion(us-east-1)의 스택에서, 이 롤이 EKS 클러스터의 master 그룹에 들어가도록 권한을 부여합니다. 프로덕션에서는 최소한의 권한을 갖는 그룹에 별도 할당되도록 해주십시오.


3. `class` 외부, 최하단
```typescript
export interface PropForCicd extends cdk.StackProps {
  cluster: eks.Cluster,
  roleFor2ndRegionDeployment: iam.Role
}
```
이 클래스에서 선언한 `cluster`와 `role`을 내보냅니다.


완성된 `cluster-stack.ts`의 코드는 아래와 같을 것입니다.

```typescript
export class ClusterStack extends cdk.Stack {
  public readonly cluster: eks.Cluster;
  public readonly roleFor2ndRegionDeployment: iam.Role;

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
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion ? new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.xlarge')
    });

    this.cluster = cluster;

    const roleForCodebuild = new iam.Role(this, 'for-2nd-region', {
      roleName: PhysicalName.GENERATE_IF_NEEDED,
      assumedBy: new iam.AccountRootPrincipal()
    });
    
    if (cdk.Stack.of(this).region!=primaryRegion) 
      this.cluster.awsAuth.addMastersRole(roleForCodebuild)
    
    this.roleFor2ndRegionDeployment = roleForCodebuild;
    
  }
}

export interface EksProps extends cdk.StackProps {
  cluster: eks.Cluster
}

export interface PropForCicd extends cdk.StackProps {
  cluster: eks.Cluster,
  roleFor2ndRegionDeployment: iam.Role
}
```

### 2. ClusterStack의 Props 주입

Construct 부분을 아래와 같이 수정해주십시오.
```typescript
    constructor(scope: cdk.Construct, id: string, props: PropForCicd) {
```

* `props: ` 부분을 EksProps에서 PropsForCicd로 수정합니다.


`bin/multi-cluster-ts.ts`로 이동하여 `CicdForPrimaryRegionStack` 부분을 수정합니다.  
입력 받아야 하는 값들이 넘어오지 않아 오류가 발생하고 있는 것을 볼 수 있습니다.  
아래와 같이 코드를 대체하여 1에서 생성한, 파이프라인에서 assume할 롤을 주입합니다.

```typescript
new CicdForPrimaryRegionStack(app, `CicdForPrimaryStack`, {env: primaryRegion, cluster: primaryCluster.cluster, roleFor2ndRegionDeployment: primaryCluster.roleFor2ndRegionDeployment});
```


### 3. 두 번째 리전에 배포하는 CodeBuild 프로젝트 생성하기

`lib/cicd-for-primary-region.ts` 파일로 이동합니다.  
지금까지 작성된 코드에 아래 코드를 붙여넣습니다. 단, CodePipeline 정의 부분보다 위에 정의해야 합니다. 

```typescript
const deployTo2ndCluster = deployTo2ndClusterspec(this, secondaryRegion, ecrForMainRegion, props.roleFor2ndRegionDeployment);

deployTo2ndCluster.addToRolePolicy(new iam.PolicyStatement({
            actions: ['sts:AssumeRole'],
            resources: [props.roleFor2ndRegionDeployment.roleArn]
        }));

```
* `/utils/buildspec.ts`에 정의된 빌드 스펙을 가지고 CodeBuild 프로젝트를 생성했습니다. 자세한 내용이 궁금하시면 해당 파일을 열어 확인하십시오.
* 이 빌드 프로젝트에서 사용할 롤이, 위의 `cluster-stack.ts`에서 정의한 롤을 assume할 권한을 부여합니다.


### 4. CodePipeline 수정
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
//...중략
const deployToMainCluster = deployToEKSspec(this, primaryRegion, props.cluster, ecrForMainRegion);
const deployTo2ndCluster = deployTo2ndClusterspec(this, secondaryRegion, ecrForMainRegion, props.roleFor2ndRegionDeployment);

        new codepipeline.Pipeline(this, 'repo-to-ecr-hello-py', {
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
```


### 5. CI/CD 파이프라인 배포하기
`cdk diff` 명령어로 생성될 자원을 확인한 뒤 `cdk deploy` 명령어를 통해 CI/CD 파이프라인을 배포합니다.
터미널에서 진행상황을 확인할 수 있습니다.

```
  ...
  8/19 | 오후 10:28:52 | UPDATE_IN_PROGRESS   | AWS::CodePipeline::Pipeline | repo-to-ecr-hello-py (repotoecrhellopy560FED9C)
  9/19 | 오후 10:28:53 | UPDATE_COMPLETE      | AWS::CodePipeline::Pipeline | repo-to-ecr-hello-py (repotoecrhellopy560FED9C)
  9/19 | 오후 10:29:00 | UPDATE_COMPLETE_CLEA | AWS::CloudFormation::Stack  | CicdForPrimaryStack
 10/19 | 오후 10:29:01 | UPDATE_COMPLETE      | AWS::CloudFormation::Stack  | CicdForPrimaryStack

 ✅  CicdForPrimaryStack

Outputs:
CicdForPrimaryStack.codecommituri = https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/hello-py-ap-northeast-1
CicdForPrimaryStack.ExportsOutputFnGetAttdeploytoeksapnortheast1Role18912335ArnB28E7CE7 = arn:aws:iam::865200059792:role/CicdForPrimaryStack-deploytoeksapnortheast1Role189-VIG05L6PETHL
```


### 6. 변경 사항 릴리즈를 통해 수정된 파이프라인 다시 트리거하기

![](/images/40-deploy-app/release-change.png)

CodePipeline 콘솔에서 새롭게 배포된 두 단계를 확인할 수 있습니다.  
위 스크린샷의 `변경 사항 릴리스`를 클릭하여 현재 시점 최신 코드를 반영하도록 합니다.  

### 7. 릴리즈 승인하기
3단계로 첫 번째 리전의 EKS 클러스터에 정상적으로 어플리케이션이 배포된 뒤에, 아래 스크린샷과 같이 승인을 대기 중인 상태가 됨을 확인할 수 있습니다.  
**<검토>** 버튼을 누른 뒤, **<승인>** 버튼을 누릅니다.

* 이 워크샵에서는 Approval 을 위한 별도의 인풋을 넣지 않았으나, 프로덕션 환경에서는 필요에 따라 이 부분에 인풋을 집어넣을 수 있습니다.

![](/images/40-deploy-app/approval-pipeline.png)


### 배포된 자원 확인하기
1. `kubectl` 명령어를 통해 배포된 컨테이너를 확인해봅시다.
{{% notice warning %}}
`kubectl config current-context` 명령어를 통해 us-east-1 리전의 클러스터에서 작업 중임을 확인하십시오.
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
```
kubectl describe service hello-py | grep Ingress

# 결과값
LoadBalancer Ingress:     aed0099fad25846a3a469d6abd64926d-847916387.ap-northeast-1.elb.amazonaws.com
```

```
curl <LoadBalancer Ingress>

# 결과값
Hello World from us-east-1
```
