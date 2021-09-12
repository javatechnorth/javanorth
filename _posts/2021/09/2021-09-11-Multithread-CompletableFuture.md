---
layout: post
title:  CompletableFuture 
tagline: by 揽月中人
categories: Thread
tags:
- 揽月中人
---

completeFuture的复杂人生！

<!--more-->

### 1 Future 和 Completable Future

不能手动完成计算

调用get()翻翻噶会阻塞程序

不能链式执行

整合多个Future执行结果方式笨重

没有异常处理

### 2  Lambda 函数

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



### 4 Completable Future 使用场景及示例

​	



### 总结

本篇为妹子讲了一下CompletableFuture的内容特点以及使用场景，并且列举了一些例子。相信聪明的美女同事已经比较熟悉CompleteableFuture的应用了！