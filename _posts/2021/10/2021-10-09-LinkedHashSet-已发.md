---
layout: post
title:  LinkedHashSet有序且不能重复的集合 -- 20211018
tagline: by IT可乐
categories: JDK 源码解读 Java基础
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
同 HashSet 与 HashMap 的关系一样，本篇文章所介绍的 LinkedHashSet 和 LinkedHashMap 也是一致的。在 JDK 集合框架中，类似 Set 集合通常都是由对应的 Map 类集合来实现的（TreeSet 和 TreeMap 同理），这里很重要的一个理论就是：Set 类集合是不允许重复的，而 Map 类集合的 key 也是不允许重复的，所以通常很容易就用 Map 类集合实现了 Set 类集合。
<!--more-->
### 1、LinkedHashSet 定义
　　LinkedHashSet 是由 LinkedHashMap 实现的集合。元素有序且不能重复。
```
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
```

![](http://www.javanorth.cn/assets/images/2021/itcore/linkedhashset-00-00.png)  
　　看上图类定义，LinkedHashSet 是由 HashSet 来实现的，其实底层是通过 LinkedHashMap 来实现的。
### 2、构造函数
　　在 LinkedHashSet 中，有如下几个构造方法：

　　①、指定初始容量和加载因子
```
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
```
　　②、指定初始容量
```
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
```
　　③、默认无参构造函数
```
    public LinkedHashSet() {
        super(16, .75f, true);
    }
```
　　④、构造包含指定集合的元素
```
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
```
　　上面所有的构造方法，都调用父类，也就是 HashSet 的 super(initialCapacity, loadFactor, true);
```
     HashSet(int initialCapacity, float loadFactor, boolean dummy) {
         map = new LinkedHashMap<>(initialCapacity, loadFactor);
     }
```
　　前面两个参数分别设置HashMap 的初始容量和加载因子。dummy 可以忽略掉，这个参数只是为了区分 HashSet 别的构造方法。

### 3、添加元素
```
     public boolean add(E e) {
         return map.put(e, PRESENT)==null;
     }
```
　　通过 map.put() 方法来添加元素，说明了该方法如果新插入的key不存在，则返回null，如果新插入的key存在，则返回原key对应的value值（注意新插入的value会覆盖原value值）。

　　也就是说 add(E e) 方法，会将 e 作为 key，PRESENT 作为 value 插入到 map 集合中，如果 e 不存在，则插入成功返回 true;如果存在，则返回false。
### 4、删除元素
```
     public boolean remove(Object o) {
         return map.remove(o)==PRESENT;
     }
```
　　调用 HashMap 的remove(Object o) 方法，该方法会首先查找 map 集合中是否存在 o ，如果存在则删除，并返回该值，如果不存在则返回 null。

　　也就是 remove(Object o) 方法，删除成功返回 true，删除的元素不存在会返回 false。
### 5、查找元素
```
     public boolean contains(Object o) {
         return map.containsKey(o);
     }
```
　　调用 HashMap 的 containsKey(Object o) 方法，找到了返回 true，找不到返回 false。

### 6、遍历元素
```
LinkedHashSet<String> hashSet = new LinkedHashSet<>();
hashSet.add("A");
hashSet.add("B");
hashSet.add("C");
//1、增强for循环
for(String str : hashSet){
    System.out.println(str);
}
//2、迭代器
Iterator<String> iterator = hashSet.iterator();
while(iterator.hasNext()){
    System.out.println(iterator.next());
}
```
### 7、小结
　　好了，这就是JDK中java.util.LinkedHashSet 类的介绍，记住有序且不能重复。

　　我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！

