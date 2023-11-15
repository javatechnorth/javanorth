
---
layout: post
title:  2023-10-28 一文带你了解SkyWalking
tagline: by 沉浮
categories: 
tags: 沉浮
---

<!--more-->
## Apache SkyWalking

   [SkyWalking](https://github.com/apache/skywalking)是一个开源可观测平台，用于收集、分析、聚合和可视化来自服务和云原生基础设施的数据。SkyWalking 提供了一种简单的方法来保持分布式系统的清晰视图，甚至跨云。它是一种现代APM，专为云原生、基于容器的分布式系统而设计。

   > 文档版本8.9.1，当前最新版本9.10

   ![架构图](/assets/images/2023/sucls/10_28/architecture.webp)


### 介绍

   SkyWalking 是一个应用性能监控系统（APM）
    
   > 为微服务、云原生和基于容器（Docker, Kubernetes, Mesos）体系结构而设计,主要实现功能包括分布式追踪，性能指标分析和服务依赖分析等
	
### 相似产品与对比

   类似功能的组件还有：Zipkin、Pinpoint 、CAT、Dapper等

   - Zipkin是Twitter开源的调用链路分析工具，目前基于Spingcloud sleuth得到了广泛的应用，特点是轻量，部署简单。
   - 一个韩国团队开源的产品，运用了字节码增强技术，只需要在启动时添加启动参数即可，对代码无侵入，目前支持Java和PHP语言，底层采用HBase来存储数据，探针收集的数据- 粒度非常细，但性能损耗大，因其出现的时间较长，完成度也很高，应用的公司较多
   - Skywalking是本土开源的基于字节码注入的调用链路分析以及应用监控分析工具，特点是支持多种插件，UI功能较强，接入端无代码侵入。
   - CAT是由国内美团点评开源的，基于Java语言开发，目前提供Java、C/C++、Node.js、Python、Go等语言的客户端，监控数据会全量统计，国内很多公司在用，例如美团点评、携程、拼多多等，CAT跟下边要介绍的Zipkin都需要在应用程序中埋点，对代码侵入性强。

|      |Cat|Zipkin|Pinpoint|skywalking|
|:---:|:---|:---|:---|:---|
|依赖   |Java 6，7，8<br/>Maven 3.2.3+<br/>mysql5.6<br/>Linux 2.6以及之上（2.6内核才可以支持epoll）|Java 6，7，8<br/>Maven3.2+<br/>rabbitMQ|Java 6，7，8<br/>maven3+<br/>Hbase0.94+|Java 6，7，8<br/>maven3.0+<br/>nodejs<br/>zookeeper<br/>elasticsearch|
| 实现方式 |代码埋点（拦截器，注解，过滤器等）|拦截请求，发送（http，mq）数据至zipkin服务|java探针，字节码增强|java探针，字节码增强|
| 颗粒度 |代码级|接口级|方法级|方法级|
| 页面UI |*****|**|*****|****|
| 存储选择 |Mysql，hdfs|In-memory，mysql，cassandra，elasticsearch|Hbase|Elasticsearch，h2|
| 通信方式 |-|http，mq|Thrift|GRPC|
| MQ监控 |不支持|不支持|不支持|RocketMq，kafka|
| 全局调用 统计|支持|不支持|支持|支持|
| Trace查询 |不持支|支持|不支持|支持|
| 报警 |支持|不支持|支持|支持|
| Jvm监控 |不支持|不支持|支持|支持|
| 优点 |功能完善|spring-cloud-sleuth可以很好的集成zipkin ， 代码无侵入，集成非常简单 ， 社区更加活跃。对外提供有query接口，更加容易二次开发|完全无侵入， 仅需修改启动方式，界面完善，功能细致。|完全无侵入，界面完善，支持应用拓扑图及单个调用链查询。功能比较完善（zipkin + pinpoint）|
| 缺点 |代码侵入性较强，需要埋点文档比较混乱，文档与发布版本的符合性较低，需要依赖点评私服 （或者需要把他私服上的jar手动下载下来，然后上传到我们的私服上去）。|默认使用的是http请求向zipkin上报信息，耗性能。跟sleuth结合可以使用rabbitMQ的方式异步来做，增加了复杂度，需要引入rabbitMQ 。数据分析比较简单。|不支持查询单个调用链， 对外表现的是整个应用的调用生态。二次开发难度较高|3.2版本之前BUG较多 ，网上反映兼容性较差 . 3.2新版本的反映情况较少依赖较多。|
| 文档 |网上资料较少，仅官网提供的文档，比较乱|文档完善|文档完善|文档完善|
| 开发者 |大众点评|Twiter|Naver|吴晟（华为开发者） ，目前已经加入Apache孵化器|
| 使用公司 |大众点评，携程，陆金所，同程旅游，猎聘网，拼多多|Twiter|Naver|华为，alibaba cloud,天源迪科，当当网，京东金融|

### 功能

   开源监控平台，用于从服务和云原生基础设施收集、分析、聚合和可视化数据。SkyWalking提供了一种简单的方法来维护分布式系统的清晰视图，甚至可以跨云查看。它是一种现代APM，专门为云原生、基于容器的分布式系统设计

   >监测对象包括：service（服务）, service instance（实例）, endpoint（端点）
	
   **功能描述**：

   - 多种监控手段，可以通过语言探针和service mesh获得监控的数据
   - 支持多重语言的自动探针，包括JAVA, .NET Core和NodeJS
   - 轻量高效，无需大数据平台和大量的服务器资源
   - 模块化，UI ,存储，集群管理都有多种机制可选
   - 支持告警
   - 优秀的可视化解决方案

### 架构

   SkyWalking 在逻辑上分为四个部分：Probes、Platform backend、Storage 和 UI。

   ![架构图](/assets/images/2023/sucls/10_28/SkyWalking_Architecture_20210424.png)

#### 探针Probe

   采集tracing（调用链数据）和metric（指标）信息并上报，上报通过HTTP或者gRPC方式按要求重新格式化数据发送数据到Skywalking Collector
	
   - 自动探针：Java支持的中间件、框架与类库列表
   - 手动探针：OpenTrackingApi、@Trace注解、trackId集成到日志中。
	
#### 后端Platform backend

	支持数据聚合、分析和流式处理，包括跟踪、度量和日志。
   基于gRpc、Http
   链路数据收集器，对agent传过来的tracing和metric数据进行整合分析通过Analysis Core模块处理并落入相关的数据存储中，同时会通过Query Core模块进行二次统计和监控告警

#### 数据存储Storage

   通过开放/可插入接口存储 SkyWalking 数据，支持多种方式存储数据 H2，ElasticSearch,MySQL, TiDB, InfluxDB或自定义

#### 可视化平台UI

   基于GraphQL Http
   高度可定制的基于 Web 的界面的可视化平台，允许 SkyWalking 最终用户可视化和管理。

### 下载安装

#### 下载

   [官方下载地址](https://skywalking.apache.org/downloads/)

   [归档地址](https://archive.apache.org/dist/skywalking/)

   [Rocketbot-UI](https://github.com/apache/skywalking-rocketbot-ui) ~~~该版本在9.0.0前使用~~~

   [Booster UI](https://github.com/apache/skywalking-booster-ui) ***9.0.0及以后版本替换成新的UI***

   > 注： APM已经集成UI，不需要单独下载与部署

#### Window安装

+ 安装APM _(8.9.1，h2)_
   
   - 下载apache-skywalking-apm-bin
   - 解压执行命名 ~/bin/startup.bat
   - 访问http://localhost:8080
    ![SkyWalking 8.9.1.png](/assets/images/2023/sucls/10_28/SkyWalking%208.9.1.png)

#### Docker安装

  + 安装OAP
  
下载镜像
```bash
 docker pull apache/skywalking-oap-server:8.9.1
```
启动容器
```bash
 docker run --name oap -p 12800:12800 -p 11800:11800 -p 1234:1234 --restart always -d apache/skywalking-oap-server:8.9.1
```
  + 安装UI
  
下载镜像  
```bash
   docker pull apache/skywalking-ui
```
启动容器
```bash
 docker run --name oap-ui -p 18080:8080 --restart always -d -e SW_OAP_ADDRESS=http://localhost:12800 apache/skywalking-ui
```
访问
   > http://localhost:18080

### 系统集成

   + 探针
  
	负责进行数据的收集，包含了Tracing和Metrics的数据，agent会被安装到服务所在的服务器上，以方便数据的获取。探针使用gRPC协议与OAP平台通信并上报数据。

   + 可观测性分析平台 OAP

	接收探针发送的数据，并在内存中使用分析引擎（Analysis Core)进行数据的整合运算，然后将数据存储到对应的存储介质上，比如 Elasticsearch、MySQL等存储服务。同时OAP还使用查询引擎(Query Core)提供HTTP查询接口。OAP默认监听两个端口gRPC协议端口11800、HTTP端口12800，gRPC用于探针上报数据，HTTP端口用于UI连接OAP平台获取数据。

   + UI

	Skywalking 提供单独的UI进行数据的查看，UI调用OAP提供的接口，获取对应的数据根据UI模板的配置进行展示。Skywalking UI与OAP之间使用Http协议进行通信。Skywalking UI默认监听8080端口提供Web服务。

   **Java Agent**
   + 下载:
	 [Java Agent v8.11.0](https://downloads.apache.org/skywalking/java-agent/8.11.0/apache-skywalking-java-agent-8.11.0.tgz.asc)
	
   + 目录结构
	  - activations                 # 工具包，默认加载。
	  - bootstrap-plugins           # 启动插件，默认加载。
	  - config                      # 配置文件
	  - logs                        # 日志
	  - optional-plugins            # 可选扩展插件，启动不加载，如需加载将其移到到plugins目录下。
	  - optional-reporter-plugins   # 可选统计类插件，启动不加载。
	  - plugins                     # 服务类插件
	  - skywalking-agent.jar        # 客户端主程序，需要被服务启动是引用。

   + 使用：

      项目启动命令添加-javaagent:/path/skywalking-agent/skywalking-agent.jar

   + 示例：
```shell
   java -javaagent:/path/skywalking-agent/skywalking-agent.jar -jar your-app.jar
```
   + 配置：

     - 系统属性：-Dskywalking.[config]=[value]
     - 代理参数: -javaagent:skywalking-agent.jar=[config]=[value],...
     - 系统环境变量：agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}，如果SW_AGENT_NAME 您的操作系统中存在环境变量，并且其值为skywalking-agent-demo，则agent.service_name此处的值将被覆盖为skywalking-agent-demo，否则将被设置为Your_ApplicationName。
     - 修改目录/path/skywalking-agent/config/agent.config文件
   ```properties
   // todo
   //服务名称
   agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}
   // OAP服务地址
   collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800}
   ```
   > 代理选项 > System.Properties(-D) > 系统环境变量 > 配置文件

   + [官方文档](https://skywalking.apache.org/docs/skywalking-java/v8.11.0/readme/)

   **Client JavaScript**
   + 安装
```bash
   npm install skywalking-client-js --save
```
   + 注册
```JavaScript
   import ClientMonitor from 'skywalking-client-js';

   // Report collected data to `http:// + window.location.host + /browser/perfData` in default
   ClientMonitor.register({
      collector: 'http://127.0.0.1:8080',
      service: 'test-ui',
      pagePath: '/current/page/name',
      serviceVersion: 'v1.0.0',
   });
```
```js
   // options
   {
      
   }
```
   + 手动收集指标
   
      页面加载时或任何其他有意义的时刻的指标，PV
     - register时设置autoTracePerf:false
     - 调用setPerformance
   ```JavaScript
      import ClientMonitor from 'skywalking-client-js';

      ClientMonitor.setPerformance({
         collector: 'http://127.0.0.1:8080',
         service: 'browser-app',
         serviceVersion: '1.0.0',
         pagePath: location.href,
         useFmp: true
      });
   ```
   + SPA

     - register时设置enableSPA:true，开启单页面应用基于hashchange event触发指标采集行为
     - 或者通过以下方法手动处理，在上报数据时手动更新页面名称，当调用该方法时，默认情况下将重新报告页面PV
   ```JavaScript
      app.on('routeChange', function (next) {
         ClientMonitor.setPerformance({
            collector: 'http://127.0.0.1:8080',
            service: 'browser-app',
            serviceVersion: '1.0.0',
            pagePath: location.href,
            useFmp: true
         });
      }); 
   ```
   + 跟踪请求数据

      支持跟踪这些(XMLHttpRequest和Fetch API)两种模式的数据请求。 同时，支持基于XMLHttpRequest和fetch的跟踪库和工具，如Axios、SuperAgent、OpenApi等
      ```js
         // Angular
         import { ErrorHandler } from '@angular/core';
         import ClientMonitor from 'skywalking-client-js';

         export class AppGlobalErrorhandler implements ErrorHandler {
         handleError(error) {
            ClientMonitor.reportFrameErrors({
               collector: 'http://127.0.0.1',
               service: 'angular-demo',
               pagePath: '/app',
               serviceVersion: 'v1.0.0',
            }, error);
         }
         }
         @NgModule({
         ...
         providers: [{provide: ErrorHandler, useClass: AppGlobalErrorhandler}]
         })
         class AppModule {}
         ```

         ```js
         // React
         class ErrorBoundary extends React.Component {
         constructor(props) {
            super(props);
            this.state = { hasError: false };
         }
         static getDerivedStateFromError(error) {
            // Update state so the next render will show the fallback UI.
            return { hasError: true };
         }
         componentDidCatch(error, errorInfo) {
            // You can also log the error to an error reporting service
            ClientMonitor.reportFrameErrors({
               collector: 'http://127.0.0.1',
               service: 'react-demo',
               pagePath: '/app',
               serviceVersion: 'v1.0.0',
            }, error);
         }
         render() {
            if (this.state.hasError) {
               // You can render any custom fallback UI
               return <h1>Something went wrong.</h1>;
            }
            return this.props.children; 
         }
         }
         <ErrorBoundary>
         <MyWidget />
         </ErrorBoundary>
         ```

         ```js
         // Vue
         Vue.config.errorHandler = (error) => {
         ClientMonitor.reportFrameErrors({
            collector: 'http://127.0.0.1',
            service: 'vue-demo',
            pagePath: '/app',
            serviceVersion: 'v1.0.0',
         }, error);
         }
         ```
   + 官方文档
  
   [skywalking-client-js](https://github.com/apache/skywalking-client-js/tree/v0.8.0)


### 监测对象

   + 服务(Service)

	对请求提供端点的单个应用或负载，在使用埋点、代理或 SDK 的时候，你可以定义服务的名字。如果不定义的话，SkyWalking 会使用在agent.conf中配置的默认服务名称。

   + 服务实例(Instance)

	服务组中的每个单独的工作负载都称为一个实例。就像pods在 Kubernetes 中一样，它不需要是单个操作系统进程，但是，如果您使用仪器代理，则实例实际上是一个真正的操作系统进程。

   + 端点(Endpoint)

	用于传入请求的服务中的路径，例如 HTTP URI 路径或 gRPC 服务类 + 方法签名。

### UI指标

   通过配置文件定义需要关注的指标
   通过特定的分析语言计算指标

   UI包括以下几个部分：
   - 仪表盘
   - 拓扑图
   - 追踪
   - 性能剖析
   - 日志
   - 告警
   - 事件
   - 调试
  
   > 仪表盘

   >> APM
   + Global
     - Services Load（CPM / PPM）：服务每分钟请求数
     - Slow Services（ms）：慢响应服务（按照响应时间排序）
     - Un-Health Services (Apdex)：Apdex分数（1为满分）
     - Slow Endpoints (ms)：慢Endpoint的平均响应时间
     - Global Response Latency（percentile in ms）：响应时间百分比
     - Global Heatmap：服务响应时间热力分布图，根据时间段内不同响应时间的数量显示颜色深度颜色越深，请求越多。
     
   + Service
     - Service Apdex 数字：当前服务的Apdex分数；
     - Successful Rate（%）：当前服务的请求成功率；
     - Service Load （CPM / PPM）数字：每分钟调用次数（CPM），如果是TCP，表示每分钟的数据包数（PPM）；
     - Service Avg Response Time（ms）：当前服务平均响应时间；
     - Service Apdex 折线图：当前服务一段时间内的Apdex分数；
     - Service Response Time Percentile（ms）：当前服务的百分比响应延时；
     - Successful Rate（%）折线图：当前服务一段时间内的请求成功率；
     - Service Load （CPM / PPM）折线图：当前服务一段时间内的每分钟调用次数；
     - Service Throughput（Bytes）：服务吞吐量，只适用于TCP服务；
     - Message Queue Consuming Count：消息队列消费数；
     - Message Queue Avg Consuming Latency（ms）：消息队列平均延迟时间；
     - Service Instances Load（CPM / PPM）：每个实例每分钟请求数；
     - Slow Service Instance（ms）：每个服务实例平均延时；
     - Service Instance Successful Rate（%）：服务实例的请求成功率。

   + Instance
     - Service Instance Load（CPM / PPM）：当前实例每分钟调用数；
     - Service Instance Throughput（Bytes）：当前实例的吞吐流量；
     - Service Instance Successful Rate（%）：当前实例调用成功比率；
     - Service Instance Latency（ms）：当前实例响应延时；
     - JVM CPU（Java Service）%：当前实例JVM的CPU占用百分比（相对于主机）；
     - JVM Memory （Java Service）(MB)：当前实例的内存占用大小；
       - instance_jvm_memory_heap（堆内存使用）
       - instance_jvm_memory_heap_max（最大堆内存）
       - instance_jvm_memory_noheap（直接内存使用）
       - instance_jvm_memory_noheap_max（最大直接内存）
     - JVM GC Time（ms）：JVM 垃圾回收时间，包含young gc和old gc；
     - JVM GC Count：JVM垃圾回收次数，包含young gc count和old gc count；
     - JVM Thread Count（java service）：当前实例的线程数；
     - JVM Thread State Count (Java Service)：当前实例的各状态线程数；
     - JVM Class Count (Java Service)：当前实例类的计数。

   + Endpoint
     - Endpoint Load in Current Service（CPM / PPM）：当前服务每个端点的每分钟请求数；
     - Slow Endpoints in Current Service（ms）：当前服务每个端点的平均响应时间；
     - Successful Rate in Current Service（%）：当前服务每个端点的请求成功率；
     - Endpoint Load：当前端点每个时间段的请求量；
     - Endpoint Avg Response Time（ms）：当前端点每个时间段的平均请求响应时间；
     - Endpoint Response Time Percentile（ms）：当前端点每个时间段的响应时间占比；
     - Endpoint Successful Rate（%）：当前端点每个时间段的请求成功率；

   >> Database

   - Database Avg Response Time（ms）：当前数据库平均响应时间；
   - Database Access Successful Rate（%）：当前数据库访问成功率；
   - Database Traffic（CPM: Calls Per Minute）：当前数据库每分钟请求数；
   - Database Access Latency Percentile（ms）：当前数据库响应延迟时间的百分比；
   - Slow Statements（ms）：慢查询，按照执行时间排序；
   - All Database Loads（CPM: Calls Per Minute）：所有数据库的请求次数排序；
   - Un-Health Databases (Successful Rate)：所有数据库请求成功率排序。

   >> Event

   >> Istio、K8s

   集成Istio、K8s

   >> SelfObservability

   自监控,OAP服务端的各项指标
   + 修改配置config/application.yml
   ```yaml
      # 将-修改为default
      prometheus-fetcher:
        selector: ${SW_PROMETHEUS_FETCHER:default}
        #default:
        #   active: ${SW_PROMETHEUS_FETCHER_ACTIVE:true}

      # 改none为prometheus
      telemetry:
        selector: ${SW_TELEMETRY:prometheus} 
        prometheus:
          host: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
          port: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
   ```
   + 如果telemetry有调整，则对应修改修改config/fetcher-prom-rules/self.yaml
   ```yaml
      fetcherInterval: PT15S
      fetcherTimeout: PT10S
      metricsPath: /metrics
      staticConfig:
         # 改为上步中telemetry配置的ip
         targets:
            - url: http://localhost:1234 
               sslCaFilePath:
         labels:
            service: oap-server
   ```
   + 检查：curl http://localhost:1234/metrics

   >> VM

   >> Web Browser

   对前端也有一定的监控，通过Skywalking-Client-js组件来操作，包括Web App、Pages两个指标。

> 拓扑图
- 服务选择器 支持显示直接关系，包括上游和下游；
- 自定义组 提供服务组的任意子拓扑功能，但是分组的信息是保存在浏览器内的；
- 服务菜单 当您单击任何服务时打开。该图形可以对所选择的服务进行度量、跟踪和告警查询；
- 服务指标的关系 提供服务RPC交互的度量以及这两个服务的实例。

> 追踪

看每个接口的调用链，每个链路耗时、状态。如果为失败展示错误信息，如果是数据库，会展示查询语句。另外可以根据追踪tid（trace id）和标记（tag）进行筛选。

> 性能剖析

> 日志

> 告警

> 事件

> 调试

### 日志集成

   支持logback、log4j、log4j2日志框架集成，基于gRpc通信协议实现日志采集。

   以logback为例：

   + 引入依赖

```xml

        <dependency>
            <groupId>org.apache.skywalking</groupId>
            <artifactId>apm-toolkit-logback-1.x</artifactId>
            <version>8.9.0</version>
        </dependency>
```
   + 修改logback.xml配置

```xml
      <!-- ... -->

      <appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
         <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
                  <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n</Pattern>
            </layout>
         </encoder>
      </appender>

      <root level="INFO">
         <!-- ... -->
         <appender-ref ref="grpc-log"/>
      </root>
```
   + 修改agent配置 ~/config/agent.config

```properties
   plugin.toolkit.log.grpc.reporter.server_host=${SW_GRPC_LOG_SERVER_HOST:0.0.0.0}
   plugin.toolkit.log.grpc.reporter.server_port=${SW_GRPC_LOG_SERVER_PORT:11800}
   plugin.toolkit.log.grpc.reporter.max_message_size=${SW_GRPC_LOG_MAX_MESSAGE_SIZE:10485760} 
   plugin.toolkit.log.grpc.reporter.upstream_timeout=${SW_GRPC_LOG_GRPC_UPSTREAM_TIMEOUT:30}
```
   + 官方示例

   [https://skywalking.apache.org/docs/skywalking-java/v8.11.0/en/setup/service-agent/java-agent/application-toolkit-logback-1.x/](https://skywalking.apache.org/docs/skywalking-java/v8.11.0/en/setup/service-agent/java-agent/application-toolkit-logback-1.x/)

### 监控方法

   通过@Trace注解标记需要追踪的方法调用情况

   + 引入依赖

```xml
      <dependency>
         <groupId>org.apache.skywalking</groupId>
         <artifactId>apm-toolkit-trace</artifactId>
         <version>8.9.0</version>
      </dependency>
```

   + 修改代码

```Java
   @Trace
   @GetMapping("/printLog")
   public String printLog() {
      Logger.info("traceId:{}",TraceContext.traceId())
      return "ok";
   }
```

### 告警接入

   - 规则  
   修改配置config/alarm-settings.yml

   - 钩子  
   支持WebHook、GRPCHook、SlackHook、WechatHook、DingtalkHook、FeishuHook实现告警信息推送


### 场景

#### 指标性统计

一个服务的 TBS 的正确率、成功率、流量等，这是我们常见的针对单个指标或者某一个数据库的，这就是 Metrics 单指标分析

#### Tracing 分布式追踪

一次请求的范围，也就是我们从浏览器或者手机端发起任何的一次调用，甚至我们可以再推广一点，是一次业务交易，比如说一次订购的过程，从浏览商品到最后下定单、支付、物流、最后交到我们的手上。这是一个流程化的东西，我们需要轨迹，需要去追踪。

#### Logging 日志记录

我们程序在执行的过程中间发生了一些日志，会一帧一帧地跳出来给大家去记录这个东西，这是日志记录。


### 关键词

   * Topology： 拓扑
   * Trace：追踪
   * Metrics：度量
   * Span：
   * Apdex：是根据设定的阈值和响应时间结合考虑的衡量标准。它是满意响应时间和不满意响应时间相对于总响应时间的比率。它衡量的是用户对你的服务的满意程度，因为传统的指标（如平均响应时间）可能很快就会容易形成偏差。
   * percentile：标签含义（p50、p75、p90、p95、p99）：例如p99为1000ms, 这意味着 99% 的请求应该比1000ms更快

### 结束语

  Apache SkyWalking是一款功能强大的APM系统，可以帮助开发人员和运维人员更好地了解分布式系统的性能状况。通过使用SkyWalking，可以提高应用程序的稳定性和性能，降低运维成本。