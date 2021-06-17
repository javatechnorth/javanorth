---
layout: post
title:  司空见惯的Exception，你了解她的秘密吗？
tagline: by simsky
categories: JDK 异常
tags: 
    - simsky

---

大家好，我是指北君，对于Exception，相信只要会Java的都知道，指北君对Java的一切东东知其然还要知其所以然的强迫症已经不可救药了，今天就带领大家谈一谈Exception的秘密，也同时缓解一下强迫症症状。

<!--more-->
### 引子
首先，指北君声明一下，这里讲的Exception不仅仅是Exception一个类，而是异常机制，就像下面图中的所有（含继承Error和Exception的）。

![异常类图](/assets/images/2021/simsky/jdk_exception_1.png)

当然JDK中更为细致的异常继承体系也不是本篇探讨内容，指北君将对异常在JVM层面的执行原理和ARM进行介绍，这里ARM不是芯片架构，是指自动资源管理。

### 异常表
要想了解异常执行原理，异常表是一个最佳的入口，异常表是什么？它是虚拟机中用于表述处理异常的一种数据结构
```c++
exception_table {
    u2 start_pc;
    u2 end_pc;
    u2 handler_pc;
    u2 catch_type;
}
```
对应的在我们编译后的字节码中也有对应的部分：

```shell
        from    to  target type
            8    41    41   Class java/lang/IllegalArgumentException
            8    41    41   Class java/io/IOException
            8    53    63   any
```

逻辑上的表述可以用下图表示：

![异常类图](/assets/images/2021/simsky/jdk_exception_2.png)

1. 监测从8-41（41不包含）号指令的IllegalArgumentException
2. 监测从8-41指令的IOException
3. 监测从8到53指令的结束动作，包含三种情况：跳出指令（比如return指令集）；athrow指令且没有匹配的异常类型；指令执行到最后一条
4. 如果监测到制定动作，则按照异常表进行跳转

现在回到异常表，我们可以看到异常表是多条异常处理数据，每条数据包含四个数据
+ from 监控的指令集起始编号
+ to 监控的指令集结束编号
+ target 满足条件后指令跳转目标
+ 异常匹配的类型
+ 特殊类型any，对应fanally块，监控try块和catch块

对于return,athrow等指令，执行顺序需要注意，指令是在对应的处理之后执行。

让你意识混乱的例子：
```java
    public String execOrder(int sn) {
        StringBuilder build = new StringBuilder();
        try {
            build.append("try-block");
            if(sn == 0) {
                return build.append(", end").toString();
            }
        }catch(RuntimeException e) {
            build.append(", catch-block");
        }finally{
            build.append(", finally-block");
        }
        
        return build.toString();
    }
```
我们之前说过finally块会在return之前执行，那是不是try->finally->end这种顺序呢？结果是：try-block,end。fnially块没有对结果形成影响，但是这并不代表它没执行，只是reutrn的结果在执行finally块时已经计算出结果了，return只是执行方法返回对应的操作而已。

### ARM 自动资源管理
讲完前面的异常执行原理后，指北君现在给大家介绍异常在JDK1.7中一个优化特性——自动资源管理。
经常使用IO的小伙伴一定对关闭IO很烦，写法繁琐，关闭前还要做判断，并且在关闭块代码还要加try-catch

```java
        InputStreamReader in = null;
        OutputStreamWriter out = null;
        try {
            in = new InputStreamReader(new FileInputStream(""));
        }catch(IOException e) {
            
        }finally {
            try {
                if(in != null) {
                    in.close();
                }
                if(out != null) {
                    in.close();
                }
            }catch(IOException e) {
                
            }
        }
```

反正指北君每次写这种代码的时候后很烦，就是那种啥事没干，占我一块黄金代码位置的感觉，对于有精简代码癖好的人来说，就像吃了一口苍蝇难受。
有了ARM后，我们再来看看新的写法：
```java
    public void sample2() {
        try (InputStreamReader in  = new InputStreamReader(new FileInputStream(""))){
            in.read();
        }catch(IOException e) {
            
        }finally {
            
        }
    }
```
嗯？try上面可以进行资源声明！怎么不关闭流！这难道就是ARM？
是的，这就是ARM自动资源管理，关键点出了在try部分声明资源外，还需要资源实现一个关键接口：AutoCloseable。JDK将为我们自动关闭资源。这种写法即使想关闭，在catch和finally部分也拿不到资源引用。
知道怎么样了，指北君还想知道怎么实现的，该怎么办呢？先看看class文件有啥变化
我们发现虽然代码少，但是编译的内容不少，而且异常表中除了finally对应的处理，还增加了额外的部分。

```shell
21: aload_3
22: invokevirtual #112                // Method java/io/InputStreamReader.read:()I
25: pop
26: ldc           #51                 // String try-block
28: astore        4
30: aload_3
31: ifnull        94
34: aload_3
35: invokevirtual #103                // Method java/io/InputStreamReader.close:()V
38: goto          94
41: astore_1
42: aload_3
43: ifnull        50
46: aload_3
47: invokevirtual #103                // Method java/io/InputStreamReader.close:()V
50: aload_1
51: athrow
52: astore_2
```
我们发现：字节码中竟然有close调用，继续往下看：

```shell
 Exception table:
    from    to  target type
       21    30    41   any
        4    52    52   any
        0    74    74   Class java/io/IOException
        0    78    85   any
```
好奇怪的异常表，我们查看指令，可以明显找出最后一条是对应finally处理。前面的呢？经过分析我们发现前面是仿照finally形式的关闭资源。
很明显，在指令层没有明显的变化ARM是编译器层面的改进，通过扩展语法，编译器增加自动的资源关闭能力（自动化编码）。



### 总结

关于Java的异常的原理，以及自动资源管理的用法和实现方式，指北君就给大家介绍到这里。

最后感谢各位人才的：点赞、收藏和评论，我们下期更精彩！

