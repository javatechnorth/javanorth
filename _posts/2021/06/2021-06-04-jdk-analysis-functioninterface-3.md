---
layout: post
title:  JDK源码解析——深入函数式接口（原理篇）
tagline: by simsky
categories: JDK 源码解读 JVM
tags: 
    - simsky

---

大家好，函数式接口的应用篇已经给大家讲完，今天，指北君和大家一同深入探索Java实现函数式接口的原理。

<!--more-->

## 概述
函数式接口将分为三个篇章来为大家介绍：
+ （应用篇一）（1）函数式接口的来源，（2）Lambda表达式，（3）双冒号运算符
+ （应用篇二）（4）详细介绍@FunctionInterface注解（5）对java.util.function包进行解读
+ （原理篇）介绍函数式接口的实现原理
应用篇将阶段相关的JDK源码以及给出典型的示例代码
原理篇则从编译、JVM维度来分析函数式接口的实现原理，具有一定深度，需要读者具备一定的底层知识。

说明：源码使用的版本为JDK-11.0.11

## 编译
无论是接口还是类，都需要经过编译，然后在运行期由JVM执行调用，现在我们来看看几个关键位置的编译结果。
先看看函数式接口编译
```sh
Classfile /O:/SCM/ws-java/sample-lambda/bin/com/tree/sample/func/IFuncInterfaceSample.class
  Last modified 2021-6-4; size 238 bytes
  MD5 checksum 58a3c8c5cbe9c7498e86d4a349554ae0
  Compiled from "IFuncInterfaceSample.java"
public interface com.tree.sample.func.IFuncInterfaceSample
  minor version: 0
  major version: 55
  flags: ACC_PUBLIC, ACC_INTERFACE, ACC_ABSTRACT
Constant pool:
   #1 = Class              #2             // com/tree/sample/func/IFuncInterfaceSample
   #2 = Utf8               com/tree/sample/func/IFuncInterfaceSample
   #3 = Class              #4             // java/lang/Object
   #4 = Utf8               java/lang/Object
   #5 = Utf8               func1
   #6 = Utf8               ()V
   #7 = Utf8               SourceFile
   #8 = Utf8               IFuncInterfaceSample.java
   #9 = Utf8               RuntimeVisibleAnnotations
  #10 = Utf8               Ljava/lang/FunctionalInterface;
{
  public abstract void func1();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_ABSTRACT
}
SourceFile: "IFuncInterfaceSample.java"
RuntimeVisibleAnnotations:
  0: #10()

```
接口的编译信息中没有任何额外的工作，如果显示声明了FunctionInterface注解，则编译信息中带有，反之则无。

那我们着重来看应用部分的代码编译的情况，先看应用部分的源代码：
```java
public class LambdaBinaryCode {
    private int lambdaVar = 100;
    
    public static void main(String[] args) {
        
        LambdaBinaryCode ins = new LambdaBinaryCode();
        ins.invokeLambda();
        ins.invokeEta();
        ins.invokeLambda2();
    }
    
    /**
     * 简单的函数式编程示例
     */
    public void invokeLambda() {
        // 准备测试数据
        Integer[] data = new Integer[] {1, 2, 3};
        List<Integer> list = Arrays.asList(data);
        
        // 简单示例：打印List数据
        list.forEach(x -> System.out.println(String.format("Cents into Yuan: %.2f", x/100.0)));
    }
    
    /**
     * 简单的函数式编程示例
     */
    public void invokeEta() {
        // 准备测试数据
        Integer[] data = new Integer[] {1, 2, 3};
        List<Integer> list = Arrays.asList(data);
        
        // 通过eta操作符访问
        list.forEach(System.out::println);
    }
    
    /**
     * 简单的函数式编程示例
     */
    public void invokeLambda2() {
        // 准备测试数据
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        int count = 10;
        Random r = new Random();
        while(count-->0) {
            map.put(r.nextInt(100), r.nextInt(10000));
        }
        
        // Lambda调用示例
        map.forEach((x, y) -> {
            System.out.println(String.format("Map key: %1s, value: %2s", x, y+lambdaVar));
        });
    }
}
```
这段源码中选取了几种典型的场景进行组合，让大家了解更多的扩展知识，因此代码稍显长。
+ invokeLambda() 单个参数的lambda表达式，省略参数括号和表达式主体的花括号。
+ invokeEta() eta方式的方法引用。
+ invokeLambda2() 两个参数的lambda表达式，lambda中使用成员变量。

### lambda表达式的编译
指北君和大家一起看看编译后的内容，使用命令查看编译后的方法结构(javap -p com.tree.sample.func.LambdaBinaryCode）
```sh
Compiled from "LambdaBinaryCode.java"
public class com.tree.sample.func.LambdaBinaryCode {
  private int lambdaVar;
  public com.tree.sample.func.LambdaBinaryCode();
  public static void main(java.lang.String[]);
  public void invokeLambda();
  public void invokeEta();
  public void invokeLambda2();
  private static void lambda$0(java.lang.Integer);
  private void lambda$2(java.lang.Integer, java.lang.Integer);
}
```
小伙伴有没发现，class文件中比源码文件中多出了两个方法：lambda$0，lambda$2。这两个方法分别对应invokeLambda和invokeLambda2中的的lambda表达式。

我们在javap命令中增加-v参数，可以查看到增加的lambda$0方法的更多细节，不熟悉JVM指令的小伙伴也不用担心，我们只是验证lambda$0就是invokeLambda中lambda表达式对应“x -> System.out.println(String.format("Cents into Yuan: %.2f", x/100.0))”。

```sh
  private static void lambda$0(java.lang.Integer);
    descriptor: (Ljava/lang/Integer;)V
    flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNTHETIC
    Code:
      stack=9, locals=1, args_size=1
         0: getstatic     #61                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #105                // String Cents into Yuan: %.2f
         5: iconst_1
         6: anewarray     #3                  // class java/lang/Object
         9: dup
        10: iconst_0
        11: aload_0
        12: invokevirtual #107                // Method java/lang/Integer.intValue:()I
        15: i2d
        16: ldc2_w        #111                // double 100.0d
        19: ddiv
        20: invokestatic  #113                // Method java/lang/Double.valueOf:(D)Ljava/lang/Double;
        23: aastore
        24: invokestatic  #118                // Method java/lang/String.format:(Ljava/lang/String;[Ljava/lang/Object;)Ljava/lang/String;
        27: invokevirtual #124                // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        30: return
      LineNumberTable:
        line 30: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      31     0     x   Ljava/lang/Integer;
```
从编译信息中我们可以看到几条明显相同的逻辑：
+ LocalVariableTable 首先包含了函数的输入参数，并且一致
+ 24行执行String.format方法
+ 27行执行PrintStream.println方法
从上面三个关键部分我们可以确定就是invokeLambda方法中的lambda表达式编译后的内容了。

仔细的小伙伴比较lambda$0，lambda$2两个方法后，可能会发现两个问题：第一，两个方法怎么一个是static一个是非static的呢？。第二，方法命名中的数字为什么不是数字连续的？
对于第一个问题，比较invokeLambda和invokeLambda2的源码，小伙伴发现有什么不同么？是否可以看到invokeLambda2中的lambda表达式引用了成员属性lambdaVar。这就是lambda生成方法的一种逻辑，**未使用成员变量的lambda表达式编译成静态方法，使用了成员变量的lambda语句则编译为成员方法**。

第二个问题我们将留待后面回答。

## Lambda调用
上面我们看到了lambda表达式的代码编译成了一个独立方法，指北君继续带领大家查看编译后的文件，我们要了解编译后lambda方法是如何调用执行的。
查看invokeLambda方法的编译后的内容（直贴出了关键部分）：

```sh
  public void invokeLambda();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=4, locals=4, args_size=1
        ... ...
        32: istore_3
        33: aload_2
        34: invokedynamic #45,  0             // InvokeDynamic #0:accept:()Ljava/util/function/Consumer;
        39: invokeinterface #49,  2           // InterfaceMethod java/util/List.forEach:(Ljava/util/function/Consumer;)
        ... ...
```
在invokeLambda中有一个指令invokedynamic，熟悉动态语言的小伙伴可能知道，这个指令是Java7为支持动态脚本语言而增加的。而函数式Java调用函数接口也正是通过invokedynamic指令来实现的。invokeLambda的详细内容指北君后续单独为大家讲解，今天我们关注函数接口的调用过程。

使用invokeLambda指令，那么该指令是直接调用的lambda$0方法么？我们知道list.forEach(xx)调用中，我们是将函数接口作为参数传递到其他类的函数中进行执行的。Java需要解决两个问题：1）如何将方法传递给被调用的外部类的方法，2）外部的内和方法如何访问我们内部私有的方法。

## 引导方法表
为解决上面两个问题，我们继续查编译后的文件，在末尾，我们看到下面的部分：

```sh
BootstrapMethods:
  0: #146 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #148 (Ljava/lang/Object;)V
      #151 invokestatic com/tree/sample/func/LambdaBinaryCode.lambda$0:(Ljava/lang/Integer;)V
      #152 (Ljava/lang/Integer;)V
  1: #146 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #153 (Ljava/lang/Object;)V
      #156 invokevirtual java/io/PrintStream.println:(Ljava/lang/Object;)V
      #157 (Ljava/lang/Integer;)V
  2: #146 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #159 (Ljava/lang/Object;Ljava/lang/Object;)V
      #162 invokespecial com/tree/sample/func/LambdaBinaryCode.lambda$2:(Ljava/lang/Integer;Ljava/lang/Integer;)V
      #163 (Ljava/lang/Integer;Ljava/lang/Integer;)V
InnerClasses:
     public static final #169= #165 of #167; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
```

这生成了三个引导方法，刚好和我们的三个函数接口调用一致，从引导方法的参数我们看出

序号|调用|调用类型
-|-|-
0|lambda$0|static
1|PrintStream.println|vertual
2|lambda$2|special

顺便回答一下之前的方法名称的数字序号不连续问题，我们看出，方法名称的序号是根据引导方法的序号来确定的，不是根据生成的lambda表达式方法序号来的。
我们看到，引导方法的逻辑似乎就是调用lambda方法或者其他的函数接口，每个引导方法中都出现了LambdaMetafactory.metafactory方法

## 动态调用
现在，我们结合invokedynamic指令来说明BootstrapMethods执行的过程

![动态调用逻辑](/assets/images/2021/simsky/jdk_src_func_mech_1.png)

上面的的流程显示了动态调用的基本逻辑
1. 执行invokedynamic
2. 检查调用点是否已连接可用
---
3. 如果未连接，构建动态调用点
4. 执行引导方法
5. 生成并加载调用点对应的动态内部类
6. 连接
---
7. 调用动态内部类方法
8. 内部类调用lambda对应的方法并执行

这两个阶段我们通过调用堆栈也能明显观察到：

![引导阶段](/assets/images/2021/simsky/jdk_src_func_mech_2.png)

![执行阶段](/assets/images/2021/simsky/jdk_src_func_mech_3.png)

我们还可以通过设置VM参数-Djdk.internal.lambda.dumpProxyClasses，查看以引导阶段动态生成的内部类：

![动态内部类列表](/assets/images/2021/simsky/jdk_src_func_mech_4.png)

打开其中一个如下：

![动态内部类详情](/assets/images/2021/simsky/jdk_src_func_mech_5.png)


## 小结
动态函数接口的调用原理，指北君就给大家介绍到这里了，相信大家看完本篇内容后，对函数式接口有了更深一层的学习。
由于涉及的内容较多，没有时间给大家逐一详细的给每个涉及到的类进行解读。后续指北君会根据小伙伴们需要对今天提及的知识点做深入的阶段，比如invokeddynamic指令，class结构，动态调用相关的各部分代码逻辑。


