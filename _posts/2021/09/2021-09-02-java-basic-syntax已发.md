---
layout: post
title: Java 基础语法
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

今天指北君要带大家来学习Java的基础语法。
<!--more-->

上一次我们学习了怎么安装JDK和开发工具IDEA，同时也给大家写了一个hello world的演示代码。今天我们给大家从 hello world 展开讲讲Java的基础语法。

话不多说，直接上代码：

```java
public class HelloWorld {
    /* 第一个Java程序
     * 它将输出字符串 Hello World
     */
    public static void main(String[] args) {
        System.out.println("Hello World"); // 输出 Hello World
    }
}
```

因为Java是面向对象的编程语言，一个程序的基本单位就是`class`，`class`是关键字，这里定义的`class`名字就是`HelloWorld`：

```java
public class HelloWorld { // 类名是HelloWorld
    // ...
} // class定义结束
```

类名要求：

- 类名必须以英文字母开头，后接字母，数字和下划线的组合
- 习惯以大写字母开头

`public`是访问修饰符，代表这个`class` 是公开的。

我们再来看看这个main函数，我画了一个图，可以先看看：

![img](http://www.javanorth.cn/assets/images/2021/feng/java-basic1.jpeg)

一个main方法有访问修饰符、关键字、返回类型、方法名、 数据类型（String）、字符串参数组成。我们一个一个展开讲讲。

### 访问修饰符

什么是访问修饰符？

像其他语言一样，Java可以使用修饰符来修饰类中方法和属性。主要有两类修饰符：

- 访问控制修饰符 : default, public , protected, private
- 非访问控制修饰符 : final, abstract, static, synchronized

### 关键字

Java关键字大概有50个左右，这些作为保留字不能用于常量、变量、和任何标识符的名称。

abstract、assert、boolean、break、byte、case、catch、char、class、continue、default、do、double、else、enum、extends、final、finally、float、for、if、implements、import、int、interface、instanceof、long、native、new、package、private、protected、public、return、short、static、strictfp、super、switch、synchronized、this、throw、throws、transient、try、void、volatile、while


### 返回类型

Java的返回类型，就是一个方法需要返回某个值的类型。如果我们不需要任何返回，我们就可以使用void。

### 方法名

是方法的实际名称，有一些规则需要遵守

- 方法的名字的第一个单词应以小写字母作为开头，后面的单词则用大写字母开头写，不使用连接符。例如：**addPerson**。
- 下划线可能出现在 JUnit 测试方法名称中用以分隔名称的逻辑组件。一个典型的模式是：**test<MethodUnderTest>_<state>**，例如 **testPop_emptyStack**。

### 参数

参数有分为参数类型和参数名称。示例中 `String[]` 作为参数类型，`args`作为参数名称。

### Java注释

Java有三种注释方式：单行注释、多行注释和文档注释。

在我们的`Hello world`示例，我们演示了多行注释和单行注释。

单行注释

以双斜杠`//`标识，只能注释一行内容，用在注释信息内容少的地方。

```java
// 输出 Hello World
```

多行注释

包含在`/*`和`*/`之间，能注释很多行的内容。

```java
/* 第一个Java程序
* 它将输出字符串 Hello World
*/
```

文档注释

包含在`/**` 和 `*/`之间，也能注释多行内容，一般用在类、方法和变量上面，用来描述其作用。

```java
/**
 * 文档注释
 */
```


### 总结

今天就是简单地给大家介绍Java的基础语法，从Hello world这个示例展开给大家讲讲Java的基础知识点。
