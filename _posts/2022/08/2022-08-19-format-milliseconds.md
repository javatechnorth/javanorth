---
layout: post
title:  Java 如何将 millisconds Duration 格式化为 HH:MM:SS
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

Duration 是以小时、分钟、秒、毫秒等表示的时间量。我们可能希望将一个Duration格式化为某种特定的时间模式。

我们可以通过在一些JDK库的帮助下编写自定义代码或利用第三方库来实现这一点。
<!--more-->
在本文中，我们将看看如何编写简单的代码，将一个给定的 Duration 格式化为 HH:MM:SS 格式。

### Java 解决方案

Duration 有多种表达方式--例如，以分、秒和毫秒为单位，或以Java `Duration`为单位，它有自己的特定格式。

本节和后续章节将重点讨论使用一些JDK库将以毫秒为单位的时间间隔（经过的时间）格式化为HH:MM:SS。在我们的例子中, 我们将把38114000ms格式化为10:35:14 (HH:MM:SS).

#### Duration

从Java 8开始，引入了 Duration 类来处理各种单位的时间间隔。 Duration 类带有很多辅助方法，可以从一个Duration中获得小时、分钟和秒。
要使用`Duration`类将时间间隔格式化为HH:MM:SS，我们需要使用`Duration`类中的工厂方法`ofMillis`将我们的时间间隔初始化为`Duration`对象。这就把时间间隔转换成了我们可以使用的`Duration`对象。

```java
Duration duration = Duration.ofMillis(38114000);
```

为了便于从秒到我们所需单位的计算，我们需要得到我们的Duration或间隔时间的总秒数。

```java
long seconds = duration.getSeconds();
```

然后，一旦我们得到秒数，我们就为我们所需的格式生成相应的小时、分钟和秒。

```java
long HH = seconds / 3600;
long MM = (seconds % 3600) / 60;
long SS = seconds % 60;
```

最后，我们对生成的数值进行格式化。

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
```

让我们来试试这个解决方案。

```java
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

如果我们使用的是Java 9或更高版本，我们可以使用一些辅助方法来直接获得单位，而不需要进行任何计算。

```java
long HH = duration.toHours();
long MM = duration.toMinutesPart();
long SS = duration.toSecondsPart();
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
```

上述片段将给我们带来与上述测试相同的结果。

```java
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

#### TimeUnit

就像上一节中讨论的`Duration`类一样，`TimeUnit`表示一个给定粒度的时间。它提供了一些辅助方法来转换不同的单位--在我们的例子中是小时、分钟和秒--并在这些单位中执行计时和延迟操作。

要把一个以毫秒为单位的Duration转换成HH:MM:SS的格式，我们所要做的就是使用`TimeUnit`中相应的辅助方法。

```java
long HH = TimeUnit.MILLISECONDS.toHours(38114000);
long MM = TimeUnit.MILLISECONDS.toMinutes(38114000) % 60;
long SS = TimeUnit.MILLISECONDS.toSeconds(38114000) % 60;
```

然后，根据上面生成的单位格式化Duration。

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

### 使用第三方库

#### Apache Commons

为了使用Apache Commons，我们需要在我们的项目中添加[commons-lang3](https://search.maven.org/search?q=g:org.apache.commons%20a:commons-lang3)。

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

正如预期的那样，这个库在其`DurationFormatUtils`类中有`formatDuration`以及其他单位格式化方法。

```java
String timeInHHMMSS = DurationFormatUtils.formatDuration(38114000, "HH:MM:SS", true);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

#### Joda Time 库

当我们使用Java 8之前的Java版本时，Joda Time库就派上用场了，因为它有方便的辅助方法来表示和格式化时间单位。为了使用Joda Time，让我们把[joda-time dependency](https://search.maven.org/search?q=g:joda-time%20a:joda-time)添加到我们的项目中。

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.10</version>
</dependency>
```

Joda Time有一个`Duration`类来表示时间。首先，我们把以毫秒为单位的时间间隔转换成Joda Time `Duration`对象的实例。

```java
Duration duration = new Duration(38114000);
```

然后，我们使用`Duration`中的`toPeriod`方法从上面的Duration中获得周期，该方法将其转换或初始化为Joda Time中`Period`类的一个实例。

```java
Period period = duration.toPeriod();
```

我们使用`Period`的相应帮助方法从它那里获得单位（小时、分钟和秒）。

```java
long HH = period.getHours();
long MM = period.getMinutes();
long SS = period.getSeconds();
```

最后，我们可以格式化Duration并测试结果。

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

### 总结

在本教程中，我们已经学会了如何将一个Duration格式化为特定的格式（在我们的例子中，是HH:MM:SS）。

首先，我们使用了Java中的`Duration`和`TimeUnit`类来获得所需的单位，并在`Formatter`的帮助下将其格式化。

最后，我们研究了如何使用一些第三方库来实现这个结果。
