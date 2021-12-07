---
layout: post
title:  ArrayBlockingQueue 入队和出队的源码分析 --20210702
tagline: by 某某白米饭
categories: ArrayBlockingQueue 源码解读
tags: 
    - ArrayBlockingQueue
---

大家好，我是指北君。

今天我们来聊一聊以数组为数据结构的阻塞队列 ArrayBlockingQueue，它实现了 BlockingQueue 接口，继承了抽象类 AbstractQueue<E>。
<!--more-->

BlockingQueue 提供了三个元素入队的方法。

```java
boolean add(E e);

boolean offer(E e);

void put(E e) throws InterruptedException;
```

三个元素出队的方法。

```java
E take() throws InterruptedException;

E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

boolean remove(Object o);
```

一起来看看，ArrayBlockingQueue 是如何实现的吧。

### 初识

首先看一下 ArrayBlockingQueue 的主要属性和构造函数。

#### 属性

```java
//存放元素
final Object[] items; 

//取元素的索引
int takeIndex;

//存元素的索引
int putIndex;

//元素的数量
int count;

//控制并发的锁
final ReentrantLock lock;

//非空条件信号量
private final Condition notEmpty;

//非满条件信号量
private final Condition notFull;

transient Itrs itrs = null;
```

从以上属性可以看出：
1. 以数组的方式存放元素。
2. 用 putIndex 和 takeIndex 控制元素入队和出队的索引。
3. 用重入锁控制并发、保证线程的安全。

#### 构造函数

ArrayBlockingQueue 有三个构造函数，其中 `public ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c)` 构造函数并不常用，暂且不提。看其中两个构造函数。

```java
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}

public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    //构造数组
    this.items = new Object[capacity];
    //默认以非公平锁初始化 ReentrantLock
    lock = new ReentrantLock(fair);
    //创建两个条件信号量
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```

可以看出 ArrayBlockingQueue 必须再创建时传入数组的大小。


#### 元素入队

ArrayBlockingQueue 有 add()、offer()、put()、offer(E e, long timeout, TimeUnit unit) 方法用来元素的入队。

##### add

```java
//ArrayBlockingQueue.add()
public boolean add(E e) {
    //调用父类的 AbstractQueue.add() 方法
    return super.add(e);
}

//AbstractQueue.add()
public boolean add(E e) {
    //调用 ArrayBlockingQueue.offer()，成功则返回 true，否则抛出异常
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}

//ArrayBlockingQueue.offer()
public boolean offer(E e) {
    //非空检查
    checkNotNull(e);
    //加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //数组满了，返回 false
        if (count == items.length)
            return false;
        else {
            //添加元素
            enqueue(e);
            return true;
        }
    } finally {
        //解锁
        lock.unlock();
    }
}

//ArrayBlockingQueue.enqueue()
private void enqueue(E x) {
    final Object[] items = this.items;
    //直接放到 putIndex 的位置
    items[putIndex] = x;
    //如果索引满了，putIndex 就从 0 开始，为什么呢？
    if (++putIndex == items.length)
        putIndex = 0;
    //数量加一
    count++;
    //数组里面有数据了，对 notEmpty 条件队列进行通知
    notEmpty.signal();
}
```

上面留下了一个坑，索引等于数组的长度的时候，索引就从 0 开始了。其实很简单，这个数组是不是先入先出的，0 索引的数组先入队，也是先出队的。这时候 0 索引的位置就空了，所以 putIndex 到达数组长度的时候就可以从 0 开始。这里可以看出，ArrayBlockingQueue 是绝对不可以修改数组长度的，一旦初始化后长度就不能再改变了。

##### put

```java
//ArrayBlockingQueue.put()
public void put(E e) throws InterruptedException {
    //非空检查
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lockInterruptibly();
    try {
        //数组满了，线程加入 notFull 队列中等待被唤醒
        while (count == items.length)
            notFull.await();
        //添加元素
        enqueue(e);
    } finally {
        //解锁
        lock.unlock();
    }
}
```

##### offer

ArrayBlockingQueue 中有两个 offer() 方法，offer(E e) 和 offer(E e, long timeout, TimeUnit unit)，add() 方法调用的就是 offer(E e) 方法。

```java
//ArrayBlockingQueue.offer(E e, long timeout, TimeUnit unit)
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    //非空检查
    checkNotNull(e);
    //将时间转换为纳秒
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lockInterruptibly();
    try {
        //当数组满了
        while (count == items.length) {
            //时间到了，元素还没有入队，则返回 false
            if (nanos <= 0)
                return false; 
            //线程加入 notFull 队列中，等待被唤醒，到达 nanos 时间返回剩余的 nanos 时间
            nanos = notFull.awaitNanos(nanos);
        }
        //元素入队
        enqueue(e);
        return true;
    } finally {
        //解锁
        lock.unlock();
    }
}
```

以上就是所有的元素入队的方法，可以得出一些结论：
1. add() 元素满了，就抛出异常。
2. offer() 元素满了，返回 false。
3. put() 元素满了，线程阻塞等待被入队。
4. offer(E e, long timeout, TimeUnit unit) 加入超时时间，如果时间到了元素还是没有被入队，则返回 false

#### 移除元素

ArrayBlockingQueue 提供了 poll()、take()、poll(long timeout, TimeUnit unit)、remove() 方法用于元素的出队。

##### poll

ArrayBlockingQueue 中有两个 poll() 方法，poll() 和 poll(long timeout, TimeUnit unit)。

```java
//ArrayBlockingQueue.poll()
public E poll() {
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lock();
    try {
        //没有元素返回 null，否则元素出队
        return (count == 0) ? null : dequeue();
    } finally {
        lock.unlock();
    }
}

//ArrayBlockingQueue.dequeue()
private E dequeue() {
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    //获取 takeIndex 上的元素
    E x = (E) items[takeIndex];
    //设置 takeIndex 索引上的元素为 null
    items[takeIndex] = null;
    //当 takeIndex 长度是数组长度，takeIndex 索引从 0 开始
    if (++takeIndex == items.length)
        takeIndex = 0;
    //元素数量 -1
    count--;

    if (itrs != null)
        //更新迭代器
        itrs.elementDequeued();
    //唤醒 notFull 的等待队列，其中等待的第一个线程可以添加元素了
    notFull.signal();
    return x;
}
```

```java
//ArrayBlockingQueue.poll(long timeout, TimeUnit unit)
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    ////将时间转换为纳秒
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lockInterruptibly();
    try {
        //数组为空，超时还没有元素出队，则返回 null
        while (count == 0) {
            if (nanos <= 0)
                return null;
            //线程加入 notEmpty 中，等待被唤醒，到达 nanos 时间返回剩余的 nanos 时间
            nanos = notEmpty.awaitNanos(nanos);
        }
        //元素出队
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

##### take

```java
//ArrayBlockingQueue.take()
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lockInterruptibly();
    try {
        //无元素
        while (count == 0)
            //将线程加入 notEmpty 的等待队列中，等待被入队的元素唤醒
            notEmpty.await();
        //元素出队
        return dequeue();
    } finally {
        //解锁
        lock.unlock();
    }
}
```

##### remove

```java
//ArrayBlockingQueue.remove()
public boolean remove(Object o) {
    //非空检查
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    //加锁
    lock.lock();
    try {

        if (count > 0) {
            //入队元素的索引
            final int putIndex = this.putIndex;
            //出队元素的索引
            int i = takeIndex;
            do {
                //找到元素
                if (o.equals(items[i])) {
                    removeAt(i);
                    return true;
                }
                //i 等于数组长度的时候，从 0 开始
                if (++i == items.length)
                    i = 0;
            // i == putIndex 说明已经遍历了一遍
            } while (i != putIndex);
        }
        return false;
    } finally {
        //解锁
        lock.unlock();
    }
}

//ArrayBlockingQueue.removeAt()
void removeAt(final int removeIndex) {
    final Object[] items = this.items;
    //需要出队的 removeIndex 正好是 takeIndex
    if (removeIndex == takeIndex) {
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        //更新迭代器
        if (itrs != null)
            itrs.elementDequeued();
    } else {
        final int putIndex = this.putIndex;
        // 循环移动元素，将 next 元素向前移动 1 个
        for (int i = removeIndex;;) {
            int next = i + 1;
            if (next == items.length)
                next = 0;
            if (next != putIndex) {
                items[i] = items[next];
                i = next;
            } else {
                //设置 i 索引的位置为空，putIndex 索引为 i
                items[i] = null;
                this.putIndex = i;
                break;
            }
        }
        count--;
        if (itrs != null)
            itrs.removedAt(removeIndex);
    }
    // 唤醒 notFull 队列中等待的线程，通知可以元素入队了
    notFull.signal();
}
```

以上就是所有的元素出队的方法，可以得出一些结论：
1. poll() 元素出队为空，则返回空
2. take() 元素出队为空的时候，会阻塞线程
3. remove() 元素出队的时候可能会移动数组
4. poll(long timeout, TimeUnit unit) 加入超时时间，如果时间到了还是没有元素需要出队，则返回 null

### 总结

ArrayBlockingQueue 可以被用在生产者和消费者模型中。

1. ArrayBlockingQueue，不能被扩容，初始化被指定容量。
2. 利用 putIndex 和 takeIndex 循环利用数组。
3. 利用了 ReentrantLock 和 两个 Condition 保证了线程的安全。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
