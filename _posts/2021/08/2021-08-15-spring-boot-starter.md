---
layout: post
title: 手把手教大家撸一个 Spring Boot Starter！
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是**指北君**。

今天**指北君**要给大家带**实习生**学习怎么创建一个`spring boot starter`。 大家都知道 starter 在Spring Boot 中是一种非常重要的机制。

**指北君**：上次我跟你说了今天要讲 `spring boot starter`，你有没有先去了解一下啊？

**实习生**：这还用说嘛，肯定是回去看过了。

<!--more-->

**指北君**：那你讲讲什么是`spring boot starter`？

**实习生**：`spring boot starter` 相当于模块，它能将模块所需的依赖整合起来并对模块内的Bean 根据环境进行自动配置。 使用者只需要依赖响应功能的starter，无需做过多的配置和依赖，Spring Boot 就能自动扫描并自动加载相应的模块。

**指北君**：可以啊，这次是真的回去好好看了嘛？

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-starter1.png)

今天我就直接带你敲代码了。

说完，**指北君**打开了Java开发IDE 神器 Intellij IDEA。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-starter2.png)

**指北君**： spring boot 创建自定义starter 有一个惯例，模块名称会在最前面，比如 我们在使用的`mybatis-spring-boot-starter`，mybatis 在前面。 所以我们新建的项目名称就是 `hello-spring-boot-starter`.

**实习生**：`spring boot` 官方建议我们创建自定义 starter 的时候，创建两个模块呢？ 一个是 autoconfiguration 和一个 starter 模块。

**指北君**：你不要急嘛，项目还是要一个一个的建立的呀。现在立刻给你建立一个 `hello-spring-boot-starter-autoconfiguration` 项目。

现在我们要配置一下依赖关系，在 `hello-spring-boot-starter` 项目中的pom文件添加依赖。

```xml
<dependency>
  <groupId>cn.javanorth</groupId>
  <artifactId>helllo-spring-boot-starter-autoconfiguration</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</dependency>
```

**实习生**：然后呢？

**指北君**： 我们现在 autoconfigration 这个模块中 添加一个配置信息类。

```java
package cn.javanorth.helllospringbootstarterautoconfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
/**
 * @author 指北君
 * @公众号 Java技术指北
 */
@ConfigurationProperties("cn.javanorth")
public class HelloConfig {
    private String hello;
    private String name;
    public String getHello() {
        return hello;
    }
    public void setHello(String hello) {
        this.hello = hello;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

HelloConfig 类使用了 @ConfigurationProperties("cn.javanorth")， 这样该类的属性可以和配置文件中以 cn.javanorth 开头的配置项绑定好了。

接下来我们在定义一个Service类，提供一个sayHello的方法。

```java
package cn.javanorth.helllospringbootstarterautoconfiguration;
import org.springframework.beans.factory.annotation.Autowired;
/**
 * @author 指北君
 * @公众号 Java技术指北
 */
public class HelloService {
    @Autowired
    private HelloConfig helloConfig;
    public String sayHello(String userName) {
        return helloConfig.getHello() + " " + helloConfig.getName();
    }
}
```

我们现在还差一个 自动化配置类，就快完成了。

```java
package cn.javanorth.helllospringbootstarterautoconfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * @author 指北君
 * @公众号 Java技术指北
 */
@Configuration
@EnableConfigurationProperties(HelloConfig.class)
public class HelloAutoConfiguration {
    @ConditionalOnMissingBean(HelloService.class)
    @Bean
    public HelloService helloService() {
        HelloService helloService = new HelloService();
        return helloService;
    }
}
```

Java 代码就这样差不多了，接下来，就是做配置了。 我们需要创建 spring.factories 文件。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-starter3.png)

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  cn.javanorth.helllospringbootstarterautoconfiguration.HelloAutoConfiguration
```

到了这一步，我们的第一个starter算是结束了。

**实习生**：这就完了吗？那个hello-spring-boot-starter 项目不是啥都没做，不用写代码吗？

**指北君**：是啊，这个是spring boot 的一个约定嘛。 我们其实也可以合并在一起的，我看你说要这么分开，我就分开写了。

**实习生**：好吧，我们打包看看会不会出错。

![](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-starter4.png)

**指北君**：你看，我们挺顺利的。要是这个项目不顺利，我们可能就不用再写代码了。

**实习生**：有道理，这个实在是太简单了。

**指北君**：接下来我们测试一下，我们的项目对不对。 先创建一个测试项目。 在pom文件添加hello-spring-boot-starter依赖。

新建完测试项目，我们新建一个HelloController 用于测试是否能调用成功

```java
package cn.javanorth.hellospringbootstartertest;
import cn.javanorth.helllospringbootstarterautoconfiguration.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
/**
 * @author 指北君
 * @公众号 Java技术指北
 */
@RestController
public class HelloController {
    @Autowired
    private HelloService helloService;
    @GetMapping("hello")
    public String sayHello() {
        return helloService.sayHello();
    }
}
```

然后在 application.properties 里面添加配置信息

```java
cn.javanorth.hello=Hello World
cn.javanorth.name=javanorth
```

搞定，启动看一下结果，我们在bash中直接 curl 操作一下

嗯，已经正常调用了。

**指北君**：那我们今天就到这里吧，下次再跟你讲。

**实习生**：好的，你把这个demo的代码也发我一下，我在我自己机器上搞搞。

**指北君**：可以的，马上发你。

### 总结

今天学习了如何建立一个简单的 spring boot starter 项目。项目真的很基础，很容易学会的。可以好好看看源代码。

本文的所有示例源代码和完整的思维导图都已上传到了 Github：

> https://github.com/javatechnorth/java-north-sample

欢迎大家 Star 关注，后续会不断更新。