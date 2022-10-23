---
layout: post
title:  Java 中的同构数
tagline: by feng
categories: java
tags: 
    - feng
---
大家好，我是指北君。

今天看到一个比较有意思的概念，叫做同构数。同构数是一个数字，它的平方数与数字本身的尾数相同。

例如，25是一个同构数，因为25的平方是625，它的结尾是25。同样地，76是一个同构数，因为76的平方是5776，同样以76结尾。

### 判断一个数是否是同构数

有许多算法可以用来确定一个数字是否是同构，接下来我们选几种来看看。

#### 在数字上循环并进行比较

验证一个数字是否同构大概有以下几个步骤：

1. 计算数字平方数
2. 获取平方数的最后一位数字并与数字的最后一位数字进行比较 *如果最后一位数字不相等，则该数字不是一个同构数* 如果最后一位数字相等，则进入下一步
3. 删除数字和平方的最后一位数字 
4. 重复步骤2/3，直到数字的所有数字都得到比较

上述方法以相反的方式对输入数字的数字进行循环。

我们现在写一个Java程序来实现， `isAutomorphicUsingLoop()`方法将一个整数作为输入，并检查它是否是同构数。

```java
public boolean isAutomorphicUsingLoop(int number) {
    int square = number * number;

    while (number > 0) {
        if (number % 10 != square % 10) {
            return false;
        }
        number /= 10;
        square /= 10;
    }
    
    return true;
}
```

在任何一个步骤，如果最后一位数字不相等，我们就返回*false* 。否则，我们就去掉最后一个数字，对*number*的剩余数字重复这个过程。

我们可以通过以下的代码来验证这个方法是否可行。

测试用例：

```java
assertTrue(AutomorphicNumber.isAutomorphicUsingLoop(76));
assertFalse(AutomorphicNumber.isAutomorphicUsingLoop(25));
```

#### 直接比较数字

当然我们还可以用更直接的方法来确定一个数字是否是同构数。

1. 得到数字并计算数字的位数n
2. 计算数字的平方数
3. 从平方数中得到最后的n个数字，如果平方数中的最后n个数字与原始数字相同，则该数字是同构的，否则就不是同构数

在这种情况下，我们不需要对数字的位数进行循环。我们可以直接使用Math库来完成数字的长度以及平方数的最后几位的计算。

代码示例：

```java
public boolean isAutomorphicUsingMath(int number) {
    int square = number * number;

    int numberOfDigits = (int) Math.floor(Math.log10(number) + 1);
    int lastDigits = (int) (square % (Math.pow(10, numberOfDigits)));

    return number == lastDigits;
}
```

与第一种方法类似，我们先计算*number*的平方。现在我们不是逐一比较*number*和*square*的最后一位数字，而是通过使用 *Math.floor()* 一次性得到 *number* 的总的*numberOfDigits*。然后通过使用[*Math.pow()*](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Math.html#pow(double,double))从*square*提取尽可能多的数字。最后，我们将输入的*number*与提取的数字*lastDigits*进行比较。

如果*number*和*lastDigits*相等，这个数字就是同构的，我们返回*true*，否则，我们返回*false*。

测试用例：

```java
assertTrue(AutomorphicNumber.isAutomorphicUsingMath(76));
assertFalse(AutomorphicNumber.isAutomorphicUsingMath(25));
```

### 总结

在这篇文章中，我们了解了什么是同构数，还学习了几种确定一个数是否为同构数的方法，以及相应的Java程序。
