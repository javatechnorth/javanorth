---
layout: post
title: 软软猿妹问我JDK中眼花缭乱的Function/Consumer/Supplier/Predicate？
tagline: by 揽月中人
categories: Algorithm
tags:
- 揽月中人
---

JDK中有许多函数式接口，也会有许多方法会使用函数式接口作为参数，同时在各种源码中也大量使用了这些方法，那么我们在实际工作中应该如何使用！我们就来盘一盘，这样也有助于写出优雅的代码，使我们在阅读源码时事半功倍。

<!--more-->

### 1 JDK中常见的Lamada表达式

Java中可以使用Lamada表达式的接口都有@FunctionalInterface注解。

先来看看util.function包下面含有FunctionalInterface注解的接口。一屏显示不全，可见功能非常齐全。

鉴于常用的一些函数式接口有Function/Consumer/Supplier/Predicate以及Runnable等。本篇介绍这几类接口。

![image-20211010224457887](E:\javaNorth\javanorth\assets\images\2021\lyj\LamadaFunctionClass1.png)

![image-20211010224559121](E:\javaNorth\javanorth\assets\images\2021\lyj\LamadaFunctionClass2.png)

####  1.1 Runnable

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

 Runnable 使用Lamada方式书写时，无参数，无返回值。最终执行的是run方法

使用Demo

```java
new Thread(() -> {
    System.out.println("JavaNorth Runnable");
}).start();
```

#### 1.2 Function 

Function 表示接受一个参数并产生结果的函数。

```java
@FunctionalInterface
public interface Function<T, R> {
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

  Function<T,R>接受一个参数T，并且有返回值 R, 其实现也主要是实现此方法 R apply(T t);

Function 的一个示例:

```java
List<String> list = new ArrayList<String>();
List<String> collect = list.stream().map((x) -> {
    return "Java North Function" + x;
}).collect(Collectors.toList());
```

上述示例中是一个stream的map方法。其中x为输入参数，『"Java North and" + x』为输出。



#### 1.3 Consumer

  Consumer表示接受一个参数，没有返回值的操作，主要方法为 void accept(T t);

```java
@FunctionalInterface
public interface Consumer<T> {
    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

##### 1.3.1 Consumer 在Java中的应用

常见的有List的forEach等。

```java
list.forEach(x -> System.out.println( "Java North Consumer test " + x));
```

x为参数，输出语句直接执行。

下面的map的forEach参数为BiConsumer，其入参有两个。

```java
Map<String,String> map = new HashMap<>();
map.forEach((K,V) -> {
    System.out.println("Java North Big Consumer MAP key : " +K + " value: "+V );
});
```

##### 1.3.2 自动义带有Consumer的方法





#### 1.4 Supplier 

  Supplier没有参数，有一个返回值。

```
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

**BiConsumer**

   BiConsumer<T,U>接受两个参数，没有返回值。

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    /**
     * Performs this operation on the given arguments.
     * @param t the first input argument
     * @param u the second input argument
     */
    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

**Predicate**

```
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }

    @SuppressWarnings("unchecked")
    static <T> Predicate<T> not(Predicate<? super T> target) {
        Objects.requireNonNull(target);
        return (Predicate<T>)target.negate();
    }
}
```

### 2 常用的Lamada参数特征

### 3 自定义Lamada函数式接口


### 总结