---
title: Clone the base repo
weight: 100
pre: "<b>4-1. </b>"
---

Let't clone the base project and open it in your favorite IDE.

### Clone the base repository
```
git clone https://github.com/yjw113080/aws-cdk-eks-multi-region-skeleton
```

### Open it up in your IDE

Open the above project folder in your favorite IDE.
For reference, this workshop was written using VS Code.


### Project Structure

The main directory structure is:
```bash
├── bin
│   └── multi-cluster-ts.ts
├── cdk.context.json
├── cdk.json
├── jest.config.js
├── lib
│   ├── cicd-stack.ts
│   ├── cluster-stack.ts
│   └── container-stack.ts
├── package-lock.json
├── package.json
├── test
│   └── ..
├── tsconfig.json
├── utils
│   ├── buildimage
│   │   ├── Dockerfile
│   │   └── entrypoint.sh
│   ├── buildspecs.ts
│   └── read-file.ts
├── yaml-ap-northeast-1
│   └── 00_ap_nginx.yaml
├── yaml-common
│   └── 00_namespaces.yaml
└── yaml-us-west-2
    └── 00_us_nginx.yaml
```
* `/bin/multi-cluster-ts`: This is the **entry point** of this CDK app. In other words, this is the first one to be executed when the CDK app is launched. This is where the stacks are registered. It means that if we want to actually deploy the stack we write, we need to add it here. If the word ‘entry point’ appears in the workshop in the future, you can think of it as referring to this file.

* `/lib`: The stacks--what we would mainly create today--would be located here. `cluster-stack` and `container-stack` are covered in Lab 1 and 2, and `cicd-stack` in Lab 3.
* `/utils`: Here are some of the tools needed to build the stack. There is a build base image required by Codebuild, a build specification, and a json converter for creating containers.
* `/yaml-*`: There are k8s manifests to be distributed for each environment.


## Installing dependency packages
Install the dependency package by running the following command on the cloned repository.


```
cd aws-cdk-eks-multi-region-skeleton/
npm i
```

## Run `npm run watch`

As described before, in typescript, you have to go through the transpilation process into javascript.
To compile changes automatically in this project directory, refer to [here](/en/30-cdk/200-watch/) and run `npm run watch` in the background.


The screenshot below is an example of running a terminal in the VScode IDE.

![](/images/20-single-region/npm-run-watch.png)
