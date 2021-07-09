---
layout: post
title:  Spring Profiles
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君。

最近公司新来了一个实习生，挺上进的，天天追着我问问题。指北君开启了带实习生打怪升级之路。吶，今天问了一个关于 `Spring Profiles` 的问题。

**实习生**：指北君，你知道 `Spring Profiles` 吗？

**指北君**：我知道啊，怎么了？有什么问题吗？

**实习生**：那你可以跟我讲讲嘛，`Spring Profiles`到底是什么？有什么用？

**指北君**：好的，Profiles 是 Spring 框架的核心特性，它允许我们在不同的 profiles 条件下，对 `Spring bean` 有不同的配置实现。 比如说，我们在生产环节用 prod 标记，那我们的 `spring bean` 构造出来之后就使用了 prod 的配置项。所以我们只要在启动的时候，设定好我们想要的 profile ，那我们就能获得不同的结果。

**实习生**：那这个 Spring Profile 在项目里怎么使用的呢？

**指北君**：好，我找个项目给你看下。

话音刚落，指北君打开了 IDEA，一顿操作，找到了一个配置类。

```java
@Component
@Profile("dev")
public class DevDatasourceConfig 
```


**指北君**：看到了吗？ 我们项目里这个数据源配置类上面有个 `@Profile` 注解，里面写了 dev 。就是说这个配置类在 `profile=dev` 的时候，才能生效。

**实习生**：那这个挺简单的呀。

**指北君**：是的呀，使用起来很简单的，哦对了，这里有个小技巧。如果我们只是不想在特定环境下配置某个配置类的话，也有很简单的操作方式。

```java
@Component
@Profile("!dev")
public class DevDatasourceConfig 
```

你看出区别了吗？

**实习生**：就是 dev 之前加了一个 ！（叹号）。其他没有变化。

**指北君**：是的，就是这样，这也是一个常用的技巧。

**实习生**：代码里虽然这样配置好了，那项目启动怎么知道启用了哪一个配置项呢？

**指北君**：启用哪一个配置，这边有很多种方式可以实现。
1、我们在项目中的 application.properties 文件里直接进行配置即可

```java
spring.profiles.active=dev
```

2、通过硬编码实现 WebApplicationInitializer 接口，配置 ServletContext 来激活配置

```java
@Configuration
public class MyWebApplicationInitializer 
  implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
 
        servletContext.setInitParameter(
          "spring.profiles.active", "dev");
    }
}
```

3、直接在 ConfigurableEnvironment 中实现设置，使用 spring boot 的朋友应该都知道，系统所有的配置项都是来自于 ConfigurableEnvironment 。

```java
@Autowired
private ConfigurableEnvironment env;
...
env.setActiveProfiles("dev");
```

4、命令行JVM启动参数

```java
-Dspring.profiles.active=dev
```

5、通过 export 环境变量来实现

```java
export spring_profiles_active=dev
```

嗯， 学会了这几种方法，你已经可以在绝大部分场景下，游刃有余地使用 profile 了。

**实习生**：那我还有个问题，既然这样配置的话，那我项目里有多个profile， 万一我手抖，没有用你说的这几种配置好呢？ 那会怎么样？

**指北君**：这个你放心，spring boot 有兜底方法，你想啊，你在一些项目没有用到 profile 的时候，spring boot 是怎么启动的呢？spring boot 项目会自动使用默认配置

```java
spring.profiles.default=none
```

也就是说，spring boot 启动了，但是没有加载任何被@Profile 等标记的配置类。只会加载没有被标记的配置类。

**实习生**：懂了，我看我们项目里，有很多类似 `@Profile("db") @Profile("mysql")` 之类的，那像这种多个的怎么使用的？ 

**指北君**：是的，我们项目中，不同的模块都分开配置了， 我给你举个例子怎么使用的。

```java
-Dspring.profiles.active=dev,db,mysql
```

就像这样，直接逗号分割，拼接上去就行了。

**实习生**：那这样好像比较麻烦啊，万一搞漏掉了就给自己挖坑了。

**指北君**：对的，这就是要仔细了，以前一直要靠自己的。现在就不会有这个问题了，spring boot 团队也意识到这个问题了，在 Spring boot 2.4 开始已经支持分组了。

```java
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

我们平时启动的时候只要指定 `spring.profiles.active=production` 就行了， prodb 和 prodmq 同样就能成功配置进来。 `Spring Profiles` 的相关内容差不多都讲完了，你还有什么疑问吗？

**实习生**：现在基本没有了，等我遇到再来找你这大佬。

**指北君**：随时欢迎，现在是我在带你，不要给我丢脸就行。哈哈哈。

### 总结

本文讲述了 `Spring Profiles` 的配置使用，和如果在启动的时候，选择特定的 profile 。也讲述了一些使用过程中的一些小技巧，希望对你有帮助。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。
