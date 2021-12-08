---
layout: post
title: 读取 classpath 资源 --20211021
tagline: by 某某白米饭
categories: Java基础
tags: 
    - 某某白米饭
---

大家好，我是指北君。

Java 程序经常要读取配置文件（properties）、图片（jpg）、文本文件（txt、csv），我们可以使用 上次所讲的 InputStream 实现。 

<!--more-->

```java
public static void main(String[] args) throws Exception {
        String path = "D:\\config.properties";

        byte[] config = new byte[1024];

        try (InputStream inputStream = new FileInputStream(path)){
            while (inputStream.read(config) != -1) {
                System.out.println(new String(config, "utf-8"));
            }
        }
    }
```


### 读取classpath

java 程序经常是部署在 Linux 上的，必然不可能使用 "D:\" 这种盘符路径。总不能在开发的时候用 windows 路径，发布到 Linux 的时候注释掉吧？最终，将配置文件放在 java 程序的 resources 下

![](http://www.javanorth.cn/assets/images/2021/classpath/0.png)

1. 使用 class 处理

class 的 getResourceAsStream() 方法可以返回一个 InputStream。

```java
public void readProperties() {
    InputStream inputStream = this.getClass().getResourceAsStream("/config.properties");
    this.parseInputStream(inputStream);

}

public void parseInputStream(InputStream inputStream) {
    try {
        byte[] config = new byte[1024];
        while (inputStream.read(config) != -1) {
            System.out.println(new String(config, "utf-8"));
        }
    } catch (Exception e) {
        e.printStackTrace();
    }

}
```

该方法接受一个文件路径字符串参数，表示文件的路径，这个路径有两种写法：

以"/"开头，表示以类路径为起始目录。
不以"/"开头，表示相对于当前类的相对路径。


2. 使用 ClassLoader 处理

ClassLoader 下也是 getResourceAsStream() 方法，这个方法的参数不能加 `/`，不然就是找不到文件。

```java
public void readProperties() {
    InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("config.properties");

    this.parseInputStream(inputStream);

}
```

### getResourceAsStream 源码分析

把 class.getResourceAsStream() 方法上路径参数中的 `/` 拿掉，就会报 `java.lang.NullPointerException` 异常，没有找到这个文件。

把 getResourceAsStream() 的源码打开，就会发现读取资源文件路径的源码中调用的居然是 ClassLoader 的 getResource() 方法。

```java
public URL getResource(String name) {
    URL url;
    if (parent != null) {
        url = parent.getResource(name);
    } else {
        url = getBootstrapResource(name);
    }
    if (url == null) {
        url = findResource(name);
    }
    return url;
}
```

也就是说 class 和 classLoader 读取文件实际上调用的是同一个方法。不过在 class.getResourceAsStream() 调用 getResource 之前会判断路径的最前面是否有 `/`，没有 `/` 就会加上包名。

![](http://www.javanorth.cn/assets/images/2021/classpath/1.png)

### 总结

在本文中学习了如何读取 classpath 下的文件，以及看了看获取文件路径的源码，用 getResourc() 方法就可以知道读取文件的路径是否正确。
