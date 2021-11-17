---
layout: post
title:  防止NullPointerException，是程序员的基本修养
tagline: by 揽月中人
categories: Java
tags:

- 揽月中人
---

<!--more-->

如果一个Java程序到处抛出NullPointerException，那么我们可能会各种吐槽代码的质量。但是我们的项目又经常会抛出空指针异常，空指针异常必然常伴Java程序员。为此我们需要了解一些它喽，NullPointerException是Java种一个常见的RuntimeException，正如阿里的Java开发手册所说，对于Java程序员来说防止NPE是一种基本素养。今天我们盘一下NullPointerException。

### 1 NullPointerException以及其产生的场景

Java中定义：在应用程序中尝试使用null时会抛出次异常。

其中以下的情况会产生NullPointerException

1. 调用空对象的方法，

2. 访问或者修改对象的字段值时

3. 获取一个空对象（数组）的长度时，

4. 修改或者回去null数组的一个元素或者值时

5. 抛出异常时，抛出null。

以上是针对各种具体发生异常的情况，而对与日常使用过程中，可能抛出空指针异常的情景非常多，我们常用的手册中提到了以下几种NPE产生的实际使用场景。

- 返回类型为基本数据类型，return包装数据类型的对象时，自动拆箱有可能产生NPE。 

- 数据库的查询结果可能为null。 

- 集合里的元素即使isNotEmpty，取出的数据元素也可能为null。 

- 远程调用返回对象时，一律要求进行空指针判断，防止NPE。

- 对于Session中获取的数据，建议进行NPE检查，避免空指针。 

- 级联调用obj.getA().getB().getC()；一连串调用，易产生NPE。



####  自动拆箱的NPE

如下代码中，代码逻辑出现问题的话，返回就是null。

```Java
public int count(){
    Integer integer = null;
    。。。。。。
    return integer;
}
```

同理如下代码也是同样的问题，均属于自动拆装箱时的NPE问题。

```java
public static Boolean callSuccess(){
    return null;

};
```

从集合取出的值直接使用会遇到NPE.

```java
Map<String,String> map= new HashMap<>();
map.get("test").equals("test");
```



### 2 NPE处理以及如何避免

一般程序中需要处理NPE的地方随处可见，常见的NPE预防介绍如下几种方法。

#### 2.1 如果是链式get这种推荐使用Optional进行处理

如下代码

```java
public class Department {
    private String departmentName;
    private Company company;
    ...
}
public class Group {
    private Department department;
    ...
}
public class Employee {
    private String staffName;
    private Group group;
    ...
}

```

如果程序中需要如此调用

```java
employee.getGroup().getDepartment().getDepartmentName();
```

那么每一处均可能出现NullPointerException，如果我们写成下面这样。

```java
if (employee != null){
    if(employee.getGroup() != null){
        if(employee.getGroup().getDepartment() != null){
            String departmentName = 	  employee.getGroup().getDepartment().getDepartmentName();
        }
    }
}
```

if嵌套大军来袭，尔等还不下马受死。

上述if嵌套看起来的确很不美观，使用Optional可以比较容易的避免这些if判断，代码也会优雅不少。

下面不管哪一层为null返回均为Default。

```java
String s = Optional.ofNullable(employee)
        .map(Employee::getGroup)
        .map(Group::getDepartment)
        .map(Department::getDepartmentName).orElse("Default");
```

或者使用如下方法，如果某一层为null则返回Supplier的执行结果。

```
String s1 = Optional.ofNullable(employee)
        .map(Employee::getGroup)
        .map(Group::getDepartment)
        .map(Department::getDepartmentName).orElseGet(() -> {
            return "Supplier default";
        });
```

#### 2.2 主动进行参数检查，对方法中传入的参数进行检验

大部分的源码中使用的基础检查均会检查null

```java
public static String testString(String str) throws Exception {
    if (str == null){
        throw new Exception("param can't be null");
    }
    return str;
}   
```

#### 2.3 在已知字符串上使用equals()，equalsIgnoreCase()等方法。

```java
"knownObject".equals(unknownObject)
```

#### 2.4 尽量避免方法中返回null

一些返回数组或者List的方法，如果没有值，尽量返回空集合，避免返回null。

#### 2.5 新版本中Java输出的NullPointException详细信息

Java14 可以使用增强异常信息来查看`NullPointerException`的详细错误信息。Java17已经默认开启。

```java
java -XX:+ShowCodeDetailsInExceptionMessages NPTDemo
```

使用Java17执行如下语句及NullPointException的输出

```java
Map<String,String> map= new HashMap<>();
map.get("test").equals("test");
```

```
E:\Java\jdk-17.0.1\bin>java NPTDemo
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.equals(Object)" because the return value of "java.util.Map.get(Object)" is null
        at NPTDemo.main(NPTDemo.java:24)
```

上述详情请见如下地址，欢迎来瓢。
https://github.com/javatechnorth/java-study-note/tree/master/multiThread/src/main/java/org/javanorth/currency/npt


### 总结

记住一句话：**避免空指针异常的最好的方法就是总是检查哪些不是自己创建的对象。**

Java新版本中的NullPointException的详细信息的输出对我们定位错误帮助很大，也是一个强有力的问题排查方法。

