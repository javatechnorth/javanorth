---
layout: post
title: 字符串拼接这种小事，我翻车了... --20211210
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

最近指北君在一个字符串拼接的小事情上翻车了，真的是万万没想到。Java 提供了多种方法和类可以用来拼接字符串。但是如果我们不注意 null 对象，则生成的 String 可能包含一些不需要的值。

<!--more-->

### 问题场景

假设我们要拼接的 String 数组的元素，其中任何元素都可能为 null。

举个例子
```java
String[] values = { "Java ", null, "", "is ", "great!" };
String result = "";

for (String value : values) {
	result = result + value;
}
```

我们简单地使用 + 运算符执行拼接，得出来的结果是

```java
Java nullis great!
```

但是，我们正常情况肯定是不喜欢在结果中包含 "null" 值。

同样，如果我们的应用程序在 Java 8 或更高版本上运行，我们使用 `String.join()` 静态方法获得相同的输出：

```java
String result = String.join("", values);
```

我们也不能避免在使用 `String.join()` 方法时连接 null 元素。

让我们看看一些方法来避免这些空元素被串联起来，并获得我们期望的结果："Java is great"

### 使用 + 运算符

加法 (+) 运算符被重载以连接 Java 中的字符串。在使用 + 运算符进行连接时，我们可以检查 String 是否为 null，并将 null 字符串替换为空 ("")字符串：

```java 
for (String value : values) {
	result = result + (value == null ? "" : value);
}

assertEquals("Java is great!", result);
```

或者，我们可以将检查空字符串的代码提取到一个 `getNonNullString()` 方法中，该方法接受一个String对象并返回一个非空字符串对象：

```java
public String getNonNullString(value) {
    return value == null ? "" : value;
}

for (String value : values) {
	result = result + getNonNullString(value);
}
```

但是 String 对象在Java中是不可变的。这意味着每次我们使用 + 运算符连接 String 对象时，都会在内存中创建一个新的 String。因此使用 + 运算符进行拼接是比较浪费资源的。

### 使用String.concat() 方法

当我们想要拼接 String 对象时，`String.concat()` 方法是一个不错的选择。

在这里，我们可以使用我们的 `getNonNullString()` 方法，该方法检查空对象并返回空字符串：
```java 
for (String value : values) {
    result = result.concat(getNonNullString(value));
}
```
getNonNullString()方法返回的空字符串与结果串联，从而忽略null对象。

### 使用StringBuilder类

StringBuilder 提供了一堆有用且方便的字符串构建方法。其中之一是 append() 方法。

在这里，我们可以使用相同的 `getNonNullString()` 方法来避免在使用 append() 方法时出现空对象：

```java
for (String value : values) {
    result = result.append(getNonNullString(value));
}
```

### 使用StringJoiner类 (Java 8+)

StringJoiner 类提供了 `String.join()` 的所有功能，以及一个以给定前缀开头并以给定后缀结尾的选项。我们可以使用它的 `add()`方法来连接字符串s。

和以前一样，我们可以使用我们的帮助器方法 `getNonNullString()` 来避免空字符串值被串联起来：

```java
StringJoiner result = new StringJoiner("");

for (String value : values) {
    result = result.add(getNonNullString(value));
}
```
`String.join()` 和 StringJoiner 之间的一个区别是，与 `String.join()` 不同，我们必须遍历集合(Array、List等)来联接所有元素。

### 使用Streams.filter (Java 8+)

Stream API 提供大量顺序和并行聚合操作。一个这样的中间流操作是过滤器，它接受一个谓词作为输入，并根据给定的谓词将流转换为另一个流。

因此，我们可以定义一个谓词，该谓词将检查字符串的空值，并将此谓词传递给filter()方法。因此，筛选器将从原始流中筛选出这些空值。

最后，我们可以使用Collectors.joining()连接所有这些非空字符串值，最后将生成的Stream收集到String变量中：
```java
result = Stream.of(values).filter(value -> null != value).collect(Collectors.joining("")); 
```

### 总结

在本文中，我们演示了避免 null 字符串对象串联的各种方法。总会有不止一种正确的方法来满足我们的要求。因此，我们必须确定哪种方法最适合给定的地方。

我们必须记住，连接String本身可能是一个昂贵的操作，特别是在循环中。因此，始终建议考虑 Java 字符串 API 的性能。
