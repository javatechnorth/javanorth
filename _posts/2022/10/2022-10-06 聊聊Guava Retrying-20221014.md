---
layout: post
title:  聊聊Guava Retrying -20221014
tagline: by 沉浮
categories: Guava 重试
tags: 
- 沉浮
---


在web应用中，由于网络原因或其他不可预测的原因，应用间会出现调用失败的情形，通过配置重试策略可以有效解决外在原因导致的系统故障
<!--more-->
## Guava Retry

在web应用中，由于网络原因或其他不可预测的原因，应用间会出现调用失败的情形，通过配置重试策略可以有效解决外在原因导致的系统故障。

### 使用场景

1. 微服务间各个服务模块间的调用
2. 第三方模块远程交易调用
3. 非业务异常导致可能失败的情况

### 示例

> 构建Retryer
```
private Retryer retryer = RetryerBuilder.newBuilder()
        .retryIfException() // 异常时重试
        .retryIfResult(input -> input!=null && input instanceof Boolean && !Boolean.valueOf((Boolean) input)) // 返回值为false时重试
        // 对应Future获取超时时间
        .withAttemptTimeLimiter(AttemptTimeLimiters.fixedTimeLimit(4, TimeUnit.SECONDS,Executors.newFixedThreadPool(2))) //重试次数限制
        .withRetryListener(new RetryListener() { // 重试执行逻辑
            @Override
            public <V> void onRetry(Attempt<V> attempt) {
                log.info("onRetry -> 重试次数：{}，距第一次重试时长：{}", attempt.getAttemptNumber(),attempt.getDelaySinceFirstAttempt());
                if(attempt.hasException()){ // 是否异常导致重试
                    Throwable exception = attempt.getExceptionCause(); // 执行的异常
                    log.info("异常：{}", exception);
                }
                if(attempt.hasResult()){ // 是否有返回
                    V result = attempt.getResult();
                    log.info("返回：{}",result);
                }
            }
        })
        // 控制每次重试间隔时间，如果AttemptTimeLimiter设置多线程
        .withWaitStrategy(WaitStrategies.fixedWait(3,TimeUnit.SECONDS)) // 等待策略
        .withBlockStrategy(BlockStrategies.threadSleepStrategy()) // 阻塞策略
        //
        .withStopStrategy(StopStrategies.stopAfterAttempt(5)) // 停止策略
        .build();
```

> 使用Retryer让业务代码拥有重试能力
>> 前两次执行时模拟返回false，则会执行重试；当第3次时，正常执行业务代码并返回true，结束重试
```
@Test
public void retryWhenResult() throws ExecutionException, RetryException {
   retryer.call(() -> {
       if(counter.incrementAndGet() == 3){// 模拟前2此返回false，触发重试
           log.info(" 执行业务代码：{}次",counter.get());
           return true;
       }
       return false; 
   });
}
```
>> 模拟前3次出现异常，则会执行重试；当第3次时，正常执行业务代码，结束重试
``` 
@Test
public void retryWhenException() throws ExecutionException, RetryException {
    retryer.call(() -> {
        if( counter.getAndIncrement() == 3 ){// 模拟前5此出现异常，触发重试
            return counter;
        }
        log.info(" 执行业务代码: {}次", counter.get());
         throw new RuntimeException("ERROR"); 
    });
}
```
>> 模拟前5此出现异常，由于Retryer配置重试次数为5，则最终业务代码不会执行
```
@Test
public void retryWhenResultOnFailure() throws ExecutionException, RetryException {
    retryer.call(() -> {
        if(counter.incrementAndGet() == 8){// 模拟前7此返回false，由于配置重试5次，因此最终失败
            log.info(" 执行业务代码：{}次",counter.get());
            return true;
        }
        return false;
    });
}
```

### 执行流程

![执行流程](/assets/images/2022/sucls/10_06/guava-retry.jpg)

通过RetryerBuilder构建Retryer，调用Retryer#call，封装业务代码为其回到函数
1. 开始循环执行
2. 由AttemptTimeLimiter#call执行回调函数
3. 将结果封装为Attempt，包括两种类型ResultAttempt，ExceptionAttempt。如果成功，记录执行结果、持续时长；如果失败，记录异常、持续时长
4. 执行监听RetyrListener#onRetry，可以配置多个监听
5. 执行拒绝断言Predicate，根据返回值、执行异常、返回异常类型判断是否终止重试
6. 如果满足条件，则继续重试；否则结束重试，并返回Attempt包含回调结果
7. 根据终止策略StopStrategy判断是否终止重试
8. 根据等待策略WaitStrategy获取等待时长
9. 根据阻塞策略BlockStrategy与上一步等待时长阻塞重试，如果出现异常则抛出RetryException
10. 重复执行以上逻辑


### 配置

构建Retryer主要通过RetryerBuilder.newBuilder()实现，其相关配置如下：

| 配置                  | 策略                       | 名称              | 描述                                                  |
|---------------------|--------------------------|-----------------|-----------------------------------------------------|
| AttemptTimeLimiters |                          | 任务执行时长限制        |                                                     |
|                     | NoAttemptTimeLimit       | 无时长限制           |                                                     |
|                     | FixedAttemptTimeLimit    | 固定时长限制          |                                                     |
| WaitStrategies      |                          | 重试等待策略          |                                                     |
|                     | ExponentialWaitStrategy  | 指数等待策略          | 按指数增加重试间隔时长，比如第一次2^1*100、2^2*100、2^3*100...最多300000 |
|                     | FibonacciWaitStrategy    | 斐波那契等待策略        | 1*100、1*100、2*100、3*100、5*100...                    |
|                     | FixedWaitStrategy        | 固定时长等待策略        | 按配置的固定间隔时间                                          |
|                     | RandomWaitStrategy       | 随机时长等待策略        | 随机间隔时间，可以设置随机值范围                                    |
|                     | IncrementingWaitStrategy | 递增等待策略          | 根据配置的初始值与增量进行累加时间                                   |
|                     | ExceptionWaitStrategy    | 异常等待策略          | 根据异常类型指定等待时间                                        |
|                     | CompositeWaitStrategy    | 复合等待策略          | 可配置多个策略进行组合                                         |
| BlockStrategies     |                          | 阻塞策略            | 根据WaitStrategies获取阻塞时长                              |
|                     | ThreadSleepStrategy      | 线程等等策略          | 通过Thread.sleet()实现                                  |
| StopStrategies      |                          | 重试停止策略          |                                                     |
|                     | NeverStopStrategy        | 无限制策略           |                                                     |
|                     | StopAfterAttemptStrategy | 限定次数策略          |                                                     |
|                     | StopAfterDelayStrategy   | 限定时长策略          |                                                     |
|                     | NoAttemptTimeLimit       | 限定次数            |                                                     |

### 注意

1. AttemptTimeLimiter中的FixedAttemptTimeLimit依赖于guava中的SimpleTimeLimiter，但是在guava高版本中该类已经成了私有类

### 总结

Guava Retrying模块能够通过简单的将代码实现业务逻辑重试的功能，并且其配置中包含了重试的次数、时长控制、重试阻塞、终止策略等等，
在项目中是非常常用的一项技术。