---
layout: post
title:   Reader 字符流基础 -20211013
tagline: by 某某白米饭
categories: java
tags:
- 某某白米饭
---

大家好，我是指北君。

上次聊到了 java 的字节流 InputStream，今天来看看它的好朋友 Reader 字符流。

<!--more-->

### Reader

java 中的 IO 输入流不是只有 InputStream 还有按字符输入的 Reader。

| InputStream | Reader |
| --- | --- |
| 字节流，以 byte 为单位 | 字符流，以 char 为单位 |
| 读取字节（-1，0~255）：int read() | 读取字符（-1，0~65535）：int read() |
| 读到字节数组：int read(byte[] b) | 读到字符数组：int read(char[] c) |

和 InputStream 一样，Reader 也是所有字符输入流的超类。主要的方法是：`public int read() throws IOException`，read() 读取字符流中的下一个字符，返回 0-65535 的 int 类型数值, 返回 -1 表示已经读取结束。

### FileReader

FileReader 打开一个文件并获取到文件的字符流。 FileReader 用于读取文件中的内容。

```java
private void fileReaderDemo() throws Exception {
    Reader reader = new FileReader("D:\\readerDemo.txt");
    int n;
    while ((n = reader.read()) != -1) {
        System.out.print((char)n);
    }
    reader.close();
}
```

Reader 实现了 Closeable 接口，可以用 `try(Reader reader = new FileReader("D:\\readerDemo.txt")) {}` 的方式关闭掉资源。

### InputStreamReader

InputStreamReader 就是将 InputStream 读取的字节流装换为 Reader 的字符流。可以把任意的 InputStream 转换为 Reader，FileReader 就继承自 InputStreamReader。在创建 InputStreamReader 实例对象的时候可以指定字符集，以防止乱码。 

```java
private void inputStreamReaderDemo() throws Exception {
    InputStream inputStream = new FileInputStream("D:\\readerDemo.txt");
    try(Reader reader = new InputStreamReader(inputStream, "utf-8")) {
        int n;
        while ((n = reader.read()) != -1) {
            System.out.print((char)n);
        }
    }
}
```

### StringReader 和 CharArrayReader

FileReader 是将文件作为一个读取源，StringReader 将 string 字符串作为一个读取源。

```java
private void stringReaderDemo() throws Exception {
    try(Reader reader =  new StringReader("这是测试代码")) {
        char[] buffer = new char[1024];
        while ((reader.read(buffer)) != -1) {
            System.out.print(buffer);
        }
    }
}

```

reader.read(char[] buffer) 是 reader 读取字符流的重载方法，将内容不在是一个 char 一个 char 的输出，而是将内容读取到缓冲区 buffer 后一次性输出。

CharArrayReader 和 StringReader 几乎一样，调用方法变成了 `try(Reader reader =  new CharArrayReader("这是测试代码".toCharArray()))`

### BufferedReader

提供通用的缓冲方式读取文本并且提供了 readLine() 读取了一个文本行。从字符输入流中读取文本，缓冲各个字符，从而提供字符、数组和行的高效读取。

```java
private void bufferedReaderDemo() throws Exception {
    try(BufferedReader reader =  new BufferedReader(new FileReader("D:\\readerDemo.txt"))) {
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
    }
}
```

### 总结

介绍了几种常用 Reader 输入流的使用方式。FileReader 用于文件读取，BufferedReader 自带缓冲区读取效率高，StringReader 和 CharArrayReader 可以读取字符串源，InputStreamReader 将 InputStream 转为 Reader。

Demo代码GitHub： https://github.com/javatechnorth/java-north-sample/tree/master/reader
