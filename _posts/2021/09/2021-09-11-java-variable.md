---
layout: post
title: Java 变量
tagline: by feng
categories: Java基础
tags: 
    - feng
---

# Java变量

大家好，我是指北君。

今天指北君来给大家讲讲Java变量。

### 什么是变量？

变量就是初中数学的代数的概念，例如一个简单的方程，x，y都是变量：

y=x^2+1

在Java程序设计中，变量是指一个包含值的存储地址以及对应的符号名称。

从定义上来看，变量大概可分为两部分：变量的名字和变量的值，可通过变量名来访问变量值。简单来说，变量就是存储数据的载体。

对于 Java 语言来讲，Java 是一种强类型的语言，因此变量都必须有一个类型。在 Java 代码中，变量是长这个样子的：
<!--more-->
```java
// =左侧的age是变量名，右侧的22是变量值
int age = 22;
```
这其中包含了变量的声明和变量的赋值。其中 `int age` 表示变量的声明，声明 `age` 的变量类型为 `int`类型。`age = 22` 的过程表示的是变量的赋值。

在Java中，变量分为两种：基本类型的变量和引用类型的变量。

### 变量命名规范

对于变量的命名，每种编程语言都有自己的规则和约定，Java 语言也不例外。命名变量的规则和约定如下：

- 变量名必须是一个以字母开头并由字母或数字构成的序列。需要注意，与大多数程序设计语言相比，Java中“字母”和“数字”的范围更大。字母包括'A'~'Z'、'a' ~'z'、'_'、'$'或在某种语言中表示字母的任何 Unicode 字符。
- 变量名不能使用Java保留字或关键字。
- 变量命名区分大小写。

### 局部变量

在Java中， 方法体内声明的变量一般被称为局部变量。该变量只能在该方法内使用，类中的其他方法并不知道。

![举个栗子_举个_栗子表情](http://www.javanorth.cn/assets/images/2021/feng/lizi.jpg)

```java
public class LocalVar {
	public static void main(String[] args) {
		int a =0, b=1;
		int c = a + b;
		System.out.println(c);
	}
}
```

其中 a、b、c就是局部变量，它们只能在当前这个 main 的方法中使用。

### 成员变量

一般来说，成员变量就是在类的内部但在方法体的外部声明的变量。我们再举个例子：

```java
public class InstanceVar {
    int data = 123;
    public static void main(String[] args) {
        InstanceVar ins = new InstanceVar();
        System.out.println(ins.data);
    }
}
```

在示例中，data 就是一个成员变量，通过InstanceVar 的实例 ins 来访问。ins 也是一个变量，它的类型就是InstanceVar，通过 new 操作之后在赋值得来的。



### 静态变量

在Java中，静态变量是通过 static 关键字指示的。

```java
static DataType 变量名 = 变量值；
```

我们再来看个例子吧：

```java
public class StaticVar {
    static int data = 100;
    public static void main(String[] args) {
        System.out.println(StaticVar.data); 
    }
}
```

在示例中, data 就是静态变量，通过`类名.变量名` 进行访问。



### 常量


在Java中，利用 `final`关键字指示变量：

```text
final DataType 常量名 = 常量值;
```

常量在程序运行过程中主要有 2 个作用:

- 代表常数，便于修改(例如:圆周率的值， final double PI = 3.14 ) 

- 增强程序的可读性(例如:常量UP、DOWN 用来代表上和下， final int UP = 0 )

如果我们尝试在代码中修改常量的值：

```text
class FinalVar {
    public static void main(String[] args) {
        // 声明并初始化常量 TOTAL_NUM
        final int TOTAL_NUM = 200;
        // 对 TOTAL_NUM 重新赋值
        TOTAL_NUM = 20;
    }
}
```

编译执行代码，编译器将会报错：

```text
FinalVar.java:6: 错误: 无法为最终变量TOTAL_NUM分配值
        TOTAL_NUM = 20;
        ^
1 个错误
```

适当地使用常量可以提高代码的安全性和可维护性。

### 总结

在本文中，我们学习了什么是变量，变量的命名规范。

Java 中变量有3个种类，分别是：局部变量、成员变量、静态变量。其中变量如果使用了`final`关键字修饰，就可定义一个不可变的常量。




