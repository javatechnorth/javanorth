---
layout: post
title:  2022-12-18 Guava RateLimiter
tagline: by 沉浮
categories: Guava RateLimiter
tags: 沉浮
---

哈喽，大家好，我是指北君。  

## Guava RateLimiter

*有没有搞错，别人都在提升系统的访问并发量，你却在这搞限制？*

我们都知道，服务器资源是有限的，当把应用部署在外网环境中，所有人都可以访问你的应用，如果访问人数上去了，你的服务器是否能够支持足够量的用户访问？在系统访问高峰时期，
仅从代码层面提供系统并发量，系统真的就能够支持突然流量的冲击？显然是不可能的，如果谁让你在不改变硬件配置的情况下，无限制的提高系统性能，你可以说他在百日做梦。

### 简介

  **限流器**
  顾名思义，就是对流量的限制，准确的说应该是流量控制，当然并不是无理由的进行流量控制，应该是在计算机硬件能够承载的范围内，防止系统突然流量过高导致系统资源耗尽，最终
系统宕机或崩溃，使得服务器上的应用全部挂掉。限流器是在保证应用能够正常提供服务的前提下，通过流量控制实现对服务的一种保护手段。

  当然流量的阈值到底是多少比较合适，这个可能需要根据实际硬件配置、系统环境以前其他相关参数经过各种测试与验证才能知道...

  本篇文章仅讨论限流中相关的技术，在实际应用中使用的限流器，除了包含流量限制的作用，为了提高用户体验，还需要对流量超出是，做出对应的应对策略，比如直接拒绝服务，让请求进行排队
  ，或者服务降级都是比较好的处理手段，这样既能给用户友好地体验，又能保证服务正常。

### 核心

有几个核心概念需要先了解到：
+ 限流的目标对象：请求数量、网络流量、用户访问次数...
+ 限流的维度：时间、IP、用户
+ 限流的实现层面：前端页面、WEB代理、服务接口...

### 应用场景

1. 防止服务接口短时间涌入大量请求导致服务器资源快速耗尽最终服务无法访问
2. 突发流量时通过排队策略实现流量削峰，杜绝对服务器的冲击
3. 针对ip、或用户控制对某个资源的访问次数
4. 针对高频接口，控制单位时间内允许的请求次数
5. 结合ip或其他因子防爬虫

### 限流方法

这里我们主要讨论后端基于请求量的限流，限流是一种非常广泛的应用技术，就比如你在登录系统时，经常会需要你输入手机验证、动态码或一些奇奇怪怪的验证方式， 来降低登录请求的频次。

+ 计数限流
  
按数量进行控制，达到设置的阈值则进行限流，其中*固定窗口*，*滑动窗口*则是通过该方法实现。

+ 固定窗口

通过控制**时间单元**内允许的**请求数量**，一旦达到阈值，则不会处理该请求后续相关的业务或者直接让请求快速失败并给予提示。
  ![FixedWindowLimiter](/assets/images/2022/sucls/12_18/fixedWindowLimiter.png)

<div style="background-color: antiquewhite">
  比如我们配置10s内允许请求的流量为1000，在第1~9s内请求为0，在第9~10秒内的请求数为1000，这样一秒内的请求就达到了1000。当然我们可以时间单元划分成更小粒度，
但是应该多小才合适呢？
</div>

  _问题：只能对时间单元内的总请求数进行控制，当请求集中在较小时间范围内时，无法达到流量限制的效果，因此这是一种粗粒度的流量限制手段_

+ 滑动窗口

为了解决固定窗口算法中存在的问题，通过滑动窗口的方法，将上述时间单元划分成多个细粒度的时间窗口，每个窗口都有自己独立的请求计数器，这样就可以让时间单元内的流量控制均匀地
落在各个时间窗口上，同时滑动的时间窗口可以形成连续时间区间控制，并不像固定窗口那样只在两个时间刻度间。
![sliding-window](/assets/images/2022/sucls/12_18/sliding-window.jpeg)
<div style="background-color: antiquewhite">
  比如时间单元为1s，每个时间窗口为100ms，在1秒内的10个时间窗口可以为09:01:01.000~09:01:02.000、09:01:01.200~09:01:02.800...
</div>

_问题：滑动窗口的区间划分的越多，则滑动窗口的滚动就越平滑，限流的统计就会越精确，但也需要更多的资源为窗口时间片段保存计数器，从而耗费系统资源_

+ 漏桶算法

如果将请求看成水滴，限流器看成一个下面开口的桶（漏桶）。漏桶算法其实就是当水滴（请求）先进入到漏桶里，漏桶以一定的速度出水，当水流入速度过大时则会超过桶的可接纳容量，
这时水将直接溢出，漏桶算法能强行限制数据的传输速率。
使用漏桶算法，可以保证接口会以一个常速速率来处理请求，所以漏桶算法必定不会出现临界问题。
![leaky-bucket](/assets/images/2022/sucls/12_18/leaky-bucket.png)

_问题：当短时间内如果有大量的突发请求时，即使服务器负载不高，每个请求也需要等待一段时间（水滴间隔）才能被响应_

+ 令牌桶算法

令牌桶算法会以一个恒定的速度往桶里放入令牌，而如果请求需要被处理，则需要先从桶里获取一个令牌，当桶里没有令牌可取时，则拒绝服务。
相比“漏桶算法”，“令牌桶算法”能够在限制数据的平均传输速率的同时，还允许能应对流量突增的情况（允许突发请求，只要有足够的令牌，支持一次拿多个令牌）。
![token-bucket](/assets/images/2022/sucls/12_18/token-bucket.jpg)

### 实现示例

#### 固定窗口
```
public class FixedWindowLimiter {
    /**
     * 时间单元 ms
     */
    private long timeUnit;

    /**
     * 时间单元内的阈值
     */
    private long limit;

    /**
     * 开始时间
     */
    private long startTime;

    /**
     * 计数器
     */
    private long count;

    public FixedWindowLimiter(long timeUnit, long limit) {
        this.timeUnit = timeUnit;
        this.limit = limit;
    }

    /**
     *
     * @return
     */
    public synchronized boolean acquire(){
        long now = System.currentTimeMillis();
        // 开始
        if( startTime == 0 ){
            startTime = now;
            count ++;
            return true;
        }
        // 在一个时间单元内
        if(now - startTime <= timeUnit){
            count ++;
            return count <= limit;
        }else{// 超过时间单元、
            startTime = now;
            count = 0;
            return true;
        }
    }
```

## Guava RateLimiter

+ 令牌桶算法实现
+ 支持预热
+ 支持突发流量

|配置| 说明          |
|---|-------------|
|permitsPerSecond| 单位时间内产生令牌数量 |
|warmupPeriod| 预热期         |
|unit| 预热期时间单位     |

创建RateLimiter，每秒发放6个令牌，平均间隔167ms一个，其中有3秒的预热期
```
    RateLimiter rateLimiter = RateLimiter.create(
            6, // 每秒发放令牌数量
            3, // 预热期，在预热期后逐步达到配置的令牌发放数量
            TimeUnit.SECONDS // 时间单位
    );
```
使用RateLimiter
```
    @Test
    public void limit() throws InterruptedException {
        Stopwatch stopwatch = Stopwatch.createStarted();
        long last;
        for (int i=0; i < 100; i++){
            last = stopwatch.elapsed(TimeUnit.MILLISECONDS);
            rateLimiter.acquire();
            long duration = stopwatch.elapsed(TimeUnit.MILLISECONDS);
            System.out.println( String.format("第%s次，距离开始的时间：%sms，间隔时间：%sms", i, duration, duration-last));

            if( i == 20 ){
                // 中间暂停5秒，看看申请令牌的时间间隔变化
                TimeUnit.SECONDS.sleep(5);
                System.out.println("暂停5秒后...");
                long last2 = stopwatch.elapsed(TimeUnit.MILLISECONDS);
                rateLimiter.acquire(10);
                long duration2 = stopwatch.elapsed(TimeUnit.MILLISECONDS);
                System.out.println( String.format("第%s次，距离开始的时间：%sms，间隔时间：%sms", i, duration2, duration2-last2));
            }
        }
    }
```
结果：大约3秒后进入平稳期
```
第0次，距离开始的时间：0ms，间隔时间：0ms
第1次，距离开始的时间：483ms，间隔时间：483ms
第2次，距离开始的时间：926ms，间隔时间：443ms
第3次，距离开始的时间：1334ms，间隔时间：408ms
第4次，距离开始的时间：1703ms，间隔时间：369ms
第5次，距离开始的时间：2036ms，间隔时间：333ms
第6次，距离开始的时间：2333ms，间隔时间：297ms
第7次，距离开始的时间：2592ms，间隔时间：259ms
第8次，距离开始的时间：2815ms，间隔时间：223ms
第9次，距离开始的时间：2999ms，间隔时间：184ms
第10次，距离开始的时间：3166ms，间隔时间：167ms
第11次，距离开始的时间：3333ms，间隔时间：167ms
第12次，距离开始的时间：3500ms，间隔时间：167ms
第13次，距离开始的时间：3666ms，间隔时间：166ms
...
暂停5秒后...
第20次，距离开始的时间：9842ms，间隔时间：0ms
第21次，距离开始的时间：13009ms，间隔时间：3166ms
第22次，距离开始的时间：13176ms，间隔时间：167ms
第23次，距离开始的时间：13342ms，间隔时间：166ms
第24次，距离开始的时间：13510ms，间隔时间：168ms
第25次，距离开始的时间：13676ms，间隔时间：166ms
第26次，距离开始的时间：13843ms，间隔时间：167ms
第27次，距离开始的时间：14009ms，间隔时间：166ms
```

+ 预热期
  系统刚启动或者长时间没有收到请求时，限流器处于冷却状态,在预热期间获取令牌的时间会比平稳期获取令牌的时间要长，随着令牌的减少，获取单个令牌的时间会慢慢变短，最终到达一个稳定值
+ acquire
  acquire是一个阻塞方法，通过RateLimiter会得到一个阻塞时间值
+ tryAcquire
  非阻塞方法，快速返回令牌申请结果

### 结束语

  作为微服务服务保证的三大利器，限流、熔断、降级，了解其大概的大概的含义是非常有必要的，虽然现在有很多封装好的限流框架，比如Sentinel、Resilience4j等，但技术是没有止境的，当你往下探索
时，更多不可思议的知识，后面有机会我们从源码，更底层来认识这些技术与设计思想。