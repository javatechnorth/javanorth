---
layout: post
title:  Java 如何验证文件名的有效性？
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在本文中，我们将讨论使用 Java 验证一个给定的字符串是否具有操作系统的有效文件名的不同方法。我们可以根据限制的字符或长度限制来检查该值。

<!--more-->

我们将只关注核心解决方案，不使用任何外部依赖。我们将使用JDK的`java.io`和NIO2包来实现我们验证方法。。

### 使用`java.io.File`

让我们从第一个例子开始，使用 `java.io.File` 类。在这个解决方案中，我们需要用一个给定的字符串创建一个`File`实例，然后在本地磁盘上创建一个文件。

```java
public static boolean validateStringFilenameUsingIO(String filename) throws IOException {
    File file = new File(filename);
    boolean created = false;
    try {
        created = file.createNewFile();
        return created;
    } finally {
        if (created) {
            file.delete();
        }
    }
}
```

当给定的文件名不正确时，它会抛出一个`IOException`。让我们注意，由于里面的文件创建，这个方法需要给定的文件名字符串没有对应存在的文件。

我们知道，不同的文件系统有自己的文件名限制。通过使用 `java.io.File` 方法，我们不需要指定每个操作系统的规则，因为Java自动为我们解决了这个问题。

然而，我们需要创建一个假文件。当我们成功后，我们必须记得在最后删除它。此外，我们必须确保我们有适当的权限来执行这些操作。任何失败也可能导致`IOException`，所以也最好检查一下错误信息。

```java
assertThatThrownBy(() -> validateStringFilenameUsingIO("javanorth?.txt"))
  .isInstanceOf(IOException.class)
  .hasMessageContaining("Invalid file path");
```

### 使用 NIO2 API

我们知道`java.io`包有很多缺点，因为它是在Java的第一个版本中创建的。NIO2 API是`java.io`包的后继者，它带来了许多改进，这也大大简化了我们之前的解决方案。

```java
public static boolean validateStringFilenameUsingNIO2(String filename) {
    Paths.get(filename);
    return true;
}
```

我们的功能现在被精简了，所以它是进行这种测试的最快方式。我们不创建任何文件，所以我们不需要有任何磁盘权限，也不需要在测试后执行清理。

无效的文件名抛出 `nvalidPathException`，它扩展了`RuntimeException`。这个的错误信息也包含了比之前更多的细节。

```java
assertThatThrownBy(() -> validateStringFilenameUsingNIO2(filename))
  .isInstanceOf(InvalidPathException.class)
  .hasMessageContaining("character not allowed");
```

但是这个解决方案有一个严重的缺点，与文件系统的限制有关。`Path`类可能表示带有子目录的文件路径。与第一个例子不同，这个方法没有检查文件名字符的溢出限制。让我们用Apache Commons的[`randomAlphabetic()`](https://commons.apache.org/proper/commons-lang/javadocs/api-3.9/org/apache/commons/lang3/RandomStringUtils.html#randomAlphabetic-int)方法生成的五百个字符的随机`String`来检查。

```java
String filename = RandomStringUtils.randomAlphabetic(500);
assertThatThrownBy(() -> validateStringFilenameUsingIO(filename))
  .isInstanceOf(IOException.class)
  .hasMessageContaining("File name too long");

assertThat(validateStringFilenameUsingNIO2(filename)).isTrue();
```

为了解决这个问题，我们应该像以前一样，创建一个文件并检查结果。

### 自定义的实现

最后，让我们尝试实现我们自己的自定义函数来测试文件名。我们还将尝试避免任何I/O功能，只使用核心的Java方法。

这类解决方案提供了更多的控制权，允许我们实现我们自己的规则。然而，我们必须考虑不同系统的许多额外限制。

#### 使用`String.contains`

我们可以使用[`String.contains()`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#contains(java.lang.CharSequence))方法来检查给定的`String`是否包含任何禁止的字符。首先，我们需要手动指定一些示例值。

```java
public static final Character[] INVALID_WINDOWS_SPECIFIC_CHARS = {'"', '`', '<', '>', '?', '|'};
public static final Character[] INVALID_UNIX_SPECIFIC_CHARS = {'\000'};
```

在我们的例子中，让我们只关注这两个操作系统，Windows的文件名比UNIX的限制更多。另外，一些的空白字符可能会有问题。

在定义了受限制的字符集之后，让我们来确定当前的操作系统。

```java
public static Character[] getInvalidCharsByOS() {
    String os = System.getProperty("os.name").toLowerCase();
    if (os.contains("win")) {
        return INVALID_WINDOWS_SPECIFIC_CHARS;
    } else if (os.contains("nix") || os.contains("nux") || os.contains("mac")) {
        return INVALID_UNIX_SPECIFIC_CHARS;
    } else {
        return new Character[]{};
    }
}
```

而现在我们可以用它来测试给定的值。

```java
public static boolean validateStringFilenameUsingContains(String filename) {
    if (filename == null || filename.isEmpty() || filename.length() > 255) {
        return false;
    }
    return Arrays.stream(getInvalidCharsByOS())
      .noneMatch(ch -> filename.contains(ch.toString()));
}
```

如果我们定义的任何字符不在给定的文件名中，这个`Stream`谓词返回真。此外，我们还实现了对`null`值和不正确长度的支持。

#### 正则表达式模式匹配

我们也可以在给定的`String`上直接使用正则表达式。让我们来实现一个只接受字母数字和点字符的模式，其长度不超过255。

```java
public static final String REGEX_PATTERN = "^[A-za-z0-9.]{1,255}$";

public static boolean validateStringFilenameUsingRegex(String filename) {
    if (filename == null) {
        return false;
    }
    return filename.matches(REGEX_PATTERN);
}
```

现在，我们可以根据先前准备的模式测试给定的值。我们还可以轻松地修改模式。在这个例子中，我们跳过了操作系统的检查功能。

#### 总结

在这篇文章中，我们集中讨论了文件名及其限制。我们介绍了不同的算法，用Java检测无效的文件名。

我们从`java.io`包开始，它为我们解决了大部分的系统限制，但执行了额外的I/O动作，可能需要一些权限。然后我们检查了NIO2 API，它是最快的解决方案，但有文件名长度检查的限制。

最后，我们实现了我们自己的方法，不使用任何I/O API，但需要自定义实现文件系统规则。
