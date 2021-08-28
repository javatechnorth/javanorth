---
layout: post
title: spring boot 面试题
tagline: by feng
categories: spring
tags: 
    - feng
---

### 1、spring boot 核心的两个配置文件

1、 bootstrap (.yml 或.properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；

2、 application (. yml 或者 . properties)：由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。

### 2、是否可以在Spring boot中更改嵌入式Tomcat服务器的端口?

是的，更改端口是可行的。可以使用application.properties文件更改端口。但需要提到“server.port”（即server.port=8081）。确保项目类路径中有application.properties；后续工作将由REST Spring框架接手。如果提到server.port=0，那么它将自动分配任何可用的端口。

### 3、如何在 SpringBoot 启动的时候运行一些特定的代码？ 

可以实现接口 ApplicationRunner 或者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个 run 方法

### 5、bootstrap.yml和application.yml有什么区别?

1、 Spring Cloud 构建于 SpringBoot 之上，在 SpringBoot 中有两种上下文，一种是 bootstrap，另外一种是 application。

2、 application 配置文件这个容易理解，主要用于 SpringBoot 项目的`自动化配置`。

3、 bootstrap 是应用程序的父上下文，也就是说 `bootstrap 加载优先于 applicaton`。

4、 bootstrap 主要用于从`额外的资源来加载配置信息`，还可以在本地外部配置文件中解密属性。

5、 这两个上下文`共用一个环境`，它是任何Spring应用程序的外部属性的来源。

6、 bootstrap 里面的属性会`优先加载`，它们默认也不能被本地相同配置覆盖。

7、 boostrap 由父 ApplicationContext 加载，`比 applicaton 优先加载`

8、 boostrap 里面的属性`不能被覆盖`

### 7、SpringBoot 的核心注解是哪个？它主要由哪几个注解组成的？

启动类上面的注解是@SpringBootApplication，它也是 SpringBoot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能：

@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。

### 9、SpringBoot 的自动配置是如何实现的？

SpringBoot 项目的启动注解是：@SpringBootApplication，其实它就是由下面三个注解组成的：

1、 @Configuration

2、 @ComponentScan

3、 @EnableAutoConfiguration

其中 @EnableAutoConfiguration 是实现自动配置的入口，该注解又通过 @Import 注解导入了AutoConfigurationImportSelector，在该类中加载 META-INF/spring.factories 的配置信息。然后筛选出以 EnableAutoConfiguration 为 key 的数据，加载到 IOC 容器中，实现自动配置功能！

### 10、SpringBoot自动配置的原理是什么？

SpringBoot启动的时候通过@EnableAutoConfiguration注解找到META-INF/spring.factories配置文件中所有的自动配置类，并对其进行加载，而这些自动配置类的类名都是以AutoConfiguration结尾来命名的，它实际上就是一个javaConfig形式的Spring容器配置类，它们都有一个@EnableConfigurationPerperties的注解，通过这个注解启动XXXProperties命名的类去加载全局配置中的属性，如server.port,而XXXProperties通过@ConfigurationProperties注解将全局配置文件中的属性与自己的属性进行绑定。

### 11、SpringBoot 配置加载顺序?

1、 properties文件 2、YAML文件 3、系统环境变量 4、命令行参数

### 12、spring boot初始化环境变量流程?

1、 调用`prepareEnvironment`方法去设置环境变量

2、 接下来有三个方法`getOrCreateEnvironment`，`configureEnvironment`，`environmentPrepared`

3、 `getOrCreateEnvironment`去初始化系统环境变量

4、 `configureEnvironment`去初始化命令行参数

5、 `environmentPrepared`当广播到来的时候调用`onApplicationEnvironmentPreparedEvent`方法去使用`postProcessEnvironment`方法`load yml`和`properties变量`

### 13、运行 SpringBoot 有哪几种方式？

1、 打包用命令或者者放到容器中运行

2、 用 Maven/ Gradle 插件运行

3、 直接执行 main 方法运行

### 14、SpringBoot 中如何解决跨域问题 ?

跨域可以在前端通过 JSONP 来解决，但是 JSONP 只可以发送 GET 请求，无法发送其他类型的请求，在 RESTful 风格的应用中，就显得非常鸡肋，因此我们推荐在后端通过 （CORS，Cross-origin resource sharing） 来解决跨域问题。这种解决方案并非 SpringBoot 特有的，在传统的 SSM 框架中，就可以通过 CORS 来解决跨域问题，只不过之前我们是在 XML 文件中配置 CORS ，现在可以通过实现WebMvcConfigurer接口然后重写addCorsMappings方法解决跨域问题。

```java 
@Configuration 
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }
}
```

### 15、SpringBoot如何配置log4j？

在引用log4j之前，需要先排除项目创建时候带的日志，因为那个是Logback，然后再引入log4j的依赖，引入依赖之后，去src/main/resources目录下的log4j-spring.properties配置文件，就可以开始对应用的日志进行配置使用。


### 16、SpringBoot运行项目的几种方式？

打包用命令或者放到容器中运行

1、 打成jar包，使用java -jar xxx.jar运行

2、 打成war包，放到tomcat里面运行

直接用maven插件运行   maven spring-boot：run

直接执行main方法运行


### 17、什么是JavaConfig？

Spring JavaConfig是Spring社区的产品，它提供了配置Spring IoC容器的纯Java方法。因此它有助于避免使用XML配置。使用JavaConfig的优点在于：

面向对象的配置。由于配置被定义为JavaConfig中的类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean方法等。

减少或消除XML配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在XML和Java之间来回切换。

JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。

从技术角度来讲，只使用JavaConfig配置类来配置容器是可行的，但实际上很多人认为将JavaConfig与XML混合匹配是理想的。

类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找

### 18、SpringBoot集成mybatis的过程

添加mybatis的starter maven依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```

在mybatis的接口中 添加@Mapper注解

在application.yml配置数据源信息


### 19、SpringBoot、Spring MVC 和 Spring 有什么区别？

1、 Spring

Spring最重要的特征是依赖注入。所有 SpringModules 不是依赖注入就是 IOC 控制反转。

当我们恰当的使用 DI 或者是 IOC 的时候，我们可以开发松耦合应用。松耦合应用的单元测试可以很容易的进行。

2、 Spring MVC

Spring MVC 提供了一种分离式的方法来开发 Web 应用。通过运用像 DispatcherServelet，MoudlAndView 和 ViewResolver 等一些简单的概念，开发 Web 应用将会变的非常简单。

3、 SpringBoot

Spring 和 SpringMVC 的问题在于需要配置大量的参数。

SpringBoot 通过一个自动配置和启动的项来目解决这个问题。为了更快的构建产品就绪应用程序，SpringBoot 提供了一些非功能性特征。

