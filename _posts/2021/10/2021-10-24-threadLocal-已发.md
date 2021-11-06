---
layout: post
title:  ThreadLocal 的使用与源码——20211103
tagline: by 某某白米饭
categories: java
tags:
- 某某白米饭
---

ThreadLocal 是一个关于创建线程局部变量的类，这个变量只能当前线程使用，其他线程不可用。
ThreadLocal 提供 get()和 set()方法创建和修改变量。

<!--more-->

### 怎么使用

ThreadLocal 有三种使用方式：

``` java
ThreadLocal threadLocal = new ThreadLocal();
```
``` java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
```
``` java
ThreadLocal threadLocal = new ThreadLocal<String>() {
    @Override
    protected String initialValue() {
        return "初始化值";
    }
};
```


### get()，set()

查看 ThreadLocal 中的 get()，set() 中有一个 ThreadLocalMap 对象

``` java
//set 方法
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

//get 方法
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

### ThreadLocalMap

ThreadLocalMap 就是一个内部静态类，没有继承也没有接口，是一个自定义的 Hash 映射，用户维护线程局部变量。

``` java
static class ThreadLocalMap
```

#### ThreadLocalMap 的内部类 Entry，继承 WeakReference 弱引用

``` java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        //key 放在 WeakReference<ThreadLocal<?>>中
        super(k);
		  //变量放在 Object value 中
        value = v;
    }
}
```

#### ThreadLocalMap 中存放线程局部变量的数据结构

``` java
private Entry[] table;
```
#### 小结：

1. ThreadLocal ——> ThreadLocalMap——> Entry[]
2. Entry 维护一个 ThreadLocal 作为 key，value 对应 ThreadLocal 的值

### 初始化方法

``` java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
	 //默认容量为 16
    table = new Entry[INITIAL_CAPACITY];
	  //threadLocalHashCode 是一个原子类 AtomicInteger 的实例，每次调用会增加 0x61c88647。&位移操作使存放分布均匀
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
	  //放入数组
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}

//nextHashCode 实现
private final int threadLocalHashCode = nextHashCode();
private static AtomicInteger nextHashCode =
    new AtomicInteger();
private static final int HASH_INCREMENT = 0x61c88647;
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```
#### 小结：

1. ThreadLocalMap 默认容量为 16，每次计算索引位置会加 0x61c88647 然后和长度-1 取模
2. 索引是原子类

### Entry 的 get

``` java
private Entry getEntry(ThreadLocal<?> key) {
	  //定位 i 的位置
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;
	  //hashcode 索引相同所以查找下一个，用循环比对取出
    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

#### 小结：

1. get 方法中先计算索引位置，如果 key 相同则返回，不同则用线性探测法取出，当 key 为 null 的时候清理 i 所在位置直到不为 null 的数据。如果找不到 key 的数据则返回 null


### Entry 的 Set
``` java
	private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
	  //hashcode 索引
    int i = key.threadLocalHashCode & (len-1);

	  //线性探测法，如果在有值的情况下，key 不同则继续下一个
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
		  //如果当前有值&&key 相同则更新 value
        if (k == key) {
            e.value = value;
            return;
        }
		 //如果 key 空，则 key-value 重新替换
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
	 //索引位置找到，插入 key-value，对 size+1
    tab[i] = new Entry(key, value);
    int sz = ++size;
	  //cleanSomeSlots 清理 key 关联的对象被回收的数据,如果没有被清理的&&size 大于扩容因子，刷新
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
#### 小结：

1.计算索引位置
2.如果当前位置有值则索引+1 判断是否为空，不为空继续+1，直到找到位置插入
3.size+1
4.是否清理 key 为 null 的数据，如果没有被清理&& size 大于列表长度的 2/3 则扩容

### 清理 key 关联的对象被回收的数据

``` java
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
		  //key 为 null，被清理
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
			  //移除 i 位置之后为 key 为 null 的元素
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```
#### expungeStaleEntry 方法

``` java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
	  //将上面 staleSlot 的数据清空，大小减去 1
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    Entry e;
    int i;
	  //以 staleSlot 往后找 key 为 null 的
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
		  //key 为 null 清空
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
			  //key 不为 null，计算当前 hashCode 索引位置，如果不相同则把当前 i 清除，当前 h 位置不为 null，再向后查找 key 合适的索引
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```
#### 小结：

1. 从 staleSlot 开始，清除 key 为 null 的 Entry，并将不为空的元素放到合适的位置，最后遍历到 Entry 为空的元素时，跳出循环返回当前索引位置


### rehash 方法

``` java
private void rehash() {
    expungeStaleEntries(); //调用 expungeStaleEntries()方法
	  //size 的长度超过容量的 3/4，则扩容
    if (size >= threshold - threshold / 4)
        resize();
}

private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
			  //key 为 null，value 也设置为 null，清理
            if (k == null) {
                e.value = null; // Help the GC
            } else {
				  //重新设置元素位置
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }
	  //设置阈值
    setThreshold(newLen);
    size = count;
    table = newTab;
}

private void expungeStaleEntries() {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}

```
#### 小结：

1. 调用 expungeStaleEntries 方法，清理整个 table 中 key 为 null 的 Entry
2. 如果清理后 size 超过阈值的 1/2，则进行扩容。
3. 新表长度为老表 2 倍，创建新表。
4. 遍历老表所有元素，如果 key 为 null，将 value 清空；否则通过 hash code 计算新表的索引位置 h，如果 h 已经有元素，则调用 nextIndex 方法直到寻找到空位置，将元素放在新表的对应位置。
5. 设置新表扩容的阈值、更新 size、table 指向新表

### 缺点

#### 内存泄露

从 Entry 源码中可以看出，Entry 继承了 WeakReference 弱引用，如果外部没有引用 ThreadLocal，则 Entry 中作为 Key 的 ThreadLocal 会被销毁成为 null，那么它所对应的 value 不会被访问到。当线程一直在执行&&没有进行 remove，rehash 等操作时，value 会一直存在内存，从而造成内存泄露

### 总结

1. Thread 中都有一个 ThreadLocalMap
2. ThreadLocalMap 的 key 是 ThreadLocal 实例
3. 默认容量大小为 16，当 size 超过 2/3 容量&&没被清理就 rehash，
4. 当 size 超过扩容因子 3/4 的时候扩容为原来的 2 倍
5. 当发现一个 key 为 null 的时候，会进行清理，直到下一个 key 不为 null
6. has 冲突的解决方法和 hashMap 不相同，ThreadLocal 是找这个冲突索引的下一个元素直到找到，hashMap 是转换为红黑树

