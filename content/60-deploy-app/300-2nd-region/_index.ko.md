---
title: 두 개 리전에 배포 테스트
weight: 300
pre: "<b>6-3. </b>"
---

첫 번째 리전에서 정상 동작하는 것을 확인했으니, 이제 두 번째 리전에도 배포를 해볼까요?

![](/images/40-deploy-app/pipeline.svg)

위와 같이 1) 첫번째 리전에 정상 동작 여부 확인한 뒤 2) 수동 승인 과정을 거쳐 3) 두번째 리전에 배포하도록 단계를 추가해보겠습니다.

### 1. PrimaryRegion, SecondaryRegion 변수 내보내기
이 파이프라인을 만들 때, 스택이 실행되는 환경이 PrimaryRegion (어플리케이션이 먼저 배포되는 환경)인지 아닌지 검사한 뒤에 권한을 주는 코드를 추가할 것입니다.  
이를 위해서는 어떤 것이 PrimaryRegion이 되는지 지정해주어야겠죠. 이 값을 [클러스터 설정 변경 챕터](content/50-deploy-to-2nd/200-change-cluster.ko.md)처럼 스택마다 하드코딩한다면, 향후 유연하게 활요하는 것이 불편해질 것입니다.  
우리는 `bin/multi-cluster-ts.ts` 파일에서 지정하는 리전값을 전달 받아서 이 작업을 수행하도록 코드를 작성할 것입니다.  

`bin/multi-cluster-ts.ts` 파일을 연 뒤, 아래 코드를 파일 끝에 붙여넣으십시오.
```typescript
export {primaryRegion, secondaryRegion}
```


### 2. 파이프라인에서 Assume할 Role 생성하기
![](/images/40-deploy-app/2nd-region-pipeline.svg)

우리가 두 번째 리전에 배포할 때 실질적으로 사용하게 될 CodeBuild는,  
그 리전에 있는 EKS 클러스터에 `kubectl` 명령을 보낼 수 있는 Role을 assume해서 애플리케이션 배포를 수행합니다.  
이 Role을 생성해봅시다.

`lib/cluster-stack.ts` 파일로 이동하여 아래 코드를 붙여넣으십시오.  
1. primaryRegion 변수 가져오기

먼저 지금 코드에 있는 `const primaryRegion...` 선언 라인을 삭제합니다.

그리고 아래 코드를 `class` 상단에 붙여넣어주십시오.

```typescript
import { primaryRegion } from '../bin/multi-cluster-ts'
```

2. `class` 내부
```typescript
public readonly roleFor2ndRegionDeployment: iam.Role;
```

3. `construct` 내부
```typescript
      const roleForCodebuild = new iam.Role(this, 'for-2nd-region', {
        roleName: PhysicalName.GENERATE_IF_NEEDED,
        assumedBy: new iam.AccountRootPrincipal()
      });

      if (cdk.Stack.of(this).region!=primaryRegion.region) 
        this.cluster.awsAuth.addMastersRole(roleForCodebuild)
      
      this.roleFor2ndRegionDeployment = roleForCodebuild;
```
* 리전, 계정 간 공유되는 자원은 `PhysicalName.GENERATE_IF_NEEDED`을 명시적으로 선언해주어야 합니다.
* secondaryRegion(us-east-1)의 스택에서, 이 롤이 EKS 클러스터의 master 그룹에 들어가도록 권한을 부여합니다. 프로덕션에서는 최소한의 권한을 갖는 그룹에 별도 할당되도록 해주십시오.


4. `class` 외부, 최하단
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
  public readonly asg: autoscaling.AutoScalingGroup;
  public readonly roleFor2ndRegionDeployment: iam.Role;

  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

      const clusterAdmin = new iam.Role(this, 'AdminRole', {
        assumedBy: new iam.AccountRootPrincipal()
      });

      const cluster = new eks.Cluster(this, 'demogo-cluster', {
        clusterName: `demogo`,
        mastersRole: clusterAdmin,
        defaultCapacity: 2,
        defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion.region ? 
                                  new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.xlarge')
      });
      this.cluster = cluster;
    

      const roleForCodebuild = new iam.Role(this, 'for-2nd-region', {
        roleName: PhysicalName.GENERATE_IF_NEEDED,
        assumedBy: new iam.AccountRootPrincipal()
      });

      if (cdk.Stack.of(this).region!=primaryRegion.region) 
        this.cluster.awsAuth.addMastersRole(roleForCodebuild)
      
      this.roleFor2ndRegionDeployment = roleForCodebuild;
  }
}

export interface CommonProps extends cdk.StackProps {
  cluster: eks.Cluster
}

export interface PropForCicd extends cdk.StackProps {
  cluster: eks.Cluster,
  roleFor2ndRegionDeployment: iam.Role
}
```

### 3. 두 번째 리전에 배포하는 CodeBuild 프로젝트 생성하기

지금까지 작성된 코드에 아래 코드를 붙여넣습니다. 단, CodePipeline 정의 부분보다 위에 정의해야 합니다. 

```typescript
const deployTo2ndCluster = deployTo2ndClusterspec(this, secondaryRegion, ecrForMainRegion, props.roleFor2ndRegionDeployment);
```
* `/utils/buildspec.ts`에 정의된 빌드 스펙을 가지고 CodeBuild 프로젝트를 생성했습니다. 자세한 내용이 궁금하시면 해당 파일을 열어 확인하십시오.


### 4. CodePipeline 수정
선언된 CodePipeline construct에 아래 스테이지를 추가합니다.
```typescript
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
`cdk diff` 명령어를 통해 생성될 자원을 확인합니다.

<<생성될 자원 입력>>

`cdk deploy` 명령어를 통해 CI/CD 파이프라인을 배포합니다.

<<결과값 보여주기>>

<<스크린샷>>


### 6. 배포된 자원 확인하기

4. `kubectl` 명령어를 통해 배포된 컨테이너를 확인해봅시다.
