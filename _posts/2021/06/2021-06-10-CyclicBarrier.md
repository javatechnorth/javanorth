---
layout: post
title:  线程：臣妾不止能抢资源还能合作共赢 --20210722
tagline: by 某某白米饭
categories: CyclicBarrier
tags: 
    - 某某白米饭
---

大家好，我是指北君。

前几天指北君的朋友小 B，写了一个导出 excel 下载太慢被客户嫌弃了。

![](http://www.javanorth.cn/assets/images/2021/CyclicBarrier/0.png)

指北君就让小 B 用 CyclicBarrier 。CyclicBarrier 是一个循环的栅栏，在多个线程完成各自的任务之后，主线程才可以开始执行任务。小 B 的情况就适用于多个线程并行查询数据库，然后写入 excel 的各个 sheet 页，在所有操作完成之后执行汇总数据的算法并将结果写入汇总的 sheet 页。
<!--more-->

下面用一个小 demo，对 CyclicBarrier 有一个初步的印象。

```java
public class Test implements Runnable{
    
    //定义一个循环栅栏
    private CyclicBarrier cyclicBarrier = new CyclicBarrier(3,this);
    //存放每个线程的数据
    private BlockingQueue<Integer> list = new ArrayBlockingQueue<>(3);

    public void count() {
        //运行 3 个线程
        for(int i = 0;i<3;i++){
            new Thread(new Runnable() {

                public void run() {
                    //这里可以存放更复杂的操作，比如查询 SQL、写入 Excel 等等
                    Random r = new Random();
                    int a = r.nextInt(100);
                    System.out.println("线程 " + Thread.currentThread().getName() + "获得数字：" + a);
                    list.add(a);
                    try {
                        //线程到达栅栏后，等待
                        cyclicBarrier.await();
                    }catch (Exception e){
                        e.printStackTrace();
                    }

                }
            }).start();
        }
    }
    //这里就是线程一起跨过栅栏后执行的任务
    public void run() {
        int result = 0;
        for (Integer i: list) {
            result += i;
        }
        System.out.println(result);
    }

    public static void main(String[] args){
        Test test = new Test();
        test.count();
    }
}
```

这个 demo 中一共 3 个线程，每个线程都随机获取一个数字(在实际生产代码中会有更复杂的操作)，最后将每个线程获取的数字相加后打印最后的结果。

### 源码分析

#### 内部类

CyclicBarrier 的一个内部类，Generation 被翻译成为“代”。当这一代的所有线程都到达栅栏后可以开启下一代，所以才被成为循环栅栏。broken 属性表示栅栏是否被打破了。

```java
private static class Generation {
    boolean broken = false;
}
```

#### 属性与构造函数

```java
//可重入的 ReentrantLock 锁，非公平锁
private final ReentrantLock lock = new ReentrantLock();
//lock 的条件队列
private final Condition trip = lock.newCondition();
//线程的数量
private final int parties;
//所有线程到达后，可执行的方法
private final Runnable barrierCommand;
//当前代
private Generation generation = new Generation();
//当前代还需要等待线程的数量
private int count;

public CyclicBarrier (int parties) {
    //调用 2 个参数的构造函数
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    //检查线程的数量是否小于 0
    if (parties <= 0) throw new IllegalArgumentException();
    //设置线程数、需要等待的线程数量
    this.parties = parties;
    this.count = parties;
    //设置所有线程都到达的时候需要运行的方法
    this.barrierCommand = barrierAction;
}
```
 
从上面的内容和 demo 粗略的可以看出，CyclicBarrier 在初始化时设置了线程数量 parties，必须等待所有的线程都到栅栏处 cyclicBarrier.await()  时才可以运行 barrierCommand 方法。

如果还有线程没有到达栅栏处，会将先到达栅栏处的线程放入 trip 条件队列中等待最后一个线程到达。

![](http://www.javanorth.cn/assets/images/2021/CyclicBarrier/1.gif)

### await()

```java
//CyclicBarrier.await()
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        //调用了 dowait()
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe);
    }
}
```

外部调用 await() 方法，等待线程到达栅栏后一起执行后续的操作。await() 可以被复用，每多调用一次 await() 就表示多增加一代，第一次调用是一代、第二次调用是二代、第三次调用是三代...。

### dowait()

dowait() 是 CyclicBarrier 的核心方法。

```java
//CyclicBarrier.dowait()
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
            TimeoutException {
    //加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        
        final Generation g = generation;
        //判断当前代的栅栏是否被打断了
        if (g.broken)
            throw new BrokenBarrierException();
        //线程是否被中断
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }
        //到达一个线程，数量就减去 1，直到最后一个线程
        int index = --count;
        if (index == 0) {
            //是否执行了 barrierCommand 的标识位
            boolean ranAction = false;
            try {
                //所有线程到达后，一起执行的方法
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                //进入下一代
                nextGeneration();
                return 0;
            } finally {
                //barrierCommand 没被执行，打破栅栏
                if (!ranAction)
                    breakBarrier();
            }
        }

        //无限循环，这里只有最后一个线程不进入循环
        for (;;) {
            try {
                //没有设置需要等待的时间
                if (!timed)
                    //到达栅栏的线程进入 trip 条件队列等待被唤醒
                    trip.await();
                //等待的时间还没有超时
                else if (nanos > 0L)
                    //等待指定的时间
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                //出了异常且当前代没有打破栅栏，那么打破栅栏并且抛出异常
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    //中断当前线程
                    Thread.currentThread().interrupt();
                }
            }
            
            //检查
            if (g.broken)
                throw new BrokenBarrierException();
            //不是当前的代
            if (g != generation)
                return index;
            //等待超时了
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        //解锁，出队
        lock.unlock();
    }
}
```

dowait() 的运行被分成了 2 部分：
1. 最后一个线程的时候，进入运行 barrierCommand 方法的流程，并且进入下一代。
2. 前面的其他线程都进入循环中，将线程添加到 trip 的条件队列中，等待最后一个线程将它们唤醒。

![](http://www.javanorth.cn/assets/images/2021/CyclicBarrier/2.png)

### nextGeneration()

```java
//CyclicBarrier.nextGeneration()
 private void nextGeneration() {
    trip.signalAll();
    count = parties;
    generation = new Generation();
}
```

CyclicBarrier 为什么会一代结束后可以开始下一代，就靠这个 nextGeneration() 方法，它干了三件事：
1. trip.signalAll() 方法将 trip() 条件队列中的线程全部转移到 AQS　队列中去。AQS 队列中出队是在 lock.unlock() 的时候。
2. 将线程的数量重置。
3. 初始化一个新的代。 

### 总结

CyclicBarrier 使用了两个队列，一个条件队列，一个 AQS 队列，在 trip.await() 出进入条件队列。当最后一个线程到达栅栏出的时候，条件队列中的线程全部移动到 AQS　队列中，要注意的是最后一个线程并没有进入 AQS 队列中。在 lock.unlock() 的时候 AQS 队列中的线程出队。

CyclicBarrier 基于 ReentrantLock 和 Condition 实现同步线程的逻辑。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
