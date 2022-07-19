---
layout: post
title:  Java record vs lombok -20220720
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

Java的 record 关键字是Java 14中引入的一个新的语义特性。record 对于创建小型不可变的对象非常有用。另一方面，Lombok 是一个Java库，可以自动生成一些已知的模式为Java字节码。尽管它们都可以用来减少模板代码，但它们是不同的工具。因此，我们应该在特定情况下使用更适合我们需求的工具。
<!--more-->
在这篇文章中，我们将探讨各种使用情况，包括java record 的一些限制。对于每个例子，我们将看到Lombok如何派上用场，并比较这两种解决方案。

### 小型不可变对象

我们的第一个例子，将使用`Color`对象。一个`Color`由三个整数值组成，分别代表红、绿、蓝三个通道。此外，一个颜色会暴露出它的十六进制表示。例如，`RGB(255,0,0)`的颜色将有一个`#FF0000` 的十六进制表示。此外，如果两种颜色具有相同的RGB值，我们希望它们是`相等的`。

由于这些原因，在这种情况下选择 record 是非常合理的。

```java
public record ColorRecord(int red, int green, int blue) {
 
    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }
}
```

同样地，Lombok允许我们使用`@Value`注解来创建不可变的对象。

```java
@Value
public class ColorValueObject {
    int red;
    int green;
    int blue;

    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }
}
```

然而，从Java 14开始，`record ` 将成为这些使用情况的最常见的方式。

### 透明的数据载体

根据JDK增强建议（[JEP 395](https://openjdk.org/jeps/359)），record 是作为不可变数据的透明载体的类。例如，我们不能强迫前面例子中的`ColorRecord`只暴露`hexString`而完全隐藏三个整数字段。

然而，Lombok允许我们自定义名称、访问级别和获取器的返回类型。让我们相应地更新`ColorValueObject`。

```java
@Value
@Getter(AccessLevel.NONE)
public class ColorValueObject {
    int red;
    int green;
    int blue;

    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }
}
```

因此，如果我们需要不可变的数据对象，record 是一个很好的解决方案。

然而，如果我们想隐藏成员字段，只暴露使用它们进行的一些操作，Lombok会更适合。

### 有许多字段的类

我们已经看到了record 是如何以一种非常方便的方式来创建小型、不可变的对象的。让我们看看如果数据模型需要更多的字段，record 会是什么样子。在这个例子中，让我们考虑`Student`的数据模型。

```java
public record StudentRecord(
  String name, 
  Long studentId, 
  String email, 
  String phoneNumber, 
  String address, 
  String country, 
  int age) {
}
```

我们已经可以猜到，StudentRecord的实例化将很难阅读和理解，尤其是如果有些字段不是强制性的。

```java
StudentRecord john = new StudentRecord(
  "John", null, "xxxx@qq.com", null, null, "sh", 20);
```

为了方便这些使用，Lombok提供了一个[Builder设计模式]（/creational-design-patterns#builder）的实现。

为了使用它，我们只需要用`@Builder:`来注释我们的类。

```java
@Getter
@Builder
public class StudentBuilder {
    private String name;
    private Long studentId;
    private String email;
    private String phoneNumber;
    private String address;
    private String country;
    private int age;
}
```
现在，让我们使用`StudentBuilder`来创建一个具有相同属性的对象。

```java
StudentBuilder john = StudentBuilder.builder()
  .name("John")
  .email("xxx@qq.com")
  .country("sh")
  .age(20)
  .build();
```

如果我们对两者进行比较，我们可以注意到，使用构建器模式是有利的，可以带来更干净的代码。

总而言之，record 对于较小的对象来说是更好的。虽然，对于有很多字段的对象来说，缺乏创建模式会使Lombok的`@Builder`成为更好的选择。

### 可变数据

我们可以使用java record 专门处理不可变的数据。如果上下文需要一个可变的java对象，我们可以使用Lombok的`@Data`对象代替：

```java
    @Data 
    @AllArgsConstructor 
    public class ColorData {

        private int red; 
        private int green; 
        private int blue;

        public String getHexString() { 
            return String.format("#%02X%02X%02X", red, green, blue); 
        }

    }
```

一些框架可能需要带有设置器或默认构造函数的对象。例如，Hibernate就属于这种类型。当创建一个`@Entity`时，我们必须使用Lombok的注解或纯Java。

### 继承性

Java record 不支持继承。因此，它们不能被扩展或继承其他类。另一方面，Lombok的`@Value`对象可以扩展其他类，但它们是最终的。

```java
@Value
public class MonochromeColor extends ColorData {
    
    public MonochromeColor(int grayScale) {
        super(grayScale, grayScale, grayScale);
    }
}
```

此外，`@Data`对象既可以扩展其他类，也可以被扩展。总之，如果我们需要继承，我们应该坚持使用Lombok的解决方案。

### 结论

在这篇文章中，我们已经看到Lombok和java records是不同的工具，有不同的用途。此外，我们发现Lombok更加灵活，它可以用于record 受到限制的场景。

