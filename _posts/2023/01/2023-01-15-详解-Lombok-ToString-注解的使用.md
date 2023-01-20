---
layout: post
title:  详解 Lombok @ToString 注解的使用
tagline: by feng
categories: java
tags: 
    - feng
---

在平时我们工作的时候，我们经常会使用`toString()` 方法来输出一个对象的一些属性信息。Lombok 给我们提供了一个自动生成 `toString()`代码的注解，可以减少代码行数，如果代码属性比较多的话，可以避免我们些代码的过程中出现属性遗漏的问题。<br />本文我们来讲讲 Lombok 的 `@ToString()`相关内容，以便于我们以后更好的使用 Lombok。

<!--more-->

<a name="Ci8Gj"></a>
### Lombok 的使用
首先我们添加一下 maven 依赖。
```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.22</version>
  <scope>provided</scope>
</dependency>
```
然后我们创建一个 Account 的 class 来演示我们一下我们的各种操作。
```java
@Getter
@Setter
@ToString
public class Account {

    private String id;

    private String name;

}
```
默认情况下，我们在调用 Account 的`toString()` 方法之后，输出的结果可能如下：
```java
Account(id=12345, name=account)
```
这是一种比较标准的格式输出。
<a name="tQwDq"></a>
### Lombok的配置
<a name="q6IGd"></a>
#### 父类 toString() 的调用
现在假设我们有一个 `SavingAccount` 的 class 继承于 Account ，我们调用 SavingAccount 的 toString() 方法时，希望把 Account 的一些属性也能够一起输出， 这个时候我们可以设置 callSupper 属性来达到我们的目的。
```java
@ToString(callSupper = true)
public class SavingAccount extends Account {
    private String savingAccountId;
    // 省略 get set
}
```
上述代码的操作，就能把父类 Account 的属性都输出出来：
```java
SavingAccount(super=Account(id=12345, name=An account), savingAccountId=6789)
```
<a name="ltQFh"></a>
#### 省略字段名称
我们知道默认输出的时候，会包含字段名称，我们可以通过设置 includeFieldNames 来控制，是否显示属性名称。
```java
@ToString(includeFieldNames = false)
public class Account {

    private String id;

    private String name;

    // 省略 get set
}
```
把 includeFieldNames 设置为 false 之后，输出结果如下
```java
Account(12345, An account)
```
<a name="UU9Go"></a>
#### 使用字段代替 Getter
我们知道 getter 方法提供了用于打印的字段值。如果该类不包含某个特定字段的getter方法，那么Lombok会直接访问该字段并获取其值。<br />我们可以通过设置 `doNotUseGetters` 属性为 true，将 Lombok 配置为总是使用直接的字段值而不是getter。
```java
@ToString(doNotUseGetters = true)
public class Account {

    private String id;

    private String name;

    // ignored getter
    public String getId() {
        return "this is the id:" + id;
    }

    // standard getters and setters
}
```
如果没有这个属性，我们会得到通过调用getters得到的输出。
```java
Account(id=this is the id:12345, name=An account)
```
相反，通过设置doNotUseGetters属性，输出实际上显示了id字段的值，而没有调用getter。
```java
Account(id=12345, name=An account)
```
<a name="TiBxk"></a>
#### 字段的包含和排除
假设我们想从字符串表示中排除某些字段，例如，密码、其他敏感信息或大的JSON结构。我们可以通过@ToString.Exclude注解来省略这些字段。<br />让我们把名字字段从我们的表示中排除
```java
@ToString
public class Account {

    private String id;

    @ToString.Exclude
    private String name;

}
```
或者，我们可以只指定输出中所需的字段，我可以通过使用 `@ToString(onlyExplicitlyIncluded = true) `和 `@ToString.Include`来实现。 
```java
@ToString(onlyExplicitlyIncluded = true)
public class Account {

    @ToString.Include
    private String id;

    private String name;
    

}
```
上述两种方法，最终输出，都只能输出 id 字段。
```java
Account(id=12345)
```
另外，Lombok 会自动忽略以$ 开头的变量，但是我们可以通过 @ToString.Include 来强制Lombok输出。
<a name="ld6Ts"></a>
#### 输出排序
默认情况下，Lombok 的输出，是按照字段定义的顺序进行输出的，我可以通过设置 @ToString.Include 来进行排序。<br />我们先修改一下 Account 的字段顺序， 然后对 id 进行标记顺序。
```java
@ToString
public class Account {

    private String name;

    @ToString.Include(rank = 1)
    private String id;

}
```
现在 id 字段输出的时候，会排在 name 的前面
```java
Account(id=12345, name=An account)
```
Lombok 输出的规则大致如下：

- rank 排名越大，排序越靠前
- 默认的排序值为0
- 相同的排序通过根据字段定义顺序输出
<a name="D8GtR"></a>
### 方法输出
除了字段之外，我们也可以包括一个不需要参数的实例方法的输出。我们可以通过用@ToString.Include标记无参数的实例方法来做到这一点。
```java
@ToString
public class Account {

    private String id;

    private String name;

    @ToString.Include
    String description() {
        return "Account description";
    }

}
```
这里 description 将会作为输出 key 进行打印输出。
```java
Account(id=12345, name=An account, description=Account description)
```
如果指定的方法名称与字段名称相匹配，那么该方法就会优先于字段。换句话说，输出包含方法调用的结果，而不是匹配字段的值。
<a name="ZIMZC"></a>
#### 修改字段名称
我们可以通过 @ToString.Include 的属性来修改字段的名称。
```java
@ToString
public class Account {

    @ToString.Include(name = "identification")
    private String id;

    private String name;

}
```
现在输出结果中，将不会包含字段名称id ，将会输出 identification。
```java
Account(identification=12345, name=An account)
```
<a name="b9nFb"></a>
#### 打印数组
Lombok 使用 Arrays.deepToString() 方法打印数组，将数组元素转换为其相应的字符串表示。但是数组有可能包含直接引用或间接循环引用。为了避免无限递归及其相关的运行时错误，该方法将任何从自身内部对数组的循环引用渲染为"[[...]]"。<br />让我们通过给我们的账户类添加一个对象数组字段来看看。
```java
@ToString
public class Account {

    private String id;

    private Object[] relatedAccounts;

}
```
这 relatedAccounts 数组的打印如下
```java
Account(id=12345, relatedAccounts=[54321, [...]])
```
重要的是，循环引用被deepToString()方法检测到，并且被Lombok适当地呈现出来，没有引起任何StackOverflowError。
<a name="Psdj4"></a>
### 有一些注意点
有几个细节值得一提，对避免产生意外的结果很重要。

- 在类中存在任何名为toString()的方法（不管返回类型如何），Lombok不会生成其 toString() 方法。
- 不同版本的Lombok可能会改变生成方法的输出格式。在任何情况下，我们应该避免依赖解析toString()方法输出的代码。所以这其实不应该是一个问题。
- 我们还可以在枚举上添加这个注解。这将产生一个枚举值跟随枚举类名称的表示，例如，AccounType.SAVING。
