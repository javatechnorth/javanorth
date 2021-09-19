---
layout: post
title:  CompletableFuture 
tagline: by 揽月中人
categories: Thread
tags:
- 揽月中人
---

completeFuture的复杂人生！ CompleteableFuture 继承自

<!--more-->

### 1 Completable Future是什么

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> 
```

CompletableFuture 实现了Future，那么也就会有Future的全部功能(其实就是5个方法)。

CompletionStage 是JDK1.8新加的一个接口，里面包含了异步计算的各种方法。

不能手动完成计算

调用get()方法会阻塞程序

不能链式执行

整合多个Future执行结果方式笨重

没有异常处理



### 2  Lambda 的几种函数

Runnable

  无参数，无返回值。

Function

  Function<T,R>接受一个参数，并且有返回值 	

Consumer

  Consumer接受一个参数，没有返回值

Supplier

  Supplier没有参数，有一个返回值。

BiConsumer

   BiConsumer<T,U>接受两个参数，没有返回值。



### 3 Completable Future 基本方法介绍

CompleteableFuture实现了CompletionStage，其相关方法比较多。我们分组来介绍。

3.1 主要参数为Runnable的方法

首先我们看两个静态方法。

```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor)
```

runAsync(Runnable runnable) 是一个静态方法，在给定的任务完成后，将使用ForkJoinPool.commonPool() 线程池来异步执行相关任务，并返回CompletableFuture，但是CompletableFuture中没有执行的结果。Runable为参数的方法中返回的CompletableFuture对象都不包含其执行结果。

runAsync(Runnable runnable,Executor executor)也是静态方法，不同的是使用了自定义的线程池。

thenRun(Runnable action) 是CompletableFuture对象的一个方法，表示在调用方即CompletionStage（CompletableFuture实现了CompletionStage，所以也就可以表示任务完成的状态）正常完成的情况下会执行这个动作。

```java

    
public CompletableFuture<Void> thenRun(Runnable action)
public CompletableFuture<Void> thenRunAsync(Runnable action) 
public CompletableFuture<Void> thenRunAsync(Runnable action,Executor executor) 

public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor)
    
public CompletableFuture<Void> runAfterEither(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor)
```



### 4 Completable Future 使用场景及示例

​	



### 总结

本篇为妹子讲了一下CompletableFuture的内容特点以及使用场景，并且列举了一些例子。相信聪明的美女同事已经比较熟悉CompleteableFuture的应用了！