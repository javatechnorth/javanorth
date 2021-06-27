---
layout: post
title:  并发编程大师(Doug Lea)也用的ThreadLocal
tagline: by 揽月中人
categories: Concurrency
tags: 
    - 揽月中人
---

大家好，我是指北君。

今天学习了ThreadLocal相关得知识，发现原来道哥(Doug Lea)也用ThreadLocal。既然大师们都喜欢用的，我们必须得研究起来。大师的背影总是需要追随。  

那么指北君给大家安排上了，如果你拥有了Java中的ThreadLocal，那麽你可以创建一个只允许同一个线程读写的变量。 因此，即使两个线程执行了相同的代码，并且引用了相同的ThreadLocal变量，这两个线程也无法看到彼此的ThreadLocal。 可以说ThreadLocal提供了一种代码线程安全的的简单方法。  

下面我们就来看看道哥都用的ThreadLocal。  
<!--more-->


### 1 ThreadLocal你来自哪里
```java
Since: 1.2
Author: Josh Bloch and Doug Lea
```
又是并发大佬们的杰作，膜拜一下。怪不得道哥也爱用，自己设计的类总得用用。下面来看看基本内容与用法吧。
![Doug Lea](http://www.javanorth.cn/assets/images/2021/ThreadLocal/Doug_Lea.jpg)
### 2 ThreadLocal原理
首先请看男神们的介绍

“This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).”

“此类提供了thread-local变量。这些变量不同于普通的类似变量，因为访问某个变量（通过其 get 或 set 方法）的每个线程都有自有的，独立初始化的变量副本，ThreadLocal实例通常是希望将状态与线程（例如，用户ID或事务ID）关联的类中的私有静态字段。”
 
通过老爷子们的描述，指北君大概也知道了ThreadLocal的推荐使用场景，
1. ThreadLocal提供了一种访问某个特有变量的方法 访问到的变量属于当前线程，同一线程在任何地方都能访问同一个线程特有变量。
2. 推荐定义为 private static 类型，但是Doug Lea老爷子在ThreadLocalRandom 和 ReentrantReadWriteLock 中使用了 private static final 类型。（肯定是当年写简介的时候手抖了）

### 2.1 Thread中如何存储

既然是线程的变量，自然是存在Thread对象中的一个变量了，但是它是通过ThreadLocal这个类来维护的。 
```java
//与此线程相关的ThreadLocal值，由ThreadLocal这个类维护
ThreadLocal.ThreadLocalMap threadLocals = null;

//与此线程相关的可继承的ThreadLocal值，由InheritableThreadLocal类来维护
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

ThreadLocal中有一个内部类来ThreadLocalMap来维护这些线程本地变量，
```java
static class ThreadLocalMap {
        private static final int INITIAL_CAPACITY = 16; //初始容量，2的n次方
        private Entry[] table; //根据需要调整数组大小，2的n次方
        private int size = 0;  //上面Entry数组中的元素数量
        private int threshold; //The next size value at which to resize  Default to 0
}
```
ThreadLocalMap中的Entry结构如下，是一种key为弱引用（其目的就是Entry对象在GC时容易回收）的hash map，其中key总是ThreadLocal。
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

### 2.2 常用方法 get，set，remove 详解

+ get()
此方法是ThreadLocal最重要的方法之一，该方法返回此线程局部变量的当前线程副本中的值。
大概可分为以下几步：  
(1) 先获取当前线程，然后再从线程中得到ThreadLocalMap。  
(2) 然后使用ThreadLocal对象的threadLocalHashCode进行散列计算，得到一个数组的index  
(3) 从Table数组中得到Entry，再对比Entry的key是不是可当前的ThreadLocal相等，如果相等就返回此Entry的value  
(4) 如果上一步中得到的Entry与当前ThreadLocal不相等，则会在方法getEntryAfterMiss中进行遍历Entry数组table中的每一个元素，如果找不到就返回null。而且在遍历的过程中会顺便清理一下废弃的Entry。  

下面可以看一下get方法的具体代码。
```java
public T get() {
        Thread t = Thread.currentThread(); //获取当前线程
        ThreadLocalMap map = getMap(t); //从当前线程中获取ThreadLocalMap
        if (map != null) {
                ThreadLocalMap.Entry e = map.getEntry(this); //获取map中当前ThreadLocal对象对应的entry
                if (e != null) {
                        @SuppressWarnings("unchecked")
                        T result = (T)e.value;
                        return result;
                }
        }
        return setInitialValue();
}

private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);//散列计算得到Entry中当前的index
        Entry e = table[i];
        if (e != null && e.get() == key) //如果Entry不是null而且key等于当前ThreadLocal对象则返回此Entry
            return e;
        else
            return getEntryAfterMiss(key, i, e);//Entry==null 或者其key不等于当前ThreadLocal对象，遍历其余Entry
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
        Entry[] tab = table;
        int len = tab.length;
        while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key) return e;
                if (k == null)
                    expungeStaleEntry(i);//如果遍历过程中发现有Entry的Key为Null，则清除掉作废的Entry
                else
                    i = nextIndex(i, len);//计算Entry数组下一个index
                    e = tab[i];
        }
        return null;
}
```
+ set(T value)
此方法将此线程局部变量的当前线程副本中的值设置为指定值。  

set线程本地变量步骤如下：  
(1) 首先依然是获取此线程的ThreadLocalMap  
(2) Map不为null时往map中插入数据，否侧创建map并插入数据  
(3) 具体的set方法依然是先遍历Entry数组中所有的的Entry，然后依次对比每个Entry的key是否等于当前ThreadLocal，如果相等则直接替换现有Entry的value。如果Entry的Key为null，则立马清理废弃的Entry，并用新的Entry来替换此卡槽。    
(4) 如果遍历完都没有return,则在在table中相应卡槽下新建Entry对象  

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
                map.set(this, value);
        else
                createMap(t, value);
 }
 
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        if (k == key) {      //如果原Entry的key就是当前ThreadLocal对象，则直接替换现有value
            e.value = value;
            return;
        }
        if (k == null) {
            replaceStaleEntry(key, value, i); // 如果Entry的Key为null， 则直接替换为新的Entry
            return;
        }
    }
    tab[i] = new Entry(key, value); // 如果前面的遍历没有return，则插入新的Entry对象到对应的卡槽
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
 ```
+ remove()
remove则相对简单，直接遍历ThreadLocalMap中Entry数组table，找到对应的Entry，将Entry的key置为null，然后再清理相应的Entry。

```java
private void remove(ThreadLocal<?> key) {
    ...
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();//Entry 的key置为null
            expungeStaleEntry(i); // 清理对应卡槽，
            return;
        }
    }
}
```

### 3 Java中使用的ThreadLocal
Java中有哪些源码使用了ThreadLocal。  

ThreadLocalRandom 中使用计算nextGaussian值时有使用到ThreadLocal。  

InheritableThreadLocal继承了ThreadLocal，线程中使用inheritableThreadLocals这个map存储线程本地变量。和ThreadLocal的区别就是子线程依然可以访问到父线程的线程本地变量，实际应用中也推荐InheritableThreadLocal  

ReentrantReadWriteLock中线程读写锁的计数器使用了ThreadLocal，其目的是记录每个线程获取读写锁的次数
```java
static final class ThreadLocalHoldCounter extends ThreadLocal<HoldCounter> {
        public HoldCounter initialValue() {
                return new HoldCounter();
        }
}
//曾经的Doug Lea老爷子推荐static field，而他默默的使用了static final。
```

### 4 如何使用ThreadLocal

ThreadLocal非常适合存储非线程安全的对象，并且不需要跨线程共享对象。很多需要线程隔离的操作都可以尝试使用它。

ThreadLocal也非常适合在WEB应用程序中使用，典型的应用就是在Web请求进来一开始就将请求状态存储在ThreadLocal中，然后参与处理的任何组件均可访问该状态。

以下是一个ThreadLocal示例：

具体使用就是配合interceptor或者filter在线程刚开始执行的时候存储SessionContext，线程执行过程中可以随时访问该变量。 然后在线程执行结束的时候再调用remove()方法移除，防止内存泄漏。

```java 
public class SessionContextHolder {
    private static final ThreadLocal<SessionContex> CONTEXHOLDER = new InheritableThreadLocal<>();
    
    public static void remove(){CONTEXHOLDER.remove();};
    
    public static SessionContex get(){return CONTEXHOLDER.get();}
    
    public static void set(SessionContex sessionContex) {CONTEXHOLDER.set(sessionContex);}
}
```


### 总结


本文介绍了ThreadLocal的原理以及解析了常用方法的实现逻辑，以及在ThreadLocal一些应用。在一步步梳理的过程中，果然看到了以往忽略的各种细节，最后给出了一个小Case。并发编程大神道哥.李都在用的ThreadLocal，不妨在自己的项目中偷偷用上，保证丝滑舒适。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
