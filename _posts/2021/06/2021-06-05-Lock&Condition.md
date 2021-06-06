---
layout: post
title:  了解这两个接口后，阿里多线程面试题秒AC
tagline: by 24只羊
categories: JDK JUC 源码解读
tags: 
    - 24只羊

---

大家好，我是指北君

我们知道，阿里面试时是非常喜欢考Java多线程编程题，如果你AC不了，那可能会给面试官留下一个基础不扎实的印象，当年指北君面阿里时就因为秒AC了一道多线程面试题，让面试官刮目相看，所以我们需要重视Java多线程编程。一般在解决多线程编程题时，我们都离不开JUC并发包下的各种工具类，特别是ReentrantLock锁，它能提供互斥与线程同步的能力，那它是如何获得这个能力的呢？今天指北君就来详细说说给它提供强大能力的两大接口。（PS：文末有当年指北君面试阿里的多线程编程原题以及答案喔）

<!--more-->

我们知道，并发领域中有两大核心问题：互斥与同步问题，Java在1.5版本之前，是提供了synchronized来实现的。synchronized是内置锁，虽然在大部分情况下它都能很好的工作，但是依然还是会存在一些局限性，除了当时1.5版本的性能问题外（1.6版本后，synchronized的性能已经得到了很大的优化），还有如下两个问题：

1. 无法解决死锁问题
2. 最多使用一个条件变量

所以针对这些问题，Doug Lea在并发包中增加了两个接口Lock和Condition来解决这两个问题，所以指北君今天就说说这两个接口是如何解决synchronized中的这两个问题的。


 
 <br/>

### 一. Lock接口

#### 1.1 介绍

在我们分析Lock接口是如何解决死锁问题之前，我们先看看死锁是如何产生的。死锁的产生需要满足下面四个条件：

1. **互斥**：共享资源同一时间只能被一个线程占用
2. **不可抢占**：其他线程不能强行占有另一个线程的资源
3. **占有且等待**：线程在等待其他资源时，不释放自己已占有的资源
4. **循环等待**：线程1和线程2互相占有对方的资源并相互等待

所以，我们只需要破坏上面条件中的任意一个，即可打破死锁。但需要注意的是，互斥条件是不能破坏的，因为使用锁的目的就是为了互斥。所以Lock接口通过破坏掉 "不可抢占"这个条件来解决死锁，具体如下：

1. **非阻塞获取锁**：尝试获取锁，如果失败了就立刻返回失败，这样就可以释放已经持有的其他锁

2. **响应中断**：如果发生死锁后，此线程被其他线程中断，则会释放锁，解除死锁

3. **支持超时**：一段时间内获取不到锁，就返回失败，这样就可以释放之前已经持有的锁

接下来我们具体看看接口代码吧。



#### 1.2 源码解读

```java
	
public interface Lock {
    /**
        阻塞获取锁，不响应中断，如果获取不到，则当前线程将进入休眠状态，直到获得锁为止。
    */
    void lock();

    /**
        阻塞获取锁，响应中断，如果出现以下两种情况将抛出异常
        1.调用该方法时，此线程中断标志位被设置为true
        2.获取锁的过程中此线程被中断，并且获取锁的实现会响应中断
    */
    void lockInterruptibly() throws InterruptedException;
  
    /**
        非阻塞获取锁，不管成功还是失败，都会立刻返回结果，成功了返回true,失败了返回false
     */
    boolean tryLock();
 
    /**
     	带超时时间且响应中断的获取锁，如果获取锁成功，则返回true,获取不到则会休眠，直到下面三个条件满足
     	1.当前线程获取到锁
     	2.其他线程中断了当前线程，并且获取锁的实现支持中断
     	3.设置的超时事件到了
     	而抛出异常的情况与lockInterruptibly一致
     	当异常抛出后中断标志位会被清除，且超时时间到了，当前线程还没有获得锁，则会直接返回false
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
   
    /**
        没啥好说，只有拥有锁的线程才能释放锁
     */
    void unlock();

    /**
        返回绑定到此Lock实例的新Condition实例。
        在等待该条件之前，该锁必须由当前线程持有。调用Condition.await（）会在等待之前自动释放锁，并在等待返回之前重新获取该锁。
        我们再下一小节再详细说说Condition接口
     */
    Condition newCondition();
}


```



还需要额外注意的一点，使用synchronized作为锁时，我们是不需要考虑释放锁的，但Lock是属于显示锁，是需要我们手动释放锁的。我们一般在finally块中调用lock.unlock()手动释放锁，具体形式如下：

```java
	 Lock l = ...;
	 l.lock();
 	 try {
             // access the resource protected by this lock
	 } finally {
  	 l.unlock();
	 }
```



我们最后通过一张图来总结下Lock接口：

![Lock](http://www.javanorth.cn/assets/images/2021/Yang24/lock.png)



 <br/>

### 二. Condition接口

#### 2.1 介绍

针对synchronized最多只能使用一个条件变量的问题，Condition接口提供了解决方案。但是为什么多个条件变量就比一个条件变量好呢？我们先来看看synchronized使用一个条件变量时会有什么弊端。

一个synchronized内置锁只对应一个等待容器（wait set），当线程调用wait方法时，会把当前线程放入到同一个等待容器中，当我们需要根据某些特定的条件来唤醒符合条件的线程时，我们只能先从等待容器里唤醒一个线程后，再看是否符合条件。如果不符合条件，则需要将此线程继续wait，然后再去等待容器中获取下一个线程再判断是否满足条件。这样会导致许多无意义的cpu开销。

我们可以看到Lock接口中有个newCondition()的方法

```java
Condition newCondition();
```

通过这个方法，一个锁可以建立多个Conditiion，每个Condtition都有一个容器来保存相应的等待线程，拿到锁的线程根据特定的条件唤醒对应的线程时，只需要去唤醒对应的Contition内置容器中的线程即可，这样就可以减少无意义的CPU开销。然后我们具体看看Condition接口的源码。



#### 2.2 源码解读

```java
public interface Condition {

    /**
	使当前线程等待，并响应中断。当当前线程进入休眠状态后，如果发生以下四种情况将会被唤醒：
	1.其他一些线程对此条件调用signal方法，而当前线程恰好被选择为要唤醒的线程；
	2.其他一些线程对此条件调用signalAll方法
	3.其他一些线程中断当前线程，并支持中断线程挂起
	4.发生“虚假唤醒”。
     */
    void await() throws InterruptedException;

    /**
	使当前线程等待，并不响应中断。只有以下三种情况才会被唤醒
	1.其他一些线程对此条件调用signal方法，而当前线程恰好被选择为要唤醒的线程；
	2.其他一些线程对此条件调用signalAll方法
	3.发生“虚假唤醒”。
     */
    void awaitUninterruptibly();

    /**
        使当前线程等待，响应中断，且可以指定超时事件。发生以下五种情况之一将会被唤醒：
        1.其他一些线程为此条件调用signal方法，而当前线程恰好被选择为要唤醒的线程；
        2.其他一些线程为此条件调用signalAll方法；
        3.其他一些线程中断当前线程，并且支持中断线程挂起；
        4.经过指定的等待时间；
        5.发生“虚假唤醒”。
     */
    long awaitNanos(long nanosTimeout) throws InterruptedException;

    /**
	与awaitNanos类似，时间单位不同
     */
    boolean await(long time, TimeUnit unit) throws InterruptedException;

    /**
	与awaitNanos类似，只不过超时时间是截止时间
     */
    boolean awaitUntil(Date deadline) throws InterruptedException;

    /**
	唤醒一个等待线程
     */
    void signal();

    /**
	唤醒所有等待线程
     */
    void signalAll();
}

```



需要注意的是，Object类的等待方法是没有返回值的，但Condtition类中的部分等待方法是有返回值的。awaitNanos(long nanosTimeout)返回了剩余等待的时间；await(long time, TimeUnit unit)返回boolean值，如果返回false，则说明是因为超时返回的，否则返回true。为什么增加返回值？为了就是帮助我们弄清楚方法返回的原因。

 <br/>
 
 

### 四. 阿里多线程考题

最后我们通过实现了Lock和Condition接口能力的ReentrantLock类来解决阿里多线程面试题的。

题目是使用三个线程循环打印ABC，一共打印50次。我们直接上答案：

```java
public class Test {


    int count = 0;
    Lock lock = new ReentrantLock();
    Condition conditionA = lock.newCondition();
    Condition conditionB = lock.newCondition();
    Condition conditionC = lock.newCondition();

    public void printA() {
        while (count < 50) {
            try {
                // 加锁
                lock.lock();
                // 打印A
                System.out.println("A");
                count ++;
                // 唤醒打印B的线程
                conditionB.signal();
                // 将自己放入ConditionA的容器中，等待其他线程的唤醒
                conditionA.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                // 释放锁
                lock.unlock();
            }
        }


    }

    public void printB() {
        while (count < 50) {
            try {
                // 加锁
                lock.lock();
                // 打印B
                System.out.println("B");
                count ++;
                // 唤醒打印C的线程
                conditionC.signal();
                // 将自己放入ConditionB的容器中，等待其他线程的唤醒
                conditionB.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                // 释放锁
                lock.unlock();
            }
        }
    }


    public void printC() {
        while (count < 50) {
            try {
                // 加锁
                lock.lock();
                // 打印B
                System.out.println("C");
                count ++;
                // 唤醒打印A的线程
                conditionA.signal();
                // 将自己放入ConditionC的容器中，等待其他线程的唤醒
                conditionC.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        Test test = new Test();
        // 建立打印ABC的三个线程
        Thread theadA = new Thread(() -> {
            test.printA();
        });
        Thread theadB = new Thread(() -> {
            test.printB();
        });
        Thread theadC = new Thread(() -> {
            test.printC();
        });
				
        // 启动线程
        theadA.start();
        theadB.start();
        theadC.start();

    }
}
```





 <br/>



### 五. 总结

Lock与Condition接口就说完了，最后指北君再总结一下：

针对synchronized内置锁无法解决死锁、只有一个条件变量等问题，Doug Lea在Java并发包中增加了Lock和Condition接口来解决。对于死锁问题，Lock接口增加了超时、响应中断、非阻塞三种方式来获取锁，从而避免了死锁。针对一个条件变量问题，Condtition接口通过一把锁可以创建多个条件变量的方式来解决。最后我们通过一个阿里面试题来说明了Lock和Condition接口所提供的能力。

今天就到这了，我是指北君，我们下篇文章见~
