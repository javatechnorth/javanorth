---
layout: post
title:  JDK源码解析——深入函数式接口（应用篇）
tagline: by simsky
categories: JDK 源码解读
tags: 
    - simsky

---

今天，指北君给大家介绍个熟悉的陌生人，为什么这么说呢？因为对于有些小伙伴来说它是挚爱，而有些小伙伴可能只是熟悉，还有一些小伙伴可能压根没用过。那到底是什么大家能有如此不同的态度呢？这就是本次JDK源码解析的主角——函数式接口。
无论是Java精英，还是刚进入Java的新同学，应该都耳有所闻这个Java8引入的新特性。当然，对于指北君这个沉浸在Java世界的这只猿来说，Java8还是具有相当的年代感了，毕竟JDK11都快一统江湖了嘛，但这依然不妨碍指北君今天给大家来解析函数式接口这个熟悉的陌生人。

<!--more-->

## 概述

口水话说了一堆，指北君还是先对本次主题做个概要说明。
文章为JDK源码解析系列，本期的主题为Java8版本中引入的“函数式接口”，介于本次主题内容涉及即有广度又有一定深度，因此文章将分为应用和原理两个篇章。读者可以根据自身的需求侧重阅读自己关注的内容。
应用篇将以java.util.function包为基础，围绕函数式接口的关键知识点进行介绍说明,将包含如下内容：java.util.function包，FunctionInterface，Lambda表达式，双冒号操作符。
原理篇则从编译、JVM维度来分析函数式接口的实现原理，具有一定深度，需要读者具备一定的底层知识。

说明：源码使用的版本为JDK-11.0.11

## 渊源

> 【阅读导引】：本节为概念性知识，纯技术向伙伴可跳过

在分析具体内容之前，指北君带领大家来对函数式接口做个基本的认知。函数式接口是JAVA语言为引入函数式编程而增加的特性，我们暂时将目标转向函数式编程这个概念。那么，函数式编程到底是什么？他和面向对象编程又有什么关系？它能为我们带来什么？我们又是否真的需要函数式编程？
有很多小伙伴，可能和指北君一样，是以面向对象语言开启的编程世界的，对于函数式编程其实很陌生。所以，指北君在这里先给大家引荐编程界的三大流派（当然还有别的流派）：过程式，函数式，对象式：

![编程范式](/assets/images/2021/simsky/jdk_src_func_if_1.png)

函数式编程的思想脱胎于数学理论，也就是我们通常所说的λ演算（λ-calculus）。这也是为什么Java8中引入的函数式编程叫Lambda表达式的原因吧。如同数学中的函数一样，函数式编程范式中的函数有独特的特性，也就是通常说的无状态或引用透明性。一个函数的输出由且仅由其输入决定，同样的输入永远会产生同样的输出。

函数式编程的定义："函数式编程是一种编程范式。它把计算当成是数学函数的求值，从而避免改变状态和使用可变数据。它是一种声明式的编程范式，通过表达式和声明而不是语句来编程。" 
函数式编程的代码通常更加简洁，但是不一定易懂。

近年来，随着多核平台和并发计算的发展，函数式编程的无状态特性，在处理这些问题时有着其他编程范式不可比拟的天然优势。这种发展也就进一步促使了Java引入函数式编程这一特性。


## java.util.function

在上一节，指北君介绍了什么是函数式编程，以及它有哪些优劣势，接下来，指北君将介绍如何使用它。

可能大家有点疑惑，指北君为啥要从java.util.function开始呢？因为，在JDK中提供一些很常用的函数式接口，可以满足我们很多通用场景的使用，这就是java.util.function这个包。

![function包](/assets/images/2021/simsky/jdk_src_func_if_4.png)

打开源码一看，足足有43个接口，要让小伙伴舞动这43把剑，是不是感觉鸭梨山大了呢？小伙伴们，别慌！在你看完指北君后面的分析后，你就可以将这43把剑化为无形，做到手中无剑心中有剑。

指北君先带领小伙伴们对接口名称进行分析。是的，是接口名称，不是源码

![接口名称解析](/assets/images/2021/simsky/jdk_src_func_if_2.png)

从思维导图中可以看到，接口的主体为最后的单词，包含：Consumer，Function，Predicate，Supplier，Operator。

![接口主体类别](/assets/images/2021/simsky/jdk_src_func_if_3.png)

除了主体外，小伙伴是不是还看到了Unary，Binary，Bi这种表示参数个数的修饰词。

修饰词|作用
-|-
Unary|一元
Binary|二元
Bi|Binary缩写

再就剩下参数类型和方向的修饰词和参数方向Int，Long，Double，Obj，(X)To(Y)。

通过以上解析，我们可以确定所有的接口都是用于描述函数的输入参数和返回，这就是java.util.function留在我们心中终极大剑。有了这把终极武器，我们可以对于这43把剑信手拈来，也随意创造自己的武器，当然，要创造新的武器，按照这43把剑依葫芦画瓢是可以，但是要打造精品武器还需要掌握我们接下来介绍的FunctionInterface接口了。

在对整个java.util.function包了然于胸后，我们再打开一个典型的接口看看源码。function包中的类型都为接口类型，并且使用了@FunctionInterface注解，且每个接口都且只有一个接口（抽象）方法，部分接口存在默认方法和静态方法。

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```
accept为核心接口方法，andThen为方便复杂组合场景提供的默认方法。function包的整个代码逻辑都很简洁易懂，部分Operator继承了Function，指北君就不一一赘述了。

## FunctionInterface
上一节，指北君给小伙伴讲了java.util.function中的43把剑，并让大家做到心中有剑，这一节指北君将和小伙伴一同研究怎么造一柄自己的剑呢。
FunctoinInterface注解就是来帮小伙伴铸造武器的工具，并且对我们的造剑质量把关。FunctionInterface用于显式声明接口为函数式接口，编译器会按照函数式接口的约束进行检查。
先看看注解的定义：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```
源码显示该注解可用于类、接口和枚举类型，注解应用于运行时阶段。

如果仅从代码层面我们可能会犯错误，比如如下情况：
```java
@FunctionalInterface
public interface IFuncInterfaceSample{
  void func1();
  void func2();
}

```
我们会发现编译器报错了，这是因为在设计FunctionInterface的时候还增加了额外的约束，这些约束无法在注解的定义中呈现，是通过编译器实现的，下面指北君就一一为小伙伴们道来。

首先、FunctionInterface是SAM接口，什么是SAM接口呢？全称Single Abstract Method，从英文字义我们就能明白，接口中只能有一个抽象方法。由于Java在接口中增加了对默认方法和静态方法的支持，因此采用SAM设计的接口也可以定义默认方法和静态方法，也就是说我们在使用了FunctionInterface注解的接口中还能够定义默认方法和静态方法。
除了默认方法和静态方法外，这里还有一种列外，我们查看java.util.Comparator源码
```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    boolean equals(Object obj);
    ...
```
咦，这怎么回事，不是说只能有一个抽象方法么？小伙伴们仔细观察接口会发现其中一个是equals接口，这不是Object中的方法么？是的，这就是另一条特殊的约束，如果抽象方法是覆盖的Object的方法，则不计入抽象方法的个数。

除了抽象方法的个数限制外FunctionInterface只能用于interface，不能用于class和enum，对于这种情形编译器也会报错。

小伙伴们有没想过FunctionInterface为什么要有上面提到的约束呢？相信有些已经隐隐感觉到了:Java是通过类来实现函数式编程的（这部分内容将在原理篇中说明）。

最后，还有一个重点知识：是不是只有使用了@FunctionalInterface才能作为函数式接口呢？相信有不少伙伴和指北君之前一样理所当然地觉得显然应该是这样嘛。但是，JDK的世界很精妙：只要符合函数式接口约束条件的接口，即使没有采用@FunctionalInterface，编译器都会处理成函数式接口，比如下面的示例代码：
```java
public interface IFuncInterfaceWithoutAn {
  boolean test(int i);
}

public class Demo {
  public static void main(String args[]) {
    IFuncInterfaceWithoutAn whithout = (x)->x%2==1;
    System.out.println(whithout.test(3));
  }
}
```

## Lambda表达式
听指北君讲了这么多，大家也做到了心中有剑以及掌握铸剑技能，这时候是不是想找个对手来练练呢？那就轮到我们的Lambda表达式登场了。
在渊源章节，指北君就介绍过函数式编程脱胎于λ演算，所以我们的战场就是Lambda。
下面是一段简单的Lambda代码示例：
```java
    List<String> list = new ArrayList<String>();
    ...
    list.forEach(x ->  System.out.print(x));
```
x -> System.out.print(x)就是Lambda表达式，从这里可以看出Lambda表达式是通过->云算法来连接的，左边为参数部分，右边为函数体。

Lambda表达式用法：
1. (parameters)->expression
2. (parameters)->{statements;}

参数：
+ 无参数，直接使用括号，例如：()->
+ 省略参数类型，省略参数类型，例如：x->, (x,y)->
+ 声明类型，完整的形式，示例：(int x)->

函数体：
+ 只有一条表达式，代码块可省略，返回关键字省略，例如：-> x*x   ->System.out.print(x)
+ 带return关键字必须用代码块 -> {return x*x}
+ 超过一个表达式，用代码块

常见的组合形式有：
```java
(int a, int b) -> {  return a + b; }

() -> System.out.println("Hello World")

(String s) -> { System.out.println(s); }

() -> 42

() -> { return 3.1415 };
```

其他的代码遵循基本的Java语法，小伙伴们现在就可以大展拳脚，只要是函数式接口或者具备函数式接口条件的地方都可以传入Lambda表达式。

## 双冒号操作符
经过上一节的实践，小伙伴们是不是很兴奋了，可能有些小伙伴会问，Java类中的的方法也是函数，我可不可以在传入Lambda表达式的地方传入方法呢？类似下面这种效果：
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
至此，指北君将函数式接口的应用知识点已经介绍和分析完，涵盖知识点包含：java.util.function包，FunctionInterface注解，Lambda表达式，双冒号操作符等知识点。函数式接口在集合，流中应用较为广泛，也证明了其在数据时候的显著优势，各位小伙伴可以结合这些JDK中的源码进行巩固加深。本篇为函数式接口的应用篇，在下一篇，指北君将从编译、JVM层面深入介绍Java中函数式编程实现原理。


