---
layout: post
title:  又被问到Java代理模式了吧，看完这篇就可以披甲再战了！
tagline: by 揽月中人
categories: 设计模式 代理模式
tags:
- 揽月中人
---

" 最近朋友小B去面试了，面试官问了代理模式，小B开心的差点笑出了声。原来就是因为小B刚好撸了这篇Java代理模式，然后就对面试官滔滔不绝。那么我们来看看Java中的代理模式是怎么回事，看完还不懂，直接来怼。

<!--more-->

### 1 代理模式的模型。

代理模式（proxy 或 surrogate）属于结构型模式。

主要目的：为其他对象提供一种代理以控制对这个对象的访问。

![img](E:\javaNorth\javanorth\assets\images\2021\lyj/proxyUML.jpg)



代理模式可以在目标对象实现的基础上，增强额外的功能操作，即扩展目标对象的功能 。被代理的对象可以是远程对象，创建开销的的对象，需要安全控制的对象。

代理模式有三种类型，静态代理，动态代理（JDK代理，接口代理）、Cglib代理（在内存中动态的创建目标对象的子类）

### 2 代理模式的示例

下面我们分别在几种场景下看各种代理。

#### 2.1 静态代理

某公司有一个产品，在当地销售需要找到一个代理销售商。那么客户需要购买产品机器的的时候，就直接通过代理商购买就可以。

![img](http://www.javanorth.cn/assets/images/2021/lyj/proxyStatic1.jpg)

静态代理需要先定义接口，被代理对象与代理对象一起实现相同的接口，然后通过调用相同的方法来调用目标对象的方法。

创建公司接口：

```java
public interface MachineCompany {
    public Machine produceMachine();
}
```

公司的工厂生产产品

```java
public class MachineFactory implements MachineCompany{
    @Override
    public Machine produceMachine() {
        System.out.println("machine factory producing machine.... ");
        return new Machine();
    }
}
```

代理商去下单拿货（静态代理类）

```java
public class MachineProxy implements MachineCompany{
    private MachineCompany machineCompany;
    public MachineProxy() {
    }

    @Override
    public Machine produceMachine() {
        System.out.println("machine proxy get order .... ");
        System.out.println("machine proxy start produce .... ");
        if(Objects.isNull(machineCompany)){
            System.out.println("machine proxy find factory .... ");
            machineCompany = new MachineFactory();
        }
        return machineCompany.produceMachine();
    }
}
```

消费者通过代理商拿货（代理类的使用）

```java
public class MachineConsumer {
    public static void main(String[] args) {
        MachineProxy proxy = new MachineProxy();
        Machine machine = proxy.produceMachine();
    }
}
```

通过上面的代理商，我们最终也拿到了产品，



#### 2.2 动态代理

有一天公司增加了业务，出售的商品越来越多，售后也需要更上。但是公司发现原来的代理商，还要再培训才能完成全部的业务，于是就找了另外的动态代理商J 。  代理商J 承诺无缝对接公司所有的业务，不管新增什么业务，均不需要额外的培训即可完成。

JDK动态代理对象不需要实现接口，只有目标对象需要实现接口。实现基于接口的动态代理需要利用JDK中的API。

需要使用到 java.lang.reflect.Proxy，和其newProxyInstance 方法。



公司增加了维修业务

```java
public interface MachineCompany {
    public Machine produceMachine();
    public Machine repair(Machine machine);
}
```



工厂也得把维修业务搞起来

```java
public class MachineFactory implements MachineCompany {
    @Override
    public Machine produceMachine() {
        System.out.println("machine factory producing machine.... ");
        return new Machine();
    }
    @Override
    public Machine repair(Machine machine) {
        System.out.println("machine is health.... ");
        return machine;
    }
}
```



J代理商全面代理公司所有的业务。使用Proxy.newProxyInstance 方法生成代理对象，实现InvocationHandler中的 invoke方法，在invoke方法中通过反射调用代理类的方法，并提供增强方法。

```java
public class MachineProxyFactory {
    private Object target;
    public MachineProxyFactory(Object object) {
        this.target = object;
    }
    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("machine proxy find factory for machine .... ");
                Object invoke = method.invoke(target, args);
                return invoke;
            }
        });
    }
}
```



购买，维修J代理就可以直接搞定了。后面公司再增加业务，J代理也可以一件搞定。

```java
public class MachineConsumer {
    public static void main(String[] args) {
        MachineCompany target = new MachineFactory();
        MachineCompany machineCompany = (MachineCompany) new MachineProxyFactory(target).getProxy();
        Machine machine = machineCompany.produceMachine();
        machineCompany.repair(machine);
    }
}
```



#### 2.3 Cglib代理

看到 J代理商承接了公司（接口）的多种业务，那么此时C代理商又从中发现新的商机， J 只能代理某个公司的产品，而我不仅想要代理公司产品，而且对接不同的工厂，拿货渠道更广，赚钱更爽快。于是Cglib就用上了。

静态代理和JDK代理都需要某个对象实现一个接口，有时候代理对象只是一个单独对象，此时可以使用Cglib代理。

Cglib代理可以称为子类代理，是在内存中构建一个子类对象，从而实现对目标对象功能的扩展。

![img](http://www.javanorth.cn/assets/images/2021/lyj\cglibproxy2.jpg)

C代理商不仅想代理公司，而且还想代理多个工厂的产品。

Cglib通过Enhancer 来生成代理类，通过实现MethodInterceptor接口，并实现其中的intercept方法，在此方法中可以添加增强方法，并可以利用反射Method或者MethodProxy继承类 来调用原方法。

```java
public class MachineProxyCglib implements MethodInterceptor {
    public Object getProxyInstance(Class claxx){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(claxx);
        enhancer.setCallback(this);
        return enhancer.create();
    }
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("MachineProxyFactory enhancement.....");
        Object object = methodProxy.invokeSuper(o, objects);
        return object;
    }
}
```



新代理的B工厂，

```java
public class MachineFactoryB {
    public Machine produceMachineB() {
        System.out.println("machine factory b producing machine.... ");
        return new Machine();
    }
    public Machine repairB(Machine machine) {
        System.out.println("machine b is health.... ");
        return machine;
    }
}
```



C代理可以直接和公司合作，也可以和工厂打交道。并且可以代理任何工厂的产品。

```java
public class MachineConsumer {
    public static void main(String[] args) {
        MachineCompany machineFactory = (MachineCompany) new MachineProxyCglib().getProxyInstance(MachineFactory.class);
        Machine machine = machineFactory.produceMachine();
        machineFactory.repair(machine);
        System.out.println("==============================");
        
        MachineFactoryB machineFactoryB = (MachineFactoryB) new MachineProxyCglib().getProxyInstance(MachineFactoryB.class);
        Machine machineB = machineFactoryB.produceMachineB();
        machineFactoryB.repairB(machineB);
    }
}
```



以上就是静态代理，JDK动态代理，Cglib动态代理的内容，以及简单的示例。

小结一下：

​	**静态代理**，需要代理类和目标类都实现接口的方法，从而达到代理增强其功能。

​	**JDK动态代理**，需要代理类实现某个接口，使用Proxy.newProxyInstance方法生成代理类，并实现InvocationHandler中的invoke方法，实现增强功能。

​	**Cglib动态代理**，无需代理类实现接口，使用Cblib中的Enhancer来生成代理对象子类，并实现MethodInterceptor中的intercept方法，在此方法中可以实现增强功能。





### 3 Spring中AOP使用代理

Spring中AOP的实现有JDK和Cglib两种，如下图：

![image-20210704041825205](http://www.javanorth.cn/assets/images/2021/lyj/proxyinspring.png)

如果目标对象需要实现接口，则使用JDK代理。

如果目标对象不需要实现接口，则使用Cglib代理。




### 总结

本篇介绍了代理模式，并且使用Java代码分别简单演示了静态代理，JDK动态代理，Cglib静态代理。

设计模式的组合使用千千万，大都是嵌套使用，单独拆开来学习可以帮助我们在实际使用的时候游刃有余！

如果还有什么想法，不妨在评论区留言。

