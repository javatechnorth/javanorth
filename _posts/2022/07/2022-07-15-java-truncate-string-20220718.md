---
layout: post
title:  Java 截取字符串的几种操作
tagline: by feng
categories: java
tags: 
    - feng
---
大家好，我是指北君。

在本文中，我们将学习在Java中把一个String截断到所需的字符数的集中方法。

首先，我们将探索使用JDK本身来实现这一目标的方法。然后，我们将研究如何使用一些流行的第三方库来实现这一目标。
<!--more-->

### 使用JDK截断一个字符串

Java提供了许多方便的方法来截断一个 String 。让我们来看看。

#### 使用 String 的 substring() 方法

 String 类有一个方便的方法，叫做 `substring` ，正如其名称所示 ， `substring()` 返回指定索引之间的 `String` 部分。

让我们来看看它的运行情况。

```java
static String usingSubstringMethod(String text, int length) {
    if (text.length() <= length) {
        return text;
    } else {
        return text.substring(0, length);
    }
}
```

在上面的例子中，如果指定的 length 大于 text 的长度，我们返回 text 本身。这是因为  传递给 `substring()` 的 length 大于 String 的字符数会导致 `IndexOutOfBoundsException`   。

否则，我们将返回从索引0开始并延伸到--但不包括--索引 length 的字符的子串。

让我们用一个测试案例来确认这一点。

```java
static final String TEXT = "Welcome to  javanorth.cn";

@Test
public void givenStringAndLength_whenUsingSubstringMethod_thenTrim() {

    assertEquals(TrimStringOnLength.usingSubstringMethod(TEXT, 10), "Welcome to");
}
```

正如我们所看到的，  的起始索引是包容的，结束索引是排他的  。因此, 索引 length 处的字符将不包括在返回的子串中。

#### 使用 String's split() 方法

  另一种截断 String 的方法是使用 split() 方法，它使用正则表达式将 String 分割成若干部分。

这里我们将使用一个叫做 positive lookbehind 的正则表达式特征来匹配从 String 开始的指定数量的字符。

```java
static String usingSplitMethod(String text, int length) {

    String[] results = text.split("(?<=\\G.{" + length + "})");

    return results[0];
}
```

 results 的第一个元素将是我们截断的 String ，如果 length 长于 text ，则是原始的 String 。

让我们测试一下我们的方法。

```java
@Test
public void givenStringAndLength_whenUsingSplitMethod_thenTrim() {

    assertEquals(TrimStringOnLength.usingSplitMethod(TEXT, 13), "Welcome to ba");
}
```

#### 使用 Pattern 类

同样，  我们可以使用 Pattern 类来编译一个正则表达式，该表达式可以匹配 String 的开头，直至指定的字符数  。

例如，让我们使用 {1," + length + "}. 这个正则表达式至少匹配一个，最多匹配 length 个字符。

```java
static String usingPattern(String text, int length) {

    Optional<String> result = Pattern.compile(".{1," + length + "}")
      .matcher(text)
      .results()
      .map(MatchResult::group)
      .findFirst();

    return result.isPresent() ? result.get() : EMPTY;

}
```

正如我们在上面看到的，在将我们的正则表达式编译成 Pattern 后，我们可以使用 Pattern的 matcher() 方法来根据该正则表达式解释我们的 String 。然后我们就可以将结果分组，并返回第一个结果，也就是我们截断的 String 。

现在让我们添加一个测试案例来验证我们的代码是否如预期那样工作。

```java
@Test
public void givenStringAndLength_whenUsingPattern_thenTrim() {

    assertEquals(TrimStringOnLength.usingPattern(TEXT, 19), "Welcome to  javanorth");
}
```

#### 使用 CharSequence 的 codePoints() 方法

Java 9提供了一个 codePoints() 方法来将一个 String 转换为一个码点值流。

让我们看看如何使用这个方法与 Stream API相结合来截断一个字符串。

```java
static String usingCodePointsMethod(String text, int length) {

    return text.codePoints()
      .limit(length)
      .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
      .toString();
}
```

在这里，我们使用 limit() 方法来限制 Stream 的长度 。然后我们使用 StringBuilder 来建立我们的截断字符串。

接下来，让我们验证一下我们的方法是否有效。

```java
@Test
public void givenStringAndLength_whenUsingCodePointsMethod_thenTrim() {
    assertEquals(TrimStringOnLength.usingCodePointsMethod(TEXT, 6), "Welcom");
}
```

### Apache Commons 库

Apache Commons Lang 库包括一个 StringUtils 类，用于操作 String。

首先，让我们把Apache Commons dependency添加到我们的 pom.xml 。

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

#### 使用 StringUtils的left() 方法

 StringUtils 有一个有用的 静态 方法叫 left() 。   StringUtils.left() 以一种安全的方式返回 String 最左边的指定字符数：  。

```java
static String usingLeftMethod(String text, int length) {

    return StringUtils.left(text, length);
}
```

#### 使用 StringUtils 的 truncate() 方法

另外，我们可以使用 StringUtils.truncate() 来达到同样的目的。

```java
public static String usingTruncateMethod(String text, int length) {
    return StringUtils.truncate(text, length);
}
```

### Guava库

除了使用核心Java方法和Apache Commons库来截断一个 String 之外，我们还可以使用 Guava。让我们首先把Guava的 dependency 添加到我们的 pom.xml 文件中。

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在我们可以使用Guava的 Splitter 类来截断我们的 String 。

```java
static String usingSplitter(String text, int length) {
    Iterable<String> parts = Splitter.fixedLength(length)
      .split(text);
    return parts.iterator()
      .next();
}
```

我们使用 Splitter.fixedLength() 将我们的 String 分割成多个给定长度的片段。 然后，我们返回结果中的第一个元素。

### 总结

在这篇文章中，我们学习了在Java中把一个 String 截断为特定数量的字符的各种方法。我们看了一些使用JDK来做这件事的方法。然后，我们使用一些第三方库来截断 String 。

