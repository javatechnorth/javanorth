---
layout: post
title:  Docker And Dockerfile
tagline: by 付义帆
categories: Docker
tags:
- Docker
---

对于开发人员来说，会Docker而不知道Dockerfile等于不会Docker，上一篇文章带大家学习了Docker的基本使用方法：[《一文带你学会Docker》](https://mp.weixin.qq.com/s?__biz=Mzg4MjYyOTgwNw==&mid=2247497758&idx=1&sn=4645ca020d3bd0305eb3753dd3c2c6b8&chksm=cf5175def826fcc881114b13e73f1f2e04d3d5d81bf207f1a77ff98daab3978ea0cc486dfe71&token=1187363738&lang=zh_CN#rd)，今天了不起带大家学习一下Dockerfile，帮助你快速上手并创建高效的 Docker 镜像。

<!--more-->

### 了解Dockerfile

Dockerfile 是一个文本文件，用于定义 Docker 镜像的构建过程。它以指令的形式描述了如何构建镜像，从基础镜像开始逐步添加配置、文件和依赖，最终形成我们所需要的镜像。为我们提供了一种简单且可重复的方式来定义镜像构建过程。

### Dockerfile 指令

- **FROM 指令：** FROM 指令是 Dockerfile 的第一条指令，用于指定基础镜像。选择合适的基础镜像非常重要，因为它将直接影响镜像的大小和性能。我们还可以利用多阶段构建来减小镜像大小。
- **RUN 指令：** RUN 指令用于在镜像构建过程中执行命令。通过 RUN，我们可以安装软件包、运行脚本以及配置环境。
- **COPY 和 ADD 指令：** 这两个指令用于将本地文件复制到镜像中。区别在于 ADD 指令支持自动解压缩和远程 URL，但推荐使用 COPY 指令，因为它更明确和可预测。
- **CMD 和 ENTRYPOINT 指令：** 这两个指令用于定义容器启动时要执行的命令。CMD 定义的命令可以被 docker run 命令行参数所覆盖，而 ENTRYPOINT 定义的命令会一直执行。



以下是一个简单的Dockerfile 示例：

```dockerfile
# 使用 openjdk 镜像作为基础镜像
FROM openjdk:latest

# 设置工作目录
WORKDIR /app

# 复制 Java 项目的 JAR 文件到镜像中
COPY target/myapp.jar /app/

# 定义容器启动时执行的命令
CMD ["java", "-jar", "myapp.jar"]
```

在上面的示例中，我们使用 `openjdk:latest` 作为基础镜像，并将 Java 项目的 JAR 文件复制到镜像中。然后，通过 CMD 指令定义了容器启动时执行的命令，即运行 `java -jar myapp.jar` 启动 Java 应用程序。

### **多阶段构建** 

多阶段构建是一种优化 Docker 镜像大小的技巧，特别适用于构建 Java 项目等编译型语言的应用。在多阶段构建中，我们可以在一个 Dockerfile 中定义多个 FROM 指令，每个指令表示一个构建阶段。最终镜像只保留最后一个 FROM 指令所定义的阶段，其他中间产物都不会包含在最终镜像中，从而减小镜像的体积。

**Dockerfile 示例：**

```dockerfile
# 第一阶段：构建 Java 项目
FROM maven:latest AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src/ /app/src/
RUN mvn package

# 第二阶段：运行 Java 项目
FROM openjdk:latest

WORKDIR /app

COPY --from=builder /app/target/myapp.jar /app/

CMD ["java", "-jar", "myapp.jar"]
```

在上面的示例中，我们使用了两个 FROM 指令：

`FROM maven:latest AS builder` 表示第一阶段构建 Java 项目，使用 Maven 镜像进行依赖安装和项目构建；

`FROM openjdk:latest` 表示第二阶段，使用 OpenJDK 镜像来运行 Java 项目。通过 COPY --from 指令，我们从第一阶段的镜像中复制构建好的 JAR 文件到第二阶段，从而减小了最终镜像的大小。

### Dockerfile 高级用法

- **使用 ARG 和 ENV：** ARG 指令用于在构建过程中传递参数，而 ENV 指令用于设置环境变量。利用这些指令，我们可以更灵活地定制镜像的构建过程。
- **使用 WORKDIR：** WORKDIR 指令用于设置工作目录，即在容器内运行命令的默认目录。这样可以使 Dockerfile 更易读和维护。
- **使用 VOLUME：** VOLUME 指令用于在容器中创建挂载点，使得容器中的数据可以持久化保存在宿主机上。

**Dockerfile 示例：**

```dockerfile
# 第一阶段：构建 Java 项目
FROM maven:latest AS builder

# 使用 ARG 指令传递构建参数
ARG APP_VERSION=1.0.0
ARG BUILD_ENV=production

# 设置工作目录
WORKDIR /app

# 复制 pom.xml 并安装项目依赖
COPY pom.xml .
RUN mvn dependency:go-offline

# 复制源代码并构建项目
COPY src/ /app/src/
RUN mvn package -DskipTests

# 第二阶段：运行 Java 项目
FROM openjdk:latest

# 使用 ENV 指令设置环境变量
ENV APP_PORT=8080
ENV BUILD_ENV=${BUILD_ENV}

# 使用 VOLUME 指令创建挂载点
VOLUME /app/logs

# 设置工作目录
WORKDIR /app

# 复制构建好的 JAR 文件到镜像中
COPY --from=builder /app/target/myapp-${APP_VERSION}.jar /app/

# 定义容器启动时执行的命令
CMD ["java", "-jar", "myapp-${APP_VERSION}.jar", "--port=${APP_PORT}", "--env=${BUILD_ENV}"]

```

在上面的示例中，我们首先使用 `ARG` 指令来定义构建参数 `APP_VERSION` 和 `BUILD_ENV`，并在 `FROM maven:latest AS builder` 阶段中使用 `ARG` 指令传递构建参数。

这样，在构建时可以通过 `--build-arg` 参数来传递具体的值，例如：

```
cssCopy code
docker build --build-arg APP_VERSION=2.0.0 --build-arg BUILD_ENV=staging -t my-java-app .
```

这样可以构建不同版本和不同环境的镜像。

同时，我们使用 `VOLUME` 指令创建了挂载点 `/app/logs`，使得容器中的日志文件可以持久化保存在宿主机上。

### 小结

Dockerfile 是构建 Docker 镜像的核心工具，它使得镜像构建过程变得简单、可重复和高效。通过本文的介绍，你已经了解了 Dockerfile 的基本语法和常用指令，以及一些最佳实践。随着你的实践和深入学习，相信你将能够创建出更加优秀的 Docker 镜像，并更好地应用 Docker 在软件开发和部署中。
