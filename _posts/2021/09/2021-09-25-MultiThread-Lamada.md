---
layout: post
title: 软软猿妹问我JDK中眼花缭乱的Function/Consumer/Supplier/Predicate？
tagline: by 揽月中人
categories: Algorithm
tags:
- 揽月中人
---

JDK中有许多函数式接口，也会有许多方法会使用函数式接口作为参数，同时在各种源码中也大量使用了这些方法，那么我们在实际工作中应该如何使用！我们就来盘一盘，这样也有助于写出优雅的代码，使我们在阅读源码时事半功倍。

<!--more-->

### 1 JDK中的Lamada表达式

 Runnable

 无参数，无返回值。

Function

  Function<T,R>接受一个参数T，并且有返回值 R	

Consumer

  Consumer接受一个参数，没有返回值

Supplier

  Supplier没有参数，有一个返回值。

BiConsumer

   BiConsumer<T,U>接受两个参数，没有返回值。

### 2 常用的Lamada参数特征
### 3 自定义Lamada函数式接口
##### 

### 总结