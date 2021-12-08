---
layout: post
title:  司空见惯的Exception，你了解她的秘密吗？--20210812
tagline: by simsky
categories: JDK 异常
tags: 
    - simsky

---

大家好，我是指北君，对于Exception，不用说Java人都知道。指北君一直以来有个不治之症：对Java的一切东东有追根寻底的强迫症，不了解其所以然就睡不了觉，这不，为了能睡个好觉就带领大家探一探Exception的秘密。

<!--more-->

### 引子
首先，指北君声明一下，这里讲的Exception不仅仅是Exception一个类，而是异常机制，就像下面图中的所有（含继承Error和Exception的）。

![异常类图](/assets/images/2021/simsky/jdk_exception_1.png)

当然JDK中更为细致的异常继承体系也不是本篇探讨内容，本次呢，指北君将对异常在JVM层面的执行原理和ARM进行介绍，这里ARM也不是芯片架构，而是指自动资源管理。

### 异常表
要想了解异常执行原理，异常表是一个最佳的入口，异常表是什么？我来看看下面的数据结构描述：
```c++
exception_table {
    u2 start_pc;
    u2 end_pc;
    u2 handler_pc;
    u2 catch_type;
}
```
它是虚拟机中用于表述异常处理的一种数据结构，我再来看异常表的样例，下面是一段包含try-catch代码编译后的字节码中的内容：

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

现在，回到异常表，我们可以看到异常表包含多条异常处理数据，每条数据包含四个属性
+ from 监控的指令集起始编号
+ to 监控的指令集结束编号
+ target 满足条件后指令跳转目标
+ 异常匹配的类型，特殊类型any，对应fanally块，监控try块和catch块

对于return,athrow等指令，执行顺序需要注意，指令是在对应的处理之后执行。

可能会让你意识混乱的例子：
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
我们之前说过finally块会在return之前执行，那是不是执行结果是：try->finally->end这种顺序呢？实际的执行的结果是：try-block,end。
可以看到，finally块没有对结果形成影响，当然，这并不说它没执行，只是reutrn的结果在执行finally块时已经计算出结果了，在执行完finally块后将之前计算的结果返回了而已。所以我们要深入理解了执行的原理，才能正确理解结果。

### ARM 自动资源管理
讲完前面的异常执行原理后，指北君现在给大家介绍异常机制在JDK1.7中一个优化特性：自动资源管理。
经常使用IO的小伙伴一定对关闭IO很烦，写法繁琐，关闭前还要做判断，并且在关闭块代码还要加try-catch，就如同下面类似的代码：

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

反正指北君每次写这种代码的时候很烦，就是那种啥事没干，占我一块黄金代码位置的感觉，对于有精简代码癖好的人来说，就像吃了一口苍蝇般难受。
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
是不是简洁多了，资源直接在try后的括号内进行声明，而且不需要显式的关闭流的代码，这难道就是ARM！是不是有点小兴奋！
对于ARM自动资源管理要点如下：
+ 在try后的括号内声明需要自动关闭的资源
+ 资源必须实现一个关键接口：AutoCloseable。
在满足上面的条件后，JDK将为我们自动关闭资源。当然这种写法即使想手动关闭，在catch和finally部分也拿不到资源引用。

又到了追根溯源环节了，自动资源管理是如何实现的呢？先看看class文件有啥变化。

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
对比之前的字节码文件，我们明显发现虽然代码行数少了，但是Class编译后的内容一点都没少，而且异常表中除了finally对应的处理，还增加了额外的部分。
具体几个不同点：
1. 字节码中竟然有close调用
2. 异常表中增加了内容

```shell
 Exception table:
    from    to  target type
       21    30    41   any
        4    52    52   any
        0    74    74   Class java/io/IOException
        0    78    85   any
```
好奇怪的异常表，我们查看指令，可以明显找出最后一条是对应finally处理。前面的呢？经过分析我们发现前面是finally形式的关闭资源。
看到这里，结果就很明显了，自动资源管理实际是上编译帮助我们做了显示关闭的逻辑，在JVM执行层面没有增加新的功能。简而言之，自动资源管理是编译器层面的改进，通过扩展语法和增强编译能力（增加自动的资源关闭能力）来实现自动化编码。

### 总结

关于Java的异常的原理，以及自动资源管理的用法和实现方式，指北君就给大家介绍到这里。

最后感谢各位人才的：点赞、收藏和评论，我们下期更精彩！

