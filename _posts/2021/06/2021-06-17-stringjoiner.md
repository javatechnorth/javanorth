---
layout: post
title:  拼接字符串，还能这么玩
tagline: by feng
categories: StringJoiner
tags: 
    - feng
---

大家好，我是指北君。

### 前言

最近呢，指北君的朋友小王在面试的时候，又遇到了问题，再次回家等通知，找个工作不容易啊，太卷了。

**面试官**：你知道使用原生 JDK 拼接字符串有多少种玩法吗？

**小王**：`StringBuilder`、`StringBuffer`、 `String.concat`、还有用 + 拼接。（这也太简单了吧，小王心理OS。）

**面试官**又问，还有其他的方法吗？

**小王**有点开始纳闷了，还有其他的吗？没见过呀。难道是字符数组来拼接出来。 

**小王**：字符数组拼接。

面试官还是继续追问，还有吗？小王心想还能有啥啊，该说的我都说了啊，更多的我也不知道啦，只能回答没有了。

面试官见状提醒了小王，你在用`JDK 1.8`及以上版本吗？ `Java 8` 里面新增了一个拼接字符串的类 ，叫 `StringJoiner` ，你可以去看一下。 小王很无奈，这个类还真没看到过。

<!--more-->

小王面完找指北君来诉苦，指北君马上安排一片文章，发给了小王。

接下来呢，指北君就要带大家来了解一下这个 `Java 8` 开始新增的 `StringJoiner` 的用法。

### 概述

> StringJoiner is used to construct a sequence of characters separated by a delimiter and optionally starting with a supplied prefix and ending with a supplied suffix.
> 
> Prior to adding something to the StringJoiner, its sj.toString() method will, by default, return prefix + suffix. However, if the setEmptyValue method is called, the emptyValue supplied will be returned instead. This can be used, for example, when creating a string using set notation to indicate an empty set, i.e. "{}", where the prefix is "{", the suffix is "}" and nothing has been added to the StringJoiner.

`StringJoiner` 源码的定义可以看出，它是 `java.util` 包中的一个类，被用来构造一个由分隔符分隔的字符串，并且可以从提供的前缀字符串开头，以提供的后缀字符串结尾。

通常我们拼接字符串都是使用 `StringBuilder` 或者 `StringBuffer` 来实现的。这个时候，我们可能就会有一个疑问了， `StringJoiner` 的价值是什么？ 到底为什么要到这个时候创造它。

### 源码解析

好，我们先看一下 `StringJoiner` 的构造函数， `StringJoiner` 一共有 2 个构造函数。 构造函数很简单，没有什么可以多讲的，就是对 分隔符、前缀和后缀字符串的初始化。

```java
    public StringJoiner(CharSequence delimiter) {
        this(delimiter, "", "");
    }
    public StringJoiner(CharSequence delimiter,
                        CharSequence prefix,
                        CharSequence suffix) {
        Objects.requireNonNull(prefix, "The prefix must not be null");
        Objects.requireNonNull(delimiter, "The delimiter must not be null");
        Objects.requireNonNull(suffix, "The suffix must not be null");
        // make defensive copies of arguments
        this.prefix = prefix.toString();
        this.delimiter = delimiter.toString();
        this.suffix = suffix.toString();
    }
```

另外 `StringJoiner` 有 5 个公有方法，其中比较常用的就是 `add` 和 `toString` 。我们也来看看这两个常用方法。

```java
public final class StringJoiner {
    private String[] elts;
    
    @Override
    public String toString() {
        final String[] elts = this.elts;
        if (elts == null && emptyValue != null) {
            return emptyValue;
        }
        final int size = this.size;
        final int addLen = prefix.length() + suffix.length();
        if (addLen == 0) {
            compactElts();
            return size == 0 ? "" : elts[0];
        }
        final String delimiter = this.delimiter;
        final char[] chars = new char[len + addLen];
        int k = getChars(prefix, chars, 0);
        if (size > 0) {
            k += getChars(elts[0], chars, k);
            for (int i = 1; i < size; i++) {
                k += getChars(delimiter, chars, k);
                k += getChars(elts[i], chars, k);
            }
        }
        k += getChars(suffix, chars, k);
        return new String(chars);
    }
    public StringJoiner add(CharSequence newElement) {
        final String elt = String.valueOf(newElement);
        if (elts == null) {
            elts = new String[8];
        } else {
            if (size == elts.length)
                elts = Arrays.copyOf(elts, 2 * size);
            len += delimiter.length();
        }
        len += elt.length();
        elts[size++] = elt;
        return this;
    }
}
```

我们来看下 `add` 方法的实现，看起来也挺简单的，就是把待拼接的字符串，放到一个字符串数组里面。`toString()` 方法的时候，才是真正做字符串拼接的过程。我例子中的代码是 `JDK 11`， 相比 `JDK 8` 中，`StringJoiner` 是通过 `StringBuilder` 来实现的。

既然 `JDK 8` 的时候，已经使用了`StringBuilder` 来实现，那么为什么还要改成 `String[]`  来缓存所有的待拼接的字符串。这个就要涉及到JVM底层的优化，我们这里暂时不展开讲这个问题了。

前面已经提过既然已经有了 `StringBuilder`，为什么还要造一个`StringJoiner`，它的优势到底在哪里，我们接着来找找原因。很快我们在代码类的注释中找到了猫腻，在注释中标记了
`Collectors#joining` 。

> A StringJoiner may be employed to create formatted output from a java.util.stream.Stream using java.util.stream.Collectors.joining(CharSequence).

```java
 * @see java.util.stream.Collectors#joining(CharSequence)
 * @see java.util.stream.Collectors#joining(CharSequence, CharSequence, CharSequence)
```

那我们就顺藤摸瓜，看看 `Collectors#joining` 有什么跟 `StringJoiner` 有关联的呢？

```java
   public static Collector<CharSequence, ?, String> joining() {
        return new CollectorImpl<CharSequence, StringBuilder, String>(
                StringBuilder::new, StringBuilder::append,
                (r1, r2) -> { r1.append(r2); return r1; },
                StringBuilder::toString, CH_NOID);
    }
    public static Collector<CharSequence, ?, String> joining(CharSequence delimiter) {
        return joining(delimiter, "", "");
    }
    public static Collector<CharSequence, ?, String> joining(CharSequence delimiter,
                                                             CharSequence prefix,
                                                             CharSequence suffix) {
        return new CollectorImpl<>(
                () -> new StringJoiner(delimiter, prefix, suffix),
                StringJoiner::add, StringJoiner::merge,
                StringJoiner::toString, CH_NOID);
    }
```

原来啊，`Java 8` 中 `Stream` 是借助了 `StringJoiner` 来实现的。 这个时候，我们可能会想，为什么不使用 `StringBuilder` 来实现呢？ 我们可以从上面的代码里看出，如果 使用 `StringBuilder` 来构造拼接的话，在没有前后缀的情况下，应该还是简单的，事实上JDK 官方组织也选择了 `StringBuilder` 。但是一旦涉及到拼接之类的操作，那如果还是使用 `StringBuilder` 的话，那就真的是太复杂了。

所以 `StringJoiner` 在 `Java 8` 的地位是 `StringBuilder` 所不能代替的。

### 总结

本文介绍了 `Java 8` 开始提供的字符串拼接类 `StringJoiner`。 `JDK 8` 中 `StringJoiner` 是通过 `StringBuilder` 实现的， 所以它的性能和 `StringBuilder` 差不多，它也是非线程安全的。`JDK 11` 中已经对其进行了优化，通过 `String[]` 来代理 `StringBuilder` 。

在日常的开发过程中，我们怎么选择字符串拼接类呢？

1. 简单的字符串拼接，直接使用 + 即可。
2. 在 for 循环之类的场景下需要字符串拼接，可以优先考虑使用 StringBuilder 。
3. 在使用 Java Stream 的场景下需要字符串拼接，可以优先考虑使用 StringJoiner。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。
