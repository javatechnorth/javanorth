---
layout: post
title:  一篇搞定@SpringBootApplication注解所有面试题
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君

### 前言

前几天，指北君有一个朋友小王在面试过程中遇到了这样一个问题，Spring Boot 的最重要的注解是什么？

小王听到这个面试题觉得这还不简单，当然是 @SpringBootApplication 啊，立即回答啦。面试官立刻追问，那 @SpringBootApplication 为什么是最重要的。

小王听后回答说：@SpringBootApplication 注解能实现Spring 配置，组建扫描，启用Spring Boot 的自动配置功能。

面试官：那 @SpringBootApplication 注解是怎么实现Spring Boot的自动配置功能。

小王这时心想，这下上套了，还真没有看过源码。支支吾吾回答不上，只好回家等通知了。
<!--more-->
指北君得知这个情况之后，跟朋友一顿操作和解释，然后就有了今天的文章，我们将学习 Spring Boot 框架中最重要的注解之一，它改变了Java开发人员使用Spring编写Java应用程序的方式。

### 概述

在这篇文章中，我将解释 @SpringBootApplication 是什么意思，以及如何在一个简单的 Spring Boot 应用程序中使用它。我们在 Application 类中使用 @SpringBootApplication 注解来启用一些功能，如基于 Java 的 Spring 配置、组件扫描，特别是用于启用 Spring Boot 的自动配置功能。

如果你已经使用 Spring Boot 很久了，那么你可能知道我们需要在 Application 类或 Main 类中注解相当多的注解才能开始使用，比如说

+ @Configuration，启用基于Java的配置。
+ @ComponentScan 来启用组件扫描。
+ @EnableAutoConfiguration启用Spring Boot的自动配置功能。

但现在你只需用 @SpringBootApplication 来注解你的 Application 类，就可以做到这一切。

顺便说一下，这个注解从 Spring Boot 1.2开始就有了，这意味着如果你运行在较低版本的 Spring Boot 上，那么如果你需要这些功能，你仍然需要使用@Configuration、@CompnentScan 和 @EnableAutoConfiguration。

### @SpringBootApplication 示例

下面是一个简单的例子，说明如何使用 @SpringBootApplication 注解来编写 Spring Boot 应用程序。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

在 Spring Boot 应用程序中，Application 类有两个用途：配置和引导。首先，它是主要的 Spring 配置类，其次，它可以实现 Spring Boot 应用程序的自动配置。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    @AliasFor(annotation = EnableAutoConfiguration.class)
    Class<?>[] exclude() default {};
    
    @AliasFor(annotation = EnableAutoConfiguration.class)
    String[] excludeName() default {};
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
    String[] scanBasePackages() default {};
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
    Class<?>[] scanBasePackageClasses() default {};
    @AliasFor(annotation = ComponentScan.class, attribute = "nameGenerator")
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;
    @AliasFor(annotation = Configuration.class)
    boolean proxyBeanMethods() default true;
}
```

从 @SpringBootApplication 源码可以看出 @SpringBootApplication = @SpringBootConfiguration + @ComponentScan + @EnableAutoConfiguration 。

@SpringBootApplication注解是以下三个Spring注解的组合，只需一行代码就能提供所有三个注解的功能。

> @SpringBootConfiguration
> 
> 这个注解将一个类标记为基于Java配置的配置类。如果你喜欢基于Java的配置而不是基于XML的配置，这一点就特别重要。
> 
> @ComponentScan
> 
> 该注解使组件扫描成为可能，这样你创建的Web控制器类和其他组件将被自动发现，并在Spring应用上下文中注册为Bean。你编写的所有@Controller类将被该注解发现。
> 
> @EnableAutoConfiguration
> 
> 这个注解可以启用Spring Boot惊人的自动配置功能，它可以为你自动配置很多东西。

另外我们还可以关注一下几个方法

> Class<?>[] exclude() default {}:
> 
> 根据 class 来排除，排除特定的类加入 spring 容器，传入参数 value 类型是 class 类型。
> 
> String[] excludeName() default {}:
> 
> 根据 class name 来排除，排除特定的类加入 spring 容器，传入参数 value 类型是 class 的全类名字符串数组。
> 
> String[] scanBasePackages() default {}:
> 
> 指定扫描包，参数是包名的字符串数组。
> 
> Class<?>[] scanBasePackageClasses() default {}:
> 
> 扫描特定的包，参数类似是 Class 类型数组。

就拿 scanBasePackages 来举个例子：

```java
@SpringBootApplication(scanBasePackages  = {"com.javanorth.controller"})
```

如果是将不需要的 bean 排除在 spring 容器中，如何操作？看看官方的代码怎么用的：

![application](http://www.javanorth.cn/assets/images/2021/feng/application1.png)

### @SpringBootConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {
    @AliasFor(annotation = Configuration.class)
    boolean proxyBeanMethods() default true;
}
```

@SpringBootConfiguration 继承自 @Configuration，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到 Spring 容器中，并且实例名就是方法名。

### @ComponentScan

可以通过该注解指定扫描某些包下包含如下注解的均自动注册为 spring beans：@Component、@Service、 @Repository、 @Controller等等 。使用方式例如：

```java
@ComponentScan(basePackages = {"com.javanorth.controller"})
```

除了可以使用 @ComponentScan  注解来加载我们的 bean，还可以在 Application 类中使用 @Import  指定该类。 例如：

```java
@Import(JdbcRepositoriesRegistrar.class)
```

### @EnableAutoConfiguration

@EnableAutoConfiguration 的作用启动自动的配置，@EnableAutoConfiguration 注解的意思就是 Spring Boot 根据你添加的 jar 包来配置你项目的默认配置，比如根据 spring-boot-starter-web ，来判断你的项目是否需要添加了web mvc 和 tomcat，就会自动的帮你配置 web 项目中所需要的默认配置。简单点说就是它会根据定义在 classpath 下的类，自动的给你生成一些 Bean，并加载到 Spring 的 Context 中。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

可以看到 import 引入了 AutoConfigurationImportSelector 类。该类使用了 Spring Core 包的 SpringFactoriesLoader 类的 loadFactoryNames() 方法。AutoConfigurationImportSelector 类实现了 DeferredImportSelector 接口，并实现了 selectImports 方法，用来导出Configuration 类。

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
        ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
    protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        }
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        ...
        return new AutoConfigurationEntry(configurations, exclusions);
    }
    protected boolean isEnabled(AnnotationMetadata metadata) {
        if (getClass() == AutoConfigurationImportSelector.class) {
            return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
        }
        return true;
    }
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                + "are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
}
```

导出的类是通过 SpringFactoriesLoader.loadFactoryNames() 读取了 classpath 下面的 META-INF/spring.factories 文件。

```java
public final class SpringFactoriesLoader {
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
    public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        ClassLoader classLoaderToUse = classLoader;
        if (classLoaderToUse == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }
        String factoryTypeName = factoryType.getName();
        return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
    }
    private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
        Map<String, List<String>> result = cache.get(classLoader);
        if (result != null) {
            return result;
        }
        result = new HashMap<>();
        try {
            Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                UrlResource resource = new UrlResource(url);
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                for (Map.Entry<?, ?> entry : properties.entrySet()) {
                    String factoryTypeName = ((String) entry.getKey()).trim();
                    String[] factoryImplementationNames =
                            StringUtils.commaDelimitedListToStringArray((String) entry.getValue());
                    for (String factoryImplementationName : factoryImplementationNames) {
                        result.computeIfAbsent(factoryTypeName, key -> new ArrayList<>())
                                .add(factoryImplementationName.trim());
                    }
                }
            }
            // Replace all lists with unmodifiable lists containing unique elements
            result.replaceAll((factoryType, implementations) -> implementations.stream().distinct()
                    .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList)));
            cache.put(classLoader, result);
        }
        catch (IOException ex) {
            throw new IllegalArgumentException("Unable to load factories from location [" +
                    FACTORIES_RESOURCE_LOCATION + "]", ex);
        }
        return result;
    }
}
```

这里列出了一部分自动配置的内容：

```java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
```

如果你发现自动装配的 Bean 不是你想要的，你也可以 disable 它。比如说，我不想要自动装配 Database 的那些Bean：

```java
@EnableAutoConfiguration(exclude  = {DataSourceAutoConfiguration.class})
```

### @AutoConfigurationPackage

@EnableAutoConfiguration 又继承了@AutoConfigurationPackage，@AutoConfigurationPackage 会引导类（@SpringBootApplication标注的类）所在的包及下面所有子包里面的所有组件扫描到Spring容器。具体怎么实现的呢，我们来看代码，原来它 import 了 AutoConfigurationPackages.Registrar.class， 我们来看看它做了什么？

```java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};
    Class<?>[] basePackageClasses() default {};
}
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
    }
    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new PackageImports(metadata));
    }
}
```

看代码就很容易理解，把注解扫描进来的package 全部给注册到 spring bean中。这样 Spring Boot 的自动配置也就完成了。

### 总结

经过这样的一番折腾，相信大家已经对 @SpringBootApplication 注解，有了一定的了解。也知道了@SpringBootApplication 怎么实现的Spring Boot 的自动配置功能。你可以只写这一行代码来启用基于Java的配置、组件扫描，并启用Spring Boot的自动配置功能。它使你的代码更具可读性。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。
