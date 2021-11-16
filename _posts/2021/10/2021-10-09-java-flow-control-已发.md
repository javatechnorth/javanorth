---
layout: post
title: 教实习生系列之Java流程控制 -- 20211011
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

**实习生**：指北君，今天能给我讲讲流程控制吗？流程控制到底有哪些东西。

**指北君** ：好，今天我就给你把把脉。Java 中的流程控制相关的东西也挺多的，比如说块作用域、if-else、switch、for循环、while循环等等。
<!--more-->
### 块作用域

**指北君**：在我们学习流程控制语句之前呢，我们先来熟悉一下什么是块作用域。

块 是指一对大括号括起来的若干条简单的Java语句，它确定了变量的作用域，多个块允许嵌套。

```java
public static void main(String[] args) {
	int a;
	...
	{
		int b;
		...
	}
}
```
但是呢， 嵌套块中的变量不能重复定义。比如下面的示例代码就不能编译成功。
```java
public static void main(String[] args) {
	int a;
	...
	{
		int b;
		int a; //这里重复定义了变量a，导致编译出错
		...
	}
}
```

### if-else 条件语句

![image-20211007105827935](https://gitee.com/274904168/image-repo/raw/master/202110071058012.png)

#### if 语句
if 语句的基本格式：

```java
if (condition) {
	statement
}
```

来个示例

```java
public static void main(String[] args) {
	int age = 18;
	if (age < 30) {
		System.out.println("年轻有为");
	} 
}
```

结果输出：

```java
年轻有为
```

#### if-else 语句
if-else 语句基本格式：

```java
if (condition) {
	statement
} else {
	statement
}
```
来个示例

```java
public static void main(String[] args) {
	int age = 20;
	if (age > 18) {
		System.out.println("我成年了");
	} else {
		System.out.println("我还没成年");
	}
}
```

结果输出：

```java
我成年了
```

#### if-else-if 语句
if-else-if 语句基本格式

```java
if (condition1) {
	statement
} else if (condition2) {
	statement
} else {
	statement
}
```

来个示例：

```java
public static void main(String[] args) {
	int age = 20;
	if (age < 18) {
		System.out.println("我还没上大学");
	} else if (age >= 18 && age < 22) {
		System.out.println("我是大学生");
	} else {
		System.out.println("我是打工人！！！");
	}
}
```

结果输出

```java
我是大学生
```

### 循环语句

![image-20211007112032419](https://gitee.com/274904168/image-repo/raw/master/202110071120005.png)

**指北君**：我们比较常见循环有 for 循环和 while 循环，do..while 是 while 循环的一个变种。

#### for 循环

for 循环一般可以分为4个部分:

1)初始变量：循环开始执行时的初始条件。

2)条件: 循环每次执行前要检测的条件，如果为 true，就执行循环体; 如果为 false，就跳出循环。当然了，条件是可选的，如果没有条件，则会一直循环。

3)循环体: 循环每次要执行的代码块，直到条件变为 false。 

4)变量更新: 更新变量。

基本格式

```java
for(初始变量; 条件; 变量更新) {
	// 循环体
}
```

来个示例：

```java
public static void main(String[] args) {
	for (int i = 0; i < 5; i++) {
		System.out.println(i);
	}
}
```

结果输出：

```java
0
1
2
3
4
```

#### while 循环
while 循环基本格式

```java
while(condition){
	statement
}
```

来个示例

```java
public static void main(String[] args) {
	int i =0;
	while (i < 5) {
		System.out.println(i);
		i++;
	}
}
```

结果输出

```java
0
1
2
3
4
```

### switch 语句
在处理多个选项时，if-else语句显得有些笨拙，switch语句刚好能够承担起这个责任。switch能够在多个选项的条件下快速的比较其相等性。

switch 基本格式

```java
switch(choice) {
	case 1: ... break;
	case 2: ... break;
	default: ...
}
```

来个示例

```java
public static void main(String[] args) {
	int age = 20;
	switch(age) {
		case 20: 
			System.out.println("学生");
			break;
		case 25:
			System.out.println("打工人");
			break;
		default:
			System.out.println("未知");
	}
}
```

结果输出

```java
学生
```

**指北君**：仔细看的话，你会发现switch 语句中有个break，它是Java的一个关键字，主要是用来中断循环或者switch语句的。 

上面的switch示例中 age 在匹配到20之后打印了“学生”，之后就不会去判断25和default的分支，直接跳出了switch语句。

**实习生**：嗯嗯，这个我能够理解，我好像有看到continue，goto之类的关键字。

**指北君**：goto，在Java中一般都不建议使用的，使用goto很容易入坑，特别是大量使用的时候，会让你对整个业务逻辑变得难以理解。所以一般来说我们不轻易使用它。 continue就相对比较简单一些，使用场景基本就是循环中，中断当前循环的意思。

**实习生**：哦哦，好的，理解了。

**指北君**：Java流程控制相关的内容就这些了。今天先到这里下次聊。

**实习生**：好的，谢指北君。

### 总结
今天我们主要是聊了下Java流程控制，比如说块作用域、if-else、switch、for循环、while循环等等。这些流程控制的东西处处可见，我们需要掌握扎实的基础，来应对复杂的场景。



