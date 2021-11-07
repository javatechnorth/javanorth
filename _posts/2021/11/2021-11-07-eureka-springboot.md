---
layout: post
title: 看得懂系列：Spring Boot 启动 Eureka Server 流程
tagline: by 某某白米饭
categories: eureka
tags: 
    - 某某白米饭
---

大家好，我是指北君。

在上篇中已经说过了 Eureka-Server 本质上是一个 web 应用的项目，今天就来看看 Spring Boot 是怎么启动 Eureka 的。
<!--more-->

### Spring Boot 启动 Eureka 流程

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer {
	public static void main(String[] args) {
		SpringApplication.run(EurekaServer.class, args);
	}
}

``` 

上面的代码是最最平常的 Spring Boot 启动类。Spring Boot 启动 eureka 的关键注解就在 @EnableEurekaServer 上面。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({EurekaServerMarkerConfiguration.class})
public @interface EnableEurekaServer {
}
```

可以看到这注解导入了一个 EurekaServerMarkerConfiguration 类。

```java
@Configuration
public class EurekaServerMarkerConfiguration {

	@Bean
	public Marker eurekaServerMarkerBean() {
		return new Marker();
	}

	class Marker {
	}
}
```

EurekaServerMarkerConfiguration 向 Spring 容器注入了一个 EurekaServerMarkerConfiguration.Marker 对象。Maker 是一个空对象，是一个标记开关的类。具体开关的类在上面的注释中。

```java
/**
 * Responsible for adding in a marker bean to activate
 * {@link EurekaServerAutoConfiguration}
 *
 * @author Biju Kunjummen
 */
```

EurekaServerMarkerConfiguration.Marker 对象用于激活 EurekaServerAutoConfiguration 类。

那 EurekaServerAutoConfiguration 类是在什么时候加载的呢？

![](http://www.javanorth.cn/assets/images/2021/eureka/springboot/0.png)

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.netflix.eureka.server.EurekaServerAutoConfiguration
```

如上图，EurekaServerAutoConfiguration 启动的调用是在 spring.factories 中的，在 Spring Boot 的启动过程中，会加载所有的 spring.factories。这个时候会读取并加载里面的内容到 Spring 中。

* @Import(EurekaServerInitializerConfiguration.class)：初始化 EurekaServerAutoConfiguration 的时候会导入 EurekaServerInitializerConfiguration 类。
* @ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class)：当 Spring 中有 EurekaServerMarkerConfiguration.Marker 类的实例存在就把 EurekaServerAutoConfiguration 也导入到 Spring 的容器中。


```java
public class EurekaServerInitializerConfiguration
		implements ServletContextAware, SmartLifecycle, Ordered
```

* ServletContextAware：实现这个类可以获取到 ServletContext 容器上下文。
* SmartLifecycle：当 Spring 容器加载所有 bean 并完成初始化之后，会接着回调实现该接口的类中对应的 start() 方法

看一下 start() 方法里面调用了什么？

```java
@Override
public void start() {
	new Thread(new Runnable() {
		@Override
		public void run() {
			try {
				eurekaServerBootstrap.contextInitialized(EurekaServerInitializerConfiguration.this.servletContext);
				log.info("Started Eureka Server");

				publish(new EurekaRegistryAvailableEvent(getEurekaServerConfig()));
				EurekaServerInitializerConfiguration.this.running = true;
				publish(new EurekaServerStartedEvent(getEurekaServerConfig()));
			}
			catch (Exception ex) {
				// Help!
				log.error("Could not initialize Eureka servlet context", ex);
			}
		}
	}).start();
}
```

start() 启动了一个线程，在线程里面 Start 了 Eureka Server。eurekaServerBootstrap 是一个自动注入 EurekaServerBootstrap 的对象。EurekaServerBootstrap 在上一篇中已经说过了，它是 Eureka Server 的启动类。最后看看它的 contextInitialized() 方法。


```java
public void contextInitialized(ServletContext context) {
	try {
		initEurekaEnvironment();
		initEurekaServerContext();

		context.setAttribute(EurekaServerContext.class.getName(), this.serverContext);
	}
	catch (Throwable e) {
		log.error("Cannot bootstrap eureka server :", e);
		throw new RuntimeException("Cannot bootstrap eureka server :", e);
	}
}
```

contextInitialized 方法调用了 initEurekaEnvironment()，初始化 Eureka 的运行环境；initEurekaServerContext()，初始化 Eureka 的上下文。


### 总结

Spring Boot 启动 Erueka Server 经历了一下步骤：
1. @EnableEurekaServer 注解
2. 注入了 EurekaServerMarkerConfiguration.Marker 对象
3. 判断容器里是否有 EurekaServerMarkerConfiguration.Marker 对象注入了  EurekaServerAutoConfiguration
4. 导入了实现 SmartLifecycle 接口的 EurekaServerInitializerConfiguration 类
5. Spring 容器在初始化后调用了 EurekaServerInitializerConfiguration 对象的 start() 方法
6. start() 中启动了一个线程，调用了 Erueka Server 的启动类：EurekaServerBootstrap。
