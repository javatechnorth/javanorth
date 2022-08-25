---
layout: post
title:  手把手教你SpringBoot整合Redis
tagline: by IT可乐
categories: redis
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
本篇文件我们来介绍如何用Springboot整合Redis。

<!--more-->
### 1、Docker 安装 Redis
#### 1.1 下载镜像

```shell
docker pull redis:6.2.6
```

#### 1.2 创建配置文件

```shell
mkdir -p /mydata/redis/conf
touch /mydata/redis/conf/redis.conf
```



#### 1.3 启动Redis

```shell
# 启动 同时 映射到对应文件夹
# 后面 \ 代表换行
docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis:6.2.6 redis-server /etc/redis/redis.conf
```



#### 1.4 进入Redis容器

```shell
docker exec -it redis redis-cli
```

![](http://www.javanorth.cn/assets/images/2021/itcore/docker-redis-01.png)

注意：新版本redis6.0 默认开启了混合持久化，重启之后依然可以看到重启之前插入的数据。

配置文件地址如下：

> https://raw.githubusercontent.com/redis/redis/6.2/redis.conf

![](http://www.javanorth.cn/assets/images/2021/itcore/docker-redis-config.png)



#### 1.5 redis 可视化工具

> https://github.com/uglide/RedisDesktopManager

下载并安装，然后连接到我们安装的 Redis，可以看到我们插入的数据。

![](http://www.javanorth.cn/assets/images/2021/itcore/docker-redis-desktop-02.png)

### 2、SpringBoot 整合Redis缓存

#### 2.1 安装Redis

之前已经通过 docker 安装好了 Redis。

![](http://www.javanorth.cn/assets/images/2021/itcore/redis-02.png)



#### 2.2 引入依赖

pom.xml

```xml
<!-- 引入redis -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



#### 2.3 配置Redis地址端口

application.yml

```yml
spring:
  redis:
    host: 192.168.88.14
    port: 6379
```



#### 2.4 测试

```java
@Autowired
StringRedisTemplate stringRedisTemplate;
@Test
public void testStringRedisTemplate() {
    stringRedisTemplate.opsForValue().set("hello","world_" + UUID.randomUUID().toString());
    String hello = stringRedisTemplate.opsForValue().get("hello");
    System.out.println("保存的数据是：" + hello);
}
```



![](http://www.javanorth.cn/assets/images/2021/itcore/redis-03.png)









