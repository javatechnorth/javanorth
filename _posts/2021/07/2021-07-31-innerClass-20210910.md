---
layout: post
title:  内部类的使用（上）
tagline: by 24只羊
categories: JDK
tags: 
    - 24只羊

---



大家好，我是指北君

内部类相信大家都应该用过，但我也相信大家应该都只是很简单的使用，所以今天，指北君会通过两篇文章详细讲解内部类的使用，这篇文章为上篇，主要讲解如何在类和接口中使用内部类，下一篇将会介绍如何使用static修饰的内部类和如何在方法中使用内部类，废话不多说，我们先赶紧来看吧。

<!--more-->


## 1 在普通类中使用内部结构

不多说，先上个代码

Outer类里面有个内部类Inner

```java
public class Outer {
    private String msg = "哈哈";   //只能在类内部访问
    public void fun(){
        Inner in = new Inner();    //实例化内部类的对象
        in.print();
    }
 
    //在Outer类中的内部类
    class Inner{
        public void print(){
            System.out.println(Outer.this.msg);   //msg是Outer类里面的属性
        }
    }
}
```



测试类

```java
public class Test {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.fun();
    }
}
```



创建外部类的实例调用外部类的方法却执行了内部类的方法，打印出“哈哈”。

从整体的代码结构来讲，内部类的结构并不合理，所以内部类本身最大的缺陷在于破坏了程序的结构，但是破坏需要有目的的破坏，那么它也一定会有其优势，如果要想更好的观察出内部类的优势，就可以将内部类拿到外面来。我将上面的代码Inner拿出来

```java
public class Outer {
    private String msg = "哈哈";   //只能在类内部访问
    public void fun(){
        Inner in = new Inner(this);    //实例化内部类的对象
        in.print();
    }
    public String getMsg(){
        return this.msg;
    }
}
```



```java
public class Inner {
    private Outer out;
    public Inner(Outer out){
        this.out = out;
    }
    public void print(){
        System.out.println(this.out.getMsg());   //msg是Outer类里面的属性
    }
}
```



```java
public class Test {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.fun();
    }
}
```



如果不是对引用和扎实，会有点懵。我们折腾了半天主要的目的就是为了让Inner这个内部类可以访问Outer这个类中的私有属性，如果不用内部类的时候整体代码会非常的麻烦，所以可以得出内部类的优点：**轻松的访问外部类中的私有属性**。

需要注意的是，**内部类虽然可以方便的访问外部类中的私有成员或私有方法**，同理**，外部类也可以轻松访问内部类中的私有成员或私有方法**。如下

```java
public class Outer {
    private String msg = "哈哈";   
    public void fun(){
        Inner in = new Inner();
        in.print();
        System.out.println(in.info);        //访问内部类的私有属性
    }
    class Inner{
        private String info = "今天天气不好";
        public void print(){
            System.out.println(Outer.this.msg);
        }
    }
}
```



使用了内部类之后，内部类与外部类之间的私有操作的访问就不再需要通过setter,getter以及其他的间接方式完成了，可以直接进行操作，但是需要注意的是，内部类本身也属于一个类，虽然在大部分情况下内部类往往是被外部类包裹的，但是外部依然可以产生内部类的实例化对象，而此时，**内部类实例化对象**的格式如下：

**外部类.内部类   内部类对象  =  new    外部类（）.new   内部类（）**；

在内部类编译完成之后会自动形成一个“Outer$Inner.class”类文件，其中“$”这个符号换到程序中就变为“.”，所以**内部类的全称：“外部类.内部类”**。内部类与外部类之间可以直接进行私有成员的访问，这样一来内部类如果要是提供有实例化对象了，一定要先保证外部类实例化了。

```java
public class Test {
    public static void main(String[] args) {
        Outer.Inner in = new Outer().new Inner();
        in.print();
    }
}
```



**如果此时Inner类只允许Outer类来使用，那么在这样的情况下就可以使用private进行私有定义**。

这样，此时的Inner类就无法再外部使用，**即在test中的这条语句 Outer.Inner in = new Outer().new  Inner()就失效**。



## 2. 在抽象类和接口中使用内部结构

在我们的java之中，类作为最基础的结构体实际上还有与之类似的抽象类或者是接口，抽象类和接口中都可以定义内部结构。



### 2.1 接口中定义内部接口

我们现在定义**内部接口**：

```java
public interface IChannel {
    public void send(IMessage msg);
    //内部接口
    interface IMessage{
        public String getContent();
    }
}
```



```java
public class ChannelImpl implements IChannel {
    @Override
    public void send(IMessage msg) {
        System.out.println(msg.getContent());
    }
 
    class MessageImpl implements IMessage{
        @Override
        public String getContent() {
            return "haha";
        }
    }
}
```



```java
public class Test {
    public static void main(String[] args) {
        IChannel channel = new ChannelImpl();
        channel.send(((ChannelImpl)channel).new MessageImpl());
    }
}
```



最后打印出结果  “哈哈”



### 2.2 接口中定义内部抽象类

下面我们继续观察一个**内部抽象类**，内部抽象类可以定义在普通类，抽象类，接口内部都可以

```java
public interface IChannel {
    public void send();
    abstract class AbstractMessage{
        public abstract String getContent();
    }
}
```



```java
public class ChannelImpl implements IChannel {
 
    @Override
    public void send() {
        AbstractMessage msg = new MessageImpl() ;
        System.out.println(msg.getContent());
 
    }
 
    class MessageImpl extends AbstractMessage {
        public String getContent() {
            return "哈哈";
        }
    }
}
```



```java
public class Test {
    public static void main(String[] args) {
        IChannel channel = new ChannelImpl();
        channel.send();
    }
}
```



结果打印出来的是“哈哈”。



### 2.3 用内部类实现外部接口

内部类还有一些更为有意思的结构，即：如果现在**定义了一个接口，那么可以在内部利用类实现该接口**，在JDK1.8之后，接口中追加了static方法可以不受到实例化对象的控制，现在就可以利用此特性来完成功能。

接口内部进行接口实现

```java
public interface IChannel {
    public void send();
 
    class ChannelImpl implements IChannel{
        public void send(){
            System.out.println("哈哈");
        }
    }
    
    public static IChannel getInstance(){
        return new ChannelImpl();
    }
}
```



```java
public class Test {
    public static void main(String[] args) {
        IChannel channel = IChannel.getInstance();
        channel.send();
    }
}
```



输出的结果为“哈哈”

从上面可以看到，内部类是非常灵活的结构，只要你的语法满足了，各种需求都可以帮你实现！



## 3. 总结：

方法，类，抽象类，接口，代码块中都可以定义内部结构-------类，抽象类，接口。今天主要讲了如何在内部类中使用内部类和接口中使用内部类。下一篇将会介绍如何使用static修饰的内部类和如何在方法中使用内部类。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
