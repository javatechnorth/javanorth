---
layout: post
title:  你是哪家的锁，这么膨胀 -- 20210830
tagline: by 某某白米饭
categories: 多线程
tags: 
    - 某某白米饭
---

大家好，我是指北君。

在面试的时候，最会被问到的多线程问题就是 synchronized，如果还只会回答 monitorenter 和 monitorexit 那就有可能通不过面试，除了 monitorenter，还可以和面试官聊聊 synchronized 的锁膨胀。
<!--more-->
### 初识

synchronized 可以加在方法和类上面，作用于类和对象。下面代码中列出了 synchronized 的用法。

```java
public class SynchronizedTest {

    public static final Object lock = new Object();

    // 锁的是SynchronizedTest.class对象
    public static synchronized void sync1() {

    }

    // 锁的是SynchronizedTest.class对象
    public static void sync2() {
        synchronized (SynchronizedTest.class) {

        }
    }

    // 锁的是当前实例this
    public synchronized void sync3() {

    }

    // 锁的是当前实例this
    public void sync4() {
        synchronized (this) {

        }
    }

    // 锁的是指定对象lock
    public void sync5() {
        synchronized (lock) {

        }
    }
}
```

synchronized 大家都知道是用 monitorenter 和 monitorexit 两个指令锁住同步块的。

那么 synchronized 是怎么膨胀的呢？为什么会膨胀呢？

先从 JVM 内存开始讲起，对象在被实例化后，是存放在堆内存中的，它由 3 部分组成：

1. 对象头：存放对象运行时的状态的信息、指向该对象所属 Class 的元数据的指针。
2. 实例数据：存放对象的属性数据信息，包括父类的信息。
3. 对齐填充字节：由于虚拟机要求对象的大小必须是 8 字节的整数倍。不是必须存在，仅仅是为了字节对齐。

其中对象头里面包含了 Mark Word（标记字段）和 Class Pointer（类型指针）

![](http://www.javanorth.cn/assets/images/2021/synchronized/1.png)

1. Mark Word 默认的存储对象的 hashcode、分代年龄、是否偏向锁、锁标识位的信息，它在运行期间的存储内容会随着锁的变化而变化。

| Mark Word (32 bits) | 是否偏向锁 | 锁标识位值 | 锁状态 |
| --- | --- | --- | --- |
| 对象的hashcode(25)、分代年龄(4)、是否偏向锁(1)、锁标识位(2) | 0 | 01 | 无锁 |
| 线程ID(23)、偏向时间戳(2)、分代年龄(4)、是否偏向锁(1)、锁标识位(2) | 1 | 01 | 偏向锁 |
| 指向栈中锁记录的指针(30)、锁标识位(2) | | 00 | 轻量级锁 |
| 指向重量级锁的指针(30)、锁标识位(2) | | 10 | 重量级锁 |

2. Class Pointer（类型指针）:对象指向类的元数据的指针，虚拟机通过这个指针来确定对象是哪一个类的实例。

### 锁膨胀

偏向锁、轻量级锁、重量级锁、自旋锁，这些都是Synchronzied的锁的实现。Synchrozied会根据不同的场景选择不同的锁，我们只使用Synchronzied，不用关心它具体使用的哪个锁。

#### 偏向锁

在java 程序中，大多数情况不存在多个线程同时竞争锁，往往都是同一个线程多次获得同一个锁。

当只有一个线程在竞争锁的时候，在线程获取到锁后，将进入偏向模式，程序会将对象的头的前 23 个字节用 CAS 的方式存储线程 ID。下次有线程竞争锁，只需要比较对象头中的线程 ID 是不是和此时获取到锁的线程 ID　相同。如果相同线程就直接进入同步代码块，不需要 CAS 竞争锁。

![](http://www.javanorth.cn/assets/images/2021/synchronized/2.png)

有另外的线程在竞争锁的时候，持有偏向锁的线程才会释放锁，持有偏向锁的线程不会主动释放偏向锁。
偏向锁的撤销，是在没有字节码执行的时候进行的。首先会暂停偏向锁的线程，判断锁对象是否被锁住。撤销偏向锁后恢复成无锁或者是轻量级锁。

#### 轻量级锁

当有另外的线程在竞争偏向锁的时候并且竞争失败了，偏向锁就会膨胀为轻量级锁，其他的线程会通过自旋的方式尝试获取锁。

JVM 会在当前线程的栈帧中创建一个叫做锁记录（Lock Record）的空间，将锁对象的 Mark Word 复制进去。这个官方称为 Displaced Mard Word。然后 JVM 将使用 CAS 操作尝试将锁对象的Mark Word 更新为指向 Lock Record 的指针。如果更新成功，锁标识位就成为 00，此时为轻量级锁。


![](http://www.javanorth.cn/assets/images/2021/synchronized/3.png)

#### 重量级锁

从上面的表格中就指出重量级锁的对象头里面存储的是指向 monitor 的指针，那 monitor 是什么呢？

monitor 又称为管程，Java 中由 ObjectMonitor 实现。 当线程要将对象加锁的时候，对象会创建一个monitor。


![](http://www.javanorth.cn/assets/images/2021/synchronized/4.png)

ObjectMonitor 主要的字段有：

1. owner：就是当前加锁的线程
2. waitSet：就是 owner的线程调用了 wait() 方法，就进入这个里面
3. entryList：加锁失败的线程阻塞在这个里面
4. recursions：锁的重入次数
5. count：用来记录是不是有对象加锁：0.当前对象没有线程加锁，1. 当前对象有线程加锁

从轻量级锁升级到重量级锁的时候，对象头 Mark Word 存储已经变成了指向 Monitor 的指针。线程可以通过这个指针找到 ObjectMonitor，放入 entryList 等待重量级锁释放后竞争。entryList 中的线程 CAS 尝试更新 count = 1，当更新成功后将 owner 设置为当前的线程。当 owner 的线程调用了 wait() 方法，线程就会释放锁，进入 waitSet 中。这个时候 count = 1，owner = null，entryList 的线程可以再次竞争锁。

![](http://www.javanorth.cn/assets/images/2021/synchronized/5.png)

### 总结

1. synchronized 不管是加在类上还是方法上，如果作用在类上，这个类的所有对象都是同一把锁，
2. 锁膨胀时不可以降级的

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
