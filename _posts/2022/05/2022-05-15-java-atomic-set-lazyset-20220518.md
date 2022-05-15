---
layout: post
title:  Java 原子变量中set()和lazySet()的区别 --20220518
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在本教程中，我们将研究 Java atomic 类（如 `AtomicInteger` 和 `AtomicReference` ）的方法 `set()` 和 `lazySet()` 之间的区别。

### 原子变量

Java中的原子变量使我们能够轻松地对类的引用或字段进行线程安全的操作，而不需要添加监视器或互斥等并发原语。

它们被定义在 `java.util.concurrent.atomic` 包下，虽然它们的API根据原子类型的不同而不同，但大多数都支持`set()`和`lazySet()`方法。

为了简单起见，我们将在本文中使用 `AtomicReference` 和 `AtomicInteger`，但同样的原则适用于其他原子类型。

### 3.The `set()` 方法

在调用`set()`后，当我们从不同的线程使用`get()`方法访问该字段时，该变化是立即可见的。这意味着该值被从CPU缓存中刷新到了所有CPU核共有的内存层。
为了展示上述功能，让我们创建一个最小的 producer-consumer 控制台应用。

```java
    public class Application {
    
        AtomicInteger atomic = new AtomicInteger(0);
    
        public static void main(String[] args) {
            Application app = new Application();
            new Thread(() -> {
                for (int i = 0; i < 10; i++) {
                    app.atomic.set(i);
                    System.out.println("Set: " + i);
                    Thread.sleep(100);
                }
            }).start();
    
            new Thread(() -> {
                for (int i = 0; i < 10; i++) {
                    synchronized (app.atomic) {
                        int counter = app.atomic.get();
                        System.out.println("Get: " + counter);
                    }
                    Thread.sleep(100);
                }
            }).start();
        }
    }
```

在控制台，我们应该看到一系列的 "设置 "和 "获取 "信息。

```
    Set: 3
    Set: 4
    Get: 4
    Get: 5
```

表明[缓存一致性](https://en.wikipedia.org/wiki/Cache_coherence)的是，"Get "语句中的值总是等于或大于其上方的 "Set "语句中的值。。

这种行为虽然非常有用，但也带来了性能上的影响。如果我们能在不需要缓存一致性的情况下避免它，那就太好了。

### The `lazySet()` 方法

`lazySet()`方法与`set()`方法相同，但没有缓存刷新。

换句话说，我们的变化最终只对其他线程可见。这意味着从不同的线程对更新的 `AtomicReference` 调用 `get()`可能会给我们带来旧的值。

为了看到这一点，让我们在之前的控制台应用程序中改变第一个线程的`Runnable`。

```java
for (int i = 0; i < 10; i++) {
    app.atomic.lazySet(i);
    System.out.println("Set: " + i);
    Thread.sleep(100);
}
```

新的 "设置 "和 "获取 "信息可能不总是递增的。

```java
Set: 4
Set: 5
Get: 4
Get: 5
```

由于线程的特性，我们可能需要重新运行几次应用程序，以便触发这种行为。尽管生产者线程已经将`AtomicInteger`设置为5，但消费者线程还是先检索到了值4，这意味着当`lazySet()`被使用时，系统最终是一致的。

在更多的技术术语中，我们说`lazySet()`方法在代码中不作为发生在前的边，与它们的`set()`对应的方法相反。

### 什么时候使用`lazySet()`？

我们并不清楚什么时候应该使用`lazySet()`，因为它与`set()`的区别很微妙。我们需要仔细分析这个问题，不仅要确保我们会得到性能上的提升，还要确保在多线程环境下的正确性。

我们可以使用的一种方式是，一旦我们不再需要一个对象的引用，就用`null`替换它。这样，我们表明该对象有资格进行垃圾回收，而不会产生任何性能上的损失。我们假设其他线程可以使用废弃的值，直到他们看到`AtomicReference`是`null`。
不过一般来说，我们应该使用`lazySet()`，当我们想对一个原子变量进行修改，而且我们知道这个修改不需要立即对其他线程可见。

### 总结

在这篇文章中，我们看了原子类的`set()`和`lazySet()`方法之间的区别。我们还学习了何时使用哪种方法。
