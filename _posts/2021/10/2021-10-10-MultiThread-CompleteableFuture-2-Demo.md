---
layout: post
title:  软软猿妹还要看CompletableFuture的各种Demo才肯放过我！
tagline: by 揽月中人
categories: Thread
tags:
- 揽月中人
---

上面一篇介绍了一下CompletableFuture的各种方法，软软猿妹还想要看更多Demo示例，那么今天就安排起来！
<!--more-->

### 1 CompletableFuture的静态方法使用

CompleteableFuture的静态方法有如下，

![image-20211024112659806](E:\javaNorth\javanorth\assets\images\2021\lyj\completableFutureStaticMethod.png)

之前的文章里面已经讲过suuplyAsync，以及runAsync。

#### delayedExcutor

delayedExcutor其作用是构建一个延迟执行任务的Excutor，默认使用ForkJoinPool. 也可以使用自定义的Excutor。

```java
一个延迟5秒执行任务的Excutor，默认使用使用ForkJoinPool.commonPool()。
Executor executor = CompletableFuture.delayedExecutor(5l, TimeUnit.SECONDS);
```



#### allof和anyof

allof和anyof 为等待多个CompletableFuture完成之后返回一个CompletableFuture，allof返回无result，anyof返回为最先完成的CompletableFuture。可以看如下示例。

```java
CompletableFuture<String> supplyAsync1 = CompletableFuture.supplyAsync(()->{
    try {Thread.sleep(4 * 1000);} catch (InterruptedException e) {e.printStackTrace();}
    return "supplyAsync1";
});
CompletableFuture<String> supplyAsync2 = CompletableFuture.supplyAsync(() -> {
    try {Thread.sleep(2 * 1000);} catch (InterruptedException e) {e.printStackTrace();}
    return "supplyAsync2";
});
CompletableFuture.anyOf(supplyAsync1,supplyAsync2).thenAccept((str)->{
    System.out.println(LocalDateTime.now() + " anyOf complete : " + str);
});
CompletableFuture.allOf(supplyAsync1,supplyAsync2).thenAccept((str)->{
    System.out.println(LocalDateTime.now() + " allOf complete "+ str );
});
```

执行结果如下:

```java
start second: 2021-10-24T12:39:40.562001600
2021-10-24T12:39:42.611118800 anyOf complete : supplyAsync2
2021-10-24T12:39:44.611233200 allOf complete null
```

failedStage和failedFuture是返回一个已知异常的CompletableFuture。这个下面和其他异常一起举例。

### 2 CompletableFuture的其余方法使用

CompletableFuture中其余方法可以大致分为run，apply，accept几个类别。 其对应的参数分别为Runnable，Function，Consummer等几个函数式表达式。

- run代表当前CompletableFuture完成后执行的一些列操作，无输入参数，无返回结果，所以只是Runnable为参数。()-> { option } 

- apply代表以当前CompletableFuture完成后的结果为参数进行的操作，并且会返回一个新的CompletableFuture，所以以Function为参数。(s)-> {return s;} 

- accept代表以当前CompletableFuture完成后的结果为参数，执行的操作，无返回结果，直接消费。以consumer为参数，(s)-> { option }。

run

将两个 CompletableFuture组合在一起(链式执行)

​	先去取快递，然后再去买菜，然后回家做饭。



在两个CompletableFuture运行后再次计算

​	和女朋友一起出去，我去取快递，女朋友去买菜，然后一起回家做饭。



​	这种场景适用于当A和B通过异步取到值后，使用A、B计算出C，比如，获取两个用户的爱好交集，首先需要获取A用户的爱好，接着获取B用户的爱好，然后在求交集。



将多个 CompletableFuture 组合在一起

​	两个人做饭，一个人打鸡蛋，一个人切西红柿，然后一起做成西红柿炒鸡蛋

​	女朋友打鸡蛋把手磕了，炒菜停止，叫外卖。



等待所有的程序都走完。

​	allof();



### 4 CompletableFuture的异常处理



##### whenComplete

handle 

exceptionally





### 总结