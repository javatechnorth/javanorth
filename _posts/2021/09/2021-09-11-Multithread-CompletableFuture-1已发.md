---
layout: post
title:  Doug Lea 大师的艺术之作CompletableFuture （一）——20211001
tagline: by 揽月中人
categories: Thread
tags:
- 揽月中人
---

completeFuture作为Doug Lea大师的有一个艺术作品，把并发编程的艺术发挥到极致。之前的Future为我们提供了多线程执行返回值的使用问题。而CompletableFuture则为我们提供了一种线程执行结果连续处理的能力。为多线程编程提供了更为简洁的方式。关于CompletableFuture，我会分为多篇来讲，今天会为大家介绍一下其基本方法以及不同名称参数的方法含义及作用。

<!--more-->

### 1 Completable Future是什么

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> 
```

CompletableFuture 实现了Future，那么也就会有Future的全部功能(其实就是5个方法)。

CompletionStage 是JDK1.8 Doug Lea 设计的一个接口，旨在提高多任务协同执行的编码效率。是一个可异步完成的Stage，可以根据当前Stage的完成情况连续调用，且支持异步计算的各种方法。并有发生异常时的处理方法。

一个Stage的action的执行，可以是在单个stage完成之后，两个stage都完成之后，或者两个stage中有一个正常完成之后。

CompletableFuture实现了CompletionStage接口，那么其stage的这种设计方式也得到了实现。



### 2 Completable Future 构造及简单使用

#### 2.1 创建CompleteableFuture的方法

- 可以直接new一个CompleteableFuture。
- 可以调用supplyAsync静态方法去异步执行，返回一个带有result的CompleteableFuture。
- 静态方法runAsync异步执行后返回一个没有result的CompleteableFuture
- 也可以使用静态方法completedFuture，直接返回一个完成状态的CompleteableFuture。

示例

```java
public class CompletableFutureDemo1 {
    public static void main(String[] args) {

        CompletableFuture<String> cf1 = new CompletableFuture<String>();
        CompletableFuture<String> completedFuture = CompletableFuture.completedFuture("completedFuture");
        CompletableFuture<String> stringCompletableFuture = CompletableFuture.supplyAsync(() -> {
            return "supplyAsyncFuture";
        });
        CompletableFuture<Void> runAsyncCompleteableFuture = CompletableFuture.runAsync(() -> {
            System.out.println("runAsyncCompleteableFuture");
        });
    }
}
```



CompleteableFuture 其主要方法大多为CompletionStage相关的方法。

- thenCompose 平行处理两个
- thenCombine 聚合两个stage的result
- thenAccept 直接消费
- thenRun 直接执行
- theApply 

我们先列出来，后面会一一讲到。

#### 2.2 从CompleteableFuture中获取执行结果

相关方法有 

- join() 没有抛出检查的异常，会阻塞等待执行结果。
- get() 会抛出检查异常，会阻塞等待执行结果
- getNow(T valueIfAbsent) 不会阻塞



### 3 Completable Future 各方法介绍

CompleteableFuture实现了CompletionStage，其相关方法比较多。我们分组来介绍。

#### 3.1 主要参数为Supplier的方法

Supplier的Lamada表达式没有输入值，但是有返回值。

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) 
```

supplyAsync 会异步执行Supplier方法，然后返回一个带有Result的CompletableFuture。



```java
public CompletableFuture<T> completeAsync(Supplier<? extends T> supplier) 
public CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)
```

**completeAsync(Supplier<? extends T> supplier)** 

使用默认的线程池来异步执行Supplier方法，将其结果作为此CompletableFuture的result，来完成此CompletableFuture. 另一个方法为使用自定义线程池。



#### 3.1 主要参数为Runnable的方法

Runnable标记了函数式接口，其唯一的方法**无参数，且无返回值**。所以相关方法中返回的的CompletableFuture也不包含返回值。相关Runable的方法如下。

##### 3.1.1 静态方法。

```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor)
```

**runAsync(Runnable runnable)** 是一个静态方法，在给定的任务完成后，将使用ForkJoinPool.commonPool() 线程池来异步执行相关任务，并返回CompletableFuture，但是CompletableFuture中没有执行的结果。Runable为参数的方法中返回的CompletableFuture对象都不包含其执行结果。

**runAsync(Runnable runnable,Executor executor)**也是静态方法，不同的是使用了自定义的线程池。



##### 3.1.2 thenRun系列

```java
public CompletableFuture<Void> thenRun(Runnable action)
public CompletableFuture<Void> thenRunAsync(Runnable action) 
public CompletableFuture<Void> thenRunAsync(Runnable action,Executor executor) 
```

**thenRun(Runnable action)** 是CompletableFuture对象的一个方法，表示在调用方即CompletionStage（CompletableFuture实现了CompletionStage，所以也就可以表示任务完成的状态）正常完成的情况下会执行这个动作。

thenRunAsync(Runnable action) 表示在当前调用方正常完成任务时异步执行此动作，默认使用ForkJoinPool.commonPool()线程池来执行任务。

同理 **thenRunAsync(Runnable action,Executor executor)** 可以使用自定义线程池来异步执行

**小结：**

- 方法中带有**Async**，则表示当前调用方正常完成任务时异步执行当前动作，默认使用ForkJoinPool.commonPool()线程池执行。

- 方法中带有Async，参数中有Executor，则表明可以使用自定义线程池。

后面的讲解就只说明不带**Async**的方法了。



##### 3.1.3 runAfterBoth

```java
public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor)
```

**runAfterBoth(CompletionStage<?> other,Runnable action)**

表示当前调用方以及参数中给定的Stage**都正常完成任务时**才执行当前动作

##### 3.1.4 runAfterEither

```java
public CompletableFuture<Void> runAfterEither(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action)
public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor)
```

**runAfterEither(CompletionStage<?> other,Runnable action)**

这个不用我说你都能才出来了吧。 表示当前调用方以及参数中给定的Stage**任何一个Stage正常完成任务**就执行当前动作。

#### 3.2 主要参数为Function的方法

```java
public interface Function<T, R>
```

Functioin接口定义T为输入值R为输出值。便是此Lamada表达式包含了输入输出值。

##### 3.2.1 thenApply

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor) 
```

**thenApply(Function<? super T,? extends U> fn)**

当前Stage正常完成后，再执行此Function任务，并且会将当前的Stage结果作为Function中的参数。最后返回一个CompletionStage。

其余方法同上面讲的一样，均是默认线程池异步执行，或使用自定义线程池异步执行。

##### 3.2.2 applyToEither

```java
public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn)
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn)
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn,Executor executor)
```

**applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn)**

当前调用的Stage或者参数中的other Stgae中任何一个正常完成，均执行给定的Function，并且使用之前相应的result作为输入值(当前调用的Stage结果优先级高于参数中的Stage)，并返回CompletionStage。

##### 3.2.3 thenCompose

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn,Executor executor)
```

**thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)**

只有一个Function参数，Function中的输入值T为当前Stage的结果，输出值为一个CompletionStage, 返回一个新的CompletionStage，

其余的方法同上。



##### 3.2.3 thenCombine

这个方法是用到了 BiFunction

```java
public interface BiFunction<T, U, R>
```

T，U为apply方法的第一个和第二个参数，R为返回值。

```java
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn, Executor executor)
```

**thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn)**

这个方法和它的名称一样，就是合并两个Stage的结果。当前Stage的result，和参数中的other Stage的result作为参数中BiFunction的两个输入参数，最终返回一个新的CompletionStage。



##### 3.2.4 handle

```java
public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn, Executor executor)
```

**handle(BiFunction<? super T, Throwable, ? extends U> fn)**

使用此Stage的Result作为一个参数，还有一个异常为参数，执行BiFunction，并将执行的结果作为返回的CompletableFuture的result。其与方法为异步执行方法。

#### 3.3 主要参数为Consumer的方法

```
public interface Consumer<T>
```

Consumer是一个只有输入参数，没有返回值的Lamada，其方法 void accept(T t)， 即表示只有输入参数没有返回值。

##### 3.3.1 thenAccept

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor)
```

**thenAccept(Consumer<? super T> action)**

看完了上面的几组方法，这个方法很容易就能明白，即当前Stage正常完成的情况下，将当前Stage的result作为参数，执行Consumer，并且返回一个没有resulet的CompletableFuture。

##### 3.3.2 thenAccept

```java
public CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action,Executor executor) 
```

**acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)**

同上面带有Either的方法差不多，即当前Stage 或者参数中的Other Stgae正常完成后执行Comsumer，并且将Stage的result作为Comsumer的输入值，（同上，调用放的Stage的result优先级更高！）



##### 3.3.3 thenAcceptBoth

```java
public <U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other,  BiConsumer <? super T, ? super U> action)
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action)
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action, Executor executor)
```

**thenAcceptBoth(CompletionStage<? extends U> other,  BiConsumer <? super T, ? super U> action)**

同上，即需要两个Stage都正常完成才会去执行 BiConsumer。由于BiConsumer有两个输入值，则Stage的result分别为BiConsumer计算的输入值，最后返回一个不包含Result的CompletableFuture。



##### 3.3.4 whenComplete

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action, Executor executor)
```

**whenComplete(BiConsumer<? super T, ? super Throwable> action)**

顾名思义，就是当前调用的stage完成的时候执行参数中的方法，并且返回包含同一个结果的CompletableFuture，如果有异常也会返回指定的异常。





### 总结

本篇为妹子讲了一下CompletableFuture的内容特点。相信聪明的美女同事已经比较熟悉CompleteableFuture的应用了！下期我们再来讲解CompletableFuture的使用场景以及应用示例。