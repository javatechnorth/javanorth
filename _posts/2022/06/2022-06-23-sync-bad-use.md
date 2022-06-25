---
layout: post
title:  synchronized 的几种错误用法
tagline: by feng
categories: java
tags: 
    - feng
---

大家好， 我是指北君。

synchronized 在我们平常工作中也是挺常用的， 对于摆脱多线程问题很有帮助。但是如果synchronized被错误使用时，可能会给我们带来很多麻烦。

在本文中，我们将讨论与同步相关的一些不好的做法，以及针对每个使用情况的更好的方法。
<!--more-->
### 同步的原则

一般来说，我们应该只对那些我们确信没有外部代码会锁定的对象进行同步。

换句话说，使用池化或可重复使用的对象进行同步是一种不好的做法。原因是池化/可重用对象可以被JVM中的其他进程访问，外部/不被信任的代码对这些对象的任何修改都会导致死锁和非确定性行为。

现在，让我们来讨论基于某些类型的同步原则，如String、Boolean、Integer和Object。

### String 字面量

#### 1.错误用法

字符串字面量是有池的，在Java中经常被重复使用。因此，不建议使用String类型与 synchronized关键字进行同步。

```java
public void stringBadPractice1() {
    String stringLock = "LOCK_STRING";
    synchronized (stringLock) {
        // ...
    }
}
```

同样地，如果我们使用private final String字面，它仍然是从常量池中引用的。

```java
private final String stringLock = "LOCK_STRING";
public void stringBadPractice2() {
    synchronized (stringLock) {
        // ...
    }
}
```

此外，为了同步，内接字符串被认为是不好的做法。

```java
private final String internedStringLock = new String("LOCK_STRING").intern();
public void stringBadPractice3() {
  synchronized (internedStringLock) {
      // ...
  }
}
```

根据Javadocs，intern方法为我们获得了String对象的规范表示。换句话说，intern方法从池中返回一个String--如果它不在池中，则明确地将它添加到池中--它的内容与这个String相同。

因此，在可重用对象上的同步问题对于内部的String对象也是存在的。

注意：所有的String字面符号和以字符串为值的常量表达式都是自动实习的。

#### 2.正确用法

为了避免在String字面上进行同步的不良做法，建议使用new关键字创建一个新的String实例。

让我们在已经讨论过的代码中解决这个问题。首先，我们将创建一个新的String对象，以拥有一个唯一的引用（避免任何重复使用）和它自己的内在锁，这有助于同步。

然后，我们保持该对象的private和final，以防止任何外部/不受信任的代码访问它。

```java
private final String stringLock = new String("LOCK_STRING");
public void stringSolution() {
    synchronized (stringLock) {
        // ...
    }
}
```

### Boolean 字面量

Boolean类型有两个值，即true和false，不适合用于锁定目的。与JVM中的String字面量类似，boolean字面量也共享Boolean类的唯一实例。

让我们来看看一个在Boolean锁对象上同步的错误用法例子。

```java
private final Boolean booleanLock = Boolean.FALSE;
public void booleanBadPractice() {
    synchronized (booleanLock) {
        // ...
    }
}
```

在这里，如果任何外部代码也在具有相同值的Boolean字面上进行同步，系统就会变得没有反应，或者导致死锁的情况。

因此，我们不建议使用Boolean对象作为同步锁。

### 原始类型的包装类

#### 1. 错误用法

与boolean字段类似，原始类型的包装类可能会重复使用某些值的实例。原因是JVM会缓存和共享可以表示为字节的值。

例如，让我们写一个在 Integer 上进行同步的错误用法例子。

```java
private int count = 0;
private final Integer intLock = count; 
public void boxedPrimitiveBadPractice() { 
    synchronized (intLock) {
        count++;
        // ... 
    } 
}
```

#### 2.正确用法

然而，与boolean字面量不同，在原始类型的包装类上同步的解决方案是创建一个新实例。

与String对象类似，我们应该使用new关键字来创建一个唯一的Integer对象的实例，该实例有自己的内在锁，并保持其private和final。

```java
private int count = 0;
private final Integer intLock = new Integer(count);
public void boxedPrimitiveSolution() {
    synchronized (intLock) {
        count++;
        // ...
    }
}
```

### 类同步

当一个类用this关键字实现方法同步或块同步时，JVM使用对象本身作为监视器（其固有锁）。

不受信任的代码可以获得并无限期地持有一个可访问类的内在锁。因此，这可能会导致死锁的情况。

#### 1.错误用法

例如，让我们创建Animal类，它有一个synchronized方法setName和一个带有synchronized块的方法setOwner。

```java
public class Animal {
    private String name;
    private String owner;
    
    // getters and constructors
    
    public synchronized void setName(String name) {
        this.name = name;
    }

    public void setOwner(String owner) {
        synchronized (this) {
            this.owner = owner;
        }
    }
}
```

现在，让我们写一些错误用法，创建一个Animal类的实例，并对其进行同步。

```java
Animal animalObj = new Animal("Tommy", "John");
synchronized (animalObj) {
    while(true) {
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

在这里，不受信任的代码例子引入了一个无限期的延迟，阻止了setName和setOwner方法的实现获得同一个锁。

#### 2.正确用法

防止这个漏洞的解决方案是私人锁对象。

我们的想法是使用与我们类中定义的Object类的private final实例相关的内在锁来代替对象本身的内在锁。

另外，我们应该使用块同步来代替方法同步，以增加灵活性，使非同步的代码不在块中。

所以，让我们对我们的Animal类进行必要的修改。

```java
public class Animal {
    // ...

    private final Object objLock1 = new Object();
    private final Object objLock2 = new Object();

    public void setName(String name) {
        synchronized (objLock1) {
            this.name = name;
        }
    }

    public void setOwner(String owner) {
        synchronized (objLock2) {
            this.owner = owner;
        }
    }
}
```

在这里，为了提高并发性，我们通过定义多个private final锁对象来细化锁定方案，以分离我们对两个方法--setName和setOwner的同步关注。

此外，如果实现同步块的方法修改了一个静态变量，我们必须通过锁定静态对象来实现同步。

```java
private static int staticCount = 0;
private static final Object staticObjLock = new Object();
public void staticVariableSolution() {
    synchronized (staticObjLock) {
        count++;
        // ...
    }
}
```

### 总结

在这篇文章中，我们讨论了一些与某些类型的同步有关的坏做法，如String、Boolean、Integer和Object。

本文最重要的启示是，不建议使用池化或可重复使用的对象进行同步。

另外，建议在Object类的private final实例上进行同步。这样的对象将无法被外部/不被信任的代码访问，否则这些代码可能会与我们的公共类交互，从而减少这种交互导致死锁的可能性。