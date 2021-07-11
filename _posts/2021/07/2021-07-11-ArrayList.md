---
layout: post
title:  ArrayList 的核心源码真的只有几行
tagline: by 某某白米饭
categories: 源码解析
tags: 
    - 某某白米饭
---

大家好，我是指北君。

前几天我的朋友小 B 给他女神在面试的时候支招，问到 ArrayList 就说是用数组实现的。最后女神被面试官要求手写 ArrayList。小 B 惨遭被分手。小 B 感到非常委屈，ArrayList 不就是数组么。
<!--more-->
ArrayList 其实可以对面试官说他的扩容，默认容量，各个方法的时间复杂度等等，这样面试成功的机率比只是数组的竞争对手大了 50%。

### 构造函数

ArrayList 有三个构造函数，默认不带参数的构造函数就是初始化一个空数组。

```java

//一个空数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//实际存储元素的数组
transient Object[] elementData; 

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

```

带一个 int 类型的容量参数的构造函数，可以指定 ArrayList 的容量大小。

```java

//空数组
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        //容量大于 0 就构建一个 Object 的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        //容量等于 0 就是一个空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        //容量小于 0 抛出异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
    }
}
```

带一个集合的构造函数, 将集合转换成 Object[] 类型后拷贝到 elementData 中。 

```java
public ArrayList(Collection<? extends E> c) {
    //集合转为数组
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        //集合是不是 ArrayList 类型
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            //将集合元素拷贝成 Object[] 到 elementData
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        //空元素
        elementData = EMPTY_ELEMENTDATA;
    }
}
```

### 常用函数

#### add()

先从 ArrayList 最常用的方法 add() 开始讲起，add() 方法就是将元素添加到 elementData 的末尾。平均时间复杂度为 O(1)。

```java
//ArrayList.add()
public boolean add(E e) {
    //检查数组是否存放的下
    ensureCapacityInternal(size + 1); 
    //添加到末尾
    elementData[size++] = e;
    return true;
}

//ArrayList.ensureCapacityInternal()
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

//ArrayList.calculateCapacity()
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    //检查时不是默认时的空数组，是默认时的空数组，初始化的容量就是 10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

//ArrayList.ensureExplicitCapacity()
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

//最大容量
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//ArrayList.grow()
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    //新长度时原来长度的 1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //新长度小于实际容量，就用实际容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //新长度太大了，就用最大的容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //copy 成一个新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

1. 扩容的大小为原长度+1/2的原长度
2. 如果扩容长度比传入的最小容量小，则使用最小容量,如果扩容长度超过设定的最大容量,则实际用最大容量
3. 初始化默认长度为10,当添加到11个长度时,容量为15


#### add(int index, E element)

将元素插入到指定的位置，主要的操作是将指定位置之后的元素往后移动一个位置，空出 index 位置。平均时间复杂度为 O(n)。

```java
//ArrayList.add(int index, E element)
public void add(int index, E element) {
    //index的越界检查
    rangeCheckForAdd(index);
    //扩容
    ensureCapacityInternal(size + 1); 
    //将 index 之后的所有元素 copy 到往后挪动一位
    System.arraycopy(elementData, index, elementData, index + 1,
                        size - index);

    //将 index 位置放入新元素
    elementData[index] = element;
    //数量 + 1
    size++;
}
```

#### get()

get() 应该是第二个常用的函数，可以返回随机位置的元素。需要注意的是越界时，超过容量返回的是 IndexOutOfBoundsException 异常，index 太小返回的是 ArrayIndexOutOfBoundsException 异常。平均时间复杂度为 O(1)。

```java
//ArrayList.get()
public E get(int index) {
    //index 越界检查
    rangeCheck(index);
    //返回指定位置的元素
    return elementData(index);
}

//ArrayList.rangeCheck()
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
//ArrayList.elementData()
E elementData(int index) {
    return (E) elementData[index];
}
```

#### remove() 

删除指定位置的元素，并返回删除的元素。平均时间复杂度为 O(n)。

```java
//ArrayList.remove()
public E remove(int index) {
    //越界检查
    rangeCheck(index);

    modCount++;
    //取出元素
    E oldValue = elementData(index);
    //拷贝数组，将 index 之后的元素往前移动一个位置
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
    //最后一个元素置 null
    elementData[--size] = null; 

    return oldValue;
}
```

#### remove(Object o)

删除指定的元素，需要循环数组删除第一个指定的元素。平均时间复杂度为 O(n)。

```java
//ArrayList.remove(Object o)
public boolean remove(Object o) {
    if (o == null) {
        //循环整个数组，找出第一个需要删除的元素
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        //循环整个数组，找出第一个需要删除的元素
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

//ArrayList.fastRemove
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
    elementData[--size] = null; 
```

### 总结

![](http://www.javanorth.cn/assets/images/2021/ArrayList/0.png)

1. ArrayList 内部用一个数组存储元素，容量不够时会扩容原来的一半。
2. ArrayList 实现了 RandomAccess，支持随机访问。
3. ArrayList 实现了 Cloneable，支持被拷贝克隆。
4. ArrayList 删除指定元素和指定位置上的元素的效率不高，需要遍历数组。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！
