---
layout: post
title: Java 运算符——20210927
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

今天指北君要带实习生小伙伴，过一遍Java运算符相关的内容。

![](https://gitee.com/274904168/image-repo/raw/master/202109261912723.png)

**指北君**: 在Java中，有很多种类型的运算符，主要可以分为：算术运算符、关系运算符、位运算符、逻辑运算符、赋值运算符、其他运算符。

**实习生**：啊，有这么多种吗？有些感觉我都没听过呀。
<!--more-->
**指北君**：哪几个你不知道？

**实习生**：位运算符。

**指北君**：对，这个在我们一般写程序的时候不怎么使用。我们先看看算术操作符。

算术操作符主要是加(+)、减(-)、乘(*)、除(/)和取模(%)。

```java
public class Test {
 
  public static void main(String[] args) {
     int a = 10;
     int b = 20;
     int c = 25;
     System.out.println("a + b = " + (a + b));
     System.out.println("a - b = " + (a - b));
     System.out.println("a * b = " + (a * b));
     System.out.println("b / a = " + (b / a));
     System.out.println("b % a = " + (b % a));
     System.out.println("c % a = " + (c % a));
  }
}
```

以上实例编译运行结果如下：

```java
a + b = 30
a - b = -10
a * b = 200
b / a = 2
b % a = 0
c % a = 5
```

这中间有一些需要注意的点：

1. 使用除的时候，除数不能等于0，以免程序出现异常。
2. 整数相除结果还是整数。比如10/3 =3， 但是正常来说应该是3.33333... ，这种场景下我们需要使用浮点数来做乘除法。

**实习生**：这些比较简单，我还是知道的。

**指北君**：那你知道算术运算符还有比较特别的两个吗？

**实习生**：额，还有啥，不就是这几个了么？

**指北君**：自增运算符(++)和自减运算符(--)呀，这两个也是算术运算符。

**实习生**：好吧，我居然没有想到。

**指北君**：直接看例子吧

```java
public class selfAddMinus{
    public static void main(String[] args){
        int a = 3;//定义一个变量；
        int b = ++a;//自增运算
        int c = 3;
        int d = --c;//自减运算
        System.out.println("进行自增运算后的值等于"+b);
        System.out.println("进行自减运算后的值等于"+d);
    }
}
```

运行结果：

```java
进行自增运算后的值等于4
进行自减运算后的值等于2
```

自增运算符也有一个需要注意的点， 前自增是先自增，再进行表达式运算。后自增是先进行表达式计算，再进行自增。自减同理。

**指北君**：我们再来看看关系运算符。

![](https://gitee.com/274904168/image-repo/raw/master/202109261810954.png)

关系运算符没有特别的难点，直接看示例吧。

```java
public class Test {
 
  public static void main(String[] args) {
     int a = 10;
     int b = 20;
     System.out.println("a == b = " + (a == b) );
     System.out.println("a != b = " + (a != b) );
     System.out.println("a > b = " + (a > b) );
     System.out.println("a < b = " + (a < b) );
     System.out.println("b >= a = " + (b >= a) );
     System.out.println("b <= a = " + (b <= a) );
  }
}
```

运行结果

```java
a == b = false
a != b = true
a > b = false
a < b = true
b >= a = true
b <= a = false
```

**指北君**：接下来看看位运算符。

![](https://gitee.com/274904168/image-repo/raw/master/202109261818580.png)

```java
public class Test {
  public static void main(String[] args) {
     int a = 60; /* 60 = 0011 1100 */ 
     int b = 13; /* 13 = 0000 1101 */
     int c = 0;
     c = a & b;       /* 12 = 0000 1100 */
     System.out.println("a & b = " + c );
 
     c = a | b;       /* 61 = 0011 1101 */
     System.out.println("a | b = " + c );
 
     c = a ^ b;       /* 49 = 0011 0001 */
     System.out.println("a ^ b = " + c );
 
     c = ~a;          /*-61 = 1100 0011 */
     System.out.println("~a = " + c );
 
     c = a << 2;     /* 240 = 1111 0000 */
     System.out.println("a << 2 = " + c );
 
     c = a >> 2;     /* 15 = 1111 */
     System.out.println("a >> 2  = " + c );
  
     c = a >>> 2;     /* 15 = 0000 1111 */
     System.out.println("a >>> 2 = " + c );
  }
} 
```

运行结果

```java
a & b = 12
a | b = 61
a ^ b = 49
~a = -61
a << 2 = 240
a >> 2  = 15
a >>> 2 = 15
```

**指北君**：逻辑运算符

![](https://gitee.com/274904168/image-repo/raw/master/202109261823531.png)

```java
public class Test {
  public static void main(String[] args) {
     boolean a = true;
     boolean b = false;
     System.out.println("a && b = " + (a&&b));
     System.out.println("a || b = " + (a||b) );
     System.out.println("!(a && b) = " + !(a && b));
  }
}
```

运行结果


```java
a && b = false
a || b = true
!(a && b) = true
```

**指北君**：接下来看看赋值运算符

![](https://gitee.com/274904168/image-repo/raw/master/202109261826547.png)

```java
public class Test {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        int c = 0;
        c = a + b;
        System.out.println("c = a + b = " + c );
        c += a ;
        System.out.println("c += a  = " + c );
        c -= a ;
        System.out.println("c -= a = " + c );
        c *= a ;
        System.out.println("c *= a = " + c );
        a = 10;
        c = 15;
        c /= a ;
        System.out.println("c /= a = " + c );
        a = 10;
        c = 15;
        c %= a ;
        System.out.println("c %= a  = " + c );
        c <<= 2 ;
        System.out.println("c <<= 2 = " + c );
        c >>= 2 ;
        System.out.println("c >>= 2 = " + c );
        c >>= 2 ;
        System.out.println("c >>= 2 = " + c );
        c &= a ;
        System.out.println("c &= a  = " + c );
        c ^= a ;
        System.out.println("c ^= a   = " + c );
        c |= a ;
        System.out.println("c |= a   = " + c );
    }
}
```

运行结果

```java
c = a + b = 30
c += a  = 40
c -= a = 30
c *= a = 300
c /= a = 1
c %= a  = 5
c <<= 2 = 20
c >>= 2 = 5
c >>= 2 = 1
c &= a  = 0
c ^= a   = 10
c |= a   = 10
```

**指北君**：我们再来看看最后一个三元运算符。

```java
public class Test {
   public static void main(String[] args){
      int a , b;
      a = 10;
      // 如果 a 等于 1 成立，则设置 b 为 20，否则为 30
      b = (a == 1) ? 20 : 30;
      System.out.println( "Value of b is : " +  b );
 
      // 如果 a 等于 10 成立，则设置 b 为 20，否则为 30
      b = (a == 10) ? 20 : 30;
      System.out.println( "Value of b is : " + b );
   }
}
```

运行结果

```java
Value of b is : 30
Value of b is : 20
```

**指北君**：好了，Java运算符就先说这么多吧，你现在应该已经全部搞清楚了吧？

**实习生**：差不多搞清楚了，谢谢指北君。

### 总结

今天指北君给实习生完整地介绍了一遍Java运算符。举例说明了各种运算符的使用方法和方式。
