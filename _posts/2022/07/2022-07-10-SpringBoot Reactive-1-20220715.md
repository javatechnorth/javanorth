---
layout: post
title: 从Functional到Stream, 再到Reactor,WebFlux-已发
tagline: by 揽月中人
categories: Reactive
tags:
- 揽月中人
---

哈喽，大家好，我是指北君。  

相信响应式编程经常会在各种地方被提到。本篇就为大家从函数式编程一直到Spring WeFlux做一次简单的讲解，并给出一些示例，希望大家可以更好的理解响应式编程，可以在合适的时机运用到实际项目中。

<!--more-->

### 1. 前言

了解响应式编程，首先我们需要了解函数式操作和Stream的操作，下面我们简单的复习一下喽。

#### 1.1 常用函数式编程

函数式接口中

我们先来回顾一下Java中的函数式接口。常见的有以下几种

- Consumer 一个输入，无输出
- Supplier  无输入，有输出
- Function<T,R>  输入T，输出R
- BiFunction<T,U,R> 输入T,U 输出R
- Predicate  有输入，输出boolean类型

上面的简单函数式接口示例如下：

```java
Consumer consumer = (i)-> System.out.println("this is " + i);
consumer.accept("consumer");

Supplier supplier  = () -> "this is supplier";
System.out.println(supplier.get());

Function<Integer,Integer> function = (i) -> i*i;
System.out.println(function.apply(8));

BiFunction<Integer,Integer,String> biFunction = (i,j)-> i+"*"+j+"="+i*j;
System.out.println(biFunction.apply(8,8));

Predicate<Integer> predicate = (i) -> i.intValue()>3;
System.out.println(predicate.test(5));
```

其执行结果如下：

```java
this is consumer
this is supplier
64
8*8=64
true
```



#### 1.2 Stream操作

对Stream进行操作，主要有几个关键点：

- **生成流**
- **流的中间操作**  其中中间操作可以有多个，中间操作会返回一个新的流（如 map ,filter,sorted等），然后交给下一个流方法使用。
- **流的终结操作**  终结操作只有一个。终结操作执行后，流就到了终止状态，无法被操作 （如forEach，toArray , findFirst 等）。

创建流的示例：

```java
String[] strArray = {"ss","ss","","sdffg"};

Arrays.stream(strArray).forEach(System.out::println);
Arrays.asList(strArray).stream().forEach(System.out::println);
Stream.of(strArray).forEach(System.out::println);
Stream.iterate(1,(i) -> i+1).limit(10).forEach(System.out::println);
Stream.generate(() -> new Random().nextInt(10)).limit(10).forEach(System.out::println);
```



简单的流处理示例：

```java
String[] strArray1 = {"ss","ss","","sdffg","bca-de","fff"};
String collect = Stream.of(strArray1)
        .filter(i -> !i.isEmpty())//过滤空字符串
        .sorted() //排序
        .limit(1) //只取第一个元素
        .map(i -> i.replace("-", ""))//替换 "-"
        .flatMap(i -> Stream.of(i.split("")))//将字符拆成字符数组
        .sorted() //排序
        .collect(Collectors.joining());//将字符拼接组合到一起
System.out.println(collect);//最后输出abcde
```



### 2. Java响应式编程

响应式编程会用到一个发布者和一个订阅者，然后通过订阅关系完成数据流的传输。订阅关系中可以处理一些背压问题，即调节消费者与生产者之间的供需平衡，让整个程序达到最大效率。

![image-20220712010907090](https://www.javanorth.cn/assets/images/2022/lyj/reactor-01.png)

Java9中java.util.concurrent.Flow接口提供响应式流编程类似的功能。

下面我们实现一个基于Java 响应式编程的示例：

其中有三个简单步骤：

1. 建立生产者
2. 构建消费者
3. 消费者订阅生产者
4. 生产者生产内容

```java
SubmissionPublisher publisher = new SubmissionPublisher<>();//建立生产者
Flow.Subscriber subscriber = new Flow.Subscriber() {...};//建立消费者 (其中的实现放到下面)
publisher.subscribe(subscriber);//订阅关系
for (int i = 0; i < 10; i++) {
	publisher.submit("test reactive java : " +i); //生产者生产内容
}


```



消费者全部代码如下：

```java
Flow.Subscriber subscriber = new Flow.Subscriber() {
    Flow.Subscription subscription;
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        System.out.println("Subscription establish first ");
        this.subscription = subscription;
        this.subscription.request(1);
    }
    @Override
    public void onNext(Object item) {
        subscription.request(10);
        System.out.println("receive :  "+ item);
    }
    @Override
    public void onError(Throwable throwable) {
        System.out.println(" onError ");
    }
    @Override
    public void onComplete() {
        System.out.println(" onComplete ");
    }
};
```

其中onSubscribe方法表示建立订阅关系

onNext接受数据，并请求生产者的数据。

onError，onComplete则是error或者完成之后的处理方法。



#### 带有中间处理器的响应式流

Reactive Stream 通常会基于如下的模型：

![image-20220712013527086](https://www.javanorth.cn/assets/images/2022/lyj/reactor-02.png)

下面我们实现一个带有中间处理功能的响应式模型：

下面的Processor 既有发布者，又有订阅者：

```java
public class ReactiveProcessor extends SubmissionPublisher implements Flow.Subscriber {
    private Flow.Subscription subscription;
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        System.out.println( Thread.currentThread().getName() +  " Reactive processor establish connection ");
        this.subscription = subscription;
        this.subscription.request(1);
    }

    @Override
    public void onNext(Object item) {
        System.out.println(Thread.currentThread().getName() + " Reactive processor receive data: "+ item);
        this.submit(item.toString().toUpperCase());
        this.subscription.request(1);
    }

    @Override
    public void onError(Throwable throwable) {
        System.out.println("Reactive processor error ");
        throwable.printStackTrace();
        this.subscription.cancel();
    }

    @Override
    public void onComplete() {
        System.out.println(Thread.currentThread().getName() + " Reactive processor receive data complete ");
    }
}
```



如上中间处理器订阅发布者， 同时消费者再订阅中间处理器。中间处理器也可以调节发布订阅的生产消费速率。

```java
SubmissionPublisher publisher = new SubmissionPublisher<>(); //创建生产者
ReactiveProcessor reactiveProcessor = new ReactiveProcessor(); // 创建中间处理器
publisher.subscribe(reactiveProcessor);	//中间处理器订阅生产者
Flow.Subscriber subscriber = new Flow.Subscriber() {...}; //创建消费者
reactiveProcessor.subscribe(subscriber); //消费者订阅中间处理器
for (int i = 0; i < 10; i++) {
    publisher.submit("test reactive java : " +i); //生产者生产数据
}
```

通过上述生产者-> 中间处理器->消费者， 可以将生产者生产的数据全部变成大写，然后再发送给最终的消费者。

以上式Java中的reactive 编程示例。Java会不同线程来分别处理消费者与生产者的消息处理

### 3. Reactor 

Reactor中两个比较关键的对象式Flux和Mono， 整个Spring的响应式编程均式基于projectreactor项目。Reactor是响应式编程的依赖，主要是基于JVM构建非阻塞程序。

根据Reactor的介绍，此类响应式编程的的三方库（Reactor）主要是解决一些JVM经典异步编程中的一些缺点，并且还可以专注于一些新的特性，如下：

- 可组合性与可读性 （Composability and readability）
- 可以使用丰富的运算操作符将数据作为流进行操作
- 订阅之前，不会有任何事
- 背压特性（Backpressure ），可以理解为消费者可以向生产者发送产出率过高的信号，从而调整生产速率。或者消费者可以选择一次性拉去一捆数据进行消费。
- 于并发无关的高度抽象的高级功能

其中有这么一段解释，可以形象的说明响应式编程。

Reactive的程序可以想象成车间的流水线，reactor既是流水线上的传送带，又是处理工作站。原料从一个原始的生产者出发，最终成为产品被推总给消费者。

#### 3.1 Flux & Mono

下面我们介绍一下Flux和Mono。

在Reactor中Flux和Mono均是Publisher，即生产者。 两者也有不同。Flux对象表示0到N个异步的响应序列，而Mono只代表0个（empty）或者1个结果。

Reactor官网上介绍的Flux示意如下：

![image-20220712235425387](https://www.javanorth.cn/assets/images/2022/lyj/reactor-03.png)

 

Mono示意如下：

![image-20220712235920120](https://www.javanorth.cn/assets/images/2022/lyj/reactor-04.png)

#### 3.2 Flux Mono创建与使用

我们也可以单独引用其依赖。

使用maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId> 
    </dependency>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId> 
        <scope>test</scope>
    </dependency>
</dependencies>
```



##### Mono创建

分别创建空Mono和一个包含一个String的Mono，并由消费者消费打印。

```java
Mono.empty().subscribe(System.out::println);
Mono.just("Hello Mono Java North").subscribe(System.out::print);
```

##### Flux创建

Flux创建有如下的一些方法，

- just（通过不定参数创建）

- range（从某个整数开始，往后的整数数量）

- fromArray，fromIterable，fromStream，从名称上就可以看出来，通过数组，迭代器，Stream流创建Flux

下面式一些Java代码示例

```java
Flux.just(1,2,3,4,5).subscribe(System.out::print);
Flux.range(1,20).subscribe(System.out::print);
Flux.fromArray(new String[]{"a1","a2","a3","a4","a5","a6"}).skip(2).subscribe(System.out::print);
Flux.fromIterable(Arrays.asList(1,2,3,4,5,6,7)).subscribe(System.out::println);
Flux.fromStream(Stream.of(Arrays.asList(1,2,3,4,5,6,7))).subscribe(System.out::print);
```



我们再举一个generate的例子

```java
public static <T, S> Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator)
```

如上代码所示，generate需要一个Callable参数，而且是supplier （即没有输入值，只有一个输出）

另一个参数是BiFunction （前面我们也介绍过，需要两个输入值，一个输出值）。 BiFunction中的其中一个输入值是SynchronousSink，下面我们给出一个generate创建Flux的示例。

```java
Flux.generate(
	() -> 0, //提供一个初始状态值0
	(i, sink) -> {
    sink.next("3*" + i + "=" + 3 * i);//使用初始值去生产一个3的乘法
    if (i > 9) sink.complete();//设置停止条件
    return i + 1;//返回一个新的状态值，以便在下一次的生产中使用，除非响应序列终止
}).subscribe(System.out::println);
```



下面我们在看一个Flux嵌套处理示例：

需求：将字符串去空格，并去重，然后排序输出。

```java
String str = "qa ws ed rf tg yh uj i k ol p za sx dc vf bg hn jm k loi yt ";
Flux.fromArray(str.split(" "))//通过数组创建Flux
    .flatMap(i -> Flux.fromArray(i.split(""))) 
    .distinct() // 去重
    .sort()	//排序
    .subscribe(System.out::print); 
    //flatMap与Stream中的flatMap类似，接受Function作为参数，输入一个值，输出一个值，此处输出均为Publisher，
```



以上就是Flux和Mono的一些简单介绍，同时Ractor也支持JDK中的FlowPubliser 和FlowSubscriber与 Reactor中的publisher, subscriber的适配等.



### 4. WebFlux

SpringBoot 2之后支持的Reactive响应式编程。

关于Reactive技术栈和经典的Servlet技术栈对比，Spring官网的这张图比较清晰。

![image-20220713015250078](https://www.javanorth.cn/assets/images/2022/lyj/reactor-05.png)

Spring响应式编程主要依赖于Reactor第三方库，即上面讲的Flux和Mono的库。

WebFlux主要有以下几个要点：

- 反应式栈web框架
- 完全异步非阻塞
- 运行在netty，undertow，Servlet3.1 + 容器
- 核心反应式库 Reactor
- 返回 Flux 或Mono
- 支持注解和函数编程两种编程模式



#### Spring WebFlux示例

下面我们给出几个SpringBoot 的响应式web示例。

可以去https://start.spring.io/ 新建webflux的项目也可以。

项目中的主要依赖就是spring-boot-starter-webflux

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webflux</artifactId>
		</dependency>
```



##### 基于注解的WebFlux：

以下是一个最简单的基于注解的WebFlux

```java
@GetMapping("/hello/mono1")
public Mono<String> mono(){
    return Mono.just("Hello Mono -  Java North");
}

@GetMapping("/hello/flux1")
public Flux<String> flux(){
    return Flux.just("Hello Flux","Hello Java North");
}
```



##### 基于函数式编程的WebFlux:

创建RouterFunction，将其注入到Spring中即可。

```java
@Bean
public RouterFunction<ServerResponse> testRoutes1() {
    return RouterFunctions.route().GET("/flux/function", new HandlerFunction<ServerResponse>() {
        @Override
        public Mono<ServerResponse> handle(ServerRequest request) {
            return ServerResponse.ok().bodyValue("hello web flux , Hello Java North");
        }
    }).build();
}

//上面的方法使用函数式编程替换之后如下
@Bean
public RouterFunction<ServerResponse> testRoutes() {
    return RouterFunctions.route().GET("/flux/function",
         request -> ServerResponse.ok()
                    .bodyValue("Hello web flux , Hello Java North")).build();
}
```



##### Flux与Mono的响应式编程延迟示例

下面我们编写一段返回Mono的响应式Web服务。

```java
@GetMapping("/hello/mono")
public Mono<String> stringMono(){
    Mono<String> from = Mono.fromSupplier(() -> {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "Hello, Spring Reactive  date time:"+ LocalDateTime.now();
    });
    System.out.println( "thread : " + Thread.currentThread().getName()+ " ===  " + LocalDateTime.now() +"  ==========Mono function complete==========");
    return from;
}
```



使用postman请求如下， 5秒钟后返回数据。后台却在5秒中之前已经处理完整个方法。

![image-20220713022916653](https://www.javanorth.cn/assets/images/2022/lyj/reactor-06.png)



后台打印日志：

![image-20220713023200713](https://www.javanorth.cn/assets/images/2022/lyj/reactor-07.png)





再看一组Flux

```java
@GetMapping(value = "/hello/flux", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> flux1(){
    Flux<String> stringFlux = Flux.fromStream(IntStream.range(1,6).mapToObj(i ->{
        mySleep(1);//表示睡1秒
        return "java north flux" + i + "date time: " +LocalDateTime.now();
    }));
    System.out.println("thread : " + Thread.currentThread().getName()+ " ===  " + LocalDateTime.now() + "  ==========Flux function complete=========");
    return stringFlux;
}
```



此次使用谷歌浏览器请求此服务：

可以发现每隔一秒就会有一条消息被生产出来。

![image-20220713023709767](https://www.javanorth.cn/assets/images/2022/lyj/reactor-08.png)

后台完成时间同样是在一开始就完成整个方法：

![image-20220713023615203](https://www.javanorth.cn/assets/images/2022/lyj/reactor-09.png)



通过上述对Flux 与 Mono的例子，可以好好体会一下响应式编程。



### 总结

本篇回顾了函数式编程，Stream操作等，然后再举例讲了Java中的Reactive编程示例， 同时也给处理Reactor三方库的Flux于Mono的示例。

最后使用了SpringBoot WebFlux 创建简单的响应式web服务。希望能让大家更好的理解响应式编程。

