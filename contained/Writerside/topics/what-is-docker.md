# 初识 Docker 

掌握好 Docker 是深入容器技术的第一步，所以我们从这篇文档开始先来聊聊什么是 Docker。

## What is Docker {id="what-is-docker"}

什么是 Docker，它是一个开源的应用容器引擎。首先它是开源的，使用 Go 语言编写。接着，他是一个应用容器引擎。容器，是用来承装内容，而此处的内容指的是应用，即我们开发出来的软件系统。既然是软件，就会有其运行环境。既然是系统，就会有很多的模块、组建、依赖。**而 Docker 就可以将软件系统(application)这一切打包、存储、发行、运行、管理、更新乃至销毁。**

什么是容器？我们来看看 Docker 官网上的一篇介绍 —— [What is a Container?](https://www.docker.com/resources/what-container) 接下来的内容，是我对这篇文章的翻译。

## What is a Container {id="what-is-container"}

> A standardized unit of software


什么是容器？软件最基本的单位。

> Package Software into Standardized Units for Development, Shipment and Deployment


将软件包装成用于开发、运维以及部署的最基本的单位。

> A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.


一个容器是软件的最基本的单位，其封装了代码以及相关的全部依赖，所以可以在一台或多台的计算机环境中快速以及稳定的运行。一个 Docker 容器镜像是轻量级、单一的、可执行的软件包，其包含了软件在运行中所需的代码、运行时、系统工具、系统库以及相关的配置。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/jMFKaFhxJdlGVOMhIhN7.png" alt="what is container"/>

> Container images become containers at runtime and in the case of Docker containers images become containers when they run on Docker Engine. Available for both Linux and Windows-based applications, containerized software will always run the same, regardless of the infrastructure. Containers isolate software from its environment and ensure that it works uniformly despite differences for instance between development and staging.


当 Docker 的容器镜像在 Docker 引擎中运行的时候会成为容器。而且无论是 Linux 还是基于 Windows 的应用，在各种基础架构中都能良好的运行。容器能够隔离软件的运行环境以确保他们的工作表现是一致的，即使是开发和生产环境是存在些许的差异。

## Comparing Containers and Virtual Machines {id="diff_containers_and_vm"}

那么容器和虚拟机之间有什么区别呢？

> Containers and virtual machines have similar resource isolation and allocation benefits, but function differently because containers virtualize the operating system instead of hardware. Containers are more portable and efficient.


容器和虚拟机都有着类似的资源隔离和分配机制，但是两者的功能不尽相同。因为容器是基于操作系统层面的虚拟化，而虚拟机是基于硬件的虚拟化。所以，相对于虚拟机，容器更加的轻量以及高效。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/B1hPoSsgE5yXNCB1QFoM.png" alt="diff-container-and-vm"/>

> Containers are an abstraction at the app layer that packages code and dependencies together. Multiple containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes in user space. Containers take up less space than VMs (container images are typically tens of MBs in size), can handle more applications and require fewer VMs and Operating systems.


容器是在应用层的抽象，它封装了代码以及相关的依赖。多个容器可以运行在数相同的机器上，并且与其他容器共享系统的内核。每个容器运行的进程都是在用户态中隔离的。容器相对于虚拟机消耗更少的空间(容器镜像通常只有十多兆的大小）。只需要更少的虚拟机和操作系统资源，就可以运行更多的应用。

> Virtual machines (VMs) are an abstraction of physical hardware turning one server into many servers. The hypervisor allows multiple VMs to run on a single machine. Each VM includes a full copy of an operating system, the application, necessary binaries and libraries - taking up tens of GBs. VMs can also be slow to boot.


虚拟机是对物理硬件的抽象，可以将一台服务器转化为多台服务器。Hypervisor 允许多个虚拟机运行在单台机子上。每个虚拟机都包含了一个完整的操作系统，以及应用程序、必不可少的二进制文件和库 - 其花费的空间通常是十多 GB。虚拟机启动的更慢。

> Containers and VMs used together provide a great deal of flexibility in deploying and managing app.


容器和虚拟机的结合，可以让我们更加灵活的部署和管理应用。

## Docker Architecture

> Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface.


Docker 使用的是 C/S (客户端/服务器) 架构。由 Docker 的客户端区请求 Docker 引擎，由其完成 Docker 容器从构建、运行到分发的生命周期。Docker 的客户端以及 Docker 引擎可以运行在同一个系统中，或者你可以使用客户端连接到远程的 Docker 引擎。Docker 的客户端和 Docker 引擎通过 REST API 来进行通信，或者通过 UNIX Sockets 或网络接口。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/LqiqVRXgzxHix0tNLL4G.png" alt="docker-arch"/>

## Install Docker

安装 Docker 在 CentOS7 中，下面给出 Shell 的脚本，已经将镜像更换外阿里云:
```bash
#!/bin/bash

## Install use yum
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y

## Start Service
systemctl start docker
systemctl enable docker
docker --version
## 如果出现没有权限，可以将用户添加到 docker 的用户组中
gpasswd -a vagrant docker

## Modify docker registry image
cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF

systemctl restart docker
```
其他的系统或发行版，请参考官方文档: [Docker Install](https://docs.docker.com/engine/install/) 。官方脚本安装:
```shell
sudo sh -c "curl -sSL https://get.docker.com/ | sh"
dockerd-rootless-setuptool.sh install
## 将下面这两行追加到 ~/.bashrc 配置文件中
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
source ~/.bashrc
```
## Summary

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/XeGGphjcYV0hLgszHuhR.jpg" alt="docker-cli-summary"/>

Docker 是 C/S 架构的，包含 Docker Cli 以及 Docker Engine，我们可以通过 Docker Cli 来操作 Docker Engine。

我们可以使用 Docker 来打包应用，打包后的应用成为了 Docker 容器镜像。而 Docker 容器镜像在 Docker Engine 中运行的时候，我们就称之为容器或者说是容器的实例。

另外我们也对比了容器和虚拟机的区别，容器相对虚拟机更加的轻量级，而在实际的生产中，他们都不是孤立存在的，而是相互结合，共同作用于我们的应用的生产、部署过程中。

**容器的本质是进程**，**在进程的基础上对资源进行隔离、限制。**
