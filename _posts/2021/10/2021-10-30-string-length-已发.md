---
layout: post
title: 面试官：String的最大长度是多少？——20211105
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

之前有提到，指北君在做面试相关的事情。有面试官问了String的最大长度是多少？指北君听到这个问题之后有点懵，还好指北君抗住了。

**指北君**：面试官你好，可以开始面试了吗？

**面试官**：你好，那我们现在开始吧。之前我们聊了new object()到底占用多少个字节？看你对JVM有一定的了解。那我今天问问你，一个 String 字符串最大长度是多少？
<!--more-->
**指北君**：从我们平常使用的角度看，String 字符串似乎是没有长度限制。所以也不存在最大长度这个事情。

**面试官**：真的这样吗？是这样的话，我们今天可以结束了。不用再聊了。

![](https://files.mdnice.com/user/15444/035f060f-d3e6-41bc-b136-453f1d678158.png)

**指北君**：那等等，我想一下。

指北君大脑中疯狂搜索 String 源代码的相关内容。

![](https://files.mdnice.com/user/15444/6eac7937-0710-4a29-8468-24a162938f10.png)

指北君终于在String 源码中找到了蛛丝马迹，

```java
public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

**指北君**：我记得String 源码中有个构造函数，有个count的参数是int类型的。在Java 中，int的最大长度是`2^31-1`。也就是说 String 的长度最大是`2^31-1`。


**面试官**：嗯，没错，这个是理论上的长度吧，实际情况能达到吗？

指北君又陷入了沉思…… 开始计算起存这么大的String需要消耗多大的内存。

![](https://files.mdnice.com/user/15444/91e4794c-79e3-426a-a60b-a78d0b1f806e.png)

**指北君**：我刚才在脑子里算了下，存储长度`2^31-1`的字符串需要4GB的内存，也就是说，我们需要有大于4GB的JVM运行内存才行。

```java
2^31-1)*2*16/8/1024/1024/1024 = 4GB
```

**面试官**：既然你已经提到了JVM，那String一般都存储在JVM的哪块区域。

**指北君**：字符串在JVM中的存储分两种情况，我上面说的String ，它是存储在JVM的堆栈中。另外还有字符串常量存储在常量池里面。

**面试官**：那你觉得常量池中的字符串最大长度是`2^31-1`吗？

**指北君**：Java中字符串在常量池中通过CONSTANT_Utf8类型表示。

```java
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```
**指北君**：我们只要重点关注`bytes[length]`即可，length在这里就是代表字符串的长度，length的类型是u2，u2是无符号的16位整数，也就是说最大长度可以做到2^16-1 即 65535。

![](https://files.mdnice.com/user/15444/57f606ac-1be2-4c8e-a88b-80a43dd87c6a.png)

**面试官**：按照你说的，我在我的机器上试了一下65535长度的字符串，编译报错了。 这是怎么回事呢？

**指北君**：这是因为javac编译器做了限制，需要length < 65535。 所以字符串常量在常量池中的最大长度是65534。你减少1个字符试试看。

**面试官**：这次编译没问题了。我们今天就先到这里吧。

**指北君**：好的，我的offer，有戏吗？

**面试官**：继续努力鸭

![](https://files.mdnice.com/user/15444/548d2a89-b3ee-4c76-a767-42d479b6f797.png)


### 总结
今天我们模拟面试了String的最大长度问题。结论是String是有长度限制的，但是不同的状态下，具有不同的长度限制。

- 字符串常量长度不能超过65534
- 堆内字符串的长度不能超过2^31-1


