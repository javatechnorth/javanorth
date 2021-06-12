---
layout: post
title:  代购 == 代理模式？
tagline: by 24只羊
categories: 设计模式
tags: 
    - 24只羊

---

大家好，我是指北君。

最近指北君的女神要生日了，所以指北君决定通过代购买个小包包当生日礼物（这半个月又白忙活了😭），在下单完的一瞬间，突然指北君发现，代购和我们Java中的静态代理模式很像啊，指北君顾不上买包的心痛，马上码了这篇文章。

<!--more-->

指北君先放一张的静态代理的结构图：

![MESA Monitor](http://www.javanorth.cn/assets/images/2021/Yang24/staticProxyStructure.png)



上图可以看到，一共有三个角色：

1. 主题角色(Subject)：可以是抽象类也可以是接口，是一个最普通的业务类型定义；
2. 真实主题角色(RealSubject)：被代理的角色，是业务逻辑的具体执行者；
3. 代理角色(Proxy)：它负责对真实主题角色的应用，并且在真实主题角色处理理完毕前后做预处理和善后处理工作。

现在不太理解不急，我们往下看。



 <br/>


## 一. 代购与静态代理

好啦，正式开始讲代购啦，咳咳。

我们都知道，一般女生都喜欢名牌包包，但名牌包包有个特点，国外的款式与价格都比国内有优势，所以以前很多多金的妹子都要亲自跑到国外的包包商店买。但是自己去买，不仅费飞机票，还费时间，因此聪明的中国人看到商机，衍生出了一个新的职业：代购。

有了代购，妹子不需要亲自去买了，只需要找代购买了（这里很关键，**妹子是找的代购小哥买的，但实际上代购小哥还是需要去到包包商店买，毕竟代购小哥自己没货啊！因此最终提供买包服务的不是代购小哥，而是包包商店**！）



我们使用代理模式将这个代购的场景用代码描述出来。

我们首先创建一个包包商店的接口（所有包包商店都需要符合这个包包商店接口的规范），接口里面定义了个buyBag的方法：

```java
public interface IBagShop {
    void buyBag(String num);
}
```


接着定义一个包包商店的实现类，这个包包商店类实现了包包商店接口，因此提供了真实的买包服务：

```java
public class BagShop implements IBagShop {
    @Override
    public void buyBag(String money) {
        System.out.println("买了一个价值"+money+"限量款包包！");
    }
}
```


最后定义我们的代购小哥，也要实现包包商店接口（毕竟也是需要提供买包服务给妹子的），且需要引用包包商店类，**因为他最终还是需要自己去到包包商店买包包：**

```java
public class PurchasingAgent implements IBagShop {
    private IBagShop bagShop;

    public PurchasingAgent(IBagShop bagShop) {
        super();
        this.bagShop = bagShop;
    }

    @Override
    public void buyBag(String num) {
        // 调用包包商店的买服务
        bagShop.buyBag(num);
    }
}
```




三个主要类就完成啦，是不是很简单？没有？那我再给你们看下面这张图，你们就一目了然了：

![MESA Monitor](http://www.javanorth.cn/assets/images/2021/Yang24/staticProxyStructure1.png)



最后我们再写个“妹子”类测测吧：

```java
public class Girl {
    public static void main(String[] args) {
        IBagShop bagShop = new BagShop();
        PurchasingAgent purchasingAgent = new PurchasingAgent(bagShop);
        purchasingAgent.buyBag("10万");
    }
}
```


![MESA Monitor](http://www.javanorth.cn/assets/images/2021/Yang24/staticProxyResult1.png)

这个妹子成功买到这么贵的包包了！


 <br/>



## 二. 优点

那代理模式有啥优点呢？

1. 由代理对象来间接访问目标对象，避免了直接访问目标对象给系统带来复杂杂性；
2. 通过代理对象对原有对象增强。

第一点我们就不解释了，主要看第二点。

代购小哥尝到了甜头以后，就想吸引更多的妹子来找他买包，于是决定提供**售前与售后服务**，售前是提供与包包商店的服务员砍价的服务，售后则是附送一些小礼品服务，妹子们看到代购小哥的服务这么好，于是也介绍其他姐妹们来找代购买东西了。你看，这就是代购小哥对包包商店业务的增强。



那如何在代码上体现呢？因为是代购小哥提供的服务，因此我们找到代购PruchasingAgent类，在其buy方法中做修改即可：

```java
public class PurchasingAgent implements IBagShop {
    private IBagShop bagShop;

    public PurchasingAgent(IBagShop bagShop) {
        super();
        this.bagShop = bagShop;
    }

    @Override
    public void buyBag(String num) {
        // 售前服务：砍价
        System.out.println(“代购砍了50%的价格，好厉害啊！");
        // 调用包包商店的买服务
        bagShop.buyBag(num);
        // 售后服务：送小礼品
        System.out.println("代购送了价值5元的小礼品，真抠");
    }
}
```


再次启动“妹子”类测试，测试结果为：

![MESA Monitor](http://www.javanorth.cn/assets/images/2021/Yang24/staticProxyResult2.png)



从上面例子可以看到，代购小哥通过增加售前售后服务增强了包包商店的买包服务业务，因此可以看到静态代理最大的优点就是：通过代理对象对原有业务进行增强。



 <br/>


## 三. 缺点

静态方法不满足设计模式原则中的开闭原则（设计模式一共有六大原则，而开闭原则是最重要的一个！纳尼，怪不得会有动态代理）。

那啥是开闭原则？程序对外扩展开放，修改关闭，换句话说，要增加新的功能时，我们可以增加新的模块代码来实现新的需求，但不能修改原有的代码来实现新需求。我们就继续通过代购来看下代理模式如果破坏了开闭原则。

经过一段时间，有妹子问代购小哥是否可以在他这里买鞋子，代购小哥虽然没有，但是代购小哥知道鞋子商店在哪啊，于是就接了这单，此后，代购小哥不仅仅代购包包，还代购鞋子了。

为了增加代购小哥的业务，我们如何修改代码呢？

首先我们需要定义一个新的鞋子商店接口，里面定义了一个购买鞋子的方法，然后再建立个实现了鞋子商店接口的鞋子商店类，并实现购买鞋子的方法，这里还是满足开闭原则的，毕竟是新建类和接口嘛。然后最头疼的地方来了，代理类PurchaingAgent需实现鞋子商店接口，且需要引入鞋子商店类，并实现购买鞋子的方法，这些都对源代码进行大量的改动，如果以后代购小哥还想扩展业务，比如代购吃的，代购奶粉，那就要继续改动，这样就违背开闭原则了。

 <br/>


## 四. 总结

最后，指北君再来总结下：

其实，计算机中的许多思想也是通过生活而来，特别是设计模式，我们可以在生活中找到许多映射。静态代理其实就是代理类通过引用被代理对象去执行方法，当我们需要对该方法前后进行其他处理时，我们只需要在代理类中进行更改就行。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
