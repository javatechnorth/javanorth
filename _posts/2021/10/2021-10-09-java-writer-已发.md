---
layout: post
title:   Writer 字符流基础
tagline: by 某某白米饭
categories: java
tags:
- 某某白米饭
---

大家好，我是指北君。

上次聊到了 java 的字符流 Reader，今天来看看它的好朋友 Writer 字符流。

<!--more-->

### Writer

java 中的 IO 输出流不是只有 OutputStream 还有按字符输出的 Writer。

| OutputStream | Writer |
| --- | --- |
| 字节流，以 byte 为单位 | 字符流，以 char 为单位 |
| 输出字节（0~255）：void write(int b) | 输出字符（0~65535）：void write(int c) |
| 输出字节数组：void write(byte[] b) | 输出字符数组：void write(char[] c) |

和 OutputStream 一样，Writer 也是所有字符输出流的超类。主要的方法是：public void write(char cbuf[]) throws IOException， write() 方法将 cbuf 参数全部输出，write(String str) 和 write(int c) 两个方法最终调用的还是 write(char cbuf[])

### FileWriter

FileWriter 就是向文件中写入字符流的 Writer。new FileWriter(fileName) 构造函数是将文件从头开始写入并不是在文件结尾处继续写入。 new FileWriter(fileName, true) 则是在文件结尾处写入。

```java
private void fileWriteDemo() throws IOException {
    Writer writer = new FileWriter("D:\\writer.txt");
    writer.write("测试写入".toCharArray());
    writer.close();
}
```

Writer 实现了 Closeable 接口，可以用 `try(Writer writer = new FileWriter("D:\\writer.txt")) {}` 的方式关闭掉资源。

### OutputStreamWriter

OutputStreamWriter 将输出的字符流转换为字节流。可以使用指定的编码字符集。new OutputStreamWriter(OutputStream out, Charset cs) 。
```java
private void outputStreamWriteDemo() throws IOException {
    // 乱码
    try(OutputStreamWriter writer = new OutputStreamWriter(new FileOutputStream("D:\\writer.txt"), "gb2312")) {
        writer.write("杺");
        writer.write(66);
    }

    // 正常
    try(OutputStreamWriter writer = new OutputStreamWriter(new FileOutputStream("D:\\writer.txt", true), "gbk")) {
        writer.write("杺");
        writer.write(66);
    }

}
```

### StringWriter 和 CharArrayWriter

StringWriter 内部有一个 StringBuffer 对象作为其缓冲区。可以利用其缓冲区中的内容来构造字符串。

```java
private void stringWriteDemo() throws IOException {
    String str = "写入测试";
    try(StringWriter writer = new StringWriter()) {
        writer.write(str);
        writer.write(str);
        System.out.println(writer.getBuffer().toString());
    }

}

```

CharArrayWriter 和 StringWriter 几乎一样，也是在内存中构造一个字符串缓冲区。不过底层不是一个 StringBuffer 了，是 char 的数组，默认 32 个长度。

### BufferedWriter

BufferedWriter 是一个缓冲的字符输出流，为其他 Writer 提供缓冲的功能。

```java
private void bufferedWriteDemo() throws IOException {
    FileWriter fileWriter = new FileWriter("D:\\writer.txt", true);
    try(BufferedWriter writer = new BufferedWriter(fileWriter)) {
        writer.write(65);
        writer.write(66);
    }
    fileWriter.close();

}
```

### 总结

介绍了几种常用 Writer  输出流的使用方式。FileWriter 用于写入文件，BufferedWriter 自带缓冲区，StringWriter 和 CharArrayWriter 基于内存，OutputStreamWriter 将字符流转为字节流。

Demo代码GitHub： https://github.com/javatechnorth/java-north-sample/tree/master/writer
