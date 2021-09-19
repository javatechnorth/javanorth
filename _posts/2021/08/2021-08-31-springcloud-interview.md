---
layout: post
title: spring cloud 面试题
tagline: by feng
categories: spring
tags: 
    - feng
---
spring cloud 面试题
<!--more-->

### 1、什么是微服务？

> 单个轻量级服务一般为一个**单独微服务**，微服务讲究的是 专注某个功能的实现，比如登录系统只专注于用户登录方面功能的实现，讲究的是**职责单一，开箱即用，可以独立运行**。微服务架构系统是一个**分布式的系统**，按照业务进行划分服务单元模块，**解决单个系统的不足**，满足越来越**复杂的业务需求**。
>
> Martin Fowler：就目前而言，对于微服务业界并没有一个统一的、标准的定义。但通常而言，微服务架构是一种架构模式或者说是架构风格，它提倡将单一应用程序划分成一组小的服务。每个服务运行在其独立的自己的进程中服务之间相互配合、相互协调，为用户提供最终价值。服务之间采用轻量级通信。每个服务都围绕具体业务进行构建，并能够独立部署到生产环境等。另外应尽量避免统一的、集中的服务管理机制。

通俗的来讲：

微服务就是一个独立的职责单一的服务应用程序。在 intellij idea 工具里面就是用maven开发的一个个独立的module，具体就是使用springboot 开发的一个小的模块，处理单一专业的业务逻辑，一个模块只做一个事情。

微服务强调的是服务大小，关注的是某一个点，具体解决某一个问题/落地对应的一个服务应用，可以看做是idea 里面一个 module。

比如你去医院：你的牙齿不舒服，那么你就去牙科。你的头疼，那么你就去脑科。一个个的科室，就是一个微服务，一个功能就是一个服务。



### 2、什么是微服务架构？

在前面你理解什么是微服务，那么对于微服务架构基本上就已经理解了。

微服务架构 就是 对微服务进行管理整合应用的。微服务架构 依赖于 微服务，是在微服务基础之上的。

**例如： **上面已经列举了什么是微服务。在医院里，每一个科室都是一个独立的微服务，那么 这个医院 就是 一个大型的微服务架构，就类似 院长 可以 对下面的 科室进行管理。微服务架构主要就是这种功能。



### 3、使用Spring Cloud有什么优势？

使用Spring Boot开发分布式微服务时，我们面临以下问题:

与分布式系统相关的复杂性-这种开销包括网络问题，延迟开销，带宽问题，安全问题。

服务发现-服务发现工具管理群集中的流程和服务如何查找和互相交谈。它涉及一个服务目录，在该目录中注册服务，然后能够查找并连接到该目录中的服务。

冗余-分布式系统中的冗余问题。

负载平衡 --负载平衡改善跨多个计算资源的工作负荷，诸如计算机，计算机集群，网络链路，中央处理单元，或磁盘驱动器的分布。

性能-问题 由于各种运营开销导致的性能问题。部署复杂性-Devops技能的要求。


### 4、微服务的优缺点是什么？说下你在项目中碰到的坑。

**优点：**松耦合，聚焦单一业务功能，无关开发语言，团队规模降低。在开发中，不需要了解多有业务，只专注于当前功能，便利集中，功能小而精。微服务一个功能受损，对其他功能影响并不是太大，可以快速定位问题。微服务只专注于当前业务逻辑代码，不会和 html、css 或其他界面进行混合。可以灵活搭配技术，独立性比较舒服。

**缺点：**随着服务数量增加，管理复杂，部署复杂，服务器需要增多，服务通信和调用压力增大，运维工程师压力增大，人力资源增多，系统依赖增强，数据一致性，性能监控。



### 5、微服务的优点缺点?说下开发项目中遇到的坑?

**优点:**

（1）每个服务直接足够内聚，代码容易理解

（2）开发效率高，一个服务只做一件事，适合小团队开发

（3）松耦合，有功能意义的服务。

（4）可以用不同语言开发，面向接口编程。

（5）易于第三方集成

（6）微服务只是业务逻辑的代码，不会和HTML,CSS或其他界

（7）可以灵活搭配，连接公共库/连接独立库

**缺点:**

（1）分布式系统的责任性

（2）多服务运维难度加大。

（3）系统部署依赖，服务间通信成本，数据一致 ，系统集成测试，性能监控。



### 6、SpringBoot和SpringCloud的区别？

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务SpringBoot可以离开SpringCloud独立使用开发项目， 但是SpringCloud离不开SpringBoot ，属于依赖的关系.

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的 治理框架



### 7、使用 Spring Cloud 有什么优势？

使用 Spring Boot 开发分布式微服务时，我们面临以下问题

**1、** 与分布式系统相关的复杂性-这种开销包括网络问题，延迟开销，带宽问题，安全问题。

**2、** 服务发现-服务发现工具管理群集中的流程和服务如何查找和互相交谈。它涉及一个服务目录，在该目录中注册服务，然后能够查找并连接到该目录中的服务。

**3、** 冗余-分布式系统中的冗余问题。

**4、** 负载平衡 --负载平衡改善跨多个计算资源的工作负荷，诸如计算机，计算机集群，网络链路，中央处理单元，或磁盘驱动器的分布。

**5、** 性能-问题 于各种运营开销导致的性能问题。

**6、** 部署复杂性 evops 技能的要求。



### 8、服务注册和发现是什么意思？Spring Cloud 如何实现？

当我们开始一个项目时，我们通常在属性文件中进行所有的配置。随着越来越多的服务开发和部署，添加和修改这些属性变得更加复杂。有些服务可能会下降，而某些位置可能会发生变化。手动更改属性可能会产生问题。Eureka 服务注册和发现可以在这种情况下提供帮助。由于所有服务都在 Eureka 服务器上注册并通过调用 Eureka 服务器完成查找，因此无需处理服务地点的任何更改和处理。



### 9、你所知道微服务的技术栈有哪些？列举一二。

| 微服务条目                     | 落地技术                                                     |
| :----------------------------- | :----------------------------------------------------------- |
| 服务开发                       | SpringBoot、Spring、SpringMVC                                |
| 服务配置与管理                 | Netfix公司的Archaius、阿里的Dlamond等                        |
| 服务注册与发现                 | Eurka、Consul、Zookeeper等                                   |
| 服务调用                       | Rest（服务通信）、RPC（Dubbo）、GRpc                         |
| 服务熔断器                     | Hystrix、Envoy等                                             |
| 负载均衡                       | Nginx、Ribbon等                                              |
| 服务接口调用（客户端简化工具） | Fegin等                                                      |
| 消息队列                       | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理               | SpringCloudConfig、Chef等                                    |
| 服务路由（API网关）            | Zuul等                                                       |
| 服务监控                       | Zabbix，Nagios，Metrics，Spectator等                         |
| 全链路追踪                     | Zipkin，Brave，Dapper等                                      |
| 服务部署                       | Docker，OpenStack，Kubernetes等                              |
| 数据流操作开发包               | SpringCloud Stream（封装与Redis，Rabbit，kafka等发送接收消息） |
| 事件消息总线                   | Spring Cloud Bus                                             |



### 10、微服务之间如何独立通讯的?

同步通信：dobbo通过 **RPC 远程过程调用、springcloud通过 REST 接口json调用** 等。

异步：消息队列，如：**RabbitMq、ActiveM、Kafka** 等。



### 11、REST 和RPC对

**1、** RPC主要的缺陷是服务提供方和调用方式之间的依赖太强，需要对每一个微服务进行接口的定义，并通过持续继承发布，严格版本控制才不会出现冲突。

**2、** REST是轻量级的接口，服务的提供和调用不存在代码之间的耦合，只需要一个约定进行规范。



### 12、你所知道的微服务技术栈？

维度(springcloud)

服务开发：springboot spring springmvc

服务配置与管理:Netﬁx公司的Archaiusm ,阿里的Diamond

服务注册与发现:Eureka,Zookeeper

服务调用:Rest RPC gRpc

服务熔断器:Hystrix

服务负载均衡:Ribbon Nginx

服务接口调用:Fegin

消息队列:Kafka Rabbitmq activemq

服务配置中心管理:SpringCloudConﬁg

服务路由（API网关）Zuul

事件消息总线:SpringCloud Bus


### 13、Spring Cloud的子项目（主要项目）

大致可分成两类，

一类是对现有成熟框架"Spring Boot化"的封装和抽象，也是数量最多的项目；

第二类是开发了一部分分布式系统的基础设施的实现，如Spring Cloud Stream扮演的就是kafka, ActiveMQ这样的角色。



**Spring Cloud Config**

集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作。



**Spring Cloud Netflix**

Netflix OSS 开源组件集成，包括Eureka、Hystrix、Ribbon、Feign、Zuul等核心组件。

- Eureka：服务治理组件，包括服务端的注册中心和客户端的服务发现机制；
- Ribbon：负载均衡的服务调用组件，具有多种负载均衡调用策略；
- Hystrix：服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力；
- Feign：基于Ribbon和Hystrix的声明式服务调用组件；
- Zuul：API网关组件，对请求提供路由及过滤功能。



**Spring Cloud Bus**

用于将服务和服务实例与分布式消息系统链接在一起的事件总线。在集群中传播状态更改很有用（例如配置更改事件）。

你可以简单理解为 `Spring Cloud Bus` 的作用就是**管理和广播分布式系统中的消息**，也就是消息引擎系统中的广播模式。当然作为 **消息总线** 的 `Spring Cloud Bus` 可以做很多事而不仅仅是客户端的配置刷新功能。

而拥有了 `Spring Cloud Bus` 之后，我们只需要创建一个简单的请求，并且加上 `@ResfreshScope` 注解就能进行配置的动态修改了。

**Spring Cloud Consul**

基于Hashicorp Consul的服务治理组件。

**Spring Cloud Security**

安全工具包，对Zuul代理中的负载均衡OAuth2客户端及登录认证进行支持。

**Spring Cloud Sleuth**

Spring Cloud应用程序的分布式请求链路跟踪，支持使用Zipkin、HTrace和基于日志（例如ELK）的跟踪。

**Spring Cloud Stream**

轻量级事件驱动微服务框架，可以使用简单的声明式模型来发送及接收消息，主要实现为Apache Kafka及RabbitMQ。

**Spring Cloud Task**

用于快速构建短暂、有限数据处理任务的微服务框架，用于向应用中添加功能性和非功能性的特性。

**Spring Cloud Zookeeper**

基于Apache Zookeeper的服务治理组件。

**Spring Cloud Gateway**

API网关组件，对请求提供路由及过滤功能。

**Spring Cloud OpenFeign**

基于Ribbon和Hystrix的声明式服务调用组件，可以动态创建基于Spring MVC注解的接口实现用于服务调用，在Spring Cloud 2.0中已经取代Feign成为了一等公民。

### 14、spring cloud 的核心组件有哪些

- Eureka：服务注册于发现。
- Feign：基于动态代理机制，根据注解和选择的机器，拼接请求 url 地址，发起请求。
- Ribbon：实现负载均衡，从一个服务的多台机器中选择一台。
- Hystrix：提供线程池，不同的服务走不同的线程池，实现了不同服务调用的隔离，避免了服务雪崩的问题。
- Zuul：网关管理，由 Zuul 网关转发请求给对应的服务。



### 15、springcloud如何实现服务的注册?

**1、**服务发布时，指定对应的服务名,将服务注册到 注册中心(eureka zookeeper)
**2、**注册中心加@EnableEurekaServer,服务用@EnableDiscoveryClient，然后用ribbon或feign进行服务直接的调用发现。


### 16、什么是 Netﬂix Feign？它的优点是什么？

Feign 是受到 Retroﬁt，JAXRS-2.0 和 WebSocket 启发的 java 客户端联编程序。Feign 的第一个目标是将约束分母的复杂性统一到 http apis，而不考虑其稳定性。在 employee-consumer 的例子中，我们使用了 emplo e-producer 使用 REST模板公开的 REST 服务。

但是我们必须编写大量代码才能执行以下步骤

**1、** 使用功能区进行负载平衡。

**2、** 获取服务实例，然后获取基本 URL。

**3、** 利用 REST 模板来使用服务。前面的代码如下

```java
@Controllerpublic class ConsumerControllerClient {
	@Autowired
	private LoadBalancerClient loadBalancer;
	public void getEmployee() throws RestClientException, IOException {
		ServiceInstance serviceInstance=loadBalancer.choose("employee-producer");
		System.out.println(serviceInstance.getUri());
		String baseUrl=serviceInstance.getUri().toString();
		baseUrl=baseUrl+"/employee";
		RestTemplate restTemplate = new RestTemplate();
		ResponseEntity<String> response=null;
		try{
			response=restTemplate.exchange(baseUrl,
			HttpMethod.GET, getHeaders(),String.class);
		}
		catch (Exception ex)
		{
			System.out.println(ex);
		}
		System.out.println(response.getBody());
	}}
```

之前的代码，有像 NullPointer 这样的例外的机会，并不是最优 。我们将看到如何使用 Netﬂix Fe n使呼叫变得更加轻松和清洁。如果 Netﬂix Ribbon 依赖关系 径中，那么 Feign 默认也会负载平衡。



### 17、分布式配置中心能干嘛？

**1、** 集中管理配置文件不同环境不同配置，动态化的配置更新，分环境部署比如

dev/test/prod/beta/release

**2、** 运行期间动态调整 置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息

**3、** 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置将配置信息以REST接口的形式暴露



###  18、什么是Eureka

eureka是Netflix开发的服务发现组件，本身是一个基于REST的服务。Spring Cloud将它集成在其子项目spring-cloud-netflix中， 以实现Spring Cloud的服务发现功能。eureka现在已经从1.0升级到2.0，可惜的是eureka2.0不在开源，但也不影响我们的使用。 由于基于REST服务，自然而然的就能想到，这个服务一定会有心跳检测、健康检查和客户端缓存等机制。

**Eureka包括两个端：**

- **Eureka Server：注册中心服务端**，用于维护和管理注册服务列表。
- **Eureka Client：注册中心客户端**，向注册中心注册服务的应用都可以叫做Eureka Client（包括Eureka Server本身）。



**Eureka Server**

- 依赖介绍
  - spring-cloud-starter-netflix-eureka-server eureka服务端的标识，标志着此服务是做为注册中心
- application.properties配置

```xml
spring.application.name=eureka-server  //服务名称
server.port=8000            //服务端口

eureka.client.register-with-eureka=false  //自身不做为服务注册到注册中心

eureka.client.fetch-registry=false     //从注册表拉取信息

eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/    服务注册地址
```

- 运行服务spring-cloud-eureka
- 运行成功后访问localhost:8000，会显示eureka提供的服务页面



**Eureka Client**

- 依赖介绍
  - spring-cloud-starter-netflix-eureka-client eureka客户端所需依赖。
  - spring-boot-starter-web web服务所需，内置tomcat服务器。
- application.properties配置

```xml
spring.application.name=eureka-client-a
server.port=8001

eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```

- 运行服务spring-cloud-clientA
- 运行成功后访问localhost:8000，会显示eureka提供的服务页面



### 19、eureka自我保护机制是什么?

当Eureka Server 点在短时间内丢失了过多实例的连接时（比如网络故障或频繁启动关闭客户端）节点会进入自我保护模式，保护注册信息，不再删除注册数据，故障恢复时，自动退出自我保护模式。



### 20、作为 务注册中心，Eureka比Zookeeper好在哪里?

**1、** Eureka保证的是可用性和分区容错性，Zookeeper 保证的是一致性和分区容错性 。

**2、** Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障。而不会像zookeeper那样使整个注册服务瘫痪。



### 21、Eureka和zookeeper都可以提供服务注册与发现的功能，请说说两个的区别？

Zookeeper保证了CP（C：一致性，P：分区容错性），Eureka保证了AP（A：高可用）

**1、** 当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的信息，但不能容忍直接down掉不可用。就是说，服务注册功能对高可用性要求比较高，但zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新选leader。问题在于，选取leader时间过长，30 ~ 120s，且选取期间zk集群都不可用，这样就会导致选取期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够恢复，但是漫长的选取时间导致的注册长期不可用是不能容忍的。

**2、** Eureka保证了可用性，Eureka各个节点是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点仍然可以提供注册和查询服务。而Eureka的客户端向某个Eureka注册或发现时发生连接失败，则会自动切换到其他节点，只要有一台Eureka还在，就能保证注册服务可用，只是查到的信息可能不是最新的。除此之外，Eureka还有自我保护机制，如果在15分钟内超过85%的节点没有正常的心跳，那么Eureka就认为 户端与注册中心发生了网络故障，此时会出现以下几种情况：

①、Eureka不 从注册列表中移除因为长时间没有收到心跳而应该过期的服务。

②、Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上（即保证当前节点仍然可用）

③、当网络稳定时，当前实例新的注册信息会被同步到其他节点。

因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像Zookeeper那样使整个微服务瘫痪。



### 22、你所知道的微服务技术栈？

当Eureka Server 节点在短时间内丢失了过多实例的连接时（比如网络故障或频繁启动关闭客户端）节点会进入自我保护模式，保护注册信息，不再删除注册数据，故障恢复时，自动退出自我保护模式。



### 23、使用Spring Cloud有什么优势？

使用Spring Boot开发分布式微服务时，我们面临以下问题

- 与分布式系统相关的复杂性-这种开销包括网络问题，延迟开销，带宽问题，安全问题。
- 服务发现-服务发现工具管理群集中的流程和服务如何查找和互相交谈。它涉及一个服务目录，在该目录中注册服务，然后能够查找并连接到该目录中的服务。
- 冗余-分布式系统中的冗余问题。
- 负载平衡 --负载平衡改善跨多个计算资源的工作负荷，诸如计算机，计算机集群，网络链路，中央处理单元，或磁盘驱动器的分布。
- 性能-问题 由于各种运营开销导致的性能问题。
- 部署复杂性-Devops技能的要求。



### 24、SpringBoot 和 SpringCloud 之间关系？

**SpringBoot：**专注于快速方便的开发单个个体微服务（关注微观）；

**SpringCloud：**关注全局的微服务协调治理框架，将SpringBoot开发的一个个单体微服务组合并管理起来（关注宏观）；

SpringBoot可以离开SpringCloud独立使用，但是SpringCloud不可以离开SpringBoot，属于依赖关系。



### 25、SpringCloud 和 Dubbo 有哪些区别?

首先，他们都是**分布式管理框架**。

dubbo 是**二进制传输**，占用带宽会少一点。SpringCloud是**http 传输**，带宽会多一点，同时使用http协议一般会使用**JSON报文**，消耗会更大。

dubbo 开发难度较大，所依赖的 jar 包有很多问题**大型工程无法解决**。SpringCloud 对第三方的继承可以**一键式生成，天然集成**。

SpringCloud 接口协议约定比较松散，**需要强有力的行政措施来限制接口无序升级**。

最大的区别: **Spring Cloud抛弃了Dubbo 的RPC通信，采用的是基于HTTP的REST方式。**

>  严格来说，这两种方式各有优劣。虽然在一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更为合适。



### 26、什么是Spring Cloud Config?

在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。在Spring Cloud中，有分布式配置中心组件spring cloud config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。在spring cloud config 组件中，分两个角色，一是config server，二是config client。

使用：

**1、**添加pom依赖

**2、**配置文件添加相关配置

**3、**启动类添加注解@EnableConfigServer



### 27、什么是 zuul路由网关

**1、** Zuul 包含了对请求的路由和过滤两个最主要的功能:其中 责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负 请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础、

**2、** Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

**注意：** Zuul服务最终还是会注册进Eureka 提供=代理+路由+过滤 三大功能



### 28、什么是Ribbon？

ribbon是一个负载均衡客户端，可以很好的控制htt和tcp的一些行为。feign默认集成了ribbon。



### 29、Ribbon负载均衡能干什么？

**1、** 将用户的请求平摊的分配到多个服务上

**2、** 集中式LB即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；

**3、** 进程内LB将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

注意：Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方 它来获取到服务提供方的地址。



###  30、什么是feigin？它的优点是什么？

**1、**feign采用的是基于接口的注解
**2、**feign整合了ribbon，具有负载均衡的能力
**3、**整合了Hystrix，具有熔断的能力

使用:
**1、**添加pom依赖。
**2、**启动类添加@EnableFeignClients
**3、**定义一个接口@FeignClient(name=“xxx”)指定调用哪个服务



### 31、Ribbon和Feign的区别？

**1、**Ribbon都是调用其他服务的，但方式不同。
**2、**启动类注解不同，Ribbon是@RibbonClient feign的是@EnableFeignClients
**3、**服务指定的位置不同，Ribbon是在@RibbonClient注解上声明，Feign则是在定义抽象方法的接口中使用@FeignClient声明。
**4、**调用方式不同，Ribbon需要自己构建http请求，模拟http请求然后使用RestTemplate发送给其他服务，步骤相当繁琐。Feign需要将调用的方法定义成抽象方法即可。



### 32、什么是Spring Cloud Gateway?

Spring Cloud Gateway是Spring Cloud官方推出的第二代网关框架，取代Zuul网关。网关作为流量的，在微服务系统中有着非常作用，网关常见的功能有路由转发、权限校验、限流控制等作用。

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters是各种过滤器，用来对请求做各种判断和修改。



### 33、什么是 Netflix Feign？它的优点是什么？

Feign 是受到 Retrofit，JAXRS-2.0 和 WebSocket 启发的 java 客户端联编程序。

Feign 的第一个目标是将约束分母的复杂性统一到 http apis，而不考虑其稳定性。

在 employee-consumer 的例子中，我们使用了 employee-producer 使用 REST模板公开的 REST 服务。

但是我们必须编写大量代码才能执行以下步骤

**1、** 使用功能区进行负载平衡。

**2、** 获取服务实例，然后获取基本 URL。

**3、** 利用 REST 模板来使用服务。前面的代码如下

```java
@Controller
public class ConsumerControllerClient {
@Autowired
private LoadBalancerClient loadBalancer;
public void getEmployee() throws RestClientException, IOException {
	ServiceInstance serviceInstance=loadBalancer.choose("employee-producer");
	System.out.println(serviceInstance.getUri());
	String baseUrl=serviceInstance.getUri().toString();
	baseUrl=baseUrl+"/employee";
	RestTemplate restTemplate = new RestTemplate();
	ResponseEntity<String> response=null;
	try{
		response=restTemplate.exchange(baseUrl,
					HttpMethod.GET, getHeaders(),String.class);
	}
	catch (Exception ex)
		{
		System.out.println(ex);
	}
	System.out.println(response.getBody());

```

之前的代码，有像 NullPointer 这样的例外的机会，并不是最优的。我们将看到如何使用 Netflix Feign 使呼叫变得更加轻松和清洁。如果 Netflix Ribbon 依赖关系也在类路径中，那么 Feign 默认也会负责负载平衡。



### 34、什么是Spring Cloud Gateway?

Spring Cloud Gateway是Spring Cloud官方推出的第二代网关框架，取代Zuul网关。网关作为流量的，在微服务系统中有着非常作用，网关常见的功能有路由转发、权限校验、限流控制等作用。

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters是各种过滤器，用来对请求做各种判断和修改。



### 35、服务注册和发现是什么意思？Spring Cloud如何实现？

当我们开始一个项目时，我们通常在属性文件中进行所有的配置。随着越来越多的服务开发和部署，添加和修改这些属性变得更加复杂。有些服务可能会下降，而某些位置可能会发生变化。手动更改属性可能会产生问题。 Eureka服务注册和发现可以在这种情况下提供帮助。由于所有服务都在Eureka服务器上注册并通过调用Eureka服务器完成查找，因此无需处理服务地点的任何更改和处理。



### 36、负载平衡的意义什么？

在计算中，负载平衡可以改善跨计算机，计算机集群，网络链接，中央处理单元或磁盘驱动器等多种计算资源的工作负载分布。负载平衡旨在优化资源使用，最大化吞吐量，最小化响应时间并避免任何单一资源的过载。使用多个组件进行负载平衡而不是单个组件可能会通过冗余来提高可靠性和可用性。负载平衡通常涉及专用软件或硬件，例如多层交换机或域名系统服务器进程。



### 37、什么是熔断？什么是服务降级？

服务熔断的作用类似于我们家用的保险丝，当某服务出现不可用或响应超时的情况时，为了防止整个系统出现雪崩，暂时停止对该服务的调用。

服务降级是从整个系统的负荷情况出发和考虑的，对某些负荷会比较高的情况，为了预防某些功能（业务场景）出现负荷过载或者响应慢的情况，在其内部暂时舍弃对一些非核心的接口和数据的请求，而直接返回一个提前准备好的fallback（退路）错误处理信息。这样，虽然提供的是一个有损的服务，但却保证了整个系统的稳定性和可用性。



### 38、什么是Netflix Feign？它的优点是什么？

Feign是受到Retrofit，JAXRS-2.0和WebSocket启发的java客户端联编程序。Feign的第一个目标是将约束分母的复杂性统一到http apis，而不考虑其稳定性。在employee-consumer的例子中，我们使用了employee-producer使用REST模板公开的REST服务。

但是我们必须编写大量代码才能执行以下步骤

- 使用功能区进行负载平衡。
- 获取服务实例，然后获取基本URL。
- 利用REST模板来使用服务。 前面的代码如下

```java
@Controller
public class ConsumerControllerClient {

@Autowired
private LoadBalancerClient loadBalancer;

public void getEmployee() throws RestClientException, IOException {

    ServiceInstance serviceInstance=loadBalancer.choose("employee-producer");

    System.out.println(serviceInstance.getUri());

    String baseUrl=serviceInstance.getUri().toString();

    baseUrl=baseUrl+"/employee";

    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<String> response=null;
    try{
    response=restTemplate.exchange(baseUrl,
            HttpMethod.GET, getHeaders(),String.class);
    }catch (Exception ex)
    {
        System.out.println(ex);
    }
    System.out.println(response.getBody());
}
```

之前的代码，有像NullPointer这样的例外的机会，并不是最优的。我们将看到如何使用Netflix Feign使呼叫变得更加轻松和清洁。如果Netflix Ribbon依赖关系也在类路径中，那么Feign默认也会负责负载平衡。


### 39、微服务是如何对外提供统一接口的(zuul具体使用)

因为每一个微服务都是独立运行的，都有自己独立的IP和端口，而当他们需要统一对外提供服务这时候就需要
SpringCloud网关 zuul网关也是netflix公司旗下的项目

使用它也很简单
在pom依赖中 引入 Spring-cloud-starter-netflix-zuul

在SpringBoot启动类中 开启 @EnableZuulProxy

然后在配置文件中定义 路由规则：

routes:
路由名称:
path: /映射路径/**
serviceId: Eureka中的服务名称

zuul也提供了过滤器功能，如果要做一些token检查 或者 过滤时可以使用
用法 就是写一个类 继承 ZuulFilter类
会要求我们实现几个方法
filterType: 过滤器什么时候执行 pre 前置 post 过程中 after 之后
shouldFilter: 过滤器是否执行 可以写判断方法 返回boolean值 true执行，false不执行此过滤器
filterOrder: 过滤器的执行顺序 排序号
run: 具体过滤器的方法


