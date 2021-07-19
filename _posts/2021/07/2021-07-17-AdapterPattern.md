---
layout: post
title:  新入职小B一顿操作猛如虎，就被领导。。。
tagline: by 揽月中人
categories: 设计模式
tags:
- 揽月中人
---

小B刚入职一家公司，领导就要求把几个接口盘一下，不然就给他一脚。小B上来一顿操作猛如虎改了几个接口，结果还是被给了一脚，小B欲哭无泪。后来小B关注了Java技术指北，看了适配器模式之后，恍然大悟。那我们拾掇一下适配器模式Adapter。

<!--more-->

### 1 适配器模式示例

适配器使一个接口与其他接口兼容。通过增加一个新的适配器类来解决接口不兼容的问题，使得原本没有任何关系的接口可以协同工作。

适配器模式主要由以下几部分内容构成：

- Target 目标接口
- Adaptee 已经存在的被适配的接口，对这个接口进行适配。
- Adapter 对Adaptee接口与Target进行适配

下面请看简单示例

Adaptee 接口及实现类

```java
public interface AdapteeInterface {
    void showLastName(String lastName);
    void showAge(int age);
}

public class Adaptee implements AdapteeInterface{
    public void showLastName(String lastName){
        System.out.println("Adapted showLastName ! Hello Mr." + lastName);
    }
    public void showAge(int age){
        System.out.println("Adapted showAge, age is :" + age);
    }
}
```

Target目标接口 

```java
public interface Target {
    void showFullName(String firstName , String lastName);
    void showLastNameAndAge(String lastName, int age);
    void showPersonalInfo(String firstName , String lastName, int age);
}
```

适配器模式可以使Target 和 AdapteeInterface两个接口进行协同工作。



使用类适配器：适配器与适配者之间是继承（或实现）关系。

```java
public class ClassAdaptor extends Adaptee implements Target{

    @Override
    public void showFullName(String firstName, String lastName) {
        System.out.println("ClassAdaptor showFullName firstName："
                           +firstName+" lastName: "+lastName);
        super.showLastName(lastName);
    }

    @Override
    public void showLastNameAndAge(String lastName, int age) {
        System.out.println("ClassAdaptor showLastNameAndAge  ， lastName："
                           +lastName+" age: "+age);
        super.showAge(age);
    }

    @Override
    public void showPersonalInfo(String firstName, String lastName, int age) {
        this.showFullName(firstName,lastName);
        this.showLastNameAndAge(lastName,age);
    }
}
```

ClassAdaptor类，通过继承Adaptee类。实现Target类接口，完成Adaptee -> Target的适配。



使用对象适配器：适配器与适配者之间是关联关系.

```java
public class ObjectAdaptor implements Target{

    private AdapteeInterface adaptee;

    public ObjectAdaptor(AdapteeInterface adaptee) {
        this.adaptee = adaptee;
    }

    public void showPersonalInfo(String firstName , String lastName, int age){
        showFullName(firstName,lastName);
        showLastNameAndAge(lastName,age);
    }

    @Override
    public void showFullName(String firstName , String lastName) {
        System.out.println("ObjectAdaptor showFullName firstName："
                           +firstName+" lastName: "+lastName);
        adaptee.showLastName(lastName);
    }

    @Override
    public void showLastNameAndAge(String lastName, int age) {
        System.out.println("ObjectAdaptor showLastNameAndAge  ， lastName："
                           +lastName+" age: "+age);
        adaptee.showAge(age);
    }
}
```

ObjectAdaptor 通过对象组合的方式，完成Adaptee -> Target的适配。



Client 调用

```java
public class Client {
    public static void main(String[] args) {
        Target objectAdaptor = new ObjectAdaptor(new Adaptee());
        objectAdaptor.showPersonalInfo("James","Gosling",66);
        System.out.println("=================");
        Target classAdaptor = new ClassAdaptor();
        classAdaptor.showLastNameAndAge("Gosling",66);
    }
}
```



最终的适配关系如下图所示

![image-20210719012409330](http://www.javanorth.cn/assets/images/2021/lyj/adapterUML.png)

### 2 适配器模式在SpringMVC中的应用

Spring定义了一个适配器接口，使得每一种Controller有一种对应的适配器实现类适配器替代controller执行相应的方法。

获取HandlerExecutionChain中的handle，然后得到HandlerAdapter 的实现类然后调用相关的方法，

在Spring MVC中，DispatcherServlet 为Client，HandlerAdapter 作为期望目标接口，具体的适配器实现类对Controller进行适配。





### 3 适配器模式的使用情景

#### 3.1 关于类适配器和对象适配器

如果被适配类中定义的方法很多，那么我们目标类接口定义的方法大部分都相同，那我们推荐使用类适配器。 适配器复用父类被适配的类的方法，比对象适配器的实现需要少写一些代码。

如果被适配类的方法很多，且目标类接口中定义的方法大部分都不相同，则推荐使用对象适配器，组合优于继承更加灵活。

#### 3.2 适配器使用情景

- 你想使用一个已经存在的类，但是它的接口不符合你的需求。
- 你想创建一个可以复用的类，该类与其他不相关的类或不可预见的类协同工作。
- 你想使用一些已经存在的类，但是不可能对每一个都进行子类化以匹配他们的接口。

### 4 适配器模式对比

适配器模式是一种对现有类的适配策略。提供一个与原始类不同的接口，适配器的意义是将一个接口转变成另一个接口，通过改变接口来达到重复使用的目的。装饰模式不改变被装饰对象的接口，而是要保持原有的接口，并增强原有对象的功能。装饰模式支持递归组合，而适配器则不能实现这一点。

与桥接模式对比，桥接模式目的是将接口和实现部分分离，从而可以比较容易的，而Adapter意味着改变一个已有对对象的接口。


### 总结

本篇讲了适配器模式的基本要点，以及在SpringMVC中的应用，适配器的使用情景，再分析了一下与装饰模式、桥接模式的不同点。实际生产中适配器模式使用的频率比较高，在新老系统兼容以及系统更新中的使用频率都特别高。所以赶紧玩起来，不久必然会用到。如果还有什么想法，不妨在评论区留言。

