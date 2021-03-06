---
layout: post
title: 都在聊云开发了，你还没听过Docker？--20210805
tagline: by 程序员七哥
categories: Docker
tags:
- 程序员七哥
- Docker
---

Docker 的概念想必大家已经了解了，最直观的好处就是可以不安装虚拟机，不需要安装软件，不需要配置环境的情况下，就将我们想用的软件跑起来。

![image-20210613163236155](http://www.javanorth.cn/assets/images/2021/sevenluo/docker-image-1.png)

<!--more-->

那你可能会问了，Docker 凭什么这么方便，它具体又是怎么玩的呢？

别急，要想了解 Docker 的整个生命周期，那了解它的三大核心概念是跑不了的。

- 镜像（Image）

- 容器（Container）

- 仓库（Repository）

诶，你是否有种似曾相识的感觉，凭借着多年来敏锐的技术嗅觉，这尼玛好像和 git 有点像呀。每天将写完的代码就是推送到远程仓库了，而这个镜像感觉也就是我们的项目。

![image-20210613163328243](http://www.javanorth.cn/assets/images/2021/sevenluo/docker-image-2.png)

下面咱继续往下整，今天绝对让你搞明白 Docker 这三大核心概念。

### Docker镜像

看到镜像，你应该能联想到我们平时安装虚拟机时下载的镜像文件，没错！这两个是类似的。Docker 镜像你可以理解为它是专门为 Docker 引擎设计的模板，里面包含了文件系统。

你可能接触过，在工作中将你们开发的 web 应用打包成一个镜像，然后在测试环境、生产环境部署启动，那么这个镜像中就只包含了你们开发的 Web 应用和所需要的运行时环境。

又比如一个镜像可以只包含一个 ubuntu 操作系统环境，那就可以把它称为一个 ubuntu 镜像。也可以只安装了 Tomcat 应用程序，那么就称它是一个 Tomcat 镜像。 所以镜像是可以包含任何用户需要的其它软件。

这个镜像呀是创建 Docker 容器的基础。而且还能通过版本管理像 Git 一样来获取各个版本的镜像。同时 Docker 也提供了一套十分简单的机制来创建和更新现有的镜像，我们也可以从网上下载别人做好的应用镜像，比如 kafka、rocketmq、redis、mysql等等应用程序的镜像，直接在本地就可以使用，不需要安装、部署、配置。

### Docker容器

Docker 容器类似于一个轻量的沙箱，将我们需要的应用程序放在里面执行。之所以说它像沙箱，是因为 Docker 利用容器来运行和隔离应用，这些容器都是相互隔离，互不可见的。

想必你也经常听运维同学说，xx 应用容器挂了，重启下 xx 容器 。这里容器就相当于一个镜像实例，镜像只是一个模板，写在文件系统中是死的，而容器就是利用这个模板启动的一个实例，容器占用了一个进程，监听在某一个端口。

你也可以将容器看做是一个简易版的 Linux 系统环境（包括进程空间、用户空间和网络空间等），以及运行在其中的应用程序打包而成的盒子。

**镜像本身是只读的。容器从镜像启动的时候，Docker 会在镜像的最上层创建一个可写层，镜像本省是保持不变的。**

### Docker仓库

说实话这个 Docker 还真和 git 是有点像的，你看着名字都叫的雷同。本质也确实一样，Docker 仓库就类似于代码仓库，是 Docker 集中存放镜像文件的地方。

这块还有一个概念需要明确给大家说下，就是这个 **注册服务器（Registry）**。很多初学者很容器将注册服务器和 Docker 仓库混为一谈。实际上它俩不是一个东西，注册服务器是存放仓库的地方，也就是说注册服务器上存放着很多个仓库。

每个仓库集中存放某一类镜像，一般包含多个镜像文件，通过标签（tag）来进行区分。 就比如存放 ubuntu 操作系统镜像的仓库中可能包含 14.04、16.04等不同版本的镜像，以方便我们按需使用。

上面注册服务器、仓库、镜像的关系我们用一张图来表示下，加深下你的理解和记忆。

![image-20210613163412881](http://www.javanorth.cn/assets/images/2021/sevenluo/docker-image-3.png)

Docker 仓库也根据所存储的镜像是否公开分享分为 **公开仓库** 和 **私有仓库**。

目前，最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供开发者下载。我们国内的公开仓库有 Docker Pool 等，可以提供稳定的国内访问。

当然，在公司里使用肯定是不希望镜像被公网访问的，因此 Docker 也支持在本地网络部署只有内部可以访问的私有仓库。

用户可以通过使用 push 命令将镜像上传，在其它机器上使用时，只需要将镜像从远程仓库 pull 下来就可以了，是不是非常方便呢？

### 小结

看到这里你应该对 Docker 的三大核心概念：**镜像、容器、仓库** 已经有了自己的理解，这三者正是构建高效工作流程的关键。毫无疑问，也正是 Docker 目前越来越流行，从众多虚拟化方案中脱颖而出的关键。
