---
layout: post
title:  为JVM应用程序添加关闭钩子
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

通常，启动一个服务是很容易的。然而，有时我们需要有一个计划来优雅地关闭一个服务。

在本教程中，我们将看一下 JVM 应用程序终止的不同方式。然后，我们将使用 Java APIs 来管理 JVM 关闭钩子。

<!--more-->

### 关闭 JVM

JVM 可以通过两种不同的方式被关闭。

- 一个受控的过程
- 一种非受控的方式

一个受控的进程在以下两种情况下关闭 JVM。

- 最后一个非 daemon 线程终止。例如，当主线程退出时，JVM 开始其关闭进程
- 从操作系统发送一个中断信号。例如，通过按 Ctrl + C 或注销操作系统
- 从Java代码中调用 System.exit()
  
虽然我们都在努力争取优雅的关闭，但有时 JVM 可能会以突然和意外的方式关闭。JVM 会在以下情况下突然关闭。

- 从操作系统发送一个杀戮信号。例如，通过发出kill -9 <jvm_pid> 的信号
- 从Java代码中调用Runtime.getRuntime().halt()。
- 主机操作系统意外死亡，例如，在电源故障或操作系统恐慌的情况下

### shutdown hook

JVM 允许在完成关机之前运行注册函数。这些函数通常是释放资源或其他类似的内部管理任务的好地方。在 JVM 的术语中，这些函数被称为关闭钩子。

关闭钩子基本上是初始化但未启动的线程。当JVM开始其关闭过程时，它将以一个未指定的顺序启动所有注册的钩子。在运行完所有钩子后，JVM 将停止运行。

#### 添加钩子

为了添加一个关闭钩子，我们可以使用 `Runtime.getRuntime().addShutdownHook()` 方法。

```java
Thread printingHook = new Thread(() -> System.out.println("我要关闭了"));
Runtime.getRuntime().addShutdownHook(printingHook);
```

在这里，我们只是在JVM自行关闭之前向标准输出端打印一些东西。如果我们像下面这样关闭JVM。

```java
> System.exit(123);
我要关闭了
```

然后我们会看到，钩子实际上是将消息打印到标准输出。

JVM负责启动钩子线程。因此，如果给定的钩子已经被启动了，Java将抛出一个异常。

```java
Thread longRunningHook = new Thread(() -> {
    try {
        Thread.sleep(300);
    } catch (InterruptedException ignored) {}
});
longRunningHook.start();

assertThatThrownBy(() -> Runtime.getRuntime().addShutdownHook(longRunningHook))
  .isInstanceOf(IllegalArgumentException.class)
  .hasMessage("钩子正在运行");
```

很明显，我们也不能多次注册一个钩子。

```java
Thread unfortunateHook = new Thread(() -> {});
Runtime.getRuntime().addShutdownHook(unfortunateHook);

assertThatThrownBy(() -> Runtime.getRuntime().addShutdownHook(unfortunateHook))
  .isInstanceOf(IllegalArgumentException.class)
  .hasMessage("钩子已经注册");
```

#### 删除钩子

Java 提供了一个孪生的移除方法，以便在注册一个特定的关闭钩子后将其移除。

```java
Thread willNotRun = new Thread(() -> System.out.println("钩子不会运行的"));
Runtime.getRuntime().addShutdownHook(willNotRun);

assertThat(Runtime.getRuntime().removeShutdownHook(willNotRun)).isTrue();
```

当关闭钩子被成功删除时，removeShutdownHook() 方法返回true。

#### 注意事项

JVM 只在正常终止的情况下运行关闭钩子。因此，当外部力量突然杀死JVM进程时，JVM将没有机会执行关闭钩子。此外，从Java代码中停止JVM也会产生同样的效果。

```java
Thread haltedHook = new Thread(() -> System.out.println("Halted abruptly"));
Runtime.getRuntime().addShutdownHook(haltedHook);
        
Runtime.getRuntime().halt(123);
```

halt 方法强行终止了当前运行的JVM。因此，注册的关闭钩子不会有机会执行。

### 总结

在本教程中，我们研究了 JVM 应用程序可能终止的不同方式。然后，我们使用一些运行时API来注册和取消注册关闭钩子。