---
layout: post
title:  新房装修与装饰模式！
tagline: by 揽月中人
categories: 设计模式
tags:
- 揽月中人
---

最近指北君忙于新房装修，那么作为程序员必然会发出一些疑问，装饰模式和装修到底有没有关系？经过一番折腾和操作就有了这篇！那么带大家看看新房装修与装饰模式如何发生关系！

<!--more-->

 

### 1 装饰模式与新房装修

最近新房交房了，装修得整起来，那么看看装饰模式和装修怎么搞在一起！

![img](http://www.javanorth.cn/assets/images/2021/lyj/house-key1.png)

首先我们有一个房子接口，或者抽象类都可以。这里我们使用接口，主要目的搞装修，安排个卧室装修，和卫生间装修。

```java
public interface House {
    void bathRoomDecoration();
    void bedRoomDecoration();
}
```



然后房子有别墅，有单元楼。虽然别墅是梦想，但是只要心中有梦，还是有机会实现的。

```java
public class Apartment implements House{
    @Override
    public void bathRoomDecoration() {
        System.out.println("[小两室，实用雅致卫生间装修]");
    }
    @Override
    public void bedRoomDecoration() {
        System.out.println("[小两室，温暖小窝装修]");
    }
}

//梦想一下下，未来是美好的！
public class Villa implements House{
    @Override
    public void bathRoomDecoration() {
        System.out.println("别墅豪华卫生间装修");
    }
    @Override
    public void bedRoomDecoration() {
        System.out.println("别墅豪华卧室装修");
    }
}
```



去网上随便搜索了一下装修的东西，装修公司很快就联系到了我，可见如今的大数据有多疯狂。装饰模式中间类。

```java
public abstract class HouseDecorator  implements House{
    House  house;
    public HouseDecorator(House house) {
        this.house = house;
    }
    public void bathRoomDecoration(){
        house.bathRoomDecoration();
    }
    public void bedRoomDecoration(){
        house.bedRoomDecoration();
    }    
}
```



装修公司让我看了卫生间，卧室的装修方案，那么大家一起来看看！

卫生间装修方案：

```java
public class BathRoomDecorate extends HouseDecorator{
    public BathRoomDecorate(House house) {
        super(house);
    }
    public void bathRoomDecoration(){
        house.bathRoomDecoration();
        bathRoomDecorate();
    }
    private void bathRoomDecorate() {
        System.out.println("========鎏金全自动智能宫殿级汤浴场=========");
    }
}
```



卧室装修方案

```java
public class BedRoomDecorator extends HouseDecorator{
    public BedRoomDecorator(House house) {
        super(house);
    }
    public void bedRoomDecoration(){
        house.bedRoomDecoration();
        bedRoomDecorate();
    }
    private void bedRoomDecorate() {
        System.out.println("========全智能温控深度睡眠房=========");
    }
}
```



虽然只能买小两室，但是装修还是要多看看。最后我们看看卧室装修方案，然后再看看卫生间装修方案。

```java
public class DecoratorConsumer {
    public static void main(String[] args) throws IOException {
        House myHouse = new Apartment();
        HouseDecorator roomDecorate = new BedRoomDecorator(myHouse);
        roomDecorate = new BathRoomDecorate(roomDecorate);

        System.out.println("--------看看卧室搞成什么样子--------");
        roomDecorate.bedRoomDecoration();

        System.out.println();
        System.out.println("--------看看卫生间整成什么样子--------");
        roomDecorate.bathRoomDecoration();
    }
}
```



输出结果

![image-20210711200102001](http://www.javanorth.cn/assets/images/2021/lyj/houseDecoratorResult1.png)

### 2 装饰模式的作用

动态地给一个对象添加一些额外的职责，且并不需要生成子类。 属于结构型设计模式。

### 3 JDK中的装饰模式

Java中InputStream的实现中有用到装饰模式，那么给大家盘一盘。下面为网上比较多的类图关系。

![image](http://www.javanorth.cn/assets/images/2021/lyj/DP-Decorator-java.png)

Java 中缓存字节流 BufferedInputStream 可以看成是 InputStream 实例的装饰类，对其中的read等方法进行了增强。

简单使用如下所示：

```java
InputStream fileInput = new FileInputStream(new File("src/main/resources/input.txt"));
BufferedInputStream bufferInput = new BufferedInputStream(fileInput);
```

BufferedInputStream 是具体的装饰类，它对InputStream实例中的一些方法进行增强，FilterInputStream为中间过渡类，负责抽象类InputStream的一些方法的默认实现，使其子类可以专注于特定功能实现。InputStream的实例 fileInput 为被修饰对象。

### 4 装饰模式在什么情况下推荐使用

以下情况可以考虑使用装饰模式

- 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加功能。

- 处理那些可以撤销的职责。

- 当不能采用生成子类的方法进行扩充时，可以考虑采用装饰模式，有以下2种情况。

  ​	1. 有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈现爆炸性增长。

  ​	2. 另外就是类定义被隐藏，或类定义不能用于生成子类。

### 5 装饰模式与代理模式

设计模式中结构型模式均有一些相似之处。其中代理模式与装饰模式对比而言，代理模式的目的主要是直接访问一个实例不方便或者不可访问时，需要为这个实例提供一个替代者。

而装饰模式中，组件一般仅提供部分功能，会有一个或者多个Decorator负责完成其功能。装饰模式适用于编译时不能确定对象的全部功能的情况。



### 总结

本篇粗略的讲了装饰模式，以及其在JDK中的应用，再分析了一下装饰模式与代理模式的不同点。实际应用过程中变化多样，可能会出现proxy-decorator这种组合，或者decorator-proxy这种组合使用的情况。拆分出来讲，我们可以更明白其中的设计思想。

那么还是老样子啦，有问题评论区见！



