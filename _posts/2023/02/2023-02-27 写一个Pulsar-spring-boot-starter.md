---
layout: post
title:  2023-02-27 写一个Pulsar-spring-boot-starter
tagline: by 沉浮
categories: 
tags: 沉浮
---

哈喽，大家好，我是指北君。  

之前写过关于Apache Pulsar的简单示例，用来了解如何使用Pulsar这个新生代的消息队列中间件，但是如果想要在项目中使用，还会欠缺很多，最明显的就是
集成复杂，如果你用过其他消息中间件，比如Kafka、RabbitMq，只需要简单的引入jar，就可以通过注解+配置快速集成到项目中。


<!--more-->
## 开始一个Pulsar Starter

既然已经了解了Apache Pulsar，又认识了spring-boot-starter，今天不妨来看下如何写一个pulsar-spring-boot-starter模块。

### 目标

写一个完整的类似kafka-spring-boot-starter（springboot项目已经集成到spring-boot-starter中），需要考虑到很多kafka的特性，
今天我们主要实现下面几个模板

+ 在项目中够通过引入jar依赖快速集成
+ 提供统一的配置入口
+ 能够快速发送消息
+ 能够基于注解实现消息的消费

## 定义结构

```
└── pulsar-starter
    ├── pulsar-spring-boot-starter
    ├── pulsar-spring-boot-autoconfigure
    ├── spring-pulsar
    ├── spring-pulsar-xx
    ├── spring-pulsar-sample
└── README.md
```
整个模块的结构如上其中**pulsar-starter**作为一个根模块，主要控制子模块依赖的其他jar的版本以及使用到的插件版本。类似于Spring-Bom，这样我们在后续升级
时，就可以解决各个第三方jar的可能存在版本冲突导致的问题。

+ pulsar-spring-boot-starter

该模块作为外部项目集成的直接引用jar，可以认为是pulsar-spring-boot-starter组件的入口，里面不需要写任何代码，只需要引入需要的依赖（也就是下面的子模块）即可
+ pulsar-spring-boot-autoconfigure

该模块主要定义了spring.factories以及AutoConfigure、Properties。也就是自动配置的核心（配置项+Bean配置）
+ spring-pulsar

该模块是核心模块，主要的实现都在这里
+ spring-pulsar-xx

扩展模块，可以对spring-pulsar做更细化的划分
+ spring-pulsar-sample

starter的使用示例项目

### 实现

上面我们说到实现目标，现在看下各个模块应该包含什么内容，以及怎么实现我们的目标

* **入口pulsar-spring-boot-starter**

上面说到starter主要是引入整个模块基础的依赖即可，里面不用写代码。

```xml
<dependencies>
    <dependency>
        <groupId>com.sucl</groupId>
        <artifactId>spring-pulsar</artifactId>
        <version>${project.version}</version>
    </dependency>

    <dependency>
        <groupId>com.sucl</groupId>
        <artifactId>pulsar-spring-boot-autoconfigure</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

* **pulsar-spring-boot-autoconfigure**

1. 添加spring-boot基础的配置
```xml
<dependencies>
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot</artifactId>
     </dependency>

     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-logging</artifactId>
     </dependency>

     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-configuration-processor</artifactId>
         <optional>true</optional>
     </dependency>
</dependencies>
```

2. 定义自动配置类*PulsarAutoConfiguration*：
   + 引入**Properties**，基于*EnableConfigurationProperties*与*spring-boot-configuration-processor*解析Properties
     生成对应*spring-configuration-metadata.json*文件，这样编写application.yml配置时就可以自动提示配置项的属性和值了。

   + 构建一些必须的Bean，如PulsarClient、ConsumerFactory、ConsumerFactory等

   + Import配置PulsarAnnotationDrivenConfiguration，这个主要是一些额外的配置，用来支持后面的功能

```java

@Configuration
@EnableConfigurationProperties({PulsarProperties.class})
@Import({PulsarAnnotationDrivenConfiguration.class})
public class PulsarAutoConfiguration {

    private final PulsarProperties properties;

    public PulsarAutoConfiguration(PulsarProperties properties) {
        this.properties = properties;
    }

    @Bean(destroyMethod = "close")
    public PulsarClient pulsarClient() {
        ClientBuilder clientBuilder = new ClientBuilderImpl(properties);
        return clientBuilder.build();
    }

    @Bean
    @ConditionalOnMissingBean(ConsumerFactory.class)
    public ConsumerFactory pulsarConsumerFactory() {
        return new DefaultPulsarConsumerFactory(pulsarClient(), properties.getConsumer().buildProperties());
    }

    @Bean
    @ConditionalOnMissingBean(ProducerFactory.class)
    public ProducerFactory pulsarProducerFactory() {
        return new DefaultPulsarProducerFactory(pulsarClient(), properties.getProducer().buildProperties());
    }

}
```

3. 配置spring.factory

在目录*src/main/resources/META-INF*下创建**spring.factories**，内容如下：
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.sucl.pulsar.autoconfigure.PulsarAutoConfiguration
```

* **spring-pulsar**

1. 添加pulsar-client相关的依赖
```xml
 <dependencies>
     <dependency>
         <groupId>org.apache.pulsar</groupId>
         <artifactId>pulsar-client</artifactId>
     </dependency>

     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-autoconfigure</artifactId>
     </dependency>

     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-messaging</artifactId>
     </dependency>
</dependencies>
```

2. 定义EnablePulsar，之前说到过，@Enable注解主要是配合AutoConfigure来做功能加强，没有了自动配置，我们依然可以使用这些模块的功能。
这里做了一件事，向Spring容器注册了两个Bean
+ _PulsarListenerAnnotationBeanProcessor_ 在Spring Bean生命周期中解析注解自定义注解PulsarListener、PulsarHandler，
+ _PulsarListenerEndpointRegistry_ 用来构建Consumer执行环境以及对TOPIC的监听、触发消费回调等等，可以说是最核心的Bean

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import({PulsarListenerConfigurationSelector.class})
public @interface EnablePulsar {

}
```

3. 定义注解，参考RabbitMq，主要针对需要关注的类与方法，分别对应注解@PulsarListener、@PulsarHandler，通过这两个注解配合可以让我们监听到关注的TOPIC，
当有消息产生时，触发对应的方法进行消费。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface PulsarListener {

    /**
     *
     * @return TOPIC 支持SPEL
     */
    String[] topics() default {};

    /**
     *
     * @return TAGS 支持SPEL
     */
    String[] tags() default {};
}

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface PulsarHandler {

}
```
4. 注解@PulsarListener的处理流程比较复杂，这里用一张图描述，或者可以通过下面github的源代码查看具体实现

![flow](/assets/images/2023/sucls/02_27/flow.png)


* **spring-pulsar-sample**

按照下面的流程，你会发现通过简单的几行代码就能够实现消息的生产与消费，并集成到项目中去。

1. 简单写一个SpringBoot项目，并添加pulsar-spring-boot-starter
```xml
    <dependencies>
    <dependency>
        <groupId>com.sucl</groupId>
        <artifactId>pulsar-spring-boot-starter</artifactId>
        <version>${project.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
1. 添加配置
```yaml
cycads:
  pulsar:
    service-url: pulsar://localhost:6650
  listener-topics: TOPIC_TEST
```
1. 编写对应消费代码
```java
@Slf4j
@Component
@PulsarListener(topics = "#{'${cycads.listener-topics}'.split(',')}")
public class PulsarDemoListener {

    @PulsarHandler
    public void onConsumer(Message message){
        log.info(">>> 接收到消息：{}", message.getPayload());
    }

}
```

4. 向Pulsar Broker发送消息进行测试

```java
@Slf4j
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {ContextConfig.class})
@Import({PulsarAutoConfiguration.class})
public class ProducerTests {

    @Autowired
    private ProducerFactory producerFactory;

    @Test
    public void sendMessage() {
        Producer producer = producerFactory.createProducer("TOPIC_TEST");
        MessageId messageId = producer.send("this is a test message");
        log.info(">>>>>>> 消息发送完成：{}", messageId);
    }

    @Configuration
    @PropertySource(value = "classpath:application-test.properties")
    static class ContextConfig {
        //
    }
}
```
5. 控制台可以看到这样的结果
```
2023-02-26 19:57:15.572  INFO 26520 --- [pulsar-01] c.s.p.s.listener.PulsarDemoListener : >>> 接收到消息：GenericMessage [payload=this is a test message, headers={id=f861488c-2afb-b2e7-21a1-f15e9759eec5, timestamp=1677412635571}]
```

### 知识点

+ Pulsar Client

基于pulsar-client提供的ConfigurationData扩展Properties；
了解Pulsar Client如何连接Broker并进行消息消费，包括同步消费、异步消费等等

+ spring.factories

实现starter自动配置的关键，基于SPI完成配置的自动加载

+ Spring Bean生命周期

通过Bean生命周期相关扩展实现注解的解析与容器的启动，比如BeanPostProcessor, BeanFactoryAware, SmartInitializingSingleton, InitializingBean, DisposableBean等

+ Spring Messaging

基于回调与MethodHandler实现消息体的封装、参数解析以及方法调用；

### 源码示例

> [https://github.com/sucls/pulsar-starter.git](https://github.com/sucls/pulsar-starter.git)

### 结束语
    
如果你看过spring-kafka的源代码，那么你会发现所有代码基本都是仿造其实现。一方面能够阅读kafka client在spring具体如何实现；同时通过编写自己的spring starter模块，学习
整个starter的实现过程。

