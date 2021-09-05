---
layout: post
title:  Thread 线程之间如何通信
tagline: by 揽月中人
categories: Netty
tags:
- 揽月中人
---

线程之间的通信以及线程之间的协作方面的面试，通常是考验一个Java程序员多线程方面的基本功。下面指北君为大家揭秘Java线程之间的通信那些事儿。

<!--more-->

相信大家都遇到过交替打印数字字母的面试题。下面我们以交替打印12A34B...来说明线程间通信的几种方式.

### 1 通过使用共享对象通信

一个简单的发送信号的方式就是在共享对象的变量里面设置信号值。线程A在同一个同步块里设置boolean成员变量，线程B也在同步块里面读取同一个线程变量。简单例子如下，一个持有信号的对象，并提供一个set和check的方法。



#### 1.1 使用synchronized,wait,notify

#### 1.2 Lock,Condition

```java
public class ThreadSignalingReentrant {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        Condition condition1 = lock.newCondition();
        Condition condition2 = lock.newCondition();
        new Thread(() -> {
            try{
                lock.lock();
                int i = 1;
                while (i <= 26) {
                    System.out.print(i * 2 - 1);
                    System.out.print(i * 2);
                    i++;
                    condition2.signal();
                    condition1.await();
                }
                condition2.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }).start();

        new Thread(() -> {
            try{
                lock.lock();
                char i = 'A';
                while (i <= 'Z') {
                    System.out.print(i);
                    i++;
                    condition1.signal();
                    condition2.await();
                }
                condition1.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

        }).start();
    }
}
```

#### 1.3 LockSupport

```java
public class ThreadSignalingLockSupport {
    private static Thread threadA = null;
    private static Thread threadB = null;
    
    public static void main(String[] args) {
        threadA = new Thread(() -> {
            int i = 1;
            while (i <= 26) {
                System.out.print(i * 2 - 1);
                System.out.print(i * 2);
                i++;
                LockSupport.unpark(threadB);
                LockSupport.park();
            }
        });
        threadB = new Thread(() -> {
            char i = 'A';
            while (i <= 'Z') {
                LockSupport.park();
                System.out.print(i);
                i++;
                LockSupport.unpark(threadA);
            }
        });
        threadA.start();
        threadB.start();
    }
}
```

#### 1.4 volatile

#### 1.5 AtomicInteger

### 2 利用 PipedInputStream

准备处理数据的线程B正等着数据变为可用。线程B一直等待线程A的信号。线程B运行在一个循环里面，以等待这个型号：

### 3 利用BlockingQueue







### 总结

今天就是给大家展示了线程间通信的几种方法，相信大家已经比较比较了解线程间通信的内容，后面会带来一些多线程的其他应用，敬请期待。

