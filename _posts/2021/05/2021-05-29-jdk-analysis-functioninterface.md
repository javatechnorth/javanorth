---
layout: post
title:  JDK源码解析——深入函数式接口（应用篇一）--20210607
tagline: by simsky
categories: JDK 源码解读
tags: 
    - simsky

---

Lambda表达式，相信大家都耳有所闻，而且不少小伙伴在日常的工作中也在使用。但说到函数式接口，可能有一些即使会使用Lambda表达式的小伙伴也会觉得陌生。今天，指北君就将带领大家对Lambda、及其所使用的一些和函数式接口相关的知识点进行一个全面的学习。
函数式接口所涉及的知识点包含：java.util.function包，@FunctoinInterface注解，Lambda表达式，双冒号操作符。同时，我们还将对函数式接口的实现原理进行深入的剖析。

<!--more-->

## 概述
函数式接口将分为三个篇章来为大家介绍：
+ （应用篇一）（1）函数式接口的来源，（2）Lambda表达式，（3）双冒号运算符
+ （应用篇二）（4）详细介绍@FunctionInterface注解（5）对java.util.function包进行解读
+ （原理篇）介绍函数式接口的实现原理
应用篇将阶段相关的JDK源码以及给出典型的示例代码
原理篇则从编译、JVM维度来分析函数式接口的实现原理，具有一定深度，需要读者具备一定的底层知识。

说明：源码使用的版本为JDK-11.0.11

## 什么是函数式接口

> 【阅读导引】：本节为概念性知识，纯技术向伙伴可跳过

在分析具体内容之前，指北君带领大家来对函数式接口做个基本的认知。函数式接口是JAVA语言为引入函数式编程而增加的特性，也即是说函数式接口式Java实现函数式编程的具体方式。那么，函数式编程到底是什么？他和面向对象编程又有什么关系？它能为我们带来什么？我们又是否真的需要函数式编程？
有很多小伙伴，可能和指北君一样，是以面向对象语言开启的编程世界的，对于函数式编程其实很陌生。所以，指北君在这里先给大家引荐编程界的三大流派（当然还有别的流派）：过程式，函数式，对象式：

![编程范式](/assets/images/2021/simsky/jdk_src_func_if_1.png)

函数式编程的思想脱胎于数学理论，也就是我们通常所说的λ演算（λ-calculus）。这也是为什么Java8中引入的函数式编程叫Lambda表达式的原因吧。如同数学中的函数一样，函数式编程范式中的函数有独特的特性，也就是通常说的无状态或引用透明性。一个函数的输出由且仅由其输入决定，同样的输入永远会产生同样的输出。

函数式编程的定义："函数式编程是一种编程范式。它把计算当成是数学函数的求值，从而避免改变状态和使用可变数据。它是一种声明式的编程范式，通过表达式和声明而不是语句来编程。" 
函数式编程的代码通常更加简洁，但是不一定易懂。

近年来，随着多核平台和并发计算的发展，函数式编程的无状态特性，在处理这些问题时有着其他编程范式不可比拟的天然优势。这种发展也就进一步促使了Java引入函数式编程这一特性。

## 一个简单示例
指北君先给大家展示一个简单的函数式编程的示例：
```java
  /**
   * 简单的函数式编程示例
   */
  public static void lambdaDemo1() {
    // 准备测试数据
    Integer[] data = new Integer[] {1, 2, 3};
    List<Integer> list = Arrays.asList(data);
    
    // 简单示例：转换单位并打印数据
    list.forEach(x -> System.out.println(String.format("Cents into Yuan: %.2f", x/100.0)));
  }
```

不熟悉Lambda表达式的小伙伴可能会好奇其中的语句：x -> System.out.println(String.format("Cents into Yuan: %.2f", x/100.0))，这是什么呢？这就是我们的Lambda表达式。
通常，我们要访问List对象，需要通过for、while等控制循环语句，并在循环中完成相关工作。有了函数式编程后，我们就可以使用Lambda表达式来完成对应的功能，是不是很简洁！
小伙伴们可能会奇怪，难道Lambda自动做了循环？当然不是，这里的循环控制并没有减少，只是在forEach方法中而已。我们打开默认的迭代器forEach实现方法（ArrayList的forEach实现有差异，总体逻辑一致），代码显示forEach循环，并在循环中执行参数的函数逻辑。
```java
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```
既然没有省略控制逻辑，难道我们费这么大的力气引入这个东东就只是为了简洁点？指北君画了一下调用逻辑，参见下图
![两种编程模式比较](/assets/images/2021/simsky/jdk_src_func_if_1-1.png)

从上图中大家是不是隐约可以看出：这种方式可以将控制部分和业务处理部分进行解耦，业务处理代码更容易集中。

我们在分析forEach源码的时候，看到forEach的参数类型为Consumer,打开Consumer源码（主要接口声明部分）：
```java
@FunctionalInterface
public interface Consumer<T> {
  ...
}
```
小伙伴们是不是发现这就是一个简单的接口，接口使用了@FunctionInterface的接口，这是不是对Lambda表达式使用位置的约束呢？这个问题我们将在接下来的几个章节给出答案。

## Lambda表达式
在示例部分，指北君展示了在Java中如何Lambda进行函数式编程，小伙伴们是不是跃跃欲试想要动手了呢？在动手之前指北君先带领大家了全面学习Lambda表达式的语法。
下面给出几种常见的Lambda代码片段（代码仅截取部分，无上下文）：
```java
    () -> System.out.println("demo")
    ...
    list.forEach(x ->  System.out.print(x));
    ...
    map.forEach((x, y) -> {
      System.out.print(x);
      System.out.println(y);
    });
    ...
    (Integer x, String y) -> System.out.println("x: " + x ", y: " + y);
```
从上面代码片段，可以看出Lambda表达式是通过->操作符来连接的，左边为参数部分，右边为表达式主体。

Lambda表达式语法：
1. (parameters)->expression
2. (parameters)->{statements;}

参数说明：
([[type] parameter [, ...]]) 
+ 参数包括在圆括号内，参数数量可以0到多个，多个参数通过逗号“,”分割，例如(x, y)->
+ 参数类型可明确声明，也可以省略，省略时根据上下问进行推断， 例如：(x)->, (int x)->
+ 无参数，直接使用括号，例如：()->
+ 一个参数时，且参数类型省略，则括号可以省略 x->

表达式主体：
+ 由0到多条语句组成 
+ 只有一条语句时，语句块符号“{}”可省略，此时语句的结果将作为返回值，例如：->x*x, ->System.out.print(x)。
+ 超过一条语句时，必须使用语句块符号“{}”包含起来。
+ 带return关键字必须用代码块，例如：->{return x+x}。

常见的组合形式：
```java
(int a, int b) -> {  return a + b; }

() -> System.out.println("Demo")

(String s) -> { 
  System.out.println(s); 
}

() -> 42

() -> { return 3.1415 };
```

启动线程
```java
new Thread(
    () -> System.out.println("start in thread.")
).start();
```

其他的代码遵循基本的Java语法，小伙伴们现在就可以大展拳脚，试试通过Lambda表达式进行函数式编程。

## 双冒号操作符
经过上一节的实践，小伙伴们是不是很兴奋了，可能有些小伙伴会问，Java类中的的方法也是函数，我可不可以在传入Lambda表达式的地方传入普通方法呢？类似下面这种效果：
```java
    List<String> list = new ArrayList<String>();
    ...
    list.forEach(xxxMethod());
```
想法是没有问题的，但是形式错误了，首先xxxMethod()会直接触方法执行，并且返回的类型也不匹配forEach方法。那么，正确的形式应该如何写呢？这就需要我们的双冒号云算法登场了。
双冒号云算符标准名称为eta-conversion，有下面四种常用场景
1. 实例方法引用 object::instanceMethod
2. 静态方法引用 Class::staticMethod
3. 实例方法引用（实例作为参数传入） Class::instanceMethod
4. 构造方法引用 Class:new
  + 无参数：Supplier
  + 一个参数：Function
  + 二个参数：BiFunction
  + 更多：自定义函数接口


示例代码
```java
public class FunctionInterfaceInvoke {

  public static void main(String[] args) {
    
    // 1-1 构造方法（无参数），编译会做参数检查（包含输入参数和返回值）
    Supplier<FunctionInterfaceInvoke> s = FunctionInterfaceInvoke::new;
    s.get();
    
    //1-2 构造方法（1个参数）
    IntFunction<FunctionInterfaceInvoke> func = FunctionInterfaceInvoke::new;
    func.apply(1);
    
    // 1-3 构造方法（多个参数）
    BiFunction<Integer, Integer, FunctionInterfaceInvoke> func2 = FunctionInterfaceInvoke::new;
    func2.apply(1, 2);
    
    // 2 静态方法
    Consumer<Integer> sta1 = FunctionInterfaceInvoke::staticMethod;
    sta1.accept(1);
    
    // 3 实例方法
    IntConsumer sta2 = new FunctionInterfaceInvoke()::instanceMethod;
    sta2.accept(2);

  }

  public FunctionInterfaceInvoke() {
    System.out.println("none parameters");
  }
  
  public FunctionInterfaceInvoke(int p1) {
    System.out.println("constructor whith one parameter: " + p1);
  }
  
  public FunctionInterfaceInvoke(Integer p1, Integer p2) {
    System.out.println(String.format("constructor whith 2 parameters %1s, %2s", p1, p2));
  } 
  
  public static void staticMethod(Integer p1) {
    System.out.println("static method:" + p1);
  }
  
  public void instanceMethod(int p1) {
    System.out.println("instance method:"+p1);
  }
}
```



## 小结
函数式接口应用篇的第一部分就给大家介绍到这里，本篇我们介绍了什么是函数式编程，一个简单示例，Lambda表达式详细说明和双冒号操作的使用。下一篇我们将会继续介绍函数式接口的应用，学习 @FunctionInterface注解和java.util.function包中的接口。

最后感谢各位小伙伴的点赞、收藏和评论，我们下期更精彩。


