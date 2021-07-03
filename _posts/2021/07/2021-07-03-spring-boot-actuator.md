---
layout: post
title:  Spring Boot Actuator
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君

不知道大家在写 Spring Boot 项目的过程中，使用过 Spring Boot Actuator 吗？ 知道 Spring Boot Actuator 是什么 ？ 干什么的吗？今天就要来给大家介绍一下 Spring Boot Actuator，学习如何在 Spring Boot 2.x 中使用、配置和扩展这个监控工具。Spring Boot 1.x 的使用就不再这边介绍了。相信大家平时使用的框架基本上都要升级到 2.x了吧。

<!--more-->

### 什么是 Actuator ？

从本质上讲，Spring Boot Actuator 为我们的应用程序带来了生产就绪的功能。监控我们的应用程序，收集指标，了解流量，或者是数据库的状态。有了它，我们就可以很简单的监控应用程序的各种指标数据。

Spring Boot Actuator 使用 http 或者 JMX 的方式来公开运行中的应用程序的操作信息--健康、指标、信息、转储、环境等，我们能够方便的与它互动。只要添加 
Spring Boot Actuator 依赖到 classpath 中，就有几个指标路径可供我们开箱使用。与大多数 Spring boot 模块一样，我们可以很容易地以多种方式配置或扩展它。

### 快速入门
要想启用 Spring Boot Actuator，我们只需在软件包管理器中添加 spring-boot-starter-actuator 依赖项。在 pom.xml 文件添加如下代码即可：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

是吧，很简单就添加好了。有的小朋友可能会又疑问了，为什么没有添加 version 节点呢？那是因为 Spring Boot 的parent pom 会制定版本号，所以这里无需再次指定了。

### Spring Boot 2.x Actuator

在 2.x 版本中，Spring Boot Actuator 保持了1.x 的基本操作，但简化了它的模型，扩展了它的能力，并加入了更好的默认值。这个版本变得与技术无关。它还简化了其安全模型，将其与应用程序模型合并。最新版本现在支持CRUD模型，而不是旧的读写模型。

#### 技术支持

在 2.x 中，Spring Boot Actuator 将其模型定义为可插拔和可扩展的，而不依赖于MVC。通过这个新的模型，我们能够利用MVC以及WebFlux作为底层Web技术的优势。此外，可以通过实现正确的适配器来添加即将到来的技术。不过 JMX 仍然被支持，无需任何额外的代码就可以暴露路径。

#### 重要变化

与以前的版本不同，Spring Boot Actuator的大多数路径都是禁用的。默认情况下，只有 /health 和 /info 两个可用。如果我们想启用所有的路径，可以设置 management.endpoints.web.exposure.include=* 来实现。Actuator 现在与常规的 App 安全规则共享安全配置，所以安全模型被大大简化。要调整 Actuator 的安全规则，我们可以只添加一个 /actuator/** 的条目。

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .pathMatchers("/actuator/**").permitAll()
      .anyExchange().authenticated()
      .and().build();
}
```

我们可以在全新的Actuator官方文档中找到进一步的细节。另外，在默认情况下，所有的Actuator路径现在都被放在/actuator路径下。和以前的版本一样，我们可以使用新的属性management.endpoints.web.base-path来调整这个路径。

#### 预定义的路径

让我们看看一些可用的路径，其中大部分在1.x中已经可用。另外，有些路径被添加了，有些被删除了，有些被重组了。

+ /auditevents 列出了安全审计相关的事件，如用户登录/注销。此外，我们还可以通过本金或类型以及其他字段进行过滤。
+ /beans 返回我们BeanFactory中所有可用的bean。与/auditevents不同，它不支持过滤。
+ /conditions，以前被称为/autoconfig，围绕自动配置建立一个条件报告。
+ /configprops 允许我们获取所有@ConfigurationProperties豆。
+ /env 返回当前的环境属性。此外，我们还可以检索单个属性。
+ /flyway 提供了关于我们Flyway数据库迁移的细节。
+ /health 总结了我们应用程序的健康状态。
+ /heapdump 构建并返回我们应用程序使用的 JVM 的堆转储。
+ /info 返回一般信息。它可能是自定义数据、构建信息或关于最新提交的细节。
+ /liquibase 的行为类似于/flyway，但针对Liquibase。
+ /logfile 返回普通的应用程序日志。
+ /loggers 使我们能够查询和修改我们应用程序的日志级别。
+ /metrics 详细说明我们应用程序的指标。这可能包括通用指标和自定义指标。
+ /prometheus 返回与前面一样的指标，但格式化为与Prometheus服务器一起工作。
+ /scheduledtasks 提供了关于我们应用程序中每个计划任务的细节。
+ /sessions 列出HTTP会话，因为我们使用的是Spring Session。
+ /shutdown 执行应用程序的优雅关闭。
+ /threaddump 转储底层JVM的线程信息。

#### 执行器路径的超媒体

Spring Boot 增加了一个所有路径的集合入口，可以返回所有可用的执行器路径的链接。这将有助于发现执行器路径及其相应的URL。默认情况下，这个集合入口可以通过/actuator路径访问。因此，如果我们向这个URL发送一个GET请求，它将返回各种路径的执行器链接。

```json
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    "features-arg0": {
      "href": "http://localhost:8080/actuator/features/{arg0}",
      "templated": true
    },
    "features": {
      "href": "http://localhost:8080/actuator/features",
      "templated": false
    },
    "beans": {
      "href": "http://localhost:8080/actuator/beans",
      "templated": false
    },
    "caches-cache": {
      "href": "http://localhost:8080/actuator/caches/{cache}",
      "templated": true
    },
    // ...
}
```

如上所示，/actuator路径在_links字段下报告所有可用的执行器路径。如果我们配置了一个自定义的管理基础路径，那么我们应该能使用该集合入口发现URL。例如，如果我们将management.endpoints.web.base-path设置为/mgmt，那么我们应该向/mgmt路径发送请求，以查看链接列表。但是如果当管理集合入口被设置为/时，集合入口被禁用，以防止与其他映射发生冲突的可能性。

#### 健康指示器

就像以前的版本一样，我们可以很容易地添加自定义指标。与其他API相反，用于创建自定义健康路径的抽象概念保持不变。然而，一个新的接口，即ReactiveHealthIndicator，已经被添加到实现反应式健康检查。让我们来看看一个简单的自定义反应式健康检查。

```java
@Component
public class DownstreamServiceHealthIndicator implements ReactiveHealthIndicator {
    @Override
    public Mono<Health> health() {
        return checkDownstreamServiceHealth().onErrorResume(
          ex -> Mono.just(new Health.Builder().down(ex).build())
        );
    }
    private Mono<Health> checkDownstreamServiceHealth() {
        return Mono.just(new Health.Builder().up().build());
    }
}
``` 

健康指标的一个方便的特点是，我们可以把它们作为一个层次结构的一部分进行汇总。因此，按照前面的例子，我们可以把所有的下游服务归入一个下游服务类别。只要每个嵌套的服务都是可以到达的，这个类别就会是健康的。

#### 健康组

从Spring Boot 2.2开始，我们可以将健康指标组织成组，并对所有组员应用相同的配置。例如，我们可以通过在application.properties中添加以下内容来创建一个名为custom的健康组。

```java
management.endpoint.health.group.custom.include=diskSpace,ping
```

这样，自定义组包含diskSpace和ping健康指标。现在，如果我们调用/actuator/health路径，它将在JSON响应中告诉我们关于新的健康组。

```json
{"status":"UP","groups":["custom"]}
```

通过健康组，我们可以看到一些健康指标的汇总结果。在这种情况下，如果我们向/actuator/health/custom发送一个请求，

```json
{"status": "UP"}
```

我们可以通过application.properties来配置该组，以显示更多细节。

```java
management.endpoint.health.group.custom.show-components=always
management.endpoint.health.group.custom.show-details=always
```

现在，如果我们向/actuator/health/custom发送同样的请求，我们会看到更多的细节。

```json
{
  "status": "UP",
  "components": {
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 499963170816,
        "free": 91300069376,
        "threshold": 10485760
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

也可以只为授权用户显示这些细节。

```java
management.endpoint.health.group.custom.show-components=when_authorized
management.endpoint.health.group.custom.show-details=when_authorized
```

我们还可以有一个自定义的状态映射。例如，它可以不返回HTTP 200 OK响应，而是返回207状态代码。

```java
management.endpoint.health.group.custom.status.http-mapping.up=207
```

在这里，我们要告诉Spring Boot，如果自定义组的状态是UP，就返回207的HTTP状态代码。

#### Spring Boot 2中的度量

在Spring Boot 2.0中，内部指标被Micrometer支持所取代，因此我们可以预期会有一些变化。如果我们的应用程序正在使用GaugeService或CounterService等度量衡服务，它们将不再可用。

相反，我们要与Micrometer直接互动。在Spring Boot 2.0中，我们会得到一个自动配置的MeterRegistry类型的bean。此外，Micrometer现在是Actuator依赖的一部分，所以只要Actuator的依赖在classpath中，我们就应该可以使用了。此外，我们将从/metrics端点得到一个全新的响应。

```json
{
  "names": [
    "jvm.gc.pause",
    "jvm.buffer.memory.used",
    "jvm.memory.used",
    "jvm.buffer.count",
    // ...
  ]
}
```

正如我们所看到的，没有像我们在1.x中得到的实际度量。为了得到一个特定指标的实际值，我们现在可以导航到所需的指标，例如，/actuator/metrics/jvm.gc.pause，然后得到一个详细的响应。

```json
{
  "name": "jvm.gc.pause",
  "measurements": [
    {
      "statistic": "Count",
      "value": 3.0
    },
    {
      "statistic": "TotalTime",
      "value": 7.9E7
    },
    {
      "statistic": "Max",
      "value": 7.9E7
    }
  ],
  "availableTags": [
    {
      "tag": "cause",
      "values": [
        "Metadata GC Threshold",
        "Allocation Failure"
      ]
    },
    {
      "tag": "action",
      "values": [
        "end of minor GC",
        "end of major GC"
      ]
    }
  ]
}
```

现在的度量标准要彻底得多，不仅包括不同的值，还包括一些相关的元数据。

#### 创建一个自定义路径

正如我们之前指出的，我们可以创建自定义路径。不过，Spring Boot 2重新设计了实现的方式，以支持新的技术无关的范式。
让我们创建一个Actuator路径，在我们的应用程序中查询、启用和禁用功能标志。

```java
@Component
@Endpoint(id = "features")
public class FeaturesEndpoint {
    private Map<String, Feature> features = new ConcurrentHashMap<>();
    @ReadOperation
    public Map<String, Feature> features() {
        return features;
    }
    @ReadOperation
    public Feature feature(@Selector String name) {
        return features.get(name);
    }
    @WriteOperation
    public void configureFeature(@Selector String name, Feature feature) {
        features.put(name, feature);
    }
    @DeleteOperation
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }
    public static class Feature {
        private Boolean enabled;
        // [...] getters and setters 
    }
}
```

为了获得路径，我们需要一个Bean。在我们的例子中，我们使用@Component来做这个。同时，我们需要用@Endpoint来装饰这个Bean。
我们的端点的路径是由@Endpoint的id参数决定的。在我们的例子中，它将把请求路由到/actuator/features。
一旦准备就绪，我们就可以开始使用定义操作了。

+ @ReadOperation。它将映射到HTTP GET。
+ @WriteOperation。它将映射到HTTP POST。
+ @DeleteOperation。它将映射到HTTP DELETE。

当我们在应用程序中使用前一个端点运行应用程序时，Spring Boot将注册它。验证这一点的一个快速方法是检查日志。

#### 扩展现有的端点

想象一下，如果我们想确保应用程序的生产实例永远不是SNAPSHOT版本。我们决定通过改变返回该信息的 Actuator 端点的HTTP状态代码，即/info来做到这一点。如果我们的应用程序碰巧是SNAPSHOT，我们会得到一个不同的HTTP状态代码。

我们可以使用 @EndpointExtension 注解，或其更具体的特殊化@EndpointWebExtension或@EndpointJmxExtension，轻松地扩展预定义端点的行为。

```java
@Component
@EndpointWebExtension(endpoint = InfoEndpoint.class)
public class InfoWebEndpointExtension {
    private InfoEndpoint delegate;
    @ReadOperation
    public WebEndpointResponse<Map> info() {
        Map<String, Object> info = this.delegate.info();
        Integer status = getStatus(info);
        return new WebEndpointResponse<>(info, status);
    }
    private Integer getStatus(Map<String, Object> info) {
        return 200;
    }
}
```

#### 启用所有端点
为了能够使用 HTTP 访问 Actuator 的端点，我们需要启用和公开它们。默认情况下，除了/shutdown，所有的端点都是启用的。默认情况下，只有/health和/info这两个端点是公开的。我们需要添加以下配置来公开所有端点。

```java
management.endpoints.web.exposure.include=*
```

要明确地启用一个特定的端点（例如，/shutdown）

```java
management.endpoint.shutdown.enabled=true
```

要公开所有已启用的端点，除了一个（例如，/loggers）

```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=loggers
```

### 总结

在这篇文章中，我们谈到了Spring Boot Actuator。我们首先解释了Actuator的含义以及它为我们做了什么。接下来，我们重点讨论了当前Spring Boot 2.x版本的Actuator，讨论了如何使用它、调整它和扩展它。我们还谈到了在这个新的迭代中我们可以发现的重要的安全变化。我们讨论了一些流行的端点，以及它们是如何变化的。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。