---
layout: post
title:  Spring Boot 入门指南 --20210601
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

### 什么是Spring Boot

Spring Boot 是 Spring 开源组织下的一个子项目，也是 Spring 组件一站式解决方案，主要是为了简化使用 Spring 框架的难度和简化 Spring 框架复杂的XML配置。使用 Spring Boot 可以很容易创建一个独立运行的、基于 Spring 的生产级应用程序，而且Spring Boot 对 Spring 平台和第三方库做了一些版本适配，这样我们就可以尽快的上手。

<!--more-->

使用 Spring Boot 来不仅可以创建基于 war 方式部署的传统Java应用程序，也可以通过创建独立的不依赖任何容器（如 tomcat 等）的应用，只需使用 “java -jar” 就能启动。Spring Boot 还提供了一个运行 "spring scripts " 的命令行工具。

Spring Boot 的设计目标

    为所有的 Spring 开发提供一个从根本上更快、更广泛的入门体验。
    开箱即用，但当需求开始偏离默认值时，可以快速从中解放出来使用个性化的定制。
    提供一系列大型应用项目中经常用到的公共的非功能特性（如嵌入式服务器、安全、度量、健康检查和外部化配置）。
    零代码生成零XML配置。

### 第一个 Spring Boot 应用程序

我们在学习一门新的开发语言，或者开发框架的时候，一般都习惯写个 Hello World 的项目。一方面可以验证基础环境的搭建是否正确，另一方面可以快速了解整个开发流程。现在我们创建一个 Hello world 的 Spring Boot 项目。我这里使用的IDE 是 Intellij IDEA 。

#### 第一步

打开Intellij IDEA 使用 Spring Initializr 向导 新建 Hello World的项目

![Spring Boot Started 1](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-started-1.png)

选择 Spring Web， 然后点击 Finish 等待项目创建完成。

![Spring Boot Started 2](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-started-2.png)

#### 第二步

项目已经创建完成。下面我们来看一下项目结构：

![Spring Boot Started 3](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-satrted-3.png)

来看下 HelloApplication 入口类的内容, HelloApplication 里面定义了一个 main 函数，一个基本的 @SpringBootApplication 注解。后续的文章会详细解释一下 @SpringBootApplication 注解。现在我们只要知道有了 @SpringBootApplication 注解，所有的 Spring Boot 项目依赖都可以扫描加入进来。

```java
@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }
}
```

再看一下 pom.xml 文件有那些依赖项目， POM 文件主要依赖了spring-boot-starter-web 项目，该项目包含了 web 项目所需的相关依赖，包括内置了 tomcat 服务器。另外还有一个比较重要的插件 spring-boot-maven-plugin，方便我们可以对 Spring Boot 项目打包成一个独立运行的 jar 包。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.javanorth</groupId>
    <artifactId>hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>hello</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 第三步

修改 HelloApplication 主类，添加一个/hello 的请求

```java
@RestController
@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }
    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

#### 第四步

在主类上，右键菜单栏里选择 Run “HelloApplication” 命令，启动项目。如下图所示，2秒钟就能启动完成。从输出日志可以看出，项目启动了内置的 tomcat 服务器，分配了8080 作为本次 web 服务器的请求端口。

![Spring Boot Started 4](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-started-4.png)

好了，我们现在访问一下 /hello 请求， 如下图所示，输出了 Hello World。

![Spring Boot Started 5](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-started-5.png)

是不是很简单，我们很快就上手完成了第一个 Hello World 应用。 

#### 第五步

我们再来看下，如何打包一个 Spring Boot 项目呢？ 前文我们已经提到了 spring-boot-maven-plugin 插件，所以我们可以使用mvn package 来对其进行打包

```shell
$ mvn package 
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------< com.javanorth:hello >-------------------------
[INFO] Building hello 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] ...
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ hello ---
[INFO] Building jar: /Users/wbf/Documents/javanorth/target/hello-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.5.0:repackage (repackage) @ hello ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.065 s
[INFO] Finished at: 2021-05-27T16:33:17+08:00
[INFO] ------------------------------------------------------------------------
```

打包完成，查看 target 目录，我们能看到 hello-0.0.1-SNAPSHOT.jar 文件。

![Spring Boot Started 6](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-started-6.png)

从上图可以看到有个名字相类似的文件，hello-0.0.1-SNAPSHOT.jar.original 文件是 Spring Boot repackage 之前的文件，也就是说这个文件其实就是 maven 创建的原始jar文件，不包含其他依赖的jar包。

如果我们想看一下 hello-0.0.1-SNAPSHOT.jar 里面包含那些内容，有没有什么快速的办法呢，我这里给大家提供一个命令行的方法，使用 “jar tvf ” 就行。
```shell
jar tvf hello-0.0.1-SNAPSHOT.jar
```
### 小结

从上面的示例可以看出开始一个新的 Spring Boot 项目非常简单，Spring Boot 提供了专门的创建向导项目，简化了大量的 Spring 项目的创建难度。全程下来不到5分钟，我们就完成了项目的创建和 hello world 的输出演示。

后续的文章会给大家讲解 Spring Boot 的更多的细节，给大家带来更详细的技术文章，包括源码分析、面试题讲解、一些有趣的实现等。关注公众号【Java技术指北】第一时间获取最新 Spring Boot 的相关技能。

[EOF]