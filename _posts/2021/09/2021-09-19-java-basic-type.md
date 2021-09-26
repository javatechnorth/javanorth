---
layout: post
title: Java 基本类型
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

今天指北君要带大家继续学习Java的基本数据类型。

大家都知道 Java 是一门强类型的编程语言，这就是说必须为每一个变量显示的声明一种类型。在 Java 中，一共有8种基本类型，其中有4种整型、2种浮点型、1种字符类型和1种布尔类型。

<!--more-->
在了解这些基本类型之前，我们先了解一下另外两个概念-- bit 和 字节。

- bit 是信息技术的最基本存储单位，非常小。计算机就是以二进制存储数据的，二进制的一位就是1bit。
- 字节与bit的换算关系是 1字节 = 8 bit。通常1一个英文字符 = 一个字节， 一个中文字符 = 2个字节。

### 整型

整型是用于表示没有小数部分的数值，它可以是负数。Java中有4种整型，具体看表格

|类型|存储大小|取值范围|
|---|---|---|
| int | 4个字节 |-2^31 ~ 2^31-1 |
| short | 2个字节 | -2^15 ~ 2^15-1 |
| long | 8个字节 | -2^63 ~ 2^63-1 |
| byte | 1个字节 ｜ -128 ~ 127 |

一般情况，需要使用整型存储数据的时候，我们可以使用int。如果int 存储不够使用的情况下，我们就使用 long。

在Java中，整型的范围与运行Java代码的机器系统平台无关。这就解决了软件在不同平台互相移植给程序员带来一系列问题。

### 浮点类型

浮点类型用于表示有小数部分的数值。在Java中，有两种浮点类型，具体我们来看下表格

|类型|存储大小|取值范围|
|---|---|---|
| float | 4个字节 |大约±3.40282347E+38F(有效位数为6~7位) |
| double | 8个字节 | 大约 ±1.79769313486231570E+308(有效位数为15位) |

double 表示这种类型的数值精度是 float 类型的两倍(有人称之为双精度数值)。 绝大部分应用程序都采用 double 类型。

float类型的数值有一个后缀F或f(例如，3.14F)。没有后缀F的浮点数值(如3.14)默认为double类型。当然，也可以在浮点数值后面添加后缀D或d(例如，3.14D)。

所有的浮点数值计算都遵循IEEE754规范。

### char类型

char 类型原本用于表示单个字符。

有些 Unicode字符也可以用一个 char来描述。说到这指北君就带大家稍微了解一下Unicode编码吧。

#### 为什么会出现Unicode编码？

在 Unicode 出现之前， 已经有许多种不同的标准: 美国的 ASCII、西欧语言中的 ISO8859-1 俄罗斯的 KOI-8、 中国的 GB18030 和 BIG-5 等。

这样就产生了下面两个问题: 一个是对于任意给定的代码值，在不同的编码方案下有可能对应不同的字母; 二是采用大字符集的语言其编码长度有可能不同。

设计 Unicode 编码的目的就是要解决这些问题。最初Unicode只有65536的一半都不到，经过一段时间的发展，65536已经不够使用了。所以出现了上述所说的部分Unicode 可以通过char来描述。

### 布尔类型

布尔(boolean)类型有两个值: false 和 true , 用来判定逻辑条件 整型值和布尔值之间不能进行相互转换。

### 总结

在本文中，指北君带大家学习了Java的基本数据类型，下一次我们继续学习运算符哦。
