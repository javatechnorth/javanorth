---
layout: post
title:  使用Java去除html标签的几种方法-已发
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在我平时的工作中，偶尔会用 Java 做一些解析html的工作。有的时候我需要删除所有的HTML标签，只保留纯文字内容。这个问题在做过一些爬虫工作的朋友来说很简单。 下面来说说，我们平时使用到的集中解析的方法。

<!--more-->

### 使用正则表达式

通过爬虫爬到的html内容，从程序角度来讲，就是一个字符串。我们可以对其按照纯文本处理的方式来处理。 

我们在做文本处理的时候，第一个想到的就是正则表达式。 从一个字符串中删除HTML，对于正则来说，还是比较简单的。毕竟还是有固定的格式，比如“<...>”。

我们常用的的正则就是 `<[^>]>` 或者 `<.*?>` 。

我们在使用正则的时候，需要注意的是正则默认是贪婪匹配。也就是说，正则表达式 `<.*>` 能够匹配到更多的html内容，而不是单个标签。

现在，让我们测试一下它是否能从HTML源中删除标签。

#### 正则测试删除标签1

在我们测试删除HTML标签之前，首先让我们创建一个HTML例子，例如`example1.html`。

```html
<!DOCTYPE html>
<html>
<head>
    <title>这是标题</title>
</head>
<body>
    <p>
        如果应用程序X没有启动，可能的原因是<br/>
        1. <a href="https://maven.apache.org">Maven</a>没有安装<br/>
        2. 磁盘空间不足<br/>
        3. 内存不足
    </p>
</body>
</html>

```

现在，让我们写一个测试，用`String.replaceAll()`来删除HTML标签。

```Java
String html = ... // load example1.html
String result = html.replaceAll("<[^>]`>", "");
System.out.println(result);
```

如果我们运行这个测试方法，我们会看到结果。

```text
    这是标题



        如果应用程序X没有启动，可能的原因是
        1.Maven没有安装
        2.磁盘空间不足
        3.没有足够的内存
```


输出结果保留了剥离后的HTML的空白处。我们在处理提取的文本时，可以很容易地删除或跳过这些空行或空白处。

#### 正则测试删除标签2

我们刚才已经看到了，通过使用Regex来删除HTML标签是非常简单。但是粗暴的使用这种方法会有很多问题，我们不能预测最终的结果会是怎么样的。

例如，一个HTML文档可能有`<script>`或`<style>`标签，而我们可能不希望在结果中出现它们的内容。

此外，`<script>`、`<style>`、甚至是`<body>`标签中的文本可能包含 `<`或 `>`字符。如果是这种情况，我们的正则方法可能会出错。

现在，让我们看看另一个例子，比如`example2.html`。

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>这是标题</title>
</head>
<script>
    // some js function
</script>
<body>
    <p>
        如果应用程序X没有启动，可能的原因是<br/>
        1. <a
            id="link"
            href="http://maven.apache.org/">
            Maven
            </a> 没有安装<br/>
        2. 磁盘空间不足 (<1G) <br/>
        3. 内存不足(<64MB)<br/>
    </p>
</body>
</html>
```
现在我们有一个`<script>`标签和 `<`字符在`<body>`标签内。

如果我们对`example2.html`使用同样的方法，我们会得到如下内容。

```HTML
   这是标题
    // some js function
        如果应用程序X没有启动，可能的原因是
        1. 
            Maven
             没有安装
        2. 磁盘空间不足 (
        3. 内存不足(
```

显然，由于"<"字符的存在，我们丢失了一些文本。 所以正则在处理文本的时候并不是万能的。 我们可以使用一些 HTML 解析器来做这些比较复杂的场景。

### 使用Jsoup

Jsoup 是一个流行的HTML解析库，如果想要从一个HTML文档中提取文本，我们可以简单地调用`Jsoup.parse(htmlString).text()`。

在项目中使用的时候，我们首先需要添加 jsoup 的依赖库，我们这里就通过maven的方式引入。

```XML
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.14.3</version>
</dependency>
```

我们用 `example2.html`来测试一下。

```Java
String html = ... // load example2.html
System.out.println(Jsoup.parse(html).text());
```

如果我们让这个方法运行，它就会打印出来。

```text
这是标题 如果应用程序X没有启动，可能的原因是 1.Maven没有安装 2.没有足够的（<1G）磁盘空间 3.没有足够的（<64MB）内存
```

从输出结果可知，Jsoup已经成功地从HTML文档中提取了文本。另外，`<script>`元素中的文本已经被忽略了。

此外，默认情况下，Jsoup会删除所有的文本格式和空白处，比如换行符。


### 使用HTMLCleaner

HTMLCleaner 也是一个HTML解析库。

首先，我们需要在`pom.xml`中添加[HTMLCleaner 依赖](https://search.maven.org/search?q=a:htmlcleaner%20g:net.sourceforge.htmlcleaner)。

```XML
<dependency>
    <groupId>net.sourceforge.htmlcleaner</groupId>
    <artifactId>htmlcleaner</artifactId>
    <version>2.25</version>
</dependency>
```

我们可以设置[各种参数]（http://htmlcleaner.sourceforge.net/parameters.php）来控制HTMLCleaner的解析行为。 我们在这里使用HTMLCleaner在解析`example2.html`时跳过`<script>`元素。

```java
String html = ... // load example2.html
CleanerProperties props = new CleanerProperties();
props.setPruneTags("script");
String result = new HtmlCleaner(props).clean(html).getText().toString();
System.out.println(result);
```

运行一下，HTMLCleaner将产生这样的输出。

```
这是标题



        如果应用程序X没有启动，可能的原因是：
        1.Maven没有安装
        2.没有足够的（<1G）磁盘空间
        3.内存不足（<64MB）
```

我们可以看到，`<script>`元素中的内容被忽略了， `<br/>`标签转换为提取的文本中的换行符。 另外， HTMLCleaner 保留了HTML的空白内容。 

### 总结

在这篇文章中，我们学习了几种去除HTML的方法，我们需要注意的是，正则在文本处理的过程中并不是万能的。
