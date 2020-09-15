---
title: K8S Manifest
weight: 320
---

## Inject EKS cluster
Modify **lib/container-stack.ts** to be injected with the EKS cluster already created by `cluster-stack.ts`. 

1. Add import
  ```typescript
  import { EksProps } from './cluster-stack'; 
  ```

2. Change `props` to EksProps
  ```typescript
  export class ContainerStack extends cdk.Stack {
      constructor(scope: cdk.Construct, id: string, props: EksProps ) 
  ```

The completed code should look like this.
```typescript
import { EksProps } from './cluster-stack';
...
export class ContainerStack extends cdk.Stack {
    constructor(scope: cdk.Construct, id: string, props: EksProps ) { 
        ...
```






## Check K8S Manifest

Then shall we look at the containers to be created?
* `yaml-xxx` folder contains a simple yaml file to use as a demo.
* `yaml-common` folder contains yaml files to be created in common in any region. There is a simple yaml file that defines a namespace.
* The `yaml-{{region}}` folder contains yaml files that should be created only in a specific region.
  This defines a yaml with two nginx replicas in Tokyo and five in Oregon.

To take the best advantage of the CDK you have to use a programming language as it is. We will check the region of the execution environment and then read only the content of the required folder. As of now (May 2020), in order to create Kubernetes objects using the CDK, json is the only format supported. Let’s first take a look at an utility function that handles yaml manifests into json format transformation.


## Check out readFile util

Go to `utils/read-file.ts`.
It reads yaml file using nodejs fs module and [js-yaml](https://www.npmjs.com/package/js-yaml) to convert to json format. After successfully converting the content into json, the function registers it as a resource in the injected EKS cluster.

Please note that this code basically does not handle any error to simplify the workshop.

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

## Create the Container Stack
Open **lib/container-stack.ts**. Paste the code below under the `super (scope, id, props)` syntax.
```typescript
    const cluster = props.cluster;
    const commonFolder = './yaml-common/';
    const regionFolder = `./yaml-${cdk.Stack.of(this).region}/`;

    readYamlFromDir(commonFolder, cluster);
    readYamlFromDir(regionFolder, cluster);
```

* `./yaml-common/`: This folder contains resource manifests to be deployed in both regions.
* `./yaml-${cdk.Stack.of(this).region}/`: This is a folder that contains resource manifests that need to be specifically deployed in that region by checking the region value where the stack is executed.
* After reading yaml files in these two folders and transforming them into json, this code then creates a resource by CDK. This allows the administrator to manage the distribution through code rather than directly controlling resources through the `kubectl` command.
  
{{% notice warning %}}
If you are doing a workshop in a region other than the Tokyo region (ap-northeast-1) or Oregon region (us-west-2), please change the folder name where the yaml files are located.
{{% /notice %}}


The completed code should look like this.
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







## Loading the stack on the entry point

Open the bin/multi-cluster-ts.ts file and load the stack. Paste the code below under the ClusterStack you just created

```typescript
new ContainerStack(app, `ContainerStack-${primaryRegion.region}`, {env: primaryRegion, cluster: primaryCluster.cluster });

```

The completed code will look like this:
```typescript
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

Unlike when the ClusterStack was first created, you can see that it takes over the `cluster` created above in addition to the environment settings (account, region) values. This is because we decided to inject the clusters into ContainerStack using our `EksProps` interface!

## Identifying resources to be created
Let's check the resources to be created with the command below.
```
cdk diff
```

You can see that KubernetesResource will be created as shown below.
```
Stack ClusterStack-ap-northeast-1
Resources
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-00_ap_nginx0/Resource demogoclustermanifest00apnginx0EE9A04C1
[+] Custom::AWSCDK-EKS-KubernetesResource demogo-cluster/manifest-00_namespaces0/Resource demogoclustermanifest00namespaces0DB969C34

Stack ContainerStack-ap-northeast-1
There were no differences
```
{{% notice info %}}
If you look closely at the output on the console, you can see that container creation is in progress in ClusterStack, not ContainerStack.
Why is that? If you want to know more, please click [here](/en/80-appendix/how-cfn-addresource/).
{{% /notice %}}





## Deploy
Let's deploy the container with the command below.

```
cdk deploy ClusterStack-ap-northeast-1
```
{{% notice info %}}
If you have created a stack in another region, make sure the correct region name comes after `ClusterStack`.
{{% /notice %}}


As in the case of creating `ClusterStack`, the progress of resource creation will be displayed in the terminal.
When the stack execution is completed, check the created resource using the following command.

1. Namespace
```
kubectl get ns
```
You should have created a namespace named management.
```
NAME              STATUS   AGE
default           Active   21m
kube-node-lease   Active   21m
kube-public       Active   21m
kube-system       Active   21m
management        Active   7m47s << Namespace we created
```

2. Pods
```
kubectl get pod
```
You can see that two pods with nginx have been created.
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5754944d6c-cnxzn   1/1     Running   0          7m51s
nginx-deployment-5754944d6c-vrrd2   1/1     Running   0          7m51s
```