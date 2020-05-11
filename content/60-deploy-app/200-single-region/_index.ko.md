---
title: 하나의 리전에 배포하는 파이프라인 생성
weight: 200
pre: "<b>6-2. </b>"

---

먼저 위에서 클론 받은 코드를 한 리전의 클러스터에만 배포해보겠습니다.  
다음과 같은 흐름을 갖는 CodePipeline을 작성합니다.

![](/images/40-deploy-app/single-region-pipeline.svg)


CDK 코드를 작성하는 IDE로 이동해서 아래와 단계에 따라 CI/CD 파이프라인을 생성하는 클래스를 생성합니다.

### 1. CI/CD Pipeline에서 사용할 IAM Role 내보내기
EKS 클러스터에 실제 어플리케이션 배포를 수행하는 CodeBuild는,  
그 리전에 있는 EKS 클러스터에 `kubectl` 명령을 보낼 수 있는 Role을 assume해서 애플리케이션 배포를 수행합니다. 이 Role을 생성해봅시다.
`lib/cluster-stack.ts`을 열어 아래 코드를 순서대로 추가합니다.


1. `constructor` 위
```typescript
  public readonly firstRegionRole: iam.Role;
```

2. `constructor` 내부
```typescript
if (cdk.Stack.of(this).region==primaryRegion) 
    this.firstRegionRole = createDeployRole(this, `for-1st-region`, cluster);

```

3. `class` 외부, 최하단
```typescript
export interface CicdProps extends cdk.StackProps {
  cluster: eks.Cluster,
  firstRegionRole: iam.Role
}

```

* skeleton 상태에서 클래스 외부에 정의된 함수가 무엇인지 궁금하셨죠? `cicd-stack`에서 배포를 수행하는 CodeBuild가 assume할 IAM 롤을 생성하되, 실행되는 스택의 리전값을 읽어 그 리전 영역의 클러스터에만 접근할 수 있는 정책을 할당하는 함수였습니다.
* 이 롤이 EKS 클러스터의 master 그룹에 들어가도록 권한을 부여합니다. 프로덕션에서는 최소한의 권한을 갖는 그룹에 별도 할당되도록 해주십시오.



완성된 `lib/cluster-stack.ts`는 아래와 같을 것입니다.
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

    const clusterAdmin = new iam.Role(this, 'AdminRole', {
      assumedBy: new iam.AccountRootPrincipal()
      });

    const primaryRegion = 'ap-northeast-1';

    const cluster = new eks.Cluster(this, 'demogo-cluster', {
      clusterName: `demogo`,
      mastersRole: clusterAdmin,
      defaultCapacity: 2,
      defaultCapacityInstance: cdk.Stack.of(this).region==primaryRegion? 
                                new ec2.InstanceType('r5.xlarge') : new ec2.InstanceType('m5.2xlarge')
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
  cluster: eks.Cluster,
  firstRegionRole: iam.Role
}
```


### 2. CicdStack 스택 뼈대 만들기
`lib` 디렉토리 아래에 `cicd-stack.ts` 파일이 아래와 같이 생성되어 있을 것입니다.

```typescript
import * as cdk from '@aws-cdk/core';
import codecommit = require('@aws-cdk/aws-codecommit');
import ecr = require('@aws-cdk/aws-ecr');
import codepipeline = require('@aws-cdk/aws-codepipeline');
import pipelineAction = require('@aws-cdk/aws-codepipeline-actions');
import { codeToECRspec, deployToEKSspec } from '../utils/buildspecs';


export class CicdStack extends cdk.Stack {

    constructor(scope: cdk.Construct, id: string, props: cdk.StackProps) {
        super(scope, id, props);

    }
}
```

1. 1번에서 생성한 props를 주입받도록 수정하겠습니다.

* 코드 최상단에 export한 props import
    ```typescript
    import { CicdProps } from './cluster-stack';
    ```

* `constructor` 부분에서 주입 받는 `props`를 `CicdProps`로 변경합니다.
    ```typescript
        constructor(scope: cdk.Construct, id: string, props: CicdProps) {
    ```

2. primaryRegion과 secondaryRegion 을 지정합니다.
    ```typescript
    const primaryRegion = 'ap-northeast-1';
    const secondaryRegion = 'us-east-1';
    ```


지금까지 완성된 코드는 아래와 같을 겁니다.
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
        const secondaryRegion = 'us-east-1';
    }
}


```

### 3. CodeCommit 생성하기
여러분들이 프로덕션 어플리케이션을 실제로 운영할 때에는 프라이빗하게 소스를 관리해야 할 것입니다.  
이 때 AWS CodeCommit 서비스를 이용할 수 있습니다. AWS CodeCommit은 뛰어난 확장성의 프라이빗 Git 리포지토리를 안전하게 호스팅하는 서비스로, 마치 git을 이용하는 것처럼 여러분들은 리포지토리 호스팅 서버, 스토리지 등을 관리하실 필요 없이 프라이빗하게 리포지토리를 이용할 수 있습니다.

아래 코드를 위에서 생성한 클래스 `construct` 부 안에 붙여넣습니다.

```typescript
const helloPyRepo = new codecommit.Repository(this, 'hello-py-for-demogo', {
    repositoryName: `hello-py-${cdk.Stack.of(this).region}`
});

new cdk.CfnOutput(this, `codecommit-uri`, {
    exportName: 'CodeCommitURL',
    value: helloPyRepo.repositoryCloneUrlHttp
});
```

* CodeCommit 레파지토리를 생성하고
* 이 레파지토리를 clone할 때 사용할 수 있는 URI를 스택 아웃풋으로 생성합니다.


### 4. ECR Repository 생성하기
이 코드를 기반으로 생성된 컨테이너 이미지는 별도의 Image Registry에 저장되어야 합니다.  
이때 Amazon ECR을 이용하실 수 있습니다. Amazon Elastic Container Registry(ECR)는 개발자가 Docker 컨테이너 이미지를 손쉽게 저장, 관리 및 배포할 수 있게 해주는 완전관리형 Docker 컨테이너 레지스트리입니다.

아래 코드를 CodeCommit 정의한 라인 아래에 붙여넣으십시오.

```typescript
const ecrForMainRegion = new ecr.Repository(this, `ecr-for-hello-py`);

```
* ECR 레지스트리를 생성합니다.


### 5. 도커 이미지를 빌드하는 CodeBuild 프로젝트 생성하기
개발자가 소스를 커밋했을 때 이 내용을 기반으로 새로운 이미지를 자동으로 빌드하게 해주어야 합니다.  
이 때 CodeBuild를 이용할 것입니다. AWS CodeBuild는 클라우드 상의 완전 관리형 빌드 서비스로, 소스 코드를 컴파일하고 단위 테스트를 실행하며 배포할 준비가 완료된 아티팩트를 생성합니다.  

buildspec 을 정의하여 CodeBuild에서 실제로 어떤 작업을 수행할 것인지 정의할 수 있습니다.  
1) 일반적인 어플리케이션에서는 코드 레파지토리의 루트 경로에 `buildspec.yml` 파일을 생성하여 정의할 수도 있고,  
2) CodeBuild 프로젝트를 정의할 때 지정할 수도 있습니다.  
이 워크샵에서는 리전 간 공통으로 사용되는 buildspec을 중앙에서 관리하기 위해 2)의 방법을 통해 CodeBuild 프로젝트에 직접 정의해줍니다.


**2. ECR Repository 생성하기** 까지 작성한 뒷 부분에, 아래 코드를 붙여 넣으십시오.
```typescript
const buildForECR = codeToECRspec(this, ecrForMainRegion.repositoryUri);
ecrForMainRegion.grantPullPush(buildForECR.role!);

```

* /utils 폴더에 이 워크샵에서 사용할 빌드 스펙을 미리 정의해두었습니다. 자세한 빌드 스펙이 궁금하신 분은 /utils/buildspec.ts 파일을 참조해주십시오. 
* 생성될 빌드 프로젝트에서 ECR에 이미지를 푸시할 수 있는 권한을 추가합니다.



### 6. EKS 클러스터에 배포하는 CodeBuild 프로젝트 생성하기
현 시점에는 AWS CodeDeploy (관리형 배포 서비스)가 아직 EKS 클러스터를 지원하지 않기 때문에 CodeBuild를 이용하여 자원을 배포해보겠습니다.


아래 코드를 위에 작성한 코드 뒤에 붙여넣으십시오.

```typescript
const deployToMainCluster = deployToEKSspec(this, primaryRegion, ecrForMainRegion, props.firstRegionRole);

```

* /utils 폴더에 이 워크샵에서 사용할 빌드 스펙을 미리 정의해두었습니다. 자세한 빌드 스펙이 궁금하신 분은 /utils/buildspec.ts 파일을 참조해주십시오. 

### 5. 한 리전의 EKS 클러스터에 애플리케이션을 배포하는 파이프라인 정의하기
그럼 지금까지 생성한 자원들을 엮어서 파이프라인을 만들어 보겠습니다.

아래 코드를 이어서 붙여넣으십시오.

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

* `sourceOutput`은 커밋된 코드를 아티팩트로 Pipeline에 전달해주기 위해 정의합니다.

{{% notice info %}} 이 워크샵에서는 워크샵 효율을 위해 빌드 이미지를 직접 업로드하여 사용합니다. 프로덕션에서는 별도로 ECR을 통해 빌드 이미지 관리를 하시기를 권고드립니다. {{% /notice %}}


`lib/cicd-stack.ts`의 완성된 코드는 아래와 같을 것입니다.
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
        const secondaryRegion = 'us-east-1';

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
        
        const deployToMainCluster = deployToEKSspec(this, primaryRegion, ecrForMainRegion, props.firstRegionRole);

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

### 6. 스택 로드하기

아래 코드를 `bin/multi-cluster-ts.ts` 파일에 붙여넣습니다.

```typescript
new CicdStack(app, `CicdStack`, {env: primaryRegion, cluster: primaryCluster.cluster ,
                                    firstRegionRole: primaryCluster.firstRegionRole});


```


### 7. CI/CD 파이프라인 배포하기
`cdk diff` 명령어를 통해 생성될 자원을 확인한 뒤, `cdk deploy` 명령어를 통해 CI/CD 파이프라인을 배포합니다.  
이때 `CicdStack`과 별개로, 이 스택의 자원에 권한 부여를 위해 `ClusterStack`에서도 변경이 발생합니다.

터미널에 완료가 뜨고 나서 콘솔에 들어가면, 현재 codecommit에 master 브랜치가 없어서 실패 상태로 되어 있는 파이프라인이 생성되어 있음을 확인할 수 있습니다.

![](/images/40-deploy-app/result-pipeline-single-region.png)


### 8. 배포 테스트하기
그러면 생성된 파이프라인을 이용해서 애플리케이션을 배포해볼까요?  
아까 [클론한 샘플 어플리케이션](https://github.com/yjw113080/aws-cdk-multi-region-sample-app) IDE를 열어주십시오.  

1. `CicdForPrimaryStack`에서 Output으로 출력된 codecommit URI 값을 복사하십시오.
```
CicdForPrimaryStack.codecommituri = https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/hello-py-ap-northeast-1
```

2. 다음 명령어를 통해 어플리케이션 프로젝트에 codecommit 레포지토리를 등록하십시오.
```
git remote add codecommit <<1번에서 카피한 codecommit URI>>
```

3. [IAM User](https://console.aws.amazon.com/iam/home?region=us-east-1) 콘솔에서, 현재 터미널이 사용 중인 User의 HTTPS Git credentials for AWS CodeCommit 를 생성하십시오.

도움이 필요하신 분들은 [이 링크](https://aws.amazon.com/ko/blogs/korea/introducing-git-credentials-a-simple-way-to-connect-to-aws-codecommit-repositories-using-a-static-user-name-and-password/)를 참조하십시오.


4. 다음 명령어를 통해 디렉토리 내의 코드를 codecommit으로 푸시하십시오.
```
git add .
git commit -am "initial commit"
git push codecommit master
```

5. CodePipeline 콘솔에서 다음과 같이 파이프라인이 트리거됨을 확인할 수 있습니다.
![](/images/40-deploy-app/result-pipeline-single-region-working.png)


6. `kubectl` 명령어를 통해 배포된 컨테이너를 확인해봅시다.
{{% notice warning %}}
`kubectl config current-context` 명령어를 통해 ap-northeast-1 리전의 클러스터에서 작업 중임을 확인하십시오.
{{% /notice %}}

```
NAME                                READY   STATUS    RESTARTS   AGE
hello-py-9b9bffb64-2f5bl            1/1     Running   0          3m7s << 어플리케이션 Pod
hello-py-9b9bffb64-btvts            1/1     Running   0          3m7s << 어플리케이션 Pod
hello-py-9b9bffb64-xttnw            1/1     Running   0          3m7s << 어플리케이션 Pod
metrics-server-6b6bbf4668-hlgmd     1/1     Running   0          24m
nginx-deployment-5754944d6c-whrmn   1/1     Running   0          24m
nginx-deployment-5754944d6c-wkkkn   1/1     Running   0          24m
```

7. 생성된 서비스 객체의 `EXTERNAL-IP`를 통해서도 정상 응답이 오는지 확인합니다.
```
kubectl describe service hello-py | grep Ingress

# 결과값
LoadBalancer Ingress:     aed0099fad25846a3a469d6abd64926d-847916387.ap-northeast-1.elb.amazonaws.com
```

```
curl <LoadBalancer Ingress>

# 결과값
Hello World from ap-northeast-1
```
