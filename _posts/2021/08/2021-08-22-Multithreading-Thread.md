---
layout: post
title:  Thread
tagline: by 揽月中人
categories: Concurrency Java 
tags:
- 揽月中人
---

我们每次讲到Java并发编程，高并发，多线程等等，都不可避免的会用到Thread。我相信部分同学对于Thread都有一定的了解，但是又不敢说精通或者十分了解。那么为了将Thread弄得比较清楚，我们接下来会共同学习，进一步深挖Thread的内容。只有到达一定深度的时候，才能有更深刻的记忆，对于多线程在开发中遇到的问题才能更快速的定位到更本原因!

<!--more-->

### 1 线程与进程

进程是资源分配的最小单位，是系统进行资源分配和调度的一个独立单元。

实现并发最直接的方式是在操作系统级别使用**进程**，进程是运行在它自己的地址空间内的自包容的程序。

多任务操作系统可以通过周期性地将CPU从一个进程切换到另一个进程，来实现同时运行多个进程即多个程序。因此每个进程看起来在其执行过程中都是停停歇歇的。

操作系统通常会将进程互相隔离，它们彼此之间不会互相干涉，这样用进程编程容易得多。

并发编程使我们可以将程序划分成多个分离的、独立运行的任务。通过使用多线程机制，这些独立线程中的每一个都有**执行线程**来驱动。

一个线程就是在进程中的一个单一的顺序控制流，  换句话说就是单个进程可以拥有多个并发执行任务，每一个任务都好像有其自己的CPU一样，其底层机制是切分CPU的时间，但是这个不需要我们来考虑。

线程模型为编程带来了便利，它简化了单一程序中同时交织在一起的多个操作的处理。在使用线程时，CPU将轮流给每个任务分配其占用时间。



### 2 Java多线程

多线程可以提高我们程序运行的效率，相对的来说，也需要系统投入一些资源来支持多线程的运行。

多线程的优点： 

1. 资源利用率好，程序相应更快
2. 进程之间不能共享内存，但是线程值之间共享内存非常容易
3. 创建线程的代价远小于创建进程，Java对多线程支持比较好。

多线程的代价：

1. 设计更复杂
2. 会增加上下文切换的开销
3. 增加资源的消耗

Java的线程的机制是抢占式，这表示调度机制会周期性地中断线程，将上下文切换到另一个线程，从而为每一个线程都提供时间片，使得每个线程都会分配到数量合理的时间去驱动它的任务。

Java中创建线程的三种方式，

1. 继承Thread类创建线程
2. 实现Runnable接口创建线程 -无返回值。Java8之后可以使用 Lambda表达式创建线程。
3. 使用Callable和Future创建线程 -有返回值Future



### 3 Thread

Thread类有很多的信息，本篇我们讲一下基本信息。

#### 3.1 Start() 与 Run()

Thread中调用start方法方可启动线程，而run方法只是thread的一个普通方法调用.

Start方法将未启动的线程加入到ThreadGroup中，线程处于可运行状态，一旦得到CPU时间片，就开始执行run()方法。 

#### 3.2 Daemon线程

后台线程（daemon）： 程序运行的时候在后台提供一种通用服务线程，并且这种线程并不属于程序中不可或缺的一部分。所有的非后台线程结束时，程序也就终止了，同时会杀死进程中的所有后台进程。后台进程在不执行finally子句的情况下就会终止run方法。

#### 3.3 join()

```java
//join 方法，超时
public final synchronized void join(long millis) 
    throws InterruptedException

 //无超时的join
public final void join() throws InterruptedException {
        join(0);//表示会一直等待
   }    

//此处时join参数为0的时候部分逻辑
//其中为0时，将一直等待。
if (millis == 0) {
    while (isAlive()) {
        wait(0);
}
```

加入一个线程： 一个线程可以在其它线程智之上用join()方法，其结果就是等待一段时间直到第二个线程结束才继续执行。

如果某一个线程在另一个线程t上调用join()，次线程将被挂起，直到目标线程t结束才恢复。

也可以在join()时带上超时参数，join()方法其实也是，这样如果目标线程在这段时间到期后还没有结束，join()方法总能返回。对join()方法的调用可以通过在调用线程上调用interrupt()方法。



#### 3.4 sleep() 

```java
`public static native void sleep(long millis) throws InterruptedException;`
```

暂停当前线程，通知线程调度器把当前线程在指定的时间周期内置为wait状态。wait时间结束之后，线程状态重新变成可执行。



#### 3.5 interrupt()

中断线程，将该线程设置为中断状态。



#### 3.6 priority

线程的优先级是线程的一个重要属性，线程调度器会倾向于让优先权更高的线程先执行。但也只是一种概率而已，不一定优先级低的线程就会最后执行。大部分的情况下，所有线程都应该以默认的优先级运行，让调度器去安排它们的执行顺序。

#### 3.7 yield()

线程协作让步，暂停当前线程，以便其他线程有机会执行线程，不能保证当前线程马上停止。yield()方法只是将Running状态转变为Runnable状态。

#### 3.8 线程捕获异常

由于线程的本质特性，使得你不能捕获到从线程中逃逸的异常。我们可以通过设置setUncaughtExceptionHandle来捕获线程异常。

#### 3.9 wait/notify/notifyAll

wait() notify() notifyAll() 这三个方法都是Object里面的方法，

wait() 方法会让当前线程处于“等待（阻塞）状态”，直到其他线程调用此对象的notify() 或notifyAll()方法，当前线程才会被唤醒，并进入“就绪状态”。

没有加锁的情况下，调用会抛出一个异常，所以调用之前需要加锁。

```java
* @throws IllegalMonitorStateException if the current thread is not
*         the owner of the object's monitor
```

### 4 线程的状态及转换

![image](E:\javaNorth\javanorth\assets\images\2021\lyj\thread status convert.png)

#### 4.1 New

初始状态：线程被创建，但是还没有被启动，没有调用start()方法。

#### 4.2 RUNNABLE

运行状态： 可执行的状态，可能正在运行，也可能在等待CPU时间片。

#### 4.3 BLOCKED

阻塞状态：线程阻塞于锁。

等待一个monitor lock 如果获得锁就进入一个同步代码块或者方法，或者在调用之后重新进入一个同步代码快或方法。

#### 4.4 WAITING

等待状态：表示线程进入等待状态，需要等待其他线程的一些特定动作。一个线程进入等待状态由于调用了以下的方法

- Object.wait with no timeout
- Thread.join with no timeout
- LockSupport.park

| 进入WAITING状态调用的方法                  | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    |                                      |

#### 4.5 TIMED_WAITING

超时等待状态：线程进入等待状态线程等待一段时间后会自动被唤醒。调用以下方法可以进入此状态：

- Thread.sleep
- Object.wait with timeout
- Thread.join with timeout
- LockSupport.parkNanos
- LockSupport.parkUntil



#### 4.6 TERMINATED

终止状态：线程执行完毕，或者或者产生异常而结束。





### 总结

本片介绍了Java多线程的本质，以及Thread的基本方法和类，相比都已近很熟悉了。巩固一下Thread相关的类，后面我们来细看Thread中的各种实际操作，了解线程之间是如何通信的。