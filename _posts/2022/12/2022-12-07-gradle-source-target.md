---
layout: post
title:  Gradle souceCompatiblity VS targetCompatibility
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

今天带大家了解一下 Gradle 中 sourceCompatiblity 和 targetCompatibility 的使用配置和区别。 如果有对 Gradle 不太了解的朋友可以看我们之前的一些文章。

<!--more-->

### Java 中的版本处理

在了解 Gradle 中的 sourceCompatiblity 和 targetCompatibility 之前， 我们先来看看 Java 在编译的时候，是怎么处理版本号的。

当我们使用javac编译一个Java程序时，我们可以为版本处理提供以下两个关闭版本的编译选项。

* -source 指的是我们的 Java 代码的语言版本和编译的 JDK 相匹配（例如，1.8代表JDK8）。我们所提供的版本值将限制源代码中使用的语言特性，使其符合各自的Java版本。 
* -target 指的是控制生成的类文件的版本。也就是说我们提供的版本值将是我们的程序可以运行的最低Java版本。

举个例子

```java
javac HelloWorld.java -source 1.6 -target 1.8
```

上面的命令的意思就是 程序的运行环境需要支持JDK 1.8 也就是 Java 8， 而源码中不能包含 Java 6 以上版本的语言特性，比如说 Lambda 表达式等等。

### Gradle 中的版本使用

Gradle 中需要依赖Java插件，然后通过一个叫 java 的 task 来配置 sourceCompatibility 和 targetCompatibility 属性，也就是 javac 中的 `-source` 和 `-target` 编译选项。

让我们来设置build.gradle文件

```groovy
plugins {
    id 'java'
}

group 'cn.javanorth'

java {
    sourceCompatibility = "1.6"
    targetCompatibility = "1.8"
}
```

### 通过一个例子来验证一下

我们创建一个叫 HelloWorld 的控制台程序来进行测试，创建一个 HelloWorldApp 的 class。

```java
public class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

好，我们使用 gradle build 命令来编译下，我们可以看到生成了一个 HelloWorldApp.class 的文件。

我们通过使用 javap 命令行工具来检查这个class 的字节码版本号。

```java
javap -verbose HelloWorldApp.class
```

输出结果如下：

```java
public class cn.javanorth.helloworld.HelloWorldApp
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
```

这里的主版本号是52，也就是 Java 8 class 文件的版本号。 这就意味着 HelloWorldApp.class 只能运行在 JDK 1.8 以上版本。

再来测试一下 sourceCompatibility， 我修改一下代码，增加一个 Java 6 没有的特性，比如说 Lambda 表达式。

```java
public class HelloWorldApp {

    public static void main(String[] args) {
        Runnable helloLambda = () -> {
            System.out.println("Hello World!");
        }
        helloLambda.run();
    }
}
```

我们尝试使用 gradle 进行编译， 可以看到有一个编译错误。

```java
error: lambda expressions are not supported in -source 1.6
```

-source选项相当于Gradle 配置中 sourceCompatibility，可以让我们的代码在编译过程中提前发现问题，如果我们不想引入更高的版本特性，使用这个选项可以确保我们不会误用这些特性。比如说我们可能希望我们的应用程序也能在Java 6 runtime 上运行。

### 总结

在这篇文章中，我们了解如何使用 `-source` 和 `-target` 编译选项来处理我们的Java源代码和目标运行时的版本。我们还可以通过Gradle 的 sourceCompatbility 和 targetCompatibility 配置使用这些编译选项。