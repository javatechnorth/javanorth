---
layout: post
title:  将Java InputStream转为字符串1
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在本教程中，我们将讲讲如何将一个 InputStream 转换为一个字符串。

我们将从使用普通的 Java 开始，包括 `Java 8+` 的解决方案，然后也会研究使用 `Guava` 和 `Apache Commons IO` 库。

<!--more-->

### 用 Java 进行转换 - StringBuilder

让我们看看一个简单的、低级别的方法，使用普通的 Java，一个 InputStream 和一个简单的 StringBuilder。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    StringBuilder textBuilder = new StringBuilder();
    try (Reader reader = new BufferedReader(new InputStreamReader
      (inputStream, Charset.forName(StandardCharsets.UTF_8.name())))) {
        int c = 0;
        while ((c = reader.read()) != -1) {
            textBuilder.append((char) c);
        }
    }
    assertEquals(textBuilder.toString(), originalString);
}
```

### 用 Java 8 进行转换 -- BufferedReader

Java 8 给 BufferedReader 带来了一个新的 lines() 方法。让我们看看如何利用它将一个 InputStream 转换为一个字符串。

```java
@Test
public void convertingAnInputStreamToAString() {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = new BufferedReader(
      new InputStreamReader(inputStream, StandardCharsets.UTF_8))
        .lines()
        .collect(Collectors.joining("\n"));

    assertThat(text, equalTo(originalString));
}
```

值得一提的是，`lines()` 使用的是 `readLine()` 方法。`readLine()` 假定一行是由换行`（"\n"）`、回车`（"\r"）`或回车后立即换行中的任何一种结束符。换句话说，它支持所有常见的行结束方式。

另一方面，当我们使用 Collectors.join() 时，我们需要明确决定我们要为创建的 String 使用哪种类型的结束符。

我们也可以使用 Collectors.join(System.lineSeparator()) ，在这种情况下，输出结果取决于系统设置。

### 用 Java 9+ 进行转换 - InputStream.readAllBytes()

如果我们在 Java 9 或以上版本，我们可以利用一个新的 readAllBytes 方法添加到 InputStream 中。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
    
    assertThat(text, equalTo(originalString));
}
```

我们需要注意的是，这段简单的代码是为那些方便将所有字节读入字节数组的简单情况准备的。我们不应该用它来读取有大量数据的输入流。

### 用 Java Scanner 进行转换

接下来，让我们看看一个使用标准文本扫描器的普通Java例子。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = null;
    try (Scanner scanner = new Scanner(inputStream, StandardCharsets.UTF_8.name())) {
        text = scanner.useDelimiter("\\A").next();
    }

    assertThat(text, equalTo(originalString));
}
```

请注意，InputStream 将被关闭的 Scanner 关闭。

同样值得澄清的是 `useDelimiter("\A")` 的作用。这里我们传递了`'\A'`，它是一个边界标记重码，表示输入的开始。本质上，这意味着 `next()` 调用读取了整个输入流。

### 使用 ByteArrayOutputStream 进行转换

最后，让我们看看另一个普通的Java例子，这次是使用 ByteArrayOutputStream 类。

```java
@Test
public void convertingAnInputStreamToString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    ByteArrayOutputStream buffer = new ByteArrayOutputStream();
    int nRead;
    byte[] data = new byte[1024];
    while ((nRead = inputStream.read(data, 0, data.length)) != -1) {
        buffer.write(data, 0, nRead);
    }

    buffer.flush();
    byte[] byteArray = buffer.toByteArray();
        
    String text = new String(byteArray, StandardCharsets.UTF_8);
    assertThat(text, equalTo(originalString));
}
```

在这个例子中，InputStream 通过读写字节块被转换为 ByteArrayOutputStream。然后 OutputStream 被转换为一个字节数组，用来创建一个字符串。

### 用java.nio进行转换

另一个解决方案是将 InputStream 的内容复制到一个文件中，然后将其转换为一个字符串。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    Path tempFile = 
      Files.createTempDirectory("").resolve(UUID.randomUUID().toString() + ".tmp");
    Files.copy(inputStream, tempFile, StandardCopyOption.REPLACE_EXISTING);
    String result = new String(Files.readAllBytes(tempFile));

    assertThat(result, equalTo(originalString));
}
```

这里我们使用 `java.nio.file.Files` 类来创建一个临时文件，同时将 InputStream 的内容复制到文件中。然后用同一个类用 readAllBytes() 方法将文件内容转换为一个字符串。

### 用Guava进行转换

让我们从一个利用 ByteSource 功能的 Guava 例子开始。

```java
@Test
public void convertingAnInputStreamToAString() 
  throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    ByteSource byteSource = new ByteSource() {
        @Override
        public InputStream openStream() throws IOException {
            return inputStream;
        }
    };

    String text = byteSource.asCharSource(Charsets.UTF_8).read();

    assertThat(text, equalTo(originalString));
}
```

让我们来看看这些步骤

- 首先，我们把我们的 InputStream 包装成一个 ByteSource.
- 其次，我们把 ByteSource 看作是一个具有 UTF8 字符集的 CharSource。
- 最后，我们使用 CharSource 将其作为一个字符串来读取。

一个更简单的转换方法是使用 Guava，但需要明确地关闭流, 我们可以简单地使用 `try-with-resources` 语法来处理这个问题。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    String text = null;
    try (Reader reader = new InputStreamReader(inputStream)) {
        text = CharStreams.toString(reader);
    }
 
    assertThat(text, equalTo(originalString));
}
```

### 用 Apache Commons IO 进行转换

现在让我们来看看如何用 Commons IO 库来做这个。

一个重要的注意事项是，与 Guava 不同的是，这些例子都不会关闭 InputStream。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = IOUtils.toString(inputStream, StandardCharsets.UTF_8.name());
    assertThat(text, equalTo(originalString));
}
```

我们也可以用一个StringWriter来做转换。

```java
@Test
public void convertingAnInputStreamToAString() throws IOException {
    String originalString = randomString(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    StringWriter writer = new StringWriter();
    String encoding = StandardCharsets.UTF_8.name();
    IOUtils.copy(inputStream, writer, encoding);

    assertThat(writer.toString(), equalTo(originalString));
}
```

### 总结

在这篇文章中，我们学习了如何将一个 InputStream 转换为一个字符串。我们从使用普通 Java 开始，然后探索了如何使用 Guava 和 Apache Commons IO 库。
