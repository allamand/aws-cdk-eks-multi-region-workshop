---
title: Docker-cli
weight: 600
pre: "<b>2-6. </b>"
---

in **Lab 3, Deploy apps into 2 regions** , you will build a docker image to use in AWS CodeBuils as a build image. This requires you to have docker-cli installed and running.


## Install Docker-cli
Install docker-cli with the following link.
* https://docs.docker.com/get-docker/


Make sure your docker-cli is successfully installed with the following command.
```
docker --version

# Example of console output
Docker version 19.03.8, build afacb8b

```

## Run Docker daemon
Run Docker daemon with the instruction:
* Mac: https://docs.docker.com/docker-for-mac/install/#install-and-run-docker-desktop-on-mac
* Windows: https://docs.docker.com/docker-for-windows/install/#start-docker-desktop

You can figure out if your docker daemon is running with the command below.

```
docker ps -q
```

* If you freshly installed docker, it will print nothting since there is no container running.
* If your docker daemon is not running you would see this error: `Error response from daemon: dial unix docker.raw.sock: connect: connection refused`