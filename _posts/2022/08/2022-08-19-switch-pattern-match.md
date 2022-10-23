---
layout: post
title:  Java 17 Switch 模式匹配
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

Java 17 引入了针对 `switch` 表达式和语句的模式匹配（[JEP 406](https://openjdk.java.net/jeps/406)）。模式匹配在 `switch` 条件定义时提供了更多的灵活性。在模式匹配之前，`switch` 只支持精确匹配一个常量值的选择器表达式。在本文中，我们来看看三种可以在`switch`语句中应用的不同模式类型。

<!--more-->

### Switch 表达式

我们在 Java 中使用`switch` 来做一些策略的选择，哪个语句被选中取决于 `switch` 表达式的值。
在 Java 17 之前版本中，`Switch` 的表达式必须是一个数字、一个字符串或一个常数。而且 `case` 只能包含常数。

```java
final String b = "B";
switch (args[0]) {
    case "A" -> System.out.println("Parameter is A");
    case b -> System.out.println("Parameter is b");
    default -> System.out.println("Parameter is unknown");
};
```

在上面例子中，如果变量b不是 final 类型的话，编译器会抛出一个常量表达式要求的错误。

### 模式匹配

模式匹配最初是在 Java 14 中首次作为预览功能引入的。但是只限于模式的一种形式 -- 类型模式。一个典型的模式由一个类型名和要绑定结果的变量组成。将类型模式应用于`instanceof` 操作简化了类型检查和构造。

```java
if (o instanceof String s) {
    System.out.printf("Object is a string %s", s);
} else if (o instanceof Number n) {
    System.out.printf("Object is a number %n", n);
}
```

这种内置的语言增强有助于我们写出更少的代码，并增强可读性。

### Switch 的模式匹配

用于`instanceof`的模式匹配在Java 16中成为一个永久性的功能。 到了Java 17， 模式匹配的应用才扩展到了`switch`表达式。 它仍然是一个预览特性，所以我们如果要使用的话，还得弃用预览功能。

```bash
java --enable-preview --source 17 PatternMatching.java
```

#### 在Switch中使用类型模式

让我们来看看类型模式和`instanceof`操作符如何在`switch`语句中应用。 我们先创建一个方法，使用`if-else`语句将不同的类型转换为`double`。如果该类型不被支持，我们的方法将简单地返回0。

```java
static double getDoubleUsingIf(Object o) {
    double result;
    if (o instanceof Integer) {
        result = ((Integer) o).doubleValue();
    } else if (o instanceof Float) {
        result = ((Float) o).doubleValue();
    } else if (o instanceof String) {
        result = Double.parseDouble(((String) o));
    } else {
        result = 0d;
    }
    return result;
}
```

我们可以通过`switch`中的类型模式来改造上面的方法，用更少的代码解决同样的问题。

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case Integer i -> i.doubleValue();
        case Float f -> f.doubleValue();
        case String s -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

在Java1 17版本之前， `Switch` 表达式只局限于几种类型。现在有了类型模式，`Switch` 表达式可以是任何类型啦。

#### 保护模式

类型模式帮助我们根据特定的类型来转移控制，然而，有时候我们也需要对传递的值进行额外的检查。我们可以在 case 中使用一个`if`语句来检查一个`String`的长度等。

```java
static double getDoubleValueUsingIf(Object o) {
    return switch (o) {
        case String s -> {
            if (s.length() > 0) {
                yield Double.parseDouble(s);
            } else {
                yield 0d;
            }
        }
        default -> 0d;
    };
}
```

我们可以用保护模式来解决同样的问题，可以使用模式和布尔表达式的组合。

```java
static double getDoubleValueUsingGuardedPatterns(Object o) {
    return switch (o) {
        case String s && s.length() > 0 -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

保护模式使我们能够避免在`switch`语句中附加`if`条件，我们可以把条件逻辑移到case标签中。

#### 括号模式

除了在case标签中设置条件逻辑外，括号模式能使我们对其分组。在执行额外的检查时，我们可以简单地在我们的布尔表达式中使用括号。

```java
static double getDoubleValueUsingParenthesizedPatterns(Object o) {
    return switch (o) {
        case String s && s.length() > 0 && !(s.contains("#") || s.contains("@")) -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

通过使用括号，我们可以避免出现额外的`if-else`语句。

### switch 的一些特殊点

现在我们来看看在`switch`中使用模式匹配时需要考虑的几个特殊情况。

### 5.1.覆盖所有的值

当在`switch`中使用模式匹配时，Java 编译器会检查类型覆盖问题。 举个例子，`switch`条件接受任何对象，但只覆盖`String`情况。

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case String s -> Double.parseDouble(s);
    };
}
```

我们的例子将导致以下编译错误

```bash
[ERROR] Failed to execute goal ... on project core-java-17: Compilation failure
[ERROR] /D:/Projects/.../HandlingNullValuesUnitTest.java:[10,16] the switch expression does not cover all possible input values
```

这是因为`switch` case标签需要包括选择器表达式的类型。

#### 对子类排序

在`switch`中使用模式匹配的子类时，case 的顺序就很重要了。举个例子，`String`案例在`CharSequence`案例之后。

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case CharSequence c -> Double.parseDouble(c.toString());
        case String s -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

由于`String`是`CharSequence`的一个子类，我们的例子将导致以下编译错误。

```bash
[ERROR] Failed to execute goal ... on project core-java-17: Compilation failure
[ERROR] /D:/Projects/.../HandlingNullValuesUnitTest.java:[12,18] this case label is dominated by a preceding case label
```

这个错误背后的原因是，没有机会执行到第二种情况，因为任何传递给方法的字符串对象都会在第一种情况下处理。

#### null 的处理

在Java17之前版本中，向`switch`语句传递一个`null`值都会导致一个`NullPointerException`。现在通过类型模式，现在可以将空值检查作为一个单独的case标签来处理。

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case String s -> Double.parseDouble(s);
        case null -> 0d;
        default -> 0d;
    };
}
```

如果没有针对空值的案例标签，那么object的模式标签也是可以匹配空值的。

```java
static double getDoubleUsingSwitchTotalType(Object o) {
    return switch (o) {
        case String s -> Double.parseDouble(s);
        case Object ob -> 0d;
    };
}
```

我们需要注意的是一个`switch`表达式不能同时具有`null`情况和 `object`情况。 如果同时存在就会报编译错误。

```bash
[ERROR] Failed to execute goal ... on project core-java-17: Compilation failure
[ERROR] /D:/Projects/.../HandlingNullValuesUnitTest.java:[14,13] switch has both a total pattern and a default label
```

最后，一个使用模式匹配的`switch`语句仍然可以抛出一个`NullPointerException`。

### 总结

在这篇文章中，我们探讨了`switch`表达式和语句的模式匹配，这是`Java 17`中的一个预览功能，通过在 `case` 标签中使用模式，来做出更多有效率的事情。