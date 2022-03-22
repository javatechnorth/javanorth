---
layout: post
title:  SpringCloudStream集成Kafka
tagline: by 揽月中人
categories: kafka
tags: 
    - 揽月中人
---

哈喽，大家好，我是指北君。  

开发中，服务与服务之间通信通常会用到消息中间件，如果我们使用了某一个MQ，那么消息中间件与我们的系统算是高耦合。将来有一天，要替换成另外的MQ，我们的改动就会比较大。为了解决这个问题，我们可以使用Spring Cloud Stream 来整合我们的消息中间件，降低耦合度，使服务可以更多关注自己的业务逻辑等。

今天为大家带来一个人人可实操的SpringCloudStream集成Kafka的快速入门示例。

<!--more-->

### 1 前言

SpringCloudStream是一个构建高扩展性的事件消息驱动的微服务框架。简单点说就是帮助你操作MQ，可以与底层MQ框架解耦。将来想要替换MQ框架的时候会比较容易。

![查看源图像](https://www.javanorth.cn/assets/images/2022/lyj/springCloudStream1-1.gif)


Kafka是一个分布式发布 - 订阅消息系统，源于LinkedIn的一个项目，2011年成为开源Apache项目。

ZooKeeper 是 Apache 软件基金会的一个软件项目，它为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册，Kafka的实现同时也依赖于zookeeper。

