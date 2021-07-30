---
layout: post
title:  Spring Boot logging
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君。

今天指北君将要带实习生来了解下 Spring Boot 中日志框架的使用。完整的思维导图文件放到文末的github地址中，可以自行获取。

**实习生**：大佬好，今天你要讲什么呀？

**指北君**：今天跟你讲讲 Spring Boot 日志是怎么回事的，先看下我整理的思维导图吧。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging1.png)

<!--more-->

今天就从这几个角度来讲。

**实习生**：好的，那我们先来看看概述吧。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging2.png)

**指北君**：spring boot 内部使用的是 apache 开源的 commons logging 来记录日志的。但是呢，它又提供了基础的日志实现接口，方便项 java util logging、 log4j、logback之类的接入。spring boot 现在已经为 Java util logging 、log4j2、logback 提供了默认的实现。我们如果需要使用的话，只要简单配置就可以了。

**实习生**：要这么说的话，spring boot 可真nb啊，轻轻松松就搞定了。你刚刚说了 spring boot 已经内置实现了 logback 这些， 那我看你图里还有starter的描述啊,默认所有的都夹在进来了吗？

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging3.png)

**指北君**：小伙子可以啊，spring boot 当然没有把所有的都加载进来，条件自动化配置你忘记了吗？ 如果我们项目只依赖了一个 spring-boot-starter 的话，你看我这个项目的依赖图，自由这么几个，如果你想依赖 log4j2 的话，你要添加 spring-boot-starter-log4j2 才行。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging4.png)

**实习生**：soga， 我懂了。

**指北君**：我们来看看日志的输出格式是怎么样的。我们平时启动项目的时候，在控制台上看到的这些内容，是有固定的格式的。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging5.png)

一开始是日期和时间，然后是日志的等级、线程ID、分割线、线程名称、logger name 、日志消息。我在思维导图里做了归纳， 日志等级这里我使用的是logback的等级制度，部分其他的会是以FATAL 代替 ERROR 的。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging6.png)

**实习生**：这个挺简单的呀。你这么一画，我感觉很清晰啊。你看这个终端输出那里有不同的颜色，这个是怎么配置的呢？

**指北君**：不要着急啊， 你看我的思维导图，下一个环节不就是要讲了吗？

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging7.png)

默认情况下，spring boot 项目的日志输出方式是控制台输出。控制台输出一般情况下只有3中输出方式，就是上面写的 ERROR、WARN、INFO。因为打开debug的话，所有其他的日志都会打出来，对我们开发来说不太方便。

**实习生**：那这个还是可以打开的吧，或者我想调试spring boot 的启动过程之类的，我就想把它打开，这样我对启动过程可以看的更加清楚一些。

**指北君**：是的，你可以打开，而且也挺方便的。有两种方式，第一种方式就是通过命令行设置

```java
java -jar myapp.jar --debug
```

第二种：在配置文件里直接配置就行

```java
application.properties 配置 debug=true
```

至于，刚才你说到的颜色问题，其实也蛮简单的，如果你的控制台支持 ASNI 的话，就配置一下就可以了，主要是配置%clr 这个轻轻松松搞定了。直接看思维导图就行了。

**实习生**：你说的没错，但是我还有一个疑问，ASNI是什么？

**指北君**：这玩意就是一种编码方式，一句两句也讲不清楚，我给你搜索一下吧。
> ANSI是一种字符代码，为使计算机支持更多语言，通常使用 0x00~0x7f 范围的1 个字节来表示 1 个英文字符。超出此范围的使用0x80~0xFFFF来编码，即扩展的ASCII编码。

**指北君**：接下来我们看看文件输出日志的配置，默认情况下我们只要在applicaiton.properties文件直接配置就行了。

```java
logging.file.name=app.log
logging.file.path=/xxx
```

**实习生**：这个我知道，但是你导图上写的文件分割，又是怎么回事呢？

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging8.png)

**指北君**：这个你都没懂吗？我们日志写入到文件里之后，如果我们不处理的话，文件可以把你电脑写爆了。另一方面，方便我们处理日志文件啊。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging9.png)

**实习生**：好的，大佬我错了。我怎么没想到呢。

**指北君**：别扯这些，我们继续来看。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging10.png)

**实习生**： 你这个写的啥意思啊？没懂，上面不是讲过了，再讲一遍？

**指北君**：上面只是提了下等级，并没有提怎么设置，你看仔细了，我们可以针对某个package来设置日志输出的等级。

**实习生**：唉，这个我怎么没想到呢，可以单独设置。我之前看项目我都没注意到这个问题。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging14.png)

**指北君**：没关系，现在不是已经知道了吗？我们继续吧。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging11.png)

**实习生**：好的，大佬，你这里写的日志关闭hook是干啥的？

**指北君**：其实这个就像我们spring boot 怎么做到优雅停机一样。我们要做到先把日志打印入口先关闭了，然后把打印的日志打印完，我再结束我这个日志打印的线程。

**实习生**：你这么说我就明白了。

**指北君**：上面讲述的都是通过配置spring boot 默认实现的方式来配置日志，我们还可以自己个性化配置日志输出的情况。在这我们以前spring 项目的时候，是很常见的。我们现在也一直在延续使用这种方式。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging12.png)

**实习生**：那你这个最后那个环节 也是和这个自定义配置有关了吧

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-logging13.png)

**指北君**：你小子眼睛还挺尖的啊，我这里主要是说明两个东西，一个是 spring boot profiles在日志文件中的应用。

```java
<springProfile name="dev | staging">
    ...
</springProfile>
```

 另外一个是怎么引入 sping的环境变量。

```java
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

**实习生**：hoho，可以啊，这操作不错。

### 总结

指北君今天就是带大家过一遍 spring boot logging 是什么一回事，怎么配置使用的。

本文的所有示例源代码和完整的思维导图都已上传到了 Github：

> https://github.com/javatechnorth/java-north-sample

欢迎大家 Star 关注，后续会不断更新。
