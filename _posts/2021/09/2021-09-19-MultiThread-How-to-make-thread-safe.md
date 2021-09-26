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

互斥的特点是在同一时刻只有一个线程获得执行权利，其余线程则会等待。其重要的一点就是加锁。

##### 4.1.1 Synchronized

synchronized是同步锁，主要用来控制线程同步，保证某个锁住的内容不被多个线程同步执行。上述买票的例子中，在buyTicket方法上加上synchronized关键字，就可以使线程同步执行了。

```java
private synchronized  void  buyTicket(){}
```

其中synchronized 使用有几点注意：

1. 加到非静态方法前，表示锁this，即当前对象
2. 加到静态方法前，表示锁当前类的所有类对象

##### 4.1.2 ReentrantLock

#### 4.2 非阻塞同步

##### 4.2.1 CAS

##### 4.2.2 Atomic 类

#### 4.3 无同步方案

##### 4.3.1 栈封闭

##### 4.3.2 线程本地存储（ThreadLocal）

##### 4.3.3 可重入代码 （Reentrant Code）

### 总结