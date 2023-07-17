---
layout: post
title:  2023-07-07 proxyBeanMethods为false时的配置类
tagline: by 沉浮
categories: 
tags: 沉浮
---


<!--more-->
## Configuration

最近看源码时，经常看了下@Configuration(proxyBeanMethods = false)这样的配置，但从命名上看应该是与代理有关的，于是抽个时间了解了下

### proxyBeanMethods

首先这个是@Configuration注解中的一个参数，我们都知道，@Configuration是Spring中的配置类，一般用来申明Bean，在默认情况下proxyBeanMethods为true

### 含义

从源码中可以看到对该参数的描述如下：

```text
Specify whether @Bean methods should get proxied in order to enforce bean lifecycle behavior, e.g. to return shared singleton bean instances even in case of direct @Bean method calls in user code. This feature requires method interception, implemented through a runtime-generated CGLIB subclass which comes with limitations such as the configuration class and its methods not being allowed to declare final.
The default is true, allowing for 'inter-bean references' via direct method calls within the configuration class as well as for external calls to this configuration's @Bean methods, e.g. from another configuration class. If this is not needed since each of this particular configuration's @Bean methods is self-contained and designed as a plain factory method for container use, switch this flag to false in order to avoid CGLIB subclass processing.
Turning off bean method interception effectively processes @Bean methods individually like when declared on non-@Configuration classes, a.k.a. "@Bean Lite Mode" (see @Bean's javadoc). It is therefore behaviorally equivalent to removing the @Configuration stereotype.
```

1. 该属性的作用是指定标注了@Bean的方法在执行生命周期的时候是否应该被代理。比如在代码中直接调用@Bean标注的方法时要返回共享的单例Bean实例（关闭代理之后不会返回共享的Bean实例）。
2. 该特性是通过运行时CGLIB子类 实现的方法拦截。该子类有一些限制，比如配置类和它的方法不允许声明为final类型。
3. 默认为true，即允许 “inter-bean references” 通过配置类内部的直接方法调用，也可以通过外部调用该配置类的@Bean标注的方法。
如果该配置类是一个特殊的配置类：每一个@Bean标注的方法 都是自包含的，并设计为供容器使用的普通工厂方法，可以设置该属性为false，以避免CGLIB子类处理。
4. 关闭Bean方法拦截可以高效的单独处理@Bean方法，就像声明在一个non-@Configuration类上一样，@Bean Lite Mode。这种行为类似于删除@Configuration原型。

当直接在Configuration中直接通过方法，实现实例件的属性依赖时，IDEA会有这样一段提示：

```text
Method annotated with @Bean is called directly in a @Configuration where proxyBeanMethods set to false. Set proxyBeanMethods to true or use dependency injection. 
```

### 示例

先通过下面的示例看下现象:

两个配置类，写法差不多，区别在与proxyBeanMethods的配置以及AnimalCage属性的注入方法。

```java
@Configuration(proxyBeanMethods = false)
public class GenericConfiguration {
    
    @Bean
    public DogCage dogCage(){
        return new DogCage();
    }

    @Bean
    public AnimalCage animalEden(){
        AnimalCage animalCage = new AnimalCage();
        animalCage.addCage(dogCage());
        return animalCage;
    }
}

@Configuration(proxyBeanMethods = true)
public class ProxyConfiguration {

    @Bean
    public DogCage dogCage(){
        return new DogCage();
    }

    @Bean
    public AnimalCage animalEden(@Autowired List<Cage> cages){
        return new AnimalCage(cages);
    }
}
```

先看下GenericConfiguration配置的情况:

```java
public class Tests {
    @Autowired
    private BeanFactory beanFactory;
    @Autowired
    private GenericConfiguration genericConfiguration;
    @Autowired
    private AnimalCage animalCage;
    @Autowired
    private DogCage dogCage;
    @Test
    public void runConfig() {
        log.info("configuration: {}", genericConfiguration); // 原始对象类型

        log.info("Configuration中的Bean: {}", genericConfiguration.dogCage() == genericConfiguration.dogCage()); // 两次结果不一样
        log.info("容器中的Bnea: {}", beanFactory.getBean(DogCage.class) == beanFactory.getBean(DogCage.class));// 从Spring容器中取值都是一样的
        animalCage.cages.forEach(cage -> {
            if (cage instanceof DogCage) {
                log.info("dogCage : {} ", cage == dogCage); // 和上面的对象不一致，非单例
            }
        });
    }
}
```

再看下ProxyConfiguration配置的情况:

```java
public class Tests {
    @Test
    public void runConfig() {
        log.info("configuration: {}", proxyConfiguration); // 1、CGLIB代理的对象

        log.info("Configuration中的Bean: {}", proxyConfiguration.dogCage() == proxyConfiguration.dogCage()); // 2、两次结果相同
        log.info("容器中的Bnea: {}", beanFactory.getBean(DogCage.class) == beanFactory.getBean(DogCage.class));// 3、从Spring容器中取值都是一样的
        animalCage.cages.forEach(cage -> {
            if (cage instanceof DogCage) {
                log.info("dogCage : {} ", cage == dogCage); // 和上面的对象不一致，非单例
            }
        });
    }
}
```

会得到这样的现象：

1. proxyBeanMethods = true时，从Spring容器中取出的Configuration是一个Cglib代理配置，否则是一个原始类型配置
2. proxyBeanMethods = true时，多次调用Bean方法，每次都是一个新对象，否则都是同一个对象
3. 从Spring容器中取出Bean，不管多少次，都是同一个对象，也就是单例的

### Lite Full Mode

看到上面的现象后，我们有必要了解下Spring配置中的Lite和Full两种模式

**lite模式包含：**

+ 被**@Component**修饰的类
+ 通过**@ComponentScan**扫描的类
+ 通过**@Import**导入的配置类
+ 通过**@ImportResource**导入的Spring配置文件
+ 没有任何Spring相关注解，类里面有@Bean修饰的方法
+ 被@Configuration修饰，但proxyBeanMethods = false

**full模式包含：**

+ 被@Configuration修饰，且属性proxyBeanMethods = true（默认）

---

**full模式使用特性：**

+ full模式下的配置类会被Cglib代理生成代理类取代原始类型保存到在容器中
+ full模式下的@Bean方法不能是private和final，因为方法会被重写
+ 单例scope下不同@Bean方法可以互相引用，实现单实例的语义

**lite模式使用特性：**

+ lite模式下的配置类不生成代理，原始类型进入容器
+ lite模式下的@Bean方法可以是private和final
+ 单例scope下不同@Bean方法引用时无法做到单例，通过@Bean方法生成的对象都是新的实例

### 结束语

@Configuration(proxyBeanMethods = false)的配置其实是Lite模式，这种模式下，配置类不会生成代理类，速度会更快，但是要注意，在配置类中的@Bean方法，不能用来实现单例级别的依赖。
