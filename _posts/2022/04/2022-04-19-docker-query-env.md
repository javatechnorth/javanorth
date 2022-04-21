---
layout: post
title:  谈谈如何从 Docker 容器中获取环境变量
tagline: by feng
categories: docker
tags: 
    - feng
---

大家好，我是指北君。

Docker是一个容器化的平台，它将一个应用程序连同其所有的依赖关系打包。大部分情况下，这些应用程序需要一个特定的环境来启动。在Linux中，我们使用环境变量来满足这一要求。这些变量决定了应用程序的行为。
<!--more-->
在本文中，我们将学习如何检索在运行 Docker 容器时设置的所有环境变量。就像有多种方法向Docker容器传递环境变量一样，一旦设置了这些变量，也有不同的方法来获取这些变量。

首先让我们了解一下环境变量的必要性。

### Linux 中的环境变量

环境变量是一组动态的键值对，可以在整个系统中访问。这些变量可以帮助系统定位软件包，配置任何服务器的行为，甚至使 bash 终端的输出更直观。

默认情况下，主机上的环境变量不会被传递给 Docker 容器。原因是 Docker 容器应该是与主机环境隔离的。因此，如果我们想在 Docker 容器中使用环境，那么我们必须明确地设置它。

现在让我们来看看从 Docker 容器内部获取环境变量的几种方法。

### 使用 docker exec 命令获取信息

为了演示，让我们首先运行一个 Docker 容器，并向它传递一些环境变量。

```shell
docker run -itd --env "my_env_var=javanorth" --name mycontainer 
```

在这里，我们将 my_env_var 的值 javanorth 传递到名为 mycontainer 的 Docker 容器中。

现在让我们使用 docker exec 命令来获取名为 my_env_var 的环境变量。

```shell
$ docker exec mycontainer /usr/bin/env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=xxxxxxxxx
my_env_var=javanorth
HOME=/root
```

我们正在 Docker 容器内执行 `/usr/bin/env` 工具。使用这个工具，你可以查看 Docker 容器内设置的所有环境变量。注意，我们的 my_env_var 也出现在输出中。

我们还可以使用下面的命令来实现类似的结果。

```shell
$ docker exec mycontainer /bin/sh -c /usr/bin/env
HOSTNAME=xxxxxxxxx
SHLVL=1
HOME=/root
my_env_var=javanorth
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
```

注意，与之前的输出相比，有更多的环境变量。这是因为这次我们是在 `/bin/sh` 二进制的帮助下执行命令的。这个二进制文件隐含地设置了一些额外的环境变量。

另外，`/bin/sh` shell并不是所有的 Docker 镜像都必须存在的。例如，在包含 `/bin/bash` shell的 centos Docker 镜像中，我们将使用以下命令来检索环境变量。

```shell
$ docker run -itd --env "container_type=centos" --name centos_container centos

$ docker exec centos_container bash -c /usr/bin/env
container_type=centos
HOSTNAME=xxxxxxxx
PWD=/
HOME=/root
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
```

我们也可以使用 docker exec 命令来获取单个环境变量的值。

```java
$ docker exec mycontainer printenv my_env_var
javanorth
```

printenv 是另一个命令行工具，用于显示 Linux 中的环境变量。在这里，我们将环境变量的名称 my_env_var 作为参数传给 printenv。这将打印出 my_env_var 的值。

这种方法的缺点是，为了检索环境变量，Docker容器必须处于运行状态。

### 使用 docker inspect 命令获取

现在让我们来看看另一种在 Docker 容器处于停止状态时获取环境变量的方法。我们将使用docker inspect 命令来实现这一目的。

docker inspect 提供所有 Docker 资源的详细信息。输出是JSON格式的。因此，我们可以根据我们的要求过滤输出。

让我们操作 docker inspect 命令，只显示容器的环境变量。

```shell
$ docker inspect mycontainer --format "{{.Config.Env}}"
[my_env_var=javanorth PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
```

在这里，我们使用-format选项过滤了 docker inspect 输出中的环境变量。同样，输出中出现了my_env_var。

与docker exec不同的是，docker inspect命令对停止和运行的容器都有效。

### 总结

在这篇文章中，我们学习了如何从 Docker 容器中检索所有的环境变量。我们首先讨论了Linux中环境变量的重要性。然后我们了解了 docker exec 和 docker inspect 命令来检索环境变量。

docker exec方法有一些限制，而docker inspect命令可以在所有情况下运行。