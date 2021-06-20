---
layout: post
title:  新人都能看懂的线程池
tagline: by 某某白米饭
categories: 线程
tags: 
    - 某某白米饭
---


大家好，我是指北君。


线程池是用来统一管理线程的，在 Java 中创建和销毁线程都是一件消耗资源的事情，线程池可以重复使用线程，不再频繁的创建、销毁线程。

<!--more-->

### 初识
Java 中的线程池是由 juc 即 java.util.concurrent 包来实现的，最主要的就是 ThreadPoolExecutor 类。


![](http://www.javanorth.cn/assets/images/2021/threadPool/0.png)

1. Executor: 代表线程池的接口，有一个 execute() 方法，给一个 Runnable 类型对象就可以分配一个线程执行。
2. ExecutorService：是 Executor 的子接口，提供了线程池的一些生命周期方法。代表了一个线程池管理器。
3. ThreadPoolExecutor：一个线程池的实现类，可以通过调用 Executors 静态工厂方法来创建线程池并返回一个 ExecutorService 对象。

### ThredadPoolExcutor

看一下最常用的 ThredadPoolExcutor ,下图是 ThreadPoolExecutor 的构造函数

![](http://www.javanorth.cn/assets/images/2021/threadPool/1.png)

从源码中可以看出每个前三个构造函数都调用了最后一个构造函数。

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
    //省略代码
    }
```

仔细分析一下构造参数：
1. corePoolSize：线程池里的核心线程数量，当正在运行的线程数量小于核心线程数量，就创建一个核心线程。
2. maximumPoolSize：线程池最多能放多少个线程。
3. keepAliveTime：线程的闲置时间，当线程池里面的线程数量大于 corePoolSize 的时候，多出来的线程在等待的时间之后会被释放掉
4. unit：keepAliveTime 的单位
5. workQueue：一个阻塞队列。
6. threadFactory：通过这个工厂模式创建线程。
7. handler：处理线程队列满了报错的。

结合线程池的参数简单的画出线程池的工作模型。

![](http://www.javanorth.cn/assets/images/2021/threadPool/2.png)

当线程池中的核心线程数量 corePoolSize 满了，就会将任务先加入到任务队列 workQueue 中。



### 常用的线程池

线程池的创建需要有 7 个参数，还是比较复杂的，JVM 为我们提供了 Executors 类中多个静态工厂，生成一些常用的线程池。

#### SingleThreadExecutor

单线程的线程池，里面就一个核心线程数。

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
ThreadPoolExecutor 参数只有一个核心线程数和一个最大线程数，这个很少用到。它保证了所有线程的执行顺序都是按照提交到线程池的顺序执行。

```java
public class TodoDemo implements Runnable {


    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        for(int i = 0; i < 10; i++) {
            executorService.execute(new TodoDemo());
        }
        executorService.shutdown();
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " Running");
    }
}
```

只有一个线程在跑。

![](http://www.javanorth.cn/assets/images/2021/threadPool/3.png)


#### FixedThreadExecutor 

固定数量的线程池

```java
    return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
}
```

这个线程池的特点就是线程的数量是固定的，超过这个数量的任务就得在 LinkedBlockingQueue 中排队等候。

```java
public class TodoDemo implements Runnable {
    
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for(int i = 0; i < 10; i++) {
            executorService.execute(new TodoDemo());
        }
        executorService.shutdown();
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " Running");
    }
}
```

可以看到就算提交 100 个任务也只有 3 个线程。

![](http://www.javanorth.cn/assets/images/2021/threadPool/4.png)

#### CachedThreadExecutor 

自动回收空闲的线程

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}
```

可以看到核心线程数量为 0， 表示不会永久保留任何的线程，最大线程的数量是 Integer.MAX_VALUE，可以无限制的创建线程，但是当有大量线程处于空闲状态的时候，超过 60s 就会被销毁。

```java
public class TodoDemo implements Runnable {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for(int i = 0; i < 20; i++) {
            executorService.execute(new TodoDemo());
        }
        executorService.shutdown();
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " Running");
    }
}
```

虽然这个线程池可以想建多少个线程就建多少个线程，但是还是会重用已经完成任务的线程。

![](http://www.javanorth.cn/assets/images/2021/threadPool/5.png)

一般最常用的是FixedThreadExecutor和CachedThreadExecutor。

### 线程池的回收策略

在线程池中任务队列已经满了，并且线程的数量已经到了最大的数量，这个时候再加任务线程池就不再接受了。

在 ThreadPoolExecutor 里有 4 种拒绝策略，都实现了 RejectedExecutionHandler：
1. AbortPolicy 表示抛出一个异常。
2. DiscardPolicy 拒绝任务但是不提示。
3. DiscardOldestPolicy 丢弃掉老的任务，执行新的任务。
4. CallerRunsPolicy 直接调用线程处理。

### 总结

线程池的作用是提高系统的性能和线程的利用率，不再需要频繁的创建和销毁线程。如果使用最简单的方式创建线程，在用户量巨大的情况下，消耗的性能是非常恐怖的，所以才有了线程池。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
