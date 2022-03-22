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

### 2 Windows搭建简单的Kafka

##### 2.1 启动zookeeper

使用Kafka首先需要启动zookeeper，windows中搭建zookeeper也很简单。以下几步即可完成：

1. 下载zookeeper （本文使用3.7.0版本，下载链接在文章末尾。）
2. 配置基本环境变量：
   1. 将conf文件夹下面的 zoo_sample.cfg 重命名zoo.cfg。并修改其工作目录dataDir。
   2. bin文件夹下面有zkEnv.cmd有zookeeper相关的配置，其中就包括JAVA_HOME，所以系统环境变量需要配置JAVA_HOME，或者直接用Java的路径来替换。
3. 启动，在bin目录下运行zkServer.cmd脚本启动zookeeper。

默认启动端口2181为。

正常启动如下：

![image-20220321004222780](https://www.javanorth.cn/assets/images/2022/lyj/springCloudStream1-2.gif)

##### 2.2 搭建Kafka

本地使用kafka同样也是如下的几个步骤：

1. 下载Kafka（本文使用2.11版本，下载链接见文章末尾）
2. 环境变量配置：

   - 查看config文件下面的 server.properties配置文件中的zookeeper的配置

      ```properties
      zookeeper.connect=localhost:2181
      ```
   - 在bin/windows文件夹下面kafka-run-class.bat文件中有JAVA_HOME的配置，同样也可以直接改成系统的Java路径.

3. 在kafka根目录下使用如下命令启动kafka，并在zookeeper中注册。
```shell
# .\bin\windows\kafka-server-start.bat .\config\server.properties
```
4. 创建topic，在bin\windows目录下使用如下命令。创建名称为“test”的topic

```shell
kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

5. 使用windows命令窗口的producer和consumer，在bin\windows目录下使用如下命令

```shell
#test topic的消息生产者
kafka-console-producer.bat --broker-list localhost:9092 --topic test
#test topic的消息消费者
kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test
#test topic的消息消费者（从头消费）
kafka-console-consumer.bat --bootstrap-server localhost:9092 --from-beginning --topic 
```