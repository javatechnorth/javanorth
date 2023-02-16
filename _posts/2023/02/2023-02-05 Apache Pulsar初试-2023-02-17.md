---
layout: post
title:  2023-02-05 Apache Pulsar初试
tagline: by 沉浮
categories: 
tags: 沉浮
---

哈喽，大家好，我是指北君。  

最近项目中准备使用消息中间件*Apache Pulsar*，借着机会先做个简单了解。

<!--more-->
## Apache Pulsar

Apache Pulsar是Apache软件基金会顶级项目，是下一代**云原生分布式消息流**平台。

Pulsar 作为下一代云原生分布式消息流平台，支持*多租户、持久化存储、多机房跨区域数据复制*，具有强一致性、高吞吐以及低延时的高可扩展流数据存储特性，
内置诸多其他系统商业版本才有的特性，是云原生时代解决实时消息流数据传输、存储和计算的最佳解决方案。

![message](/assets/images/2023/sucls/02_05/message.png)

### Pulsar简介  

+ 系统架构

![pulsar-system-architecture](/assets/images/2023/sucls/02_05/pulsar-system-architecture.png)

+ 功能特色
  - 支持多租户
  
  租户和命名空间（namespace）是 Pulsar 支持多租户的两个核心概念。在租户级别，Pulsar 为特定的租户预留合适的存储空间、应用授权与认证机制。
  在命名空间级别，Pulsar 有一系列的配置策略（policy），包括存储配额、流控、消息过期策略和命名空间之间的隔离策略。

  - 灵活的消息系统

  Pulsar 做了队列模型和流模型的统一，在 Topic 级别只需保存一份数据，同一份数据可多次消费。以流式、队列等方式计算不同的订阅模型大大提升了灵活度。

  - 云原生架构
  
  Pulsar 使用计算与存储分离的云原生架构，数据从 Broker 搬离，存在共享存储内部。上层是无状态 Broker，复制消息分发和服务；下层是持久化的存储层 Bookie 集群。
  Pulsar 存储是分片的，这种构架可以避免扩容时受限制，实现数据的独立扩展和快速恢复。

  - 支持跨地域复制

  Pulsar 原生支持跨地域复制，因此 Pulsar 可以跨不同地理位置的数据中心复制数据。当数据中心中断或网络分区时，在多个数据中心存有消息副本尤为重要，提高可用性。

  - Pulsar Functions

  Pulsar Functions 是基于 Pulsar 的轻量级流处理方式。Pulsar Functions 直接部署在 broker 节点上（或作为 Kubernetes 集群中的容器）。
  通过 Pulsar Functions，Pulsar 可以直接解决许多流处理任务，简化操作。

+ 支持客户端
  - Java 客户端
  - C++ 客户端
  - .Net/C# 客户端
  - Go 客户端
  - NodeJS 客户端
  - Ruby 客户端

### Pulsar安装与部署

目前Pulsar不支持Window，下面通过Docker进行安装，可以参考官方，https://pulsar.apache.org/docs/next/getting-started-docker/
同时可以安装Pulsar Manager，具体操作可以参考官方文档 https://pulsar.apache.org/docs/next/administration-pulsar-manager/

*其中Pulsar Manager 是一个网页式可视化管理与监测工具，支持多环境下的动态配置。可用于管理和监测租户、命名空间、topic、订阅、broker、集群等。*

1. window环境使用docker可以使用Docker Desktop，和linux一样可以通过docker命令管理镜像、部署容器等操作。
> 打开并启动Docker Desktop后，打开终端，执行 _>docker search pulsar 可以查询到pulsar相关的镜像

![pulsar-docer](/assets/images/2023/sucls/02_05/pulsar-docer.png)

2. 镜像下载
> 这里我们选择分别下载红框的两个镜像，执行命令 
> _> docker pull apachepulsar/pulsar
> _> docker pull apachepulsar/pulsar-manager

3. 启动
+ 启动Pulsar
```shell
docker run -it -p 6650:6650 -p 8080:8080 \
      --mount source=pulsardata,target=/pulsar/data \
      --mount source=pulsarconf,target=/pulsar/conf \
      apachepulsar/pulsar bin/pulsar standalone
```
 

+ 启动Pulsar Manager
```shell
docker run --name pulsar-manager -dit \
      -p 9527:9527 -p 7750:7750 \
      -e SPRING_CONFIGURATION_FILE=/pulsar-manager/pulsar-manager/application.properties \
      apachepulsar/pulsar-manager
```
添加用户：
```shell
for /f "tokens=1" %A in ('curl http://localhost:7750/pulsar-manager/csrf-token') do set CSRF_TOKEN=%A
curl -X PUT "X-XSRF-TOKEN: %CSRF_TOKEN%"   -H "Cookie: XSRF-TOKEN=%CSRF_TOKEN%;" 
  -H "Content-Type: application/json" -d "{\"name\": \"admin\", \"password\": \"123456\", \"description\": \"super user admin\", \"email\": \"admin@test.com\"}" 
  "http://localhost:7750/pulsar-manager/users/superuser"
```

访问：
```
http://localhost:9527/ 
用户名密码：admin/123456
```

配置environments：

```
这里需要保证Pulsar Manager应用服务能够访问到Pulsar应用，由于都是通过Docker部署，配置Service URL需要使用网络IP，不要用localhost。
```
![pulsar-manager-env](/assets/images/2023/sucls/02_05/pulsar-manager-env.png)

管理界面：
![pulsar-manager](/assets/images/2023/sucls/02_05/pulsar-manager.png)

### Pulsar与Springboot集成

+ springboot version : 2.3.7.RELEASE
+ pulsar client: 2.10.2

1. 定义Properties
主要简单定义了Broker相关的属性
```java
@Data
@ConfigurationProperties(prefix = "pulsar")
public class PulsarProperties {
    
    private String cluster;
    
    private String namespace;

    private String serverUrl;

    private String token;
}
```
2. 定义配置
定义了一些常用的组件，比如生产、消费工厂
```java
@Configuration
@EnableConfigurationProperties({PulsarProperties.class})
public class PulsarBootstrapConfiguration {

    private final PulsarProperties properties;

    public PulsarBootstrapConfiguration(PulsarProperties properties) {
        this.properties = properties;
    }

    @Bean(destroyMethod = "close")
    public PulsarClient pulsarClient() throws PulsarClientException {
        ClientBuilder clientBuilder = PulsarClient.builder().serviceUrl(properties.getServerUrl());
        return clientBuilder.build();
    }

    @Bean
    public PulsarProducerFactory pulsarProducerFactory() throws PulsarClientException {
        return new PulsarProducerFactory(pulsarClient(), properties);
    }

    @Bean
    public PulsarConsumerFactory pulsarConsumerFactory() throws PulsarClientException {
        return new PulsarConsumerFactory(pulsarClient(), properties);
    }

}
```
3. 启动服务
在服务启动后，通过实现SmartInitializingSingleton接口，完成容器基本启动(不包含Lazy的Bean)后，开始对消费者Consumer监听
```java
@Slf4j
@SpringBootApplication
public class PulsarApplication implements SmartInitializingSingleton {

    @Autowired
    private PulsarConsumerFactory consumerFactory;

    public static void main(String[] args) {
        SpringApplication.run(PulsarApplication.class,args);
    }

    @Override
    public void afterSingletonsInstantiated() {
        startConsumerListener();
    }

    private void startConsumerListener(){
        Consumer<String> consumer = createConsumer();
        if( consumer != null ){
            while (!Thread.currentThread().isInterrupted()){
                CompletableFuture<? extends Message<?>> completableFuture = consumer.receiveAsync();
                Message<?> message = null;
                try {
                    message = completableFuture.get();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    log.error("错误",e);
                } catch (ExecutionException e) {
                    log.error("错误",e);
                }

                if( message!=null ){
                    try {
                        log.info(" 接收消息：{} ", message.getValue() );
                        consumer.acknowledge(message);
                    } catch (PulsarClientException e) {
                        consumer.negativeAcknowledge(message);
                        throw new RuntimeException(e);
                    }
                }
            }
        }
    }

    private Consumer<String> createConsumer() {
        try {
            return consumerFactory.getConsumer(Constants.TOPIC_DEMO);
        } catch (PulsarClientException e) {
            log.error("创建consumer出错:{}", e.getMessage(),e);
        }
        return null;
    }
}
```
4. 消息发送
```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class PulsarBootTests {

    @Autowired
    private PulsarProducerFactory producerFactory;

    @Test
    public void sendMessage() throws PulsarClientException {
        Producer producer = producerFactory.getProducer(Constants.TOPIC_DEMO);

        producer.send(" 测试消息: " + new Date());

        producer.close();
    }

}
```
5. 检查消息接收情况
```
2023-02-05 12:05:14.043  INFO 23472 --- [ulsar-timer-6-1] o.a.p.c.impl.ConsumerStatsRecorderImpl   : [TOPIC_DEMO] [sub-TOPIC_DEMO] [7c2b2] Prefetched messages: 0 --- Consume throughput received: 0.02 msgs/s --- 0.00 Mbit/s --- Ack sent rate: 0.02 ack/s --- Failed messages: 0 --- batch messages: 0 ---Failed acks: 0
2023-02-05 12:06:16.425  INFO 23472 --- [           main] com.sucl.pulsar.PulsarApplication        :  接收消息： 测试消息: Sun Feb 05 12:06:16 CST 2023 
```

### 结束语
    
  该篇主要通过官网对Pulsar做个简单的了解与尝试，以及简单的示例实现消息的发送与接收，其中各个组件（生产者、消费者）没有过多的配置，同时基于Springboot也没有过多的封装，
  在生产环境需要根据Pulsar的特性以及官方API将各个环节补充完整。
  
