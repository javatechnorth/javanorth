---
layout: post
title:  泡茶喝水？用FutureTask吧！
tagline: by 24只羊
categories: JDK JUC 源码解读
tags: 
    - 24只羊
---



大家好，我是指北君。

告诉大家一个小秘密，其实指北君没事就会出去面试，目的并不是找工作，而是想看看市场行情。最近面了一圈发现，现在的面试题真是“稀奇古怪”，各式各样，这不，指北君就碰到一个大佬，一上来就问我喜不喜欢喝茶，我猜大佬应该喜欢喝茶，就附和说喜欢，结果大佬转手就是一句，"那我们做个题吧，用程序实现一个烧水泡茶的程序"，擦，原来这是挖坑啊，早知道我就说我只喝咖啡了🐶！不过还好，指北君基础扎实😎，用FutureTask实现了这个功能，所以今天指北君就说说这道面试题。

<!--more-->




但在说面试题之前，我们还需要先掌握FutureTask这个类，文章的末尾我会给出面试题以及指北君的答案，当然，如果你很好奇这个面试题，也可以先划到文章底部一览为快。

<br/>


## **一. FutureTask简介**

我们都知道，Java中生成线程两种最常见的方式是继承Thread，和实现Runnable接口。而Thread其实也是实现了Runnable接口，因此这两种启动线程方式最终执行的都是重写了Runnable接口里面的run()方法，但是我们知道，run方法的返回值是void，所以我们通过这两种方式无法获取线程的执行结果。因此，java提供了FutureTask类和Callable接口来满足我们对线程执行结果的需要。FutureTask类实现了RunnableFuture接口，而这个接口实现了Runnable接口和Future接口。

实现了Runnable接口意味着FutureTask也可以传给Thread来启动线程，但你可能会有疑问？这FutureTask不还是继承了Runnable接口，重写了run方法吗？嗯嗯，确实没错，但是我们后面可以看看FutureTask的两个构造方法，其中一个传入的是啥？Callable，而Clllable接口中call方法可以有返回值的，而FutureTask中实现的run方法最终调用的是Callable实例的call方法（另一个构造方法传入的是Runnable，但最终还是通过适配模式将其转变为了Callable）。

好了，那Future接口又是干啥的呢？它其实就是定义了对并发任务的执行及获取其结果的一些操作方法，FutureTask对这些方法进行了实现，现在我们就好好来看看FutureTask的源码吧！

 
 <br/>
 

## **二. 源码解析**

 

## **1. 属性**

```java
private volatile int state;
private static final int NEW          = 0; //新任务
private static final int COMPLETING   = 1; //任务执行中
private static final int NORMAL       = 2; //任务正常结束
private static final int EXCEPTIONAL  = 3; //任务异常
private static final int CANCELLED    = 4; //任务取消
private static final int INTERRUPTING = 5; //任务被中断中
private static final int INTERRUPTED  = 6; //任务已中断


private Callable<V> callable; //被提交的任务
private Object outcome; //任务完成后返回的结果或是异常抛出的错误
private volatile Thread runner; //执行任务的线程
private volatile WaitNode waiters; //等待的线程，是单向链表结构
```

 

state表示任务的执行状态，状态一共有上面的6种，而状态的流转过程一共有下面四种情况：

1. NEW -> COMPLETING -> NORMAL ：任务正常执行并返回
2. NEW -> COMPLETING -> EXCEPTIONAL ：执行中出现异常
3. NEW -> CANCELLED ：任务执行过程中被取消，并且不响应中断
4. NEW -> INTERRUPTING -> INTERRUPTED ：任务执行过程中被取消，并且响应中断

需要注意的是，只要state不为NEW，就说明任务已经执行完了（等看后面的代码就清楚了）。

waiters表示所有等待任务执行完毕的线程的集合，我们看下它的结构：

```java
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```



这是一个典型的单向链表结构，但是这个单向链表在FutureTask中是当做栈使用的，这个栈的出栈与入栈是使用CAS来完成的，所以是线程安全的。

使用线程安全的栈是因为在同一时刻，可能有多个线程在获取执行任务（对任务进行操作，如get，cancel等），如果任务还在执行中，就会将此线程包装成WaitNode放入栈顶，因此需要保证线程安全。出栈同理。waiters就是永远指向栈顶的。

 

## **2. 方法**

方法有许多，我们主要看构造方法、run方法，get方法和cancel方法：

 

### **2.1 构造方法**

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;  
}
```



```java
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

都是初始化属性callable和state，需要注意的是，如果构造函数传入的是Callable对象，则需要通过Executors将其适配成Callable对象。



### **2.2 run方法**

```java
public void run() {
    //如果状态不为NEW 或者 使用CAS操作将runner属性设置为当前线程操作失败的话 则直接返回
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                // 执行任务，获取任务结果（阻塞）
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                // 将抛出的异常过setException方法赋给outcome属性
                setException(ex);
            }
            if (ran)
                // 将获取的结果通过set方法赋给outcome属性
                set(result);
        }
    } finally {
        runner = null;
        int s = state;
        // 防止其他线程将state更改，自旋判断
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```



首先，我们会判断state状态是否为NEW，并通过CAS操作将runner置为本线程（runner此时必须为null，如果不为null，则说明此时有线程在调用），可以看到，runner是在运行时被初始化的。

接着就调用Callable对象的call方法来执行方法，如果执行成功，则调用set方法，否则调用setException方法。

 

### **2.2.1** set**方法、**setException方法

接着我们就来看下set方法和setException方法：

```java
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```

```java
protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}
```

set方法中，我们先将state属性从NEW变为COMPLETING，然后将结果赋给属性outcome，然后再将属性置为NORMAL，最后执行finishCompletion()方法。

我们可以看到，当任务执行完成后，我们才将state从NEW变为COMPLETING，然后赋值完outcome后，又马上变为NORMAL，因此得出两点：

1. 所以state只要不是NEW，就表明任务已经完成了
2. COMPLETING只是一个很短暂的中间状态

setException方法和set方法大同小异，状态变化不同而已。

 

### **2.2.1.1** finishCompletion()方法

我们再看下finishCompletion()方法，此时，任务都执行完了，因此这个方法和run方法finally块里面的代码都是进行善后处理的。finishCompletion()是对属性waiters进行善后（waiters置null并唤醒栈中线程），而finally块里面是对属性runner和states进行善后，我们先说finishCompletion()：

```java
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }
    done();
    callable = null;        // to reduce footprint
}
```

for循环是判断Treiber栈的栈顶节点是否为null，不为null就继续循环，而里面的if条件则是将waiters属性的值置为null，如果不成功，则继续跳到外层for循环，直到waiters为null（所以这个for循环相当于一个自旋操作，目的是为了确保waiters为null）

waiters为null后，我们将进入里面的for循环来遍历整个Treiber栈，将栈里面的线程通过LockSupport.unpart方法一一唤醒，最后执行done方法（是个空方法，提供给子类覆写来执行结束前的额外操作），将callable清理。

 

最后我们跳回到run方法看下finally里面的程序：

```java
finally {
        runner = null;
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
}
```



我们回想下set方法和setException方法，里面已经把status状态转换成COMPLETING或EXCEPTIONAL了，这里为什么还要判断状态是否>=INTERRUPTING，因为多线程环境下，当前线程在执行run方法时，可能另一个线程执行了cancel方法，取消了任务的执行，因此将stats的值改了，所以这也是为什么在set或setException方法中，改变COMPLETING状态时为什么使用了putOrderedInt直接更改status，而不是用compareAndSwapInt比较后再更改，因为此时我们根本不确定原值是COMPLETING还是INTERUPING，可能此时COMPLETING已经被另一个线程更改了。

这里需要特别注意，我们FutureTask中会涉及两种线程，第一种是执行任务的线程，这种一般只有一个，而获取结果的线程则会有多个。

handlePossibleCancellationInterrupt方法里面相当于一个自旋，直到当status不为INTERUPING时就完了。

 

**总结下run方法，一共完成了下面几件事：**

1. runner初始化
2. 调用callable对象的call方法执行任务
3. 任务结束后将state置为中间态COMPLETING，并任务结果赋值给outcome
4. 将state置为终止态NORMAL或EXCEPTIONAL
5. 唤醒Treiber栈中的所有线程
6. 将runner，callable置为null
7. 验证states是否为终止态

 

### **2.3 get方法**

get分为无参和有参，我们看下无参的get方法：

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果state不是属于最终状态，则会执行awaitDone的方法

 

### **2.3.1** awatiDone**方法**

awatiDone方法里面完成了获取结果，响应中断，挂起线程等功能。

```java
private int awaitDone(boolean timed, long nanos) throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```



初始化变量后，我们进入for循环，如果此时任务还未完成，则会进入到下面if分支：

```java
else if (q == null)
    q = new WaitNode();
else if (!queued)
    queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                         q.next = waiters, q);
```



首先在第一个if分支生成一个WaitNode节点，然后在第二个分支将此节点放入栈首。因为调用的是无参构造方法，所以传入的timed==false，则又返回到for开始处，假设此时state的状态变为了中间态COMPLIETING，则会执行下面分支将线程挂起：

```java
else if (s == COMPLETING) // cannot time out yet
    Thread.yield();
```



如果state为终止态，则执行下面分支，q不为null时则将其thread属性置为null，然后返回此时的状态states：

```java
if (s > COMPLETING) {
     if (q != null)
         q.thread = null;
     return s;
 }
```



当检测到线程中断时，则执行下面分支：

```java
if (Thread.interrupted()) {
      removeWaiter(q);
      throw new InterruptedException();
 }
```

 

### **2.3.1.1** removeWaiter**方法**

我们看下removeWaiter方法：

```java
private void removeWaiter(WaitNode node) {
    if (node != null) {
        node.thread = null;
        retry:
        for (;;) {          // restart on removeWaiter race
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                s = q.next;
                if (q.thread != null)
                    pred = q;
                else if (pred != null) {
                    pred.next = s;
                    if (pred.thread == null) // check for race
                        continue retry;
                }
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset, q, s))
                    continue retry;
            }
            break;
        }
    }
}
```

 

我们先将出栈的node的thread属性设置为null，为啥要这样做，是因为我们此时不知道此WaitNode是否在栈顶，所以我们需要在后面的for循环中遍历栈找到此WaitNode位置并移除，而属性thead为null就是我们遍历过程中定位此WaitNode的依据。

如果node在栈顶，则for循环中直接执行最后一个else if ，将栈顶节点的下一个节点变成栈顶节点。需要注意的是，不管此CAS操作是否成功，都需要跳回到for循环外的retry位置，然后执行for循环，遍历完栈中的所有节点。

假如node不在栈顶，则最终会执行第一个else if，将出栈节点的前一个节点的next指向出栈节点的后一个节点（队列的删除操作）。可是为什么后面还有一个if判断呢？因为removeWaiter没有加锁，如果多个线程同时执行，前面一个节点此时被另一个线程将标记为要拿出去栈的节点（因为thred和next都是volatile修饰，因此它们的状态具有可见性），则此时我们需要回到for循环外，再从头遍历栈，删除此节点。所以removeWaiter方法不仅删除传入的节点，可能还会删除在其他线程中标记为需要删除的节点，这样就提升了效率。

我们最后再回到awaitDone方法，如果上面条件都不满足，我们就执行最后一个分支，并执行LockSupport.park(this)，将自己挂起，当任务执行完或调用取消操作时，会调用我们前面讲的finishCompletion方法将所有挂起的线程唤醒，当然，如果有中断，该线程也会被唤醒。

 

### **2.4 cancel方法**

下面就是Cancel方法的源码

```java
public boolean cancel(boolean mayInterruptIfRunning) {
     if (!(state == NEW &&
           UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
               mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
         return false;
     try {    // in case call to interrupt throws exception
         if (mayInterruptIfRunning) {
             try {
                 Thread t = runner;
                 if (t != null)
                     t.interrupt();
             } finally { // final state
                 UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
             }
         }
     } finally {
         finishCompletion();
     }
     return true;
 }
```



首先看第一个if，如此时的state不是NEW状态，则会直接返回false，这不就对应着前面所讲的"如果此任务处于已经完成、已被取消过、或其他原因不能被取消这三种情况的一种，则此次取消操作失败"吗？

我们继续看if中的代码：

```java
UNSAFE.compareAndSwapInt(this, stateOffset, NEW, 
                         mayInterruptIfRunning ? INTERRUPTING : CANCELLED
```



我们会根据传入布尔值mayInterruptIfRunning来决定将NEW状态置为中间态INTERRUPTING或终止态CANCELLED，你看这里是不是和前面讲的run方法中finally块中的内容对上了，是不是很爽。

然后继续看try代码块中的内容：

```java
try {    // in case call to interrupt throws exception
     if (mayInterruptIfRunning) {
         try {
             Thread t = runner;
             if (t != null)
                 t.interrupt();
         } finally { // final state
             UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);          }
         }
 } finally {
          finishCompletion();
 }
```



前面讲了，runner就是真正执行任务的线程，所以此时调用此线程的interrupt方法，最后在finnally块中将状态置为INTERRUPTED，这里我们需要注意的是，我们知道Thread的interrupt方法不一定会中断线程，那大家可能会想，那这cancel方法还有啥用啊？因为FutureTask是提供给我们获取线程任务结果的，我们只要使FutureTask的结果为null，管它任务真结束还是假结束。还记得run方法中的set方法吗，只有当此时state为NEW，才会把任务执行结果赋值给outcome，但此时如果cancel方法中的if方法成功了，那states就不是NEW了，则outcome是不会被赋值的。所以是不是前后都串起来了？！

最后返回true给用户告诉他执行cancel方法成功了。

 
 <br/>
 

## **三. 面试题之烧水喝茶**

 

最后来看看我们的面试题：烧水喝茶

题目是这样的，我们喝茶之前一般都会有准备工作，一是洗杯子，二是烧水，而且这两个是可以同时进行的，当这两步都完成后，我们才可以泡水喝茶了，所以我们怎么通过代码实现这个喝茶的步骤？这个例子的关键就是运用FutureTask来获取洗杯子和烧水线程的结果，当结果都为ture时，我们才能喝茶。具体程序如下：

```java
public class FutureTaskDemo {

    public static final int SLEEP_TIME = 10000;

    // 清洗杯子
    static class ClearCup implements Callable<Boolean> {

        @Override
        public Boolean call() throws Exception {
            System.out.println("洗杯子啦");
            Thread.sleep(SLEEP_TIME);
            System.out.println("杯子洗完啦");
            return true;
        }
    }

    // 烧热水
    static class BoilWater implements Callable<Boolean> {
        @Override
        public Boolean call() throws Exception {
            System.out.println("开始烧水啦");
            Thread.sleep(SLEEP_TIME);
            System.out.println("烧水完成案例");
            return true;
        }
    }

   static void drinkWater(boolean clearCupIsOk, boolean boilWaterIsOk) {
        if (clearCupIsOk && boilWaterIsOk) {
            System.out.println("可以泡茶喝啦");
        } else {
            if (!clearCupIsOk) {
                System.out.println("茶杯清洗失败");
            }
            if (!boilWaterIsOk) {
                System.out.println("烧水失败");
            }
        }
    }

    public static void main(String[] args) throws Exception {
        // 建立清洗杯子线程
        Callable<Boolean> clearCup = new ClearCup();
        FutureTask<Boolean> cTask = new FutureTask(clearCup);
        Thread clearCupThread = new Thread(cTask);
        // 建立烧水线程
        Callable<Boolean> boilCup = new BoilWater();
        FutureTask<Boolean> bTask = new FutureTask(boilCup);
        Thread boilCupThread = new Thread(bTask);

        // 开启两个线程
        clearCupThread.start();
        boilCupThread.start();

        // 获取线程结果
        boolean clearCupIsOk =  cTask.get();
        boolean boilWaterIsOk = bTask.get();

        //喝水
        drinkWater(clearCupIsOk, boilWaterIsOk);

    }

}
```

<br/>


## 四. 小结

好了，FutureTask源码和如何用FutureTask解决泡茶喝水这道面试题就讲完了，指北君这里还有许多碰到的有趣的面试题，如果大家感兴趣，可以关注我哈，后期指北君会一一告诉大家。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！


