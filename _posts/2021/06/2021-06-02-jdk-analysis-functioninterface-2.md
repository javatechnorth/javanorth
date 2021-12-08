---
layout: post
title:  JDK源码解析——深入函数式接口（应用篇二） -- 20210705
tagline: by simsky
categories: JDK 源码解读
tags: 
    - simsky

---

大家好，上次指北君给大家开启了函数式接口的介绍，今天，指北君将在第一篇基础上继续为大家解读函数式接口涉及到的知识点。本篇文章为函数接口的应用篇二，将会为各位小伙伴详细介绍“@FunctionInterface”注解，java.util.function包中所有接口。

<!--more-->

### 概述
函数式接口将分为三个篇章来为大家介绍：
+ （应用篇一）（1）函数式接口的来源，（2）Lambda表达式，（3）双冒号运算符
+ （应用篇二）（4）详细介绍@FunctionInterface注解（5）对java.util.function包进行解读
+ （原理篇）介绍函数式接口的实现原理
应用篇将阶段相关的JDK源码以及给出典型的示例代码
原理篇则从编译、JVM维度来分析函数式接口的实现原理，具有一定深度，需要读者具备一定的底层知识。

说明：源码使用的版本为JDK-11.0.11


### FunctionInterface
这一节，指北君给大家介绍如何声明一个函数式接口，FunctionInterface注解就是用于来干这件事情的，当我们为接口增加FunctionInterface注解后，编译器会按照函数式接口的约束进行检查。
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
咦，这怎么回事，不是说只能有一个抽象方法么？小伙伴们仔细观察接口会发现其中一个是equals接口，这不是Object中的方法么？是的，这就是另一条特殊的约束，如果抽象方法是覆盖的是Object的方法，则不计入抽象方法的个数。

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

### java.util.function

之前的学习，指北君介绍了什么是函数式编程，如果写一段Lambda表达式，以及学习如何申明函数式接口。本节我们将学习JDK为我们提供的基础函数式接口，这些接口位于java.util.function包中，它们可以满足我们很多通用场景的使用需要。

![function包](http://www.javanorth.cn/assets/images/2021/simsky/jdk_src_func_if_4.png)

打开java.util.function包，我们发现里面足足有43个接口，这43个接口就是JDK提供给我们43把剑，如果要让小伙伴舞动这43把剑，是不是感觉鸭梨山大了呢？小伙伴们，别慌！在你看完指北君后面的分析后，你就可以将这43把剑化为无形，做到手中无剑心中有剑。

指北君先带领小伙伴们对接口名称进行分析。是的，是接口名称，不是源码

![接口名称解析](http://www.javanorth.cn/assets/images/2021/simsky/jdk_src_func_if_2.png)

从思维导图中可以看到，接口的主体为最后的单词，包含：Consumer，Function，Predicate，Supplier，Operator。

![接口主体类别](http://www.javanorth.cn/assets/images/2021/simsky/jdk_src_func_if_3.png)

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


### 小结
至此，指北君将函数式接口的应用知识点已经介绍和分析完，涵盖知识点包含：java.util.function包，FunctionInterface注解，Lambda表达式，双冒号操作符等知识点。函数式接口在集合，流中应用较为广泛，也证明了其在数据时候的显著优势，各位小伙伴可以结合这些JDK中的源码进行巩固加深。本篇为函数式接口的应用篇，在下一篇，指北君将从编译、JVM层面深入介绍Java中函数式编程实现原理。


