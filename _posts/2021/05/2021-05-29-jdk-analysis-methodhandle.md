---
layout: post
title:  JDK源码解读——MethodHandle
tagline: by simsky
categories: JDK 源码解析
tags: 
    - 反射
    - MethodHandle

---

哈喽，大家好，我是指北君。反射（Reflect）作为Java最重要的一种机制，相信大家一定都很熟悉了，今天指北君要介绍另一种和反射机制类似的方法调用机制——MethodHandle。

MethodHandle是Java7引入的一种机制，主要是为了JVM支持动态语言，
<!--more-->

## 一个MethodHandle调用示例

### 创建查找对象：Lookup

```java
		// 获取Look用于查找方法句柄
		MethodHandles.Lookup lookup = MethodHandles.lookup();
```

### 创建方法类型：MethodType
构造方法的返回值类型，参数类型

```java
		// 方法类型，描述返回值和参数，第一个参数为返回值类型，void则为void.class。第二个参数开始为被调用方法的参数类型
		MethodType mt = MethodType.methodType(boolean.class, String.class, int.class);
```

### 查找方法句柄
Lookup的findVirtual查找成员方法

```java
		//查找方法句柄，参数1：调用类，参数2：方法名，参数3：方法类型
		MethodHandle handle = lookup.findVirtual(String.class, "startsWith", mt);
```

### 方法调用
通过MehodHandle的invoke方法执行，并返回结果

```java
		//方法调用，参数1：实例，参数2..n：方法参数
		Boolean value = (Boolean)handle.invoke("the i am in the room", "the", 0);
```

以上就是一个简单的调用示例。

## 核心类MethodType
MethodType构造方法为私有，只能通过MethodType提供的静态工具方法来获取实例

```java
    /**
     * Constructor that performs no copying or validation.
     * Should only be called from the factory method makeImpl
     */
    private MethodType(Class<?> rtype, Class<?>[] ptypes) {
        this.rtype = rtype;
        this.ptypes = ptypes;
    }
```

常用的工具方法：
 * 大于一个参数
```java
    public static MethodType methodType(Class<?> rtype, Class<?> ptype0, Class<?>... ptypes) {
        Class<?>[] ptypes1 = new Class<?>[1+ptypes.length];
        ptypes1[0] = ptype0;
        System.arraycopy(ptypes, 0, ptypes1, 1, ptypes.length);
        return makeImpl(rtype, ptypes1, true);
    }
```

 * 无参数
```java
    public static MethodType methodType(Class<?> rtype) {
        return makeImpl(rtype, NO_PTYPES, true);
    }
```
 

静态工具方法都通过makeImple创建实例，该方法做如下几件事情：
1. 参数检查
	+ 返回值的类型不能为null，如果无返回使用void.class
	+ 参数类型不能为null，且不能为void.class
2. 使用缓存表缓存MethodType实例，优化处理
3. 如果非信任模式(trusted==false),则克隆参数数组

这里需要注意MethodType重写了hashCode方法，从逻辑看参数数组克隆不影响同类型的缓存机制。

```java
static
    MethodType makeImpl(Class<?> rtype, Class<?>[] ptypes, boolean trusted) {
        if (ptypes.length == 0) {
            ptypes = NO_PTYPES; trusted = true;
        }
        MethodType primordialMT = new MethodType(rtype, ptypes);
        MethodType mt = internTable.get(primordialMT);
        if (mt != null)
            return mt;

        // promote the object to the Real Thing, and reprobe
        MethodType.checkRtype(rtype);
        if (trusted) {
            MethodType.checkPtypes(ptypes);
            mt = primordialMT;
        } else {
            // Make defensive copy then validate
            ptypes = Arrays.copyOf(ptypes, ptypes.length);
            MethodType.checkPtypes(ptypes);
            mt = new MethodType(rtype, ptypes);
        }
        mt.form = MethodTypeForm.findForm(mt);
        return internTable.add(mt);
    }
```

除了静态工具方法外，MethodType还有几个实例方法，主要为差异性较小的MethodType提供快速获取实例的方法。包含：
+ changeParameterType
+ insertParameterTypes
+ appendParameterTypes
+ dropParameterTypes

## 核心类MethodHandle
MethodHandle为抽象类，但是里面提供了大量的原生方法，提供底层访问，也是方法调用的核心逻辑。这部分涉及MethodHandle的机制实现，对于使用功能来说指北君就不在此展开了。
```java
    @HotSpotIntrinsicCandidate
    public final native @PolymorphicSignature Object invokeExact(Object... args) throws Throwable;

    @HotSpotIntrinsicCandidate
    public final native @PolymorphicSignature Object invoke(Object... args) throws Throwable;
```
![调用方法](/assets/images/2021/simsky/jdk_src_methodhandle_2.png)

## 核心类MethodHandles, Lookup
MethodHandles不是MethodHandle的实现，他提供工具用于帮助获取MethodHandle，我们主要使用到lookup(),publicLookup()

@CallerSensitive注解，可以使Reflection.getCallerClass()获取到调用lookup()方法的类
```java
    @CallerSensitive
    @ForceInline // to ensure Reflection.getCallerClass optimization
    public static Lookup lookup() {
        return new Lookup(Reflection.getCallerClass());
    }
```

```java
    public static Lookup publicLookup() {
        return Lookup.PUBLIC_LOOKUP;
    }
```
Lookup构造方法主要传入：lookupClass搜寻的类, allowedModes:许可模式。
通过Lookup的findXXX获取到MethodHandle

![查找方法](/assets/images/2021/simsky/jdk_src_methodhandle_1.png)


## 小结

关于MethodHandle的基本使用就基本讲完，这里指北君附上一张类图便大家理解：
![类图](/assets/images/2021/simsky/jdk_src_methodhandle_3.png)

最后感谢各位小伙伴的点赞、收藏和评论，我们下期更精彩。
