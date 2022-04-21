---
layout: post
title:  聊聊如何格式化 Instant
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

今天我们将聊聊如何在Java中把一个 Instant 格式化为一个字符串。我们将展示如何使用 Java 原生和第三方库（如Joda-Time）来处理这个事情。

### 使用 Java 原生格式化Instant

在 Java 8 中有个名为 Instant 类。通常情况下，我们可以使用这个类来记录我们应用程序中的事件时间戳。

让我们看看如何把它转换成一个字符串对象。

#### 使用 DateTimeFormatter 类

一般来说，我们将需要一个格式化器来格式化一个即时对象。 Java 8引入了DateTimeFormatter类来统一格式化日期和时间。

DateTimeFormatter 提供了 format() 方法来完成这项工作。

简单地说，DateTimeFormatter 需要一个时区来格式化一个 Instant 。没有它，它将无法将Instant 转换为人类可读的日期/时间域。

例如，让我们假设我们想用 dd.MM.yyyy 格式来显示我们的即时信息实例。

```java
public class FormatInstantUnitTest {
    
    private static final String PATTERN_FORMAT = "dd.MM.yyyy";

    @Test
    public void givenInstant_whenUsingDateTimeFormatter_thenFormat() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(PATTERN_FORMAT)
        .withZone(ZoneId.systemDefault());

        Instant instant = Instant.parse("2022-04-21T15:35:24.00Z");
        String formattedInstant = formatter.format(instant);

        assertThat(formattedInstant).isEqualTo("21.04.2022");
    }
}
```

如上所示，我们可以使用withZone()方法来指定时区。

请记住，如果不能指定时区将导致 UnsupportedTemporalTypeException。

```java
@Test(expected = UnsupportedTemporalTypeException.class)
public void givenInstant_whenNotSpecifyingTimeZone_thenThrowException() {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern(PATTERN_FORMAT);

    Instant instant = Instant.now();
    formatter.format(instant);
}
```

#### 使用toString()方法

另一个解决方案是使用toString()方法来获得即时对象的字符串表示。

让我们用一个测试案例举例说明toString()方法的使用。

```java
@Test
public void givenInstant_whenUsingToString_thenFormat() {
    Instant instant = Instant.ofEpochMilli(1641828224000L);
    String formattedInstant = instant.toString();

    assertThat(formattedInstant).isEqualTo("2022-01-10T15:23:44Z");
}
```

这种方法的局限性在于，我们不能使用自定义的、对人友好的格式来显示即时信息。

### Joda-Time库

另外，我们也可以使用 Joda-Time API 来实现同样的目标。这个库提供了一套随时可用的类和接口，用于在Java中操作日期和时间。

在这些类中，我们发现DateTimeFormat类。顾名思义，这个类可以用来格式化或解析进出字符串的日期/时间数据。

因此，让我们来说明如何使用DateTimeFormatter来将一个瞬间转换为一个字符串。

```java
@Test
public void givenInstant_whenUsingJodaTime_thenFormat() {
    org.joda.time.Instant instant = new org.joda.time.Instant("2022-03-20T10:11:12");
        
    String formattedInstant = DateTimeFormat.forPattern(PATTERN_FORMAT)
        .print(instant);

    assertThat(formattedInstant).isEqualTo("20.03.2022");
}
```

我们可以看到，DateTimeFormatter提供forPattern()来指定格式化模式，print()来格式化即时对象。

### 总结

在这篇文章中，我们了解了如何在Java中把一个 Instant 格式化为一个字符串。

在这一过程中，我们了解了一些使用Java 原生方法来实现这一目标的方法。然后，我们解释了如何使用Joda-Time库来完成同样的事情。
