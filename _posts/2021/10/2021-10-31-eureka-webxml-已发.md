---
layout: post
title:  erueka源码之 web.xml --20211111
tagline: by 某某白米饭
categories: eureka
tags: 
    - 某某白米饭
---

eureka 作为 Spring Cloud 的核心组件，学习他的源码是非常有必要的。
<!--more-->

### 导入到 idea

第一步要导入到 idea，为学习源码打造一把好用的工具。

源码地址： https://github.com/Netflix/eureka，建议 fork 到自己的 git 仓库，自己添加的注释、修改等等都可以提交，以后阅读也会方便很多。

小编这里选用的是 eureka 的 v1.10.10 版本。其他高版本在编译源码的是总会报各种错误。这个版本编译一次通过。因为小编的电脑问题，只能通过下载 Releases 的方式得到源码。

![](http://www.javanorth.cn/assets/images/2021/eureka/xml/0.png)

下载源码后，解压缩到文件夹。

![](http://www.javanorth.cn/assets/images/2021/eureka/xml/1.png)

双击文件夹中的 gradlew.bat 文件或者打开 cmd 命令行在里面运行也可以，这个时候它会先下载 gradle，然后才开始编译源码。

![](http://www.javanorth.cn/assets/images/2021/eureka/xml/2.png)

最后导入到 idea 中如下图：

![](http://www.javanorth.cn/assets/images/2021/eureka/xml/3.png)

### web.xml

在 eureka-server 包中的 WEB-INF 下有个 web.xml 文件，学过 java 的童鞋都应该知道 web.xml 是一个网站应用的标配。web.xml 中有 servlet、listener、filter 等信息。

1、web.xml 配置了一个 listener，是 eureka 的核心的启动入口，随着 web 应用的启动而启动。EurekaBootStrap 在 eureka-core 包下面，负责的是 eureka-server 的初始化操作。

```xml
  <listener>
    <listener-class>com.netflix.eureka.EurekaBootStrap</listener-class>
  </listener>
```

2、配置了 5 个 filter，分别是 statusFilter、requestAuthFilter、rateLimitingFilter、gzipEncodingEnforcingFilter、jersey。

```xml
  <filter>
    <filter-name>statusFilter</filter-name>
    <filter-class>com.netflix.eureka.StatusFilter</filter-class>
  </filter>

  <filter>
    <filter-name>requestAuthFilter</filter-name>
    <filter-class>com.netflix.eureka.ServerRequestAuthFilter</filter-class>
  </filter>
  <filter>
    <filter-name>rateLimitingFilter</filter-name>
    <filter-class>com.netflix.eureka.RateLimitingFilter</filter-class>
  </filter>
  <filter>
    <filter-name>gzipEncodingEnforcingFilter</filter-name>
    <filter-class>com.netflix.eureka.GzipEncodingEnforcingFilter</filter-class>
  </filter>

  <filter>
    <filter-name>jersey</filter-name>
    <filter-class>com.sun.jersey.spi.container.servlet.ServletContainer</filter-class>
    ....
  </filter>
```

1. statusFilter：负责状态相关的处理逻辑，对所有的请求开放。
2. requestAuthFilter：负责对请求进行授权认证，对所有的请求开放。
3. rateLimitingFilter：负责限流相关逻辑的，默认不开启，如果要打开 eureka-server 内置的限流功能，需要把web文件中 rateLimitingFilter 的的注释打开，让这个 filter 生效。
4. gzipEncodingEnforcingFilter：拦截 /v2/apps 相关的请求。
5. jersey：核心的 filiter，拦截所有请求。

3、配置了status.jsp 是欢迎页面、eureka-server 的控制台页面，展示注册服务的信息。

```xml
  <welcome-file-list>
    <welcome-file>jsp/status.jsp</welcome-file>
  </welcome-file-list>
```

### 总结

eureka-server 其实是一个 web 应用，可以被打成 war 包。可以被放在 tomcat 中运行。最重要的核心是 EurekaBootStrap 类。
