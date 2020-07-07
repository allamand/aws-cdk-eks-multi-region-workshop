---
title: K8S Manifest 이용하기
weight: 320
---

## EKS 클러스터 주입 받기
앞서 ClusterStack에서 정의한 Interface를 이용하여, 이 `ContainerStack` 클래스가 생성되는 시점에 이미 생성된 EKS 클러스터를 전달 받을 수 있도록 다음과 같이 **lib/container-stack.ts**를 수정합니다.

1. import 추가
  ```typescript
  import { EksProps } from './cluster-stack'; 
  ```

2. 주입 받는 props를 EksProps로 변경
  ```typescript
  export class ContainerStack extends cdk.Stack {
      constructor(scope: cdk.Construct, id: string, props: EksProps ) 
  ```

완성된 코드는 아래와 같을 것입니다.
```typescript
import { EksProps } from './cluster-stack';
...
export class ContainerStack extends cdk.Stack {
    constructor(scope: cdk.Construct, id: string, props: EksProps ) { // props 부분 변경!!
        ...
```






## K8S Manifest 확인하기
그럼 생성할 컨테이너들을 살펴볼까요?
* `yaml-xxx` 폴더에 보면 데모로 사용할 간단한 yaml 파일이 들어 있습니다. 
* `yaml-common` 폴더에는 어느 리전에든 공통적으로 생성될 yaml 파일이 위치합니다. 
  여기에는 namespace를 하나 정의하는 간단한 yaml 파일이 있습니다.
* `yaml-{{region}}` 폴더에는 특정 리전에만 생성되어야 하는 yaml 파일이 들어있습니다.
  여기에는 nginx 를 세 개(replicas) 띄우는 yaml이 정의되어 있습니다.


프로그래밍 언어를 그대로 이용할 수 있다는 CDK의 장점을 살려서, 우리는 실행 환경의 리전을 확인한 뒤 필요한 폴더의 내용만 읽어서 자원을 배포하도록 할 것입니다. 그런데 현재 (2020년 5월) 시점으로는 CDK를 이용해 쿠버네티스 오브젝트를 생성하기 위해서는, Manifest를 json 형태로 바꾸어주어야 합니다. 이를 처리해줄 유틸 함수를 먼저 살펴봅시다.



## readFile util 확인하기
`utils/read-file.ts`로 이동합니다.  
nodejs fs 모듈과 [js-yaml](https://www.npmjs.com/package/js-yaml) 을 이용해서 yaml 파일을 읽어 json 형태로 변환하고,  
주입 받은 EKS 클러스터에 자원으로 등록해주는 코드입니다.  
이 워크샵에서는 에러 처리를 최소한으로만 한 것임에 유의해주십시오.

```typescript
export function readYamlFromDir(dir: string, cluster: eks.Cluster) {
  fs.readdirSync(dir, "utf8").forEach(file => {
    if (file!=undefined && file.split('.').pop()=='yaml') {
      let data = fs.readFileSync(dir+file, 'utf8');
      if (data != undefined) {
        let i=0;
        yaml.loadAll(data).forEach((item) => {
          cluster.addResource(file.substr(0,file.length-5)+i, item);
          i++;
        })
      }
    }
  })
}
```

## Manifest 를 이용한 자원 생성
**lib/container-stack.ts**을 여십시오.  
아래 코드를 `super(scope, id, props)` 구문 아래에 붙여넣습니다.

```typescript
    const cluster = props.cluster;
    const commonFolder = './yaml-common/';
    const regionFolder = `./yaml-${cdk.Stack.of(this).region}/`;

    readYamlFromDir(commonFolder, cluster);
    readYamlFromDir(regionFolder, cluster);
```

* `./yaml-common/`: 멀티 리전으로 확장할 때, 두 리전 모두에 배포될 자원 manifest가 들어 있는 폴더입니다.
* `./yaml-${cdk.Stack.of(this).region}/`: 스택이 실행되는 리전값을 확인하여, 그 리전에만 특수하게 배포해야 하는 자원 manifest를 담은 폴더입니다.
* 이 두 폴더의 yaml 파일을 읽어서 json으로 변형한 뒤 CDK를 통해 자원을 생성합니다. 이를 통해 운영자가 직접 `kubectl` 명령어를 통해 자원을 컨트롤 하는 것이 아니라, 코드를 통해 배포 관리를 할 수 있게 됩니다.

{{% notice warning %}}
도쿄 리전 (ap-northeast-1), 오레곤 리전 (us-west-2)이 아닌 다른 리전에서 워크샵을 수행하시는 경우, yaml 파일이 있는 폴더명을 변경해주십시오.  
{{% /notice %}}


완성된 코드는 아래와 같을 것입니다.
```typescript
import * as cdk from '@aws-cdk/core';
import { readYamlFromDir } from '../utils/read-file';
import { EksProps } from './cluster-stack';

export class ContainerStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: EksProps) {
    super(scope, id, props);

    const cluster = props.cluster;
    const commonFolder = './yaml-common/';
    const regionFolder = `./yaml-${cdk.Stack.of(this).region}/`;

    readYamlFromDir(commonFolder, cluster);
    readYamlFromDir(regionFolder, cluster);

  }
}
```







## 엔트리포인트에 스택 로드하기
**bin/multi-cluster-ts.ts** 파일을 열어 스택을 로드합시다.
아까 생성한 CluterStack 아래에 아래 코드를 붙여넣습니다.

```typescript
new ContainerStack(app, `ContainerStack-${primaryRegion.region}`, {env: primaryRegion, cluster: primaryCluster.cluster });

```

완성된 코드는 아래와 같을 것입니다.
```typescript
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { ClusterStack } from '../lib/cluster-stack';
import { ContainerStack } from '../lib/container-stack';
import { CicdStack } from '../lib/cicd-stack';


const app = new cdk.App();

const account = app.node.tryGetContext('account') || process.env.CDK_INTEG_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT;
const primaryRegion = {account: account, region: 'ap-northeast-1'};
const secondaryRegion = {account: account, region: 'us-west-2'};

const primaryCluster = new ClusterStack(app, `ClusterStack-${primaryRegion.region}`, {env: primaryRegion })
new ContainerStack(app, `ContainerStack-${primaryRegion.region}`, {env: primaryRegion, cluster: primaryCluster.cluster });
```


최초로 ClusterStack을 생성했을 때와는 다르게, 환경 설정(계정, 리전) 값 외에 위에서 생성한 `cluster`를 넘겨받는 것을 알 수 있습니다.



## 생성될 자원 확인하기
아래 명령어로 생성될 자원을 확인해봅시다.
```
cdk diff
```

아래 결과와 같이 KubernetesResource 들이 생성될 것임을 확인할 수 있습니다.
```
Stack ClusterStack-ap-northeast-1
Resources
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-00_ap_nginx0/Resource demogoclustermanifest00apnginx0EE9A04C1
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-00_namespaces0/Resource demogoclustermanifest00namespaces0DB969C34

Stack ContainerStack-ap-northeast-1
There were no differences
```
{{% notice info %}}
콘솔에 출력되는 결과를 잘 보면, ContainerStack이 아니라 ClusterStack에서 컨테이너 생성이 진행되는 것을 볼 수 있습니다.
왜 그럴까요? 더 자세히 알고 싶으신 분들은 [여기](/ko/80-appendix/how-cfn-addresource/)를 눌러주십시오.
{{% /notice %}}





## 배포하기
아래 명령어로 컨테이너를 배포해봅시다.

```
cdk deploy ClusterStack-ap-northeast-1
```
{{% notice info %}}
혹시 다른 리전에 스택을 생성하셨다면 `ClusterStack` 뒤에 정확한 리전 이름이 오도록 해주세요.
{{% /notice %}}

`ClusterStack`을 생성했을 때처럼 터미널에 자원 생성 진행상황이 출력될 것입니다.  
스택 실행이 완료되면 다음 명령어를 이용해 생성된 자원을 확인하십시오.


1. Namespace
```
kubectl get ns
```
management 라는 이름의 네임스페이스가 생성되었을 것입니다.
```
NAME              STATUS   AGE
default           Active   21m
kube-node-lease   Active   21m
kube-public       Active   21m
kube-system       Active   21m
management        Active   7m47s << 우리가 생성한 네임스페이스
```

2. Pods
```
kubectl get pod
```
이름에 nginx를 포함하는 pod가 2개 생성된 것을 확인할 수 있습니다.
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5754944d6c-cnxzn   1/1     Running   0          7m51s
nginx-deployment-5754944d6c-vrrd2   1/1     Running   0          7m51s
```