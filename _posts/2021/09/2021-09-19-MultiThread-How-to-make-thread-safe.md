---
layout: post
title: 软软猿妹问我遇到线程安全怎么办？
tagline: by 揽月中人
categories: Algorithm
tags:
- 揽月中人
---

线程安全一直是多线程开发中需要注意的地方，可以说，并发安全保证了所有的数据都安全。

<!--more-->

### 1 线程不安全示例

线程安全其实是多线程编程里面的一个核心点，所有的设计和代码都是为了实现线程的高效与安全。

多线程中有几个比较核心概念，即原子性，可见性，顺序性。那么线程安全也会围绕着这三个核心来展开喽。

下面我们看一两个简单的问题多线程。

**简单买票线程安全问题**

```java
public class ThreadSafeDemo3 {
    public static void main(String[] args) throws InterruptedException {
        TicketStation station = new TicketStation();
        new Thread(station,"软软").start();
        new Thread(station,"冰冰").start();
        new Thread(station,"指北君").start();
    }
}
class TicketStation implements Runnable{
    int ticketCount = 10;
    boolean hasTicket = true;
    @Override
    public void run() {
        while(hasTicket){buyTicket();}
    }
    private void buyTicket(){
        if (ticketCount < 1) {
            hasTicket = false;
            return;
        }
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " get the ticket"+ ticketCount--);
    }
}
```

运行几遍就有可能会出现下面的错误不预期的结果。一个线程卖完了票，但是另外两个线程都还不知道。

![image-20210926221216385](E:\javaNorth\javanorth\assets\images\2021\lyj\treadSafeTicketResult.jpg)



**多线程操非线程安全对象问题**

```java
public class ThreadSafeDemo2 {
    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<>();
        for (int i =0 ;i< 20 ;i++) {
            new Thread(() -> {
                for (int j = 0; j < 5; j++) {
                    list.add(Thread.currentThread().getName() + j);
                }
            }, "thread" + i).start();
        }
        Thread.sleep(1000*3);
        System.out.println(list.size());
    }
}
```

以上代码多执行几次之后会，所得list的size不会等于100。问题就在于多线程操作同一个线程不安全的List的时候，会是结果与预期不符，出现线程安全问题。



以上是两个线程不安全的示例，而对于线程安全，应该要做到如下：

当多个线程访问某个方法的时候，不管你通过怎样的调用方法，或者说这些线程如何交替执行，我们在主程序中不需要去做任何的同步，这个类的行为都是我们设想的正确行为，那么我们可以说这个类是线程安全的。即可以保证原子性，可见性，顺序性。

### 2 并发安全的问题根源

线程不安全指的是多线程并发执行某个代码时，产生了逻辑上的错误，结果和预期值不相同。

其原因可以总结如下： 

- Java线程是抢占执行的。
- 有些操作不是原子的，cpu在处理某一个线程的时候，有可能被其他线程抢去做工。
- 内存共享可变。
- 指令重排序：Java编译器在编译代码时，会对最终执行的指令重排序，它会保证原有逻辑不变的情况下，提高程序的运行效率。

### 3 线程安全不是绝对的

《深入理解JVM》中有讲到如果要保证绝对线程安全，在大多数的应用场景下是难以做到的，或者说很难做到，即使做到，也会付出很大的代价。而且在Java中标注的某些线程安全的类也不是绝对的线程安全，也需要在调用时使用一些额外操作。

我们大多时候都是尽量保证线程的相对安全，对一个对象单独操作的时候保证线程安全，而对于一些特殊的调用情况，我们则需要采取一些同步操作付诸。

### 4 线程安全的实现方法

我们在写代码的时候，保证线程安全的方法有多种，下面我们介绍几种方式。

#### 4.1 互斥同步

互斥的特点是在同一时刻只有一个线程获得执行权利，其余线程则会等待。(**同一时刻，只有一个线程在操作共享数据**)  互斥是实现同步的一种手段， 临界区、互斥量、信号量都是主要的互斥实现方式。 即通过实现互斥来最终完成同步的目的。

##### 4.1.1 Synchronized

synchronized是同步锁，主要用来控制线程同步，保证某个锁住的内容不被多个线程同步执行。上述买票的例子中，在buyTicket方法上加上synchronized关键字，就可以使线程同步执行了。

```java
private synchronized  void  buyTicket(){}
```

其中synchronized 使用有几点注意：

1. 加到非静态方法前，表示锁this，即当前对象
2. 加到静态方法前，表示锁当前类的所有类对象

##### 4.1.2 Lock

Lock 是Java1.6之后引入的。使用Lock可以对锁进行多种操作，可以手动的获取锁，释放锁。

我们用ReentrantLock （ReentrantLock传送门。。。）改写上述购票行为。

```java
class TicketLockStation implements Runnable{
    private Lock lock = new ReentrantLock();
    int ticketCount = 10;
    boolean hasTicket = true;
    @Override
    public void run() {
        while(hasTicket){buyTicket();}
    }
    private void  buyTicket(){
        lock.lock();
        try {
        if (ticketCount < 1) {
            hasTicket = false;
            return;
        }
        Thread.sleep(1000);
        System.out.println(Thread.currentThread().getName() + " get the ticket"+ ticketCount--);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```

以上为简单Lock示例，其中Lock中还有tryLock()（如果获取不到锁立即返回）， tryLock(long time, TimeUnit unit)（一段时间后获取不到锁则返回）等方法。

上边就是Lock的简单示例。

#### 4.2 非阻塞同步

非阻塞同步可以描述为**基于冲突检测的乐观并发策略**，关键点就是冲突检测以及乐观的并发策略。

冲突检测是指当发生共享数据抢夺的话，我们会进行重试检测，直到成功为止。而乐观的并发策略的实现大多时候都不需要挂起线程。 

##### 4.2.1 CAS

CAS（Compare  And Swap）是非阻塞的一个实现，其核心指令有3个操作数，分别为内存地址V， 旧值A，新值B。当CAS执行时是有当 V的内存地址对应的值与A匹配时，本操作就会用B来更新V对应的值，否则不执行更新。但无论是否更新了V对应的值，都会返回V处对应的旧值。而且此操作为原子操作。

CAS有一个缺点就是ABA问题，V处的值原来是A，后来变成了B,然后又变成了A。 使用CAS检查的时候发现其值没变化，然而实际上已经发生了变化。对于解决ABA问题，可以使用版本号的思路来解决，在更新变量的时候把版本号加一。然后对比的时候也对比版本号，版本号与值全都相等则执行更新。

##### 

#### 4.3 无同步方案

保证线程安全的方法中，也并不是一定要使用同步。 同步只是在保证共享数据在有竞争条件的时候使用。如果有方法可以避免共享数据的竞争，那么自然就不需要任何同步操作去保证数据的正确。所以在有一些场景下，代码自身就已经保证了线程安全，而无须使用同步方法。

##### 4.3.1 栈封闭

##### 4.3.2 线程本地存储

如果代码中所需的数据必须与其他代码共享，那么就可以看看这些共享数据的代码是否能保证在同一个线程中执行。如果可以保证共享数据在同一个线程之内是可见的，那么线程之间也就不会出现数据的竞争。

ThreadLocal就是一个最典型的例子，Web应用中的Request也是这样的思路。

##### 4.3.3 可重入代码 （Reentrant Code）

可重入代码指在代码执行的任何时刻中断它，转而去执行另外一段代码，而控制权返回后，原来的程序不会出现任何错误。所有的可重入代码都是线程安全的。一般而言可重入代码不依赖存储在堆上的数据以及公共的系统资源，用到的状态量都是有参数传入，或者说不调用，非可重入的方法等。

### 总结

关于线程安全，我们可以总结以下的一些思路。

1. 使用互斥同步的方法。
   - 使用Synchronized
   - 使用Lock

2. 使用非阻塞同步方案。

   - CAS 等。

3. 无同步方案

   其实就是在设计上尽量避免共享变量的使用，这样也就可以避免线程安全问题的发生。

