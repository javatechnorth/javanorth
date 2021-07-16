---
layout: post
title:  Spring Boot Context Path
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君。

这段时间有点忙，写文章有点拉下啦。实习生的问题真的挺多的，指北君有点 hold 不住了。 今天给实习生介绍 `Context Path` (上下文路径)。起因是这样的，实习生见公司项目都部署在同一个域名下，过来问我为什么那么多项目都能搞到一起，挺神奇的。

**指北君**：你前几天问了我，那么多项目怎么搞在一起的问题，因为那时候挺忙的，没有回答你，你自己搞清楚了吗？

**实习生**：有个大致的概念了吧，`nginx` 反向代理是吧。

**指北君**：嗯，是那么一回事情，那你知道 `Java` 程序也有类似的东西，叫做 `Context Path` (上下午路径)。你懂吗？

**实习生**：这个我好像有印象，用 `tomcat` 的时候，可以配置那玩意。

**指北君**：年轻人可以啊，还用过 `tomcat` 。那你知道在 `Java web` 里面怎么配置的吗？

**实习生**：哦，你说到 `Java web`，我有印象了，在学校里的时候，学习 `JSP` 的时候，有这个东西，但是具体怎么配置已经忘记了，还给老师了。

**指北君**：哈哈哈，`JSP` 好古老的东西啊，学校里还教啊。不过没事，今天我给你讲讲在 `Spring Boot` 中怎么配置使用 `context path` 。 

在 `Spring Boot 2.x` 配置其实挺方便的，只要设置 `server.servlet.context-path` 就行啦。在 `Spring Boot 1.x` 中有一点点小区别，配置字段有变化 设置 `server.context-path` 就好了。

**实习生**：大佬，你先等等，我有个疑问，你说的这个看起来挺简单的，1.x 和 2.x 为什么变路径了啊？2.x 中间多了一层 servlet，`Java web` 不是都是 `servlet` 吗？某非这 2.x 有其他的出现了？

**指北君**：年轻人，不错啊，嗅觉很灵敏啊。2.x新增了 webflux，抽空可以了解下，只是现在好像没啥使用场景。也没啥大厂去用这玩意。有时间我给你讲讲。感觉扯远了。接下来跟你说说在项目里怎么去设置。

#### 使用 `application.properties` 设置

在 `application.properties` 设置这个属性，是更改上下文路径最简单方法啦。一行代码搞定它。

```java
server.servlet.context-path=/javanorth
```

我们可以不把 `application.properties` 文件放在 `src/main/resources`中，也可以把它放在当前工作目录中（classpath之外）。

**实习生**：为什么可以不放在 `src/main/resources` 。放在其他地方，Spring Boot 项目怎么读取文件啊？不都是从 jar 包里读取的吗？

**指北君**：那你可小看spring boot 团队了，他们怎么可能没考虑这个问题呢？现在我要怀疑你在学校里到底有没有学Spring了，spring 读取的配置文件是不是散落在各地呢？

**实习生**：好像是的哦。居然可以放在外面读取应该还有其他的配置方式吧？

**指北君**：那当然了，再教你几个方法。

#### Java系统属性

我们也可以在上下文被初始化之前将上下文路径设置为一个Java系统属性。 直接在代码里写，只是这样不太友好。

```java
public static void main(String[] args) {
    System.setProperty("server.servlet.context-path", "/javanorth")。
    SpringApplication.run(Application.class, args)。
}
```

#### 操作系统环境变量

Spring Boot也可以依赖操作系统环境变量。在基于Unix的系统上，我们可以这样写。

```shell
$ export SERVER_SERVLET_CONTEXT_PATH=/javanorth
```

在Windows上，设置环境变量的命令是。

```shell
> set SERVER_SERVLET_CONTEXT_PATH=/javanorth
```

上述环境变量适用于Spring Boot 2.x，如果我们有1.x，变量是SERVER_CONTEXT_PATH。

#### 命令行参数

我们也可以通过命令行参数动态地设置属性。

```java
$ java -jar app.jar --server.servlet.context-path=/javanorth
```

#### 使用 Java Config 设置

现在让我们通过向Bean工厂填充配置Bean来设置上下文路径。

通过Spring Boot 2，我们可以使用WebServerFactoryCustomizer。

```java
@Bean
public WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> webServerFactoryCustomizer() {
    return factory -> factory.setContextPath("/javanorth")。
}
```

通过Spring Boot 1，我们可以创建EmbeddedServletContainerCustomizer的实例。

```java
@Bean
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer() {
    return container -> container.setContextPath("/javanorth")。
}
```

**指北君**：一口气说了这么多，累死我了。这几种方法你都知道了吧？

**实习生**：大佬NB，我知道了。有这么多中设置方法，那你跟我讲讲万一我多设置了几种方式，配置生效的优先级是怎么样的？

**指北君**：哟，不错嘛，还能想到这个问题。

**实习生**：大佬带的好。

**指北君**：好了好了，这个就别吹了。先来看看吧。

#### 配置的优先级顺序

有了这么多的选项，我们最终可能会对同一个属性有不止一个的配置。

下面是降序排列的优先顺序，Spring Boot用它来选择有效的配置。

1. Java 配置
2. 命令行参数
3. Java系统属性
4. 操作系统环境变量
5. 当前目录中的application.properties
6. classpath（src/main/resources或打包的jar文件）中的application.properties

**指北君**： 懂了没？

**实习生**：给大佬点赞👍 。我懂了。

**指北君**：好了，你自己去干活吧，别来影响我摸鱼了。

### 总结

在这篇文章中，我们快速地介绍了在Spring Boot应用程序中设置上下文路径或任何其他配置属性的不同方法。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。
