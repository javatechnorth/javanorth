---
title: IDEA WARNING：@Autowired 犯了什么错？ 2023-05-17已发
date: 2023-05-17 08:15:00
author: gotanks广楠
categories: gotanks广楠
tagline: by 广楠
tags: ["@Autowired", "gotanks广楠"]
---

哈喽，大家好，我是了不起。  

做开发的同学可能都会发现, IDEA 在我们经常使用的`@Autowired`注解上添加了警告: Field injection is not recommended, 即: 不推荐使用属性注入

---


<!--more-->

## 问题原因

### 违背单一职责原则

因为现在的业务一般都会使用很多依赖, 但拥有太多的依赖通常意味着承担更多的责任,而这显然违背了单一职责原则。

### 依赖Spring

`@Autowired`由 Spring 提供，而`@Resource`是JSR-250提供的，它是Java标准。前者会警告，而后者不警告，就是因为前者导致了应用与框架的强绑定，若是换成其他IOC框架，则不能够成功注入了。其实对于这方面，我认为在大多数情况时是不会有什么问题的。

### 其他

我看到网络上有一些其他方面的总结，比如：因为是 ByType 注入, 因此有可能会出现两个相同的类型bean，进而导致Spring装配失败；不能像构造器那样注入不可变的对象等，这类问题需要结合个人实际开发进行判断。

对于`@Autowired`使用方面，它虽然是将业务代码和框架进行了强绑定，但字段注入确实大幅简化了代码。能够有效提高代码简洁性，让依赖注入的事情交给IOC容器，省时省力，这也是它的优点，我们应该在实际使用中追求平衡，否则将为了过度追求松耦合而得不偿失。


## 其他注入方法

除了使用`@Autowired`以外，我们其实也有几种好用的方式。使用`@Resource`替代`@Autiwired`方法是其中一种，只需要改变一个注解，这里就不展示了。

### Setter注入

能够通过懒加载的方式解决循环依赖，类中的依赖在需要用到的时候才会注入。另外，setter注入方式很灵活，注入的对象还能改变。

```java
@RestController
public class DemoController {

    private DemoService demoService;

    /*
     * 基于set注入
     * */
    @Autowired
    public void setDemoService(DemoService demoService) {
        this.demoService = demoService;
    }

}
```
这种方法也使用了`@Autowired`注解，但是它是作用于成员变量的Setter函数上，而不是像Field注入一样作用于成员变量上。

这是三种注入方式中最灵活的，这个灵活就是它的缺点。Setter注入的依赖不能保证依赖不可变。

### 构造器

```java
@RestController
public class DemoController {

    private DemoService demoService;

    /*
    * 基于构造方法的注入
    * */
    public DemoController(DemoService demoService) {
        this.demoService = demoService;
    }
}
```

它的好处在于，采用了构造方法注入，这种方式对对象创建的顺序会有要求，它将避免循环依赖问题。是最可靠的方法。

但其也有缺点，假如类中需要注入的依赖比较多，就会显得构造方法很臃肿，缺乏可读性。另外，构造器注入不能解决循环依赖问题。


### 构造器简化版（推荐）

首先，需要引入lombok依赖。

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

随后，我们在创建时就可以使用`@RequiredArgsConstructor`注解，它将帮我们创建构造器，注意，属性必须添加 **final** 关键字。

```java
@RestController
@RequiredArgsConstructor
public class DemoController {
    /*
     * 用@RequiredArgsConstructor注解，这个使用方式也可以应用于service层
     * */
    private final DemoService demoService;

}
```

通过查看编译结果，发现Lombok自动生成了构造方法，非常简便。


## 总结

最不推荐使用的是 **属性注入** ，除了省力以外没有好处。

**Setter注入** 和 **构造器注入** 各有优劣，需要根据实际情况选择。

最推荐使用 **Lombok版的构造器注入** 方式，既简单又可靠。




