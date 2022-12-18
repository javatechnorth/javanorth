---
layout: post
title:  connection pool setting 干货必须收藏！-20221216
tagline: by 揽月中人
categories: 性能优化
tags:
- 揽月中人
---

![](http://www.javanorth.cn/assets/images/2021/lyj/huahuizhou.jpg)

如果很多资源的使用如果不从共享资源池中里面获取，极容易造成内存泄漏和内存溢出。要想实现高并发并且合理利用资源，大部分设计方案都会用到各种连接池，线程池等等。所有的可重复利用资源均从一组资源池中进行调用。也类似于近几年火爆的共享经济，然而共享经济就和软件设计中的共享资源池类似。不单独持有某个资源，在需要使用的时候再去资源池中进行申请。

下面我们盘一盘各种资源共享池的一些配置，以及优化策略！

<!--more-->



### 1 Tomcat中的各种connection

废话少说，我们看一下一个简单的SpringBoot tomcat配置

```yaml
server:
  tomcat:
    accept-count: 500 //accept队列长度
    max-connections: 1000//最大连接数
    threads:
      max: 200 //最大工作线程数量
      min-spare: 10 //最小工作线程数量
```

HTTP Connector 

其工作流程如下：

1. 每个非异步请求都需要一个线程来处理，如果并发请求大于当前可处理的线程数量，则会创建额外的线程来处理，至多创建到maxThreads 的数量。
2. 此时仍然接收到更多的并发请求，Tomcat会接受新的connection，直到connection数到达最大数maxConnections。此时这些connection会在Connector中创建的 server socket中排队，直到有线程可以来处理这些connection。
3. 一旦上面的排队数量达到maxC onnections，然后还有新的请求进来，那么新进来的connection会在OS中排队，操作系统提供的排队数量为acceptCount。如果这个队列满了的话，后面进来的请求有可能被拒绝或者超时timeout

关于这个咱们讲一个食堂干饭的例子： 

![img](http://www.javanorth.cn/assets/images/2021/lyj/school-canteen.jpg)

- 某学校有一个食堂，大厅里面日常至少摆100把椅子（min-spare）供学生们吃饭。

- 然而当同时吃饭的同学大于100人的时候，食堂会增加一些椅子（创建线程），并且这些椅子也不会立马收回去，一段时间没有人使用才会收回。
- 但是食堂里面最多可以摆500把椅子（maxThreads）。 然后超过500人吃饭同时吃饭的话，其他人就只能在大厅里面排队等别人吃完。食堂大厅里面可以容纳1000人进行排队等候（maxConnections）。 
- 当食堂大厅1000人都排满了，那么就只能到食堂外面排队了，外面排队最多一直能排200人（acceptCount）。 这个时候如果再有人过来要吃饭，而且还排不上队，就会等到不耐烦（time out），也会有人来告诉后来的同学，别来了人都满了，上其他地方吃饭去吧。（reject）

通过上面的例子，我相信大家都能清楚tomcat的一些基本参数配置作用，并且针对不同的情况进行调优了。

### 2 ThreadPool

关于Java线程池，大家都比较熟悉了吧。下面是基本参数

```java
public ThreadPoolExecutor(
    int corePoolSize, //核心线程数
    int maximumPoolSize,//最大线程数
    long keepAliveTime, //大于核心线程数量的线程存活时间，如果没有新任务就会关闭
    TimeUnit unit, // 时间单位
    BlockingQueue<Runnable> workQueue, //线程等待队列
    ThreadFactory threadFactory,//创建线程的工厂
    RejectedExecutionHandler handler//拒绝策略
) {
```

线程池基本运行原理介绍

1. 提交任务给线程池后，线程池会检查线程池中正在运行的线程数量，如果线程数量小于核心线程，则创建一个新的线程来处理任务。
2. 如果线程池中线程数量达到和corePoolSize的大小，则将线程放入等待队列BlockingQueue中。
3. 如果提交任务时连等待队列都已经满了的话，线程池会继续创建新的线程来处理任务，直到线程池数量达到maximumPoolSize。
4. 如果线程数量达到了最大容量，则会执行拒绝策略。



这里线程池的方案和tomcat Connector 的方案稍微有点不同。前者是先排队然后再把池子容量扩大代最大，后者是先扩大池子，然后再排2个队。

我觉得对于ThreadPoolExecutor线程池的理解，用工厂工人的例子比较好理解。

- 有一家工厂建立，开始的时候只有10个工人，然后工厂的活越来越多，招聘新的工人肯定不是最好的策略，所以多出来的活暂时只能等着，进行排队。（这个例子中工厂的活多了，立马去招人肯定是不可能，只能先排单）
- 后面工厂的业务越来越多，任务挤压过多，原来的工人干活已经不能满足业务需求了。为了最大化效益，招聘新的工人势在必行，于是就招聘了新的工人，所有的工人一起来干活，加快效率。
- 当工厂的工人数量达到饱和之后，仍然不停的新增业务，此时工厂已经饱和，没有办法再继续接单。那么只能采取别的方案（拒绝策略），找别的工厂干，或者新建工厂。
- 当后面业务量比较小的时候，新招的工人就会慢慢的裁剪（线程一段时间不使用就会关掉！）。



对线程池的优化思路：

- 如果线程需要执行的任务耗时比较少，是High CPU类型，则核心线程数量可以根据CPU的核数来进行设置。最大线程数量也不应该设置的太大。线程队列可以根据使用场景设置大一点，提高线程池效率。
- 如果线程需要执行的任务耗时比较长，是High IO型，依赖其他系统，CPU需要等待的时间比较长，则核心线程数可以大一点，相应的线程队列长度也应该针对不同的使用场景进行调整。
- 线程数量也不宜设置过大，不然会导致频繁的GC。

### 3 RestTemplate的坑与优化

SpringBoot微服务与其他Restful的资源进行交互的时候会使用到RestTemplate。如果你直接new RestTemplate，那么就需要特别注意了。使用不慎就会造成内存泄漏，引发GC等。

RestTemplate底层依旧是使用org.apache.http包下的HttpClient。

SpringBoot中可以通过PoolingHttpClientConnectionManager设置一些connection pool 的参数

```java
PoolingHttpClientConnectionManager connectionPoolManager = new PoolingHttpClientConnectionManager();
connectionPoolManager.setMaxTotal(100);//最大连接数
connectionPoolManager.setDefaultMaxPerRoute(200);//
```

通过HttpRequestFactory可以设置connectTimeOut，connectionRequestTimeout，SocketTimeout

```java
HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        httpRequestFactory.setConnectionRequestTimeout(3000);//获取链接超时时间
        httpRequestFactory.setConnectTimeout(3000);// 指客户端和服务器建立连接的timeout
        httpRequestFactory.setReadTimeout(120000);// 读取数据的超时时间
```



小结一下比较重要的几个参数如下：

maxTotal : 连接池里面的最大连接数

defaultMaxPerRoute  : 每个路由默认接收的最大连接数

socketTimeout ：它是指客户端和服务器建立连接后，客户端从服务器读取数据的超时时间，超出后会抛出SocketTimeOutException。

connectionRequestTimout：指从连接池获取连接的timeout

connetionTimeout：指客户端和服务器建立连接的timeout。



可以通过如下方式构建RestTemplate，其中的参数也可以自定以从配置文件中引入。

```java
@Bean
public  RestTemplate buildPoolingRestTemplate(RestTemplateBuilder builder){
    PoolingHttpClientConnectionManager connectionPoolManager = new PoolingHttpClientConnectionManager();
    connectionPoolManager.setMaxTotal(100);//最大连接数
    connectionPoolManager.setDefaultMaxPerRoute(200);//每个路由默认接收的最大连接数

    HttpClient httpClient = HttpClientBuilder.create()
            .setConnectionManager(connectionPoolManager).build();

    HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
    httpRequestFactory.setHttpClient(httpClient);
    httpRequestFactory.setConnectionRequestTimeout(3000);//获取链接超时时间
    httpRequestFactory.setConnectTimeout(3000);// 指客户端和服务器建立连接的timeout
    httpRequestFactory.setReadTimeout(120000);// socketTimeout 读取数据的超时时间
    
    return builder.requestFactory(()-> httpRequestFactory).build();
}
```



对于RestTemplate的一些建议

- 应该从资源池中获取RestTemplate（PoolingHttpClientConnectionManager）
- 使用RestTemplateBuilder来创建RestTemplate
- 针对maxTotal ，defaultMaxPerRoute  ，可以增大maxTotal以增大并发量，同时也需要调整每个路由的最大并发连接数，此时也可以提高某条路由的并发量。
- connectionRequestTimeout和connectTimeout设置不要太长，socketTimeout根据需求可以设置相应的时间。
- 当然还有其他的一些优化的地方，比如使用不同的ConnectionKeepAliveStrategy等，设置maxIdleTime最大空闲时间等。




### 总结

本篇总结了Tomcat，线程池，RestTemplate 的一些日常优化策略。平时应该多注意总结，在不同的情况下，优化参数均有不同。所以就要多一些测试，才能得到最好的配置。看完这些不妨在项目中试一下，增墙记忆。如果还有什么想法，不妨在评论区留言。

