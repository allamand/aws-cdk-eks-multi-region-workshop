---
title: 하나의 리전에 배포하는 파이프라인 생성
weight: 200
pre: "<b>6-2. </b>"

---

먼저 위에서 클론 받은 코드를 한 리전의 클러스터에만 배포해보겠습니다.  
다음과 같은 흐름을 갖는 CodePipeline을 작성합니다.

![](/static/images/40-deploy-app/single-region-pipeline.svg)


CDK 코드를 작성하는 IDE로 이동해서 아래와 단계에 따라 CI/CD 파이프라인을 생성하는 클래스를 생성합니다.

### CicdForPrimaryRegion 스택 뼈대 만들기
`lib` 디렉토리 아래에 `cicd-for-primary-region.ts` 파일을 생성합니다.  
아래 코드를 붙여 넣어 스택 클래스를 생성합니다.

```typescript
import * as cdk from '@aws-cdk/core';
import codecommit = require('@aws-cdk/aws-codecommit');
import ecr = require('@aws-cdk/aws-ecr');
import codebuild = require('@aws-cdk/aws-codebuild');
import codepipeline = require('@aws-cdk/aws-codepipeline');
import { CommonProps } from './cluster-stack';
import pipelineAction = require('@aws-cdk/aws-codepipeline-actions');
import * as iam from '@aws-cdk/aws-iam';
import { codeToECRspec, deployToEKSspec, deployTo2ndClusterspec } from '../utils/buildspecs';


export class CicdForPrimaryRegionStack extends cdk.Stack {

    constructor(scope: cdk.Construct, id: string, props: CommonProps) {
    }
}
```

### 1. CodeCommit 생성하기
여러분들이 프로덕션 어플리케이션을 실제로 운영할 때에는 프라이빗하게 소스를 관리해야 할 것입니다.  
이 때 AWS CodeCommit 서비스를 이용할 수 있습니다. AWS CodeCommit은 뛰어난 확장성의 프라이빗 Git 리포지토리를 안전하게 호스팅하는 서비스로, 마치 git을 이용하는 것처럼 여러분들은 리포지토리 호스팅 서버, 스토리지 등을 관리하실 필요 없이 프라이빗하게 리포지토리를 이용할 수 있습니다.

아래 코드를 위에서 생성한 클래스 `construct` 부 안에 붙여넣습니다.

```typescript
        const helloPyRepo = new codecommit.Repository(this, 'hello-py-for-demogo', {
            repositoryName: `hello-py-${cdk.Stack.of(this).region}`
        });
```

### 2. ECR Repository 생성하기
이 코드를 기반으로 생성된 컨테이너 이미지는 별도의 Image Registry에 저장되어야 합니다.  
이때 Amazon ECR을 이용하실 수 있습니다. Amazon Elastic Container Registry(ECR)는 개발자가 Docker 컨테이너 이미지를 손쉽게 저장, 관리 및 배포할 수 있게 해주는 완전관리형 Docker 컨테이너 레지스트리입니다.

아래 코드를 CodeCommit 정의한 라인 아래에 붙여넣으십시오.

```typescript
const ecrForMainRegion = new ecr.Repository(this, `ecr-for-hello-py`);
ecrForMainRegion.grantPull(props.asg.role);
```
* ECR 레지스트리를 생성합니다.
* 그리고 우리가 생성한 EKS 클러스터가 이 ECR로부터 컨테이너 이미지를 pull 할 수 있도록 권한을 부여합니다.

### 3. 도커 이미지를 빌드하는 CodeBuild 프로젝트 생성하기
개발자가 소스를 커밋했을 때 이 내용을 기반으로 새로운 이미지를 자동으로 빌드하게 해주어야 합니다.  
이 때 CodeBuild를 이용할 것입니다. AWS CodeBuild는 클라우드 상의 완전 관리형 빌드 서비스로, 소스 코드를 컴파일하고 단위 테스트를 실행하며 배포할 준비가 완료된 아티팩트를 생성합니다.  

buildspec 을 정의하여 CodeBuild에서 실제로 어떤 작업을 수행할 것인지 정의할 수 있습니다.  
1) 일반적인 어플리케이션에서는 코드 레파지토리의 루트 경로에 `buildspec.yml` 파일을 생성하여 정의할 수도 있고,  
2) CodeBuild 프로젝트를 정의할 때 지정할 수도 있습니다.  
이 워크샵에서는 리전 간 공통으로 사용되는 buildspec을 중앙에서 관리하기 위해 2)의 방법을 통해 CodeBuild 프로젝트에 직접 정의해줍니다.

**2. ECR Repository 생성하기**아래 코드를 붙여 넣으십시오.
```typescript
const buildForECR = codeToECRspec(this, ecrForMainRegion.repositoryUri);
ecrForMainRegion.grantPullPush(buildForECR.role!);

```

* /utils 폴더에 이 워크샵에서 사용할 빌드 스펙을 미리 정의해두었습니다. 자세한 빌드 스펙이 궁금하신 분은 /utils/buildspec.ts 파일을 참조해주십시오. 
* 생성될 빌드 프로젝트에서 ECR에 이미지를 푸시할 수 있는 권한을 추가합니다.

### 4. EKS 클러스터에 배포하는 CodeBuild 프로젝트 생성하기
현 시점에는 AWS CodeDeploy (관리형 배포 서비스)가 아직 EKS 클러스터를 지원하지 않기 때문에 CodeBuild를 이용하여 자원을 배포해보겠습니다.

아래 코드를 위에 작성한 코드 뒤에 붙여넣으십시오.

```typescript
const deployToMainCluster = deployToEKSspec(this, primaryRegion, props.cluster, ecrForMainRegion);
```

* /utils 폴더에 이 워크샵에서 사용할 빌드 스펙을 미리 정의해두었습니다. 자세한 빌드 스펙이 궁금하신 분은 /utils/buildspec.ts 파일을 참조해주십시오. 

### 5. 한 리전의 EKS 클러스터에 애플리케이션을 배포하는 파이프라인 정의하기
그럼 지금까지 생성한 자원들을 엮어서 파이프라인을 만들어 보겠습니다.

아래 코드를 이어서 붙여넣으십시오.

```typescript
const sourceOutput = new codepipeline.Artifact();

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
                }
                
            ]
        });
```

* `sourceOutput`은 커밋된 코드를 아티팩트로 Pipeline에 전달해주기 위해 정의합니다.

{{% notice info %}} 이 워크샵에서는 워크샵 효율을 위해 빌드 이미지를 직접 업로드하여 사용합니다. 프로덕션에서는 별도로 ECR을 통해 빌드 이미지 관리를 하시기를 권고드립니다. {{% /notice %}}

### 6. CI/CD 파이프라인 배포하기
`cdk diff` 명령어를 통해 생성될 자원을 확인합니다.

<<생성될 자원 입력>>

`cdk deploy` 명령어를 통해 CI/CD 파이프라인을 배포합니다.

<<결과값 보여주기>>

<<스크린샷>>

### 7. 배포 테스트하기
그러면 생성된 파이프라인을 이용해서 애플리케이션을 배포해볼까요?  
아까 [클론한 샘플 어플리케이션](https://github.com/yjw113080/aws-cdk-multi-region-sample-app) IDE를 열어주십시오.  

1. 다음 명령어를 통해 codecommit 레포지토리를 등록하십시오.


2. 다음 명령어를 통해 디렉토리 내의 코드를 codecommit으로 푸시하십시오.


3. CodePipeline 콘솔에서 다음과 같이 파이프라인이 트리거됨을 확인할 수 있습니다.
4. `kubectl` 명령어를 통해 배포된 컨테이너를 확인해봅시다.
