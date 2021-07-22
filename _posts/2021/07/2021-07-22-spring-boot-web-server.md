---
layout: post
title:  Spring Boot 嵌入式 Web 服务器
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君。

今天指北君要带实习生来了解一下 `Spring Boot` 嵌入式 Web 服务器的相关内容。好，上思维导图目录。文末有完整的思维导图哦。

![概览目录](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server1.png)

**指北君**：实习生，你过来下，我今天给你讲讲 `Spring Boot` 嵌入式 Web 服务器。

**实习生**：好的，大佬，马上就来。

**指北君**：你先看下我这个思维导图，然后我一个一个的给你讲。

**实习生**：边看边讲吧。

**指北君**：那行，我直接来了。Spring Boot 现在有4种 web 服务器。分别是 tomcat ，jetty，undertow 和 netty。

![web服务器种类](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server2.png)

**实习生**：我们现在的项目只使用tomcat，其他的我就听过 Jetty 和 netty 。 另外的 Undertow 我听都没听过。

![一脸懵逼](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server3.png)

**指北君**：你以前不知道，现在知道了不就行了么。Undertow 是Jboss 旗下的开源服务器产品，根据GitHub上一些开发者的性能压测，Undertow 在 `Spring Boot` 里可是最强的。

**实习生**：那我们为什么不用 Undertow？

**指北君**：那你去问上面？别扯这些了，下面还要讲呢。现在我来教你我们平时怎么切换web服务器。

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

一般情况，我们引入 web 服务器是这样的，因为 `spring-boot-starter-web` 默认内置了tomcat。比如说我们现在要切换到 Jetty ，我们要怎么操作呢？

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- 排除tomcat的maven依赖 -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 添加Jetty 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

这里有一点需要注意下，因为 jetty 9.4 是不支持 servlet 4.0 的，所以我们在使用的时候，需要做 Jetty 和servlet-api 的版本适配。

![Jetty](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server4.png)

**实习生**：既然后 Jetty 有这个问题， 另外的 undertow 和 netty 有这个问题吗？

**指北君**：undertow 适配的很好，没有这个问题， netty 这个你搞错了，netty 是在 webflux 的场景下才使用的，所以不存在你的疑问。

**实习生**：那我换成 undertow 是不是直接把 jetty 改成 undertow 就行了。

**指北君**：是的，没毛病。接下来，我们来看下一个分支，如何禁用 web 服务器。

**实习生**：我不懂，为什么要禁用web服务器？平时不用的话，不把 `spring-boot-starter-web` 引入进来不就好了么。

**指北君**：我们有些时候会封装一些基础库，放到我们的基础框架里面。假如我们要做一个定时任务的程序，这个时候我们其实是不需要有web功能的。但是项目依赖已经有了，拆出去可能也比较麻烦。幸好 `spring boot` 团队给我们提供了一个开关可以控制。

```java
spring.main.web-application-type=none
```

这样简单的配置一下，搞定了我们拆分框架封装一堆的事情。

**实习生**：给 `spring boot` 团队点赞👍 。

![点赞](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server5.gif)

**指北君**：好，下一个端口修改。这个功能比较简单了。只要改一下 `application.properties` 文件就行了。

**实习生**：你是说那个 `server.port` 的配置项？这玩意还有啥可以讲的。

**指北君**：你不要小看它， 你知道怎么配置随机端口吗？ 你知道怎么配置启动web容器，但不暴露吗？

**实习生**：我……

![疑问](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server6.png)

**指北君**：其实也挺简单的。随机端口只要配置 `server.port=0` 就可以了。不暴露端口的话， 配置 `server.port=-1` 这样就OK了。

**实习生**：好棒，我学废了。

**指北君**：可以可以，再来看看http2 的支持情况。总体来说，对http2支持最好的是undertow，原生支持，完全不需要一些特殊处理。

![http2的支持](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server7.png)

**实习生**：这看起来有点复杂啊，脑壳疼。

**指北君**：没事，等我讲完了，我把这个思维导图发给你。 再来看看日志配置吧，http的访问日志，有些时候也挺重要的，需要分析之类的。

![日志配置](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server8.png)

**实习生**：哇哦，so easy。我学废了。

**指北君**：是不是，思维导图这么一画，内容就清晰很多了，讲起来也不费力，你听起来也比较轻松。

**实习生**：是的，感谢大佬，别忘了，把这个导图发我。 我先走了。

**指北君**：好嘞好嘞。

![开始摸鱼](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server10.png)

### 总结

今天我跟实习生梳理了一下，Spring Boot 项目中嵌入式 web 服务器的相关内容，包括web服务器种类、如何切换web服务器、如何在不删除依赖的条件下禁用服务器、如何修改端口，HTTP2的支持情况以及日志配置的处理。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。

![完整导图](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-web-server9.png)
