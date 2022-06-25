---
layout: post
title:  聊聊 Java File分隔符和 Path分隔符
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

不同的操作系统使用不同的字符作为文件和路径分隔符。当我们的应用程序需要在多个平台上运行时，我们需要正确处理这些问题。

Java帮助我们选择一个合适的分隔符，并提供了一些函数来帮助我们创建能在主机操作系统上运行的路径。

在本文中，我们将了解如何编写代码来使用正确的文件和路径分隔符。
<!--more-->

### 2.文件分隔符

文件分隔符是用来分隔构成特定位置路径的目录名的字符。

#### 1.获取文件分隔符

在Java中，有几种方法可以获得文件分隔符。

我们可以用*File.separator*来获取*String*的分隔符。

```java
String fileSeparator = File.separator;
```

我们也可以用*File.separatorChar*获得这个分隔符的字符。

```java
char fileSeparatorChar = File.separatorChar;
```

从Java 7开始，我们也可以使用*文件系统*。

```java
String fileSeparator = FileSystems.getDefault().getSeparator();
```

输出结果将取决于主机操作系统。文件分隔符在Windows上为`\`，在macOS和基于Unix的操作系统上为`/`。

#### 2.构造一个文件路径

Java提供了一些方法来从目录列表中构建文件路径。

这里，我们使用*Paths*类。

```java
Path path = Paths.get("dir1", "dir2");
```

让我们在Microsoft Windows上测试一下。

```java
assertEquals("dir1\\dir2", path.toString());
```

同样，我们也可以在Linux或Mac上测试它。

```java
assertEquals("dir1/dir2", path.toString());
```

我们也可以使用*文件*类。

```java
File file = new File("file1", "file2");
```

让我们在Microsoft Windows上测试一下。

```java
assertEquals("file1\\file2", file.toString());
```

同样，我们也可以在Linux或Mac上测试它。

```java
assertEquals("file1/file2", file.toString());
```

正如我们所看到的，我们可以只提供路径字符串来构建一个文件路径---我们不需要明确地包括一个文件分隔符。

3.路径分隔符

路径分隔符是操作系统常用的一个字符，用于分隔路径列表中的各个路径。

### 3.1.获取路径分隔符

我们可以用*File.pathSeparator*来获取路径分隔符的*String*。

```java
String pathSeparator = File.pathSeparator;
```

我们也可以以*char*的形式获得路径分隔符。

```java
char pathSeparatorChar = File.pathSeparatorChar;
```

这两个例子都返回了路径分隔符。在Windows上是分号`;`，在Mac和基于Unix的操作系统上是冒号`:`。

### 3.2.构造一个文件路径

我们可以用分隔符作为分隔符来构建一个*String*的文件路径。

让我们试试*String.join*方法：

```java
String[] pathNames = { "path1", "path2", "path3" };
String path = String.join(File.pathSeparator, pathNames);
```

这里我们在Windows上测试我们的代码。

```java
assertEquals("path1;path2;path3", path);
```

而文件路径在Linux或Mac上看起来会有所不同。

```java
assertEquals("path1:path2:path3", path);
```

类似地，我们可以使用*StringJoiner*类：

```java
public static StringJoiner buildPathUsingStringJoiner(String path1, String path2) {
    StringJoiner joiner = new StringJoiner(File.pathSeparator);
    joiner.add(path1);
    joiner.add(path2);
    return joiner;
}
```

让我们在Microsoft Windows上测试我们的代码。

```java
assertEquals("path1;path2", buildPathUsingStringJoiner("path1", "path2"));
```

而且它在Mac或Unix上的表现会有所不同。

```java
assertEquals("path1:path2", buildPathUsingStringJoiner("path1", "path2"));
```

### 总结

在这篇短文中，我们学习了如何使用系统特定的文件分隔符来构建路径，这样我们的代码就可以在多个操作系统上工作。

我们看到了如何使用内置类*Path*和*File*来构造文件路径，我们还看到了如何获得必要的分隔符来使用*String*连接工具。