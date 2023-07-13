# 软件安装
现阶段微纳光学智能设计软件安装是将不同算法环境下/不同Dockerfile制作的Docker镜像部署到Linux主机或虚拟机，通过启动yml文件来运行智能设计软件。因此本地Linux主机或虚拟机需安装Docker和Docker-compose环境.
## 什么是Docker？
Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

因此docker可以屏蔽环境差异，只要程序被docker打包后，那么无论运行在什么环境下程序的行为都是一致的，可以做到真正实现“build once, run everywhere”。

此外docker的另一个好处就是快速部署，一个原因是容器启动速度非常快，另一个原因在于只要确保一个容器中的程序正确运行，那么你就能确信无论在生产环境部署多少都能正确运行。

Docker包括三个基本概念：
+ 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
+ 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
+ 仓库（Repository）：仓库可看成一个代码控制中心，用来保存镜像。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。
## Linux本地安装Docker engine及Docker compose
根据本地Linux系统选择对应的docker engine及docker-compose版本进行安装，下载链接：[ubuntu系统下安装docker engine](https://download.docker.com/linux/ubuntu/dists/ )，[centos系统下安装docker engine](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)，[安装docker-compose](https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-linux-x86_64)。

## 拉取代码，启动微纳光学智能设计软件
通过Git拉取微纳光学智能设计软件后端代码，通过CMD








得到算法的Dockerfile部署在后端，基于Docker镜像构建相对于的Python 3运行环境、Jupyter kernel服务器、一些接口代码用于前端展示以及一些基础设施比如mongo DB的连接，Websocket Server相关的连接配置等。前端使用Websocket连接后端服务器，需执行的代码中通过提供的Python接口与前端实时交互，可视化建模中Python脚本窗口使用React插件React-Code Mirror来实现Python代码编写和输出的窗口环境。


