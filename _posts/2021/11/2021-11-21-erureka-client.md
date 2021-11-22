---

layout: post
title:  Eureka Client的创建
tagline: by 某某白米饭
categories: erueka
tags:
- 某某白米饭
---
<!--more-->

### EurekaClient 创建

在上篇中已经讲了 Eureka Server 的配置文件读取。接下来讲讲 Eureka Client 的创建。

每一个 eureka server 都是一个 eureka client，是用来和其他 eureka-server 节点注册和通信的。

### EurekaInstanceConfig 和 EurekaClientConfig

Instance 是实例的意思， EurekaInstanceConfig 就是 eureka client 的配置文件中的服务实例信息。
```java
 EurekaInstanceConfig instanceConfig = isCloud(ConfigurationManager.getDeploymentContext())
                    ? new CloudInstanceConfig()
                    : new MyDataCenterInstanceConfig();

applicationInfoManager = new ApplicationInfoManager(
        instanceConfig, new EurekaConfigBasedInstanceInfoProvider(instanceConfig).get());

EurekaClientConfig eurekaClientConfig = new DefaultEurekaClientConfig();
```

EurekaInstanceConfig 就是将 eureka-client.properties 文件中的服务实例信息加载到 ConfigManager 中，然后基于 EurekaInstanceConfig 对外暴露接口，并且提供配置项的默认值。

![](http://www.javanorth.cn/assets/images/2021/eureka/eurekaclient/0.png)

ApplicationInfoManager，包含了服务实例的信息、配置，作为服务实例管理的一个组件，由 EurekaInstanceConfig 和 InstanceInfo 构建完成，InstanceInfo 的创建是在 `new EurekaConfigBasedInstanceInfoProvider(instanceConfig).get()` 中，用了构造器模式 LeaseInfo.Builder.newBuilder() 构造了一个复杂的代表一个服务实例的 InstanceInfo 对象。核心思想就是在 EurekaInstanceConfig 中读取各种实例相关的配置信息，再构造了一些其他的对象，最终完成 InstanceInfo 对象的构建。

```java
public synchronized InstanceInfo get() {
    if (instanceInfo == null) {
        // Build the lease information to be passed to the server based on config
        LeaseInfo.Builder leaseInfoBuilder = LeaseInfo.Builder.newBuilder()
                .setRenewalIntervalInSecs(config.getLeaseRenewalIntervalInSeconds())
                .setDurationInSecs(config.getLeaseExpirationDurationInSeconds());

        if (vipAddressResolver == null) {
            vipAddressResolver = new Archaius1VipAddressResolver();
        }
        //代码太长，省略其他代码
    }
}
```

DefaultEurekaClientConfig 就是和 DefaultEurekaServerConfig 一样，读取 erueka-client.properties 配置文件的内容。具体方法在 com.netflix.discovery.internal.util.initConfig() 中。
最后都是交给 ConfigurationManager 管理。

EurekaInstanceConfig 和 EurekaClientConfig 虽然都是读取的 erueka-client.propeties ，但是读取的内容是不一样的。

EurekaInstanceConfig 接口

![](http://www.javanorth.cn/assets/images/2021/eureka/eurekaclient/1.png)

EurekaClientConfig 接口

![](http://www.javanorth.cn/assets/images/2021/eureka/eurekaclient/2.png)


### DiscoveryClient

DiscoveryClient 是创建 Erueka Client 的类，由 ApplicationInfoManager 和 EurekaClientConfig 组成。

```java
eurekaClient = new DiscoveryClient(applicationInfoManager, eurekaClientConfig);
```

DiscoveryClient 代码很长。

1. appPathIdentifier，服务实例的标识符，  appName，代表了一个服务名称，但是一个服务可能部署多台机器，每台机器上部署的就是一个服务实例，如：serviceABC/001

```java
appPathIdentifier = instanceInfo.getAppName() + "/" + instanceInfo.getId();
```

2. 是否需要注册到别的注册中心。eurekaServer 有个配置：eureka.client.fetchRegistry，单机情况下为 false。false 表示自己就是注册中心。职责就是维护服务实例，并不需要去检索服务。

```java
if (!config.shouldRegisterWithEureka() && !config.shouldFetchRegistry()) {}
```

3. 支持调度的线程池。

```java
scheduler = Executors.newScheduledThreadPool(2,
        new ThreadFactoryBuilder()
                .setNameFormat("DiscoveryClient-%d")
                .setDaemon(true)
                .build());
```

4. 支持心跳的线程池。

```java
heartbeatExecutor = new ThreadPoolExecutor(
            1, clientConfig.getHeartbeatExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(),
            new ThreadFactoryBuilder()
                    .setNameFormat("DiscoveryClient-HeartbeatExecutor-%d")
                    .setDaemon(true)
                    .build()
    ); 
```

5. 支持缓存刷新的线程池。

```java
cacheRefreshExecutor = new ThreadPoolExecutor(
            1, clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>(),
            new ThreadFactoryBuilder()
                    .setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d")
                    .setDaemon(true)
                    .build()
    );
```

6. 支持底层的 eureka client 和 eureka server 进行通信的组件。

```java
eurekaTransport = new EurekaTransport();
```

7. 初始化调度任务。

```java
initScheduledTasks();
```

### 总结

先创建了 EurekaInstanceConfig，基于 EurekaInstanceConfig 和 InstanceInfo 创建 ApplicationInfoManager，之后才创建 DiscoveryClient。
