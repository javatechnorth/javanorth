---
layout: post
title:  LinkedBlockingQueue 的入队与出队解析
tagline: by 某某白米饭
categories: LinkedBlockingQueue
tags: 
    - LinkedBlockingQueue
---

大家好，我是指北君。

在前面的文章中，已经对 ArrayBlockingQueue 进行了一次源码分析，对它的核心源码做了分析，今天来解析一波同为 BlockingQueue 家族中的一员的 LinkedBlockingQueue。它的底层基于单向链表实现。
<!--more-->
![](http://www.javanorth.cn/assets/images/2021/LinkedBlockingQueue/0.png)

先看一看它的 Node 内部类和主要属性、构造函数。

### Node

```java
static class Node<E> {
    E item;
    Node<E> next;

    Node(E x) { item = x; }
}
```

Node 是 LinkedBlockingQueue 的基石。 它如第一张图所示的一个单向链表形式的内部类，item 是当前节点的内容，next 指向的是下一个 Node 节点。

### 属性

```java
//容量
private final int capacity;

//队列中元素的数量
private final AtomicInteger count = new AtomicInteger();

//链表的头节点
transient Node<E> head;

//链表的尾节点
private transient Node<E> last;

//出队锁
private final ReentrantLock takeLock = new ReentrantLock();

//当队列没有元素了，出队锁的线程会加入 notEmpty 条件队列中被阻塞，等待其它线程唤醒
private final Condition notEmpty = takeLock.newCondition();

//入队锁
private final ReentrantLock putLock = new ReentrantLock();

//当队列满了，入队锁的线程会加入 notFull 条件队列中被阻塞，等待其它线程唤醒
private final Condition notFull = putLock.newCondition();

```

1. 从属性中就可以看出 LinkedBlockingQueue 和 ArrayBlockingQueue 的差异点：
    ArrayBlockingQueue 只有一把 ReentrantLock 锁，入队和出队相互互斥。
    LinkedBlockingQueue 分了出队锁 takeLock 和入队锁 putLock，两把锁相互不阻塞。
2. capacity 是 LinkedBlockingQueue 的容量，表示 LinkedBlockingQueue 是一个有界队列。

### 构造函数

LinkedBlockingQueue 提供了三个构造函数。

![](http://www.javanorth.cn/assets/images/2021/LinkedBlockingQueue/1.png)

#### LinkedBlockingQueue()

```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}
```

LinkedBlockingQueue() 构造函数直接调用带参数的构造函数，参数被设置为 2 的 31 次方 - 1

#### LinkedBlockingQueue(int capacity)

```java
public LinkedBlockingQueue(int capacity) {
    //检查容量大小
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    //构造一个 null 节点
    last = head = new Node<E>(null);
}
```

LinkedBlockingQueue(int capacity) 构造函数做了三件事：
1. 先检查参数是否大于 0，不大于 0 就抛出异常。
2. 设置 capacity 容量为参数大小。
3. 构造了一个 null 节点，赋值给 last 和 head。
4. head 的 item 元素永远是一个 null。

#### LinkedBlockingQueue(Collection<? extends E> c)

```java
public LinkedBlockingQueue(Collection<? extends E> c) {
    //容量是最大值
    this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
    //加锁
    putLock.lock();
    try {
        int n = 0;
        for (E e : c) {
            if (e == null)
                throw new NullPointerException();
            if (n == capacity)
                throw new IllegalStateException("Queue full");
            
            //真实的入队操作
            enqueue(new Node<E>(e));
            ++n;
        }
        //设置元素的数量
        count.set(n);
    } finally {
        //解锁
        putLock.unlock();
    }
}
```

1. LinkedBlockingQueue(Collection<? extends E> c) 调用了 LinkedBlockingQueue(int capacity) 构造函数并将参数设置成了最大值。
2. putLock 加锁后，将集合中的元素一个个遍历并且入队。
3. 设置元素的数量就是集合中元素的数量。
4. 在遍历元素时，会将 null 元素抛出异常并且检查队列是否满了。

### 入队

LinkedBlockingQueue 有多种入队方法，当队列满了之后它们的处理方法各不相同。在入队之前和入队之后的操作都是相同的。

#### offer

```java
//LinkedBlockingQueue.offer()
public boolean offer(E e) {
    //检查是否为 null
    if (e == null) throw new NullPointerException();
    //检查队列是否满了，队列满了返回 fasle
    final AtomicInteger count = this.count;
    if (count.get() == capacity)
        return false;
    //初始化一个 size
    int c = -1;
    //创建一个 Node 节点
    Node<E> node = new Node<E>(e);
    //putLock 加锁
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        //如果没有满队，则入队
        if (count.get() < capacity) {
            //入队
            enqueue(node);
            //返回元素的数量并且元素的数量 + 1
            c = count.getAndIncrement();
            //元素的数量 + 1 还没有满队，还有剩余的容量空间，唤醒 notFull 队列中因为队列满了而等待入队的线程。
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        //解锁
        putLock.unlock();
    }
    //当 c == 0 表示元素入队之前是一个空的队列
    //现在队列不是空的了，唤醒阻塞在 notEmpty 条件队列中因为空队列而等待元素出队的线程
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}

//LinkedBlockingQueue.enqueue()
private void enqueue(Node<E> node) {
    //直接加到 last 的 next 属性中，并替换 last 属性
    last = last.next = node;
}

//LinkedBlockingQueue.signalNotEmpty()
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
```

offer() 方法做了以下几件事情：
1. 检查需要入队的元素是否为 null。
2. 判断队列是否满了，满了就返回 false。
3. 队列没有满，创建一个新的 Node 节点。
4. putLock 锁加锁，不让其他线程操作队列。
5. 当队列没有满队的时候，将元素入队并且将局部变量 c 设置为入队之前元素的数量，元素数量 + 1。
6. 再次判断队列是否满了，如果队列中还有空位，则唤醒正在阻塞的入队线程。这些阻塞的线程来自 put()、offer(E e, long timeout, TimeUnit unit) 方法。
7. 释放 putLock 锁
8. 当入队之前是一个空队列的时候，调用 signalNotEmpty() 方法开启 takeLock 出队锁，将阻塞在 notEmpty 条件队列中的线程唤醒。

enqueue() 方法的源码比较简单，就是将 last 节点的 next 指向需要入队的元素，如下图所示。

![](http://www.javanorth.cn/assets/images/2021/LinkedBlockingQueue/2.png)

#### add()

```java
//LinkedBlockingQueue.add()
public boolean add(E e) {
    //调用 offer()
    if (offer(e))
        return true;
    else
        //队列满了，就抛出异常
        throw new IllegalStateException("Queue full");
}
```
add() 方法调用的是 offer() 方法，它在队列满了的时候不是返回 false 而是抛出一个 `Queue full` 异常。

#### put()

```java
//LinkedBlockingQueue.put()
public void put(E e) throws InterruptedException {
    //检查是否为 null
    if (e == null) throw new NullPointerException();
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    //putLock 加锁
    putLock.lockInterruptibly();
    try {
        //如果队列满了，就把线程加入 notFull 队列，阻塞线程
        while (count.get() == capacity) {
            notFull.await();
        }
        //入队
        enqueue(node);
        //返回元素的数量并且元素的数量 + 1
        c = count.getAndIncrement();
        //元素的数量 + 1 还没有满队，还有剩余的容量空间，唤醒 notFull 队列中因为队列满了而等待入队的线程
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        //解锁
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```

put() 方法和 offer()、and() 的方法大致相同，不同的是对队列满了之后的操作，offer() 是直接返回 false，and() 是抛出异常，put() 则是将线程加入到 notFull 条件队列中阻塞入队线程。


#### offer(E e, long timeout, TimeUnit unit)

这是一个带超时时间的 offer() 方法。

```java
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    //检查是否为 null
    if (e == null) throw new NullPointerException();
    //将 timeout 超时转换为毫秒数
    long nanos = unit.toNanos(timeout);
    int c = -1;
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    //加锁
    putLock.lockInterruptibly();
    try {
        //当队列满了之后，等待超时时间，如果超时时间到了，还没入队则返回 false
        while (count.get() == capacity) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        //入队
        enqueue(new Node<E>(e));
        //返回元素的数量并且元素的数量 + 1
        c = count.getAndIncrement();
        //元素的数量 + 1 还没有满队，还有剩余的容量空间，唤醒 notFull 队列中因为队列满了而等待入队的线程
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        //解锁
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
    return true;
}
```

这个方法是在一定时间内元素等待入队，就是在 timeout 时间内队列中有空余位置就将元素加入单向链表的队尾，如果在 timeout 时间内元素还没有入队，就返回 false。

#### 入队总结

LinkedBlockingQueue 的入队一共有 offer()、add()、put()、offer(E e, long timeout, TimeUnit unit) 四种方法。这四种方法在队列满了之后的处理是不同的：

1. offer() 方法在队列满了之后就返回 false。
2. add() 方法在内部调用的是 offer() 方法，当队列满了就抛出 `Queue full` 异常。
3. put() 方法在队列满了之后会将线程加入 notFull 中，等待被唤醒后加入队列。
4. offer(E e, long timeout, TimeUnit unit) 方法在队列满了之后会等待一段 timeout 的时间，在这时间内入队就返回 true，在这段时间内未能入队就返回 false。
5. 每个方法在入队完后都会唤醒在 notEmpty 队列中等待出队的线程。

### 出队

LinkedBlockingQueue 的出队也有几种不同的方法，它们对于空队列的处理方式各不相同。

#### poll()

```java
//LinkedBlockingQueue.poll()
public E poll() {
    //元素的数量
    final AtomicInteger count = this.count;
    //队列中的非空检查
    if (count.get() == 0)
        return null;
    //初始化一个 null 对象
    E x = null;
    //初始化元素的个数
    int c = -1;
    //takeLock 出队加锁
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        //当队列中有元素
        if (count.get() > 0) {
            //出队
            x = dequeue();
            //取出出队之前的元素数量并且元素的数量 - 1
            c = count.getAndDecrement();
            //当队列中还有元素，唤醒 notEmpty 条件队列中的线程
            if (c > 1)
                notEmpty.signal();
        }
    } finally {
        //解锁
        takeLock.unlock();
    }
    //如果出队之前队列是满的，就唤醒 putLock 锁中的 notFull 条件队列中等待的线程
    if (c == capacity)
        signalNotFull();
    return x;
}

//LinkedBlockingQueue.dequeue()
private E dequeue() {
    //head 节点没有存储元素，head.next 是第一个元素
    Node<E> h = head;
    //取出第一个元素
    Node<E> first = h.next;
    //将 head 节点的 next 指向自己
    h.next = h;
    //将第一个元素变成新的 head
    head = first;
    //取出内容
    E x = first.item;
    first.item = null;
    return x;
}
```

poll() 方法在出队的时候做了以下几件事情：
1. 先取出队列中元素的数量
2. 队列的非空检查，当队列是空的，则返回 false。
3. 初始化一个局部的元素变量。
4. takeLock 出队锁加锁，不让其他线程操作队列的出队。
5. 当队列中有元素的时候，将队列中的第一个元素弹出。
6. 判断队列中是否还有元素，唤醒阻塞在 notEmpty 条件队列上的线程。
7. takeLock 出队锁解锁
8. 在原队列是满队的情况下，唤醒阻塞在 notFull 条件队列上的线程。

dequeue() 方法会将 LinkedBlockingQueue 中第一个元素取出。取的并不是 head 中的item，而是 head.next 中的 item。

![](http://www.javanorth.cn/assets/images/2021/LinkedBlockingQueue/3.png)

#### poll(long timeout, TimeUnit unit)

```java
//LinkedBlockingQueue.poll(long timeout, TimeUnit unit)
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E x = null;
    int c = -1;
    //将超时时间转换为毫秒数
    long nanos = unit.toNanos(timeout);
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    //takeLock 出队加锁
    takeLock.lockInterruptibly();
    try {
        //队列中是空的，没有元素
        while (count.get() == 0) {
            //等待超时时间， 超时后返回 null
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        //出队
        x = dequeue();
        //取出出队之前的元素数量并且元素的数量 - 1
        c = count.getAndDecrement();
        //当队列中还有元素，唤醒 notEmpty 条件队列中的线程
        if (c > 1)
            notEmpty.signal();
    } finally {
        //解锁
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

poll(long timeout, TimeUnit unit) 方法是在一定时间内出队还未取到元素就阻塞线程，时间到了还没取到元素就返回 null，并不会一直阻塞线程。

#### take()

```java
//LinkedBlockingQueue.take()
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    //加锁
    takeLock.lockInterruptibly();
    try {
        //队列是空的，将线程加入到 notEmpty 队列中等待并且阻塞线程
        while (count.get() == 0) {
            notEmpty.await();
        }
        //出队
        x = dequeue();
        //取出出队之前的元素数量并且元素的数量 - 1
        c = count.getAndDecrement();
        //当队列中还有元素，唤醒 notEmpty 条件队列中的线程
        if (c > 1)
            notEmpty.signal();
    } finally {
        //解锁
        takeLock.unlock();
    }
    //如果出队之前队列是满的，就唤醒 putLock 锁中的 notFull 条件队列中等待的线程
    if (c == capacity)
        signalNotFull();
    return x;
}
```
take() 方法和 poll() 最大的不同就是在空队列的时候会一直阻塞线程，poll() 则返回 null，poll(long timeout, TimeUnit unit) 则在一定时间内阻塞线程，超时后返回的 null。


#### remove()

```java
//LinkedBlockingQueue.remove()
public boolean remove(Object o) {
    //非空检查
    if (o == null) return false;
    //将入队锁和出队锁都加锁
    fullyLock();
    try {
        //遍历所有元素
        for (Node<E> trail = head, p = trail.next;
                p != null;
                trail = p, p = p.next) {
            if (o.equals(p.item)) {
                //删除节点
                unlink(p, trail);
                return true;
            }
        }
        return false;
    } finally {
        //将入队锁和出队锁都解锁
        fullyUnlock();
    }
}

//LinkedBlockingQueue.unlink()
void unlink(Node<E> p, Node<E> trail) {
    //p 是需要删除的节点，trail 是 p 的上一个节点
    p.item = null;
    //将 trail.next 指向 p 的下一个节点
    trail.next = p.next;
    //要删除的就是最后一个节点
    if (last == p)
        //将 trail 设置为最后一个节点
        last = trail;
    //原队列是满的队列，唤醒 notFull 条件队列中的线程
    if (count.getAndDecrement() == capacity)
        notFull.signal();
}
```

remove() 并不会弹出元素，它是删除一个元素。遍历整个单向链表，找到需要删除的元素后，将元素前一个节点的next 指向删除元素的 next。将需要删除的元素设置为 null。

#### peek()

```java
//LinkedBlockingQueue.peek()
public E peek() {
    //非空检查
    if (count.get() == 0)
        return null;
    final ReentrantLock takeLock = this.takeLock;
    //加锁
    takeLock.lock();
    try {
        //取出第一个元素，为 null 则返回 null
        Node<E> first = head.next;
        if (first == null)
            return null;
        else
            return first.item;
    } finally {
        //解锁
        takeLock.unlock();
    }
}
```

peek() 方法仅仅是取出第一个元素，没有修改节点的任何一个 next 属性，所以并不会将元素从队列中移除。

#### 出队总结

LinkedBlockingQueue 的出队一共有 poll()、take()、poll(long timeout, TimeUnit unit) 三种方法，移除元素用 remove() 方法，取出第一个元素用 peek() 方法。

出队方法在遇到空队列的时候操作不同：

1. poll() 方法遇到空队列就返回 null。
2. take() 方法遇到空队列就将线程加入到 notEmpty 条件队列中并且阻塞线程。
3. poll(long timeout, TimeUnit unit) 方法在遇到空队列就阻塞一段时间，这期间没获取到元素就返回 null。

### 总结

1. LinkedBlockingQueue 是基于单向链表的，线程安全的。
2. 是一个有界的队列，最大的容量是最大的 int 值。
3. 出队入队基于两把锁。互不阻塞。

LinkedBlockingQueue 被用在线程池中，也可以用在生产者-消费者模式中。

```java
ExecutorService executorService = Executors.newFixedThreadPool(10);

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
