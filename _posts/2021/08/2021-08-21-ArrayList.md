---
layout: post
title:  ArrayList 集合类，你知多少？
tagline: by IT可乐
categories: JDK 源码解读
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
说到集合类，java.utils.ArrayList类可能是大家日常用的最多的类，这个容器，可能对于很多Java Coder 来说，这个集合可以一把梭，但是对于它是怎么实现的，你真的明白吗？不知道不要紧，善解人意的指北君写下了这篇文章，包你一看就明白了。
<!--more-->


### 1、ArrayList 定义
　　ArrayList 是一个用数组实现的集合，支持随机访问，元素有序且可以重复。
```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

![](http://www.javanorth.cn/assets/images/2021/itcore/arraylist-01-00.png)
　　**①、实现 RandomAccess 接口**

　　这是一个标记接口，一般此标记接口用于 List 实现，以表明它们支持快速（通常是恒定时间）的随机访问。该接口的主要目的是允许通用算法改变其行为，以便在应用于随机或顺序访问列表时提供良好的性能。

　　比如在工具类 Collections(这个工具类后面会详细讲解)中，应用二分查找方法时判断是否实现了 RandomAccess 接口：
```
int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```
　　**②、实现 Cloneable 接口**

　　这个类是 java.lang.Cloneable，前面我们讲解深拷贝和浅拷贝的原理时，我们介绍了浅拷贝可以通过调用 Object.clone() 方法来实现，但是调用该方法的对象必须要实现 Cloneable 接口，否则会抛出 CloneNoSupportException异常。

　　Cloneable 和 RandomAccess 接口一样也是一个标记接口，接口内无任何方法体和常量的声明，也就是说如果想克隆对象，必须要实现 Cloneable 接口，表明该类是可以被克隆的。

　　**③、实现 Serializable 接口**

　　这个没什么好说的，也是标记接口，表示能被序列化。

　　**④、实现 List 接口**

　　这个接口是 List 类集合的上层接口，定义了实现该接口的类都必须要实现的一组方法，如下所示，下面我们会对这一系列方法的实现做详细介绍。

![](http://www.javanorth.cn/assets/images/2021/itcore/arraylist-01-01.png)
### 2、字段属性
```
//集合的默认大小
        private static final int DEFAULT_CAPACITY = 10;
        //空的数组实例
        private static final Object[] EMPTY_ELEMENTDATA = {};
        //这也是一个空的数组实例，和EMPTY_ELEMENTDATA空数组相比是用于了解添加元素时数组膨胀多少
        private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
        //存储 ArrayList集合的元素，集合的长度即这个数组的长度
        //1、当 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时将会清空 ArrayList
        //2、当添加第一个元素时，elementData 长度会扩展为 DEFAULT_CAPACITY=10
        transient Object[] elementData;
        //表示集合的长度
        private int size;
```
### 3、构造函数
```
     public ArrayList() {
         this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
     }
```
　　此无参构造函数将创建一个 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 声明的数组，注意此时初始容量是0，而不是大家以为的 10。

　　**注意**：根据默认构造函数创建的集合，ArrayList list = new ArrayList();此时集合长度是0.
```
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
　　初始化集合大小创建 ArrayList 集合。当大于0时，给定多少那就创建多大的数组；当等于0时，创建一个空数组；当小于0时，抛出异常。
```
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
　　这是将已有的集合复制到 ArrayList 集合中去。
### 4、添加元素
　　通过前面的字段属性和构造函数，我们知道 ArrayList 集合是由数组构成的，那么向 ArrayList 中添加元素，也就是向数组赋值。我们知道一个数组的声明是能确定大小的，而使用 ArrayList 时，好像是能添加任意多个元素，这就涉及到数组的扩容。

　　扩容的核心方法就是调用前面我们讲过的Arrays.copyOf 方法，创建一个更大的数组，然后将原数组元素拷贝过去即可。下面我们看看具体实现：
```
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  //添加元素之前，首先要确定集合的大小
        elementData[size++] = e;
        return true;
    }
```
　　如上所示，在通过调用 add 方法添加元素之前，我们要首先调用 ensureCapacityInternal 方法来确定集合的大小，如果集合满了，则要进行扩容操作。
```
private void ensureCapacityInternal(int minCapacity) {//这里的minCapacity 是集合当前大小+1
        //elementData 是实际用来存储元素的数组，注意数组的大小和集合的大小不是相等的，前面的size是指集合大小
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {//如果数组为空，则从size+1的值和默认值10中取最大的
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;//不为空，则返回size+1
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```
　　在 ensureExplicitCapacity 方法中，首先对修改次数modCount加一，这里的modCount给ArrayList的迭代器使用的，在并发操作被修改时，提供快速失败行为（保证modCount在迭代期间不变，否则抛出ConcurrentModificationException异常，可以查看源码865行），接着判断minCapacity是否大于当前ArrayList内部数组长度，大于的话调用grow方法对内部数组elementData扩容，grow方法代码如下：
```
private void grow(int minCapacity) {
        int oldCapacity = elementData.length;//得到原始数组的长度
        int newCapacity = oldCapacity + (oldCapacity >> 1);//新数组的长度等于原数组长度的1.5倍
        if (newCapacity - minCapacity < 0)//当新数组长度仍然比minCapacity小，则为保证最小长度，新数组等于minCapacity
            newCapacity = minCapacity;
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 = 2147483639
        if (newCapacity - MAX_ARRAY_SIZE > 0)//当得到的新数组长度比 MAX_ARRAY_SIZE 大时，调用 hugeCapacity 处理大数组
            newCapacity = hugeCapacity(minCapacity);
        //调用 Arrays.copyOf 将原数组拷贝到一个大小为newCapacity的新数组（注意是拷贝引用）
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // 
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ? //minCapacity > MAX_ARRAY_SIZE,则新数组大小为Integer.MAX_VALUE
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
　　对于 ArrayList 集合添加元素，我们总结一下：

　　①、当通过 ArrayList() 构造一个空集合，初始长度是为0的，第 1 次添加元素，会创建一个长度为10的数组，并将该元素赋值到数组的第一个位置。

　　②、第 2 次添加元素，集合不为空，而且由于集合的长度size+1是小于数组的长度10，所以直接添加元素到数组的第二个位置，不用扩容。

　　③、第 11 次添加元素，此时 size+1 = 11，而数组长度是10，这时候创建一个长度为10+10*0.5 = 15 的数组（扩容1.5倍），然后将原数组元素引用拷贝到新数组。并将第 11 次添加的元素赋值到新数组下标为10的位置。

　　④、第 Integer.MAX_VALUE - 8 = 2147483639，然后 2147483639%1.5=1431655759（这个数是要进行扩容） 次添加元素，为了防止溢出，此时会直接创建一个 1431655759+1 大小的数组，这样一直，每次添加一个元素，都只扩大一个范围。

　　⑤、第 Integer.MAX_VALUE - 7 次添加元素时，创建一个大小为 Integer.MAX_VALUE 的数组，在进行元素添加。

　　⑥、第 Integer.MAX_VALUE + 1 次添加元素时，抛出 OutOfMemoryError 异常。

　　注意：能向集合中添加 null 的，因为数组可以有 null 值存在。
```
Object[] obj = {null,1};

ArrayList list = new ArrayList();
list.add(null);
list.add(1);
System.out.println(list.size());//2
```
### 5、删除元素
　　**①、根据索引删除元素**
```
public E remove(int index) {
        rangeCheck(index);//判断给定索引的范围，超过集合大小则抛出异常

        modCount++;
        E oldValue = elementData(index);//得到索引处的删除元素

        int numMoved = size - index - 1;
        if (numMoved > 0)//size-index-1 > 0 表示 0<= index < (size-1),即索引不是最后一个元素
            //通过 System.arraycopy()将数组elementData 的下标index+1之后长度为 numMoved的元素拷贝到从index开始的位置
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; //将数组最后一个元素置为 null，便于垃圾回收

        return oldValue;
    }
```
　　remove(int index) 方法表示删除索引index处的元素，首先通过 rangeCheck(index) 方法判断给定索引的范围，超过集合大小则抛出异常；接着通过 System.arraycopy 方法对数组进行自身拷贝。关于这个方法的用法可以参考这篇博客。

　　**②、直接删除指定元素**
```
public boolean remove(Object o) {
        if (o == null) {//如果删除的元素为null
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {//不为null，通过equals方法判断对象是否相等
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }


    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // 
    }
```
　　remove(Object o)方法是删除第一次出现的该元素。然后通过System.arraycopy进行数组自身拷贝。
### 6、修改元素
　　通过调用 set(int index, E element) 方法在指定索引 index 处的元素替换为 element。并返回原数组的元素。
```
public E set(int index, E element) {
        rangeCheck(index);//判断索引合法性

        E oldValue = elementData(index);//获得原数组指定索引的元素
        elementData[index] = element;//将指定所引处的元素替换为 element
        return oldValue;//返回原数组索引元素
    }
```
　　通过调用 rangeCheck(index) 来检查索引合法性。
```
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```
　　当索引为负数时，会抛出 java.lang.ArrayIndexOutOfBoundsException 异常。当索引大于集合长度时，会抛出 IndexOutOfBoundsException 异常。
### 7、查找元素
　　**①、根据索引查找元素**
```
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```
　　同理，首先还是判断给定索引的合理性，然后直接返回处于该下标位置的数组元素。

　　**②、根据元素查找索引**
```
public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
　　**注意**：indexOf(Object o) 方法是返回第一次出现该元素的下标，如果没有则返回 -1。

　　还有 lastIndexOf(Object o) 方法是返回最后一次出现该元素的下标。
### 8、遍历集合
　　**①、普通 for 循环遍历**
　　前面我们介绍查找元素时，知道可以通过get(int index)方法，根据索引查找元素，那么遍历同理：
```
ArrayList list = new ArrayList();
list.add("a");
list.add("b");
list.add("c");
for(int i = 0 ; i < list.size() ; i++){
    System.out.print(list.get(i)+" ");
}
```
　　**②、迭代器 iterator**
　　先看看具体用法：
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String str = it.next();
    System.out.print(str+" ");
}
```
　　在介绍 ArrayList 时，我们知道该类实现了 List 接口，而 List 接口又继承了 Collection 接口，Collection 接口又继承了 Iterable 接口，该接口有个 Iterator<T> iterator() 方法，能获取 Iterator 对象，能用该对象进行集合遍历，为什么能用该对象进行集合遍历？我们再看看 ArrayList 类中的该方法实现：
```
     public Iterator<E> iterator() {
         return new Itr();
     }
```
　　该方法是返回一个 Itr 对象，这个类是 ArrayList 的内部类。
```
private class Itr implements Iterator<E> {
        int cursor;       //游标， 下一个要返回的元素的索引
        int lastRet = -1; // 返回最后一个元素的索引; 如果没有这样的话返回-1.
        int expectedModCount = modCount;

        //通过 cursor ！= size 判断是否还有下一个元素
        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();//迭代器进行元素迭代时同时进行增加和删除操作，会抛出异常
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;//游标向后移动一位
            return (E) elementData[lastRet = i];//返回索引为i处的元素，并将 lastRet赋值为i
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);//调用ArrayList的remove方法删除元素
                cursor = lastRet;//游标指向删除元素的位置，本来是lastRet+1的，这里删除一个元素，然后游标就不变了
                lastRet = -1;//lastRet恢复默认值-1
                expectedModCount = modCount;//expectedModCount值和modCount同步，因为进行add和remove操作，modCount会加1
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {//便于进行forEach循环
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        //前面在新增元素add() 和 删除元素 remove() 时，我们可以看到 modCount++。修改set() 是没有的
        //也就是说不能在迭代器进行元素迭代时进行增加和删除操作，否则抛出异常
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```
　　注意在进行 next() 方法调用的时候，会进行 checkForComodification() 调用，该方法表示迭代器进行元素迭代时，如果同时进行增加和删除操作，会抛出 ConcurrentModificationException 异常。比如：
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String str = it.next();
    System.out.print(str+" ");
    list.remove(str);//集合遍历时进行删除或者新增操作，都会抛出 ConcurrentModificationException 异常
    //list.add(str);
    list.set(0, str);//修改操作不会造成异常
}
```

![](http://www.javanorth.cn/assets/images/2021/itcore/arraylist-01-02.png)
　　解决办法是不调用 ArrayList.remove() 方法，转而调用 迭代器的 remove() 方法：
```
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String str = it.next();
    System.out.print(str+" ");
    //list.remove(str);//集合遍历时进行删除或者新增操作，都会抛出 ConcurrentModificationException 异常
    it.remove();
}
```
　　**注意**：迭代器只能向后遍历，不能向前遍历，能够删除元素，但是不能新增元素。

　　**③、迭代器的变种 forEach**
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
for(String str : list){
    System.out.print(str + " ");
}
```
　　这种语法可以看成是 JDK 的一种语法糖，通过反编译 class 文件，我们可以看到生成的 java 文件，其具体实现还是通过调用 Iterator 迭代器进行遍历的。如下：
```
ArrayList list = new ArrayList();
        list.add("a");
        list.add("b");
        list.add("c");
        String str;
        for (Iterator iterator1 = list.iterator(); iterator1.hasNext(); System.out.print((new StringBuilder(String.valueOf(str))).append(" ").toString()))
            str = (String)iterator1.next();
```
　　**④、迭代器 ListIterator**
　　还是先看看具体用法：
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
ListIterator<String> listIt = list.listIterator();

//向后遍历
while(listIt.hasNext()){
    System.out.print(listIt.next()+" ");//a b c
}

//向后前遍历,此时由于上面进行了向后遍历，游标已经指向了最后一个元素，所以此处向前遍历能有值
while(listIt.hasPrevious()){
    System.out.print(listIt.previous()+" ");//c b a
}
```
　　还能一边遍历，一边进行新增或者删除操作：
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
ListIterator<String> listIt = list.listIterator();

//向后遍历
while(listIt.hasNext()){
    System.out.print(listIt.next()+" ");//a b c
    listIt.add("1");//在每一个元素后面增加一个元素 "1"
}

//向后前遍历,此时由于上面进行了向后遍历，游标已经指向了最后一个元素，所以此处向前遍历能有值
while(listIt.hasPrevious()){
    System.out.print(listIt.previous()+" ");//1 c 1 b 1 a 
}
```
　　也就是说相比于 Iterator 迭代器，这里的 ListIterator 多出了能向前迭代，以及能够新增元素。下面我们看看具体实现：

　　对于  Iterator 迭代器，我们查看 JDK 源码，发现还有 ListIterator 接口继承了 Iterator:
> public interface ListIterator<E> extends Iterator<E> 

　　该接口有如下方法：

![](http://www.javanorth.cn/assets/images/2021/itcore/arraylist-01-03.png)
　　我们看在 ArrayList 类中，有如下方法可以获得 ListIterator 接口：
```
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
```
　　这里的 ListItr 也是一个内部类。
```
//注意 内部类 ListItr 继承了另一个内部类 Itr
    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {//构造函数，进行游标初始化
            super();
            cursor = index;
        }

        public boolean hasPrevious() {//判断是否有上一个元素
            return cursor != 0;
        }

        public int nextIndex() {//返回下一个元素的索引
            return cursor;
        }

        public int previousIndex() {//返回上一个元素的索引
            return cursor - 1;
        }

        //该方法获取当前索引的上一个元素
        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();//迭代器进行元素迭代时同时进行增加和删除操作，会抛出异常
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;//游标指向上一个元素
            return (E) elementData[lastRet = i];//返回上一个元素的值
        }

        
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        //相比于迭代器 Iterator ，这里多了一个新增操作
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```
 ### 9、SubList
　　在 ArrayList 中有这样一个方法：
```
     public List<E> subList(int fromIndex, int toIndex) {
         subListRangeCheck(fromIndex, toIndex, size);
         return new SubList(this, 0, fromIndex, toIndex);
     }
```
　　作用是返回从 fromIndex(包括) 开始的下标，到 toIndex(不包括) 结束的下标之间的元素视图。如下：
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");

List<String> subList = list.subList(0, 1);
for(String str : subList){
    System.out.print(str + " ");//a
}
```
　　这里出现了 SubList 类，这也是 ArrayList 中的一个内部类。

　　注意：返回的是原集合的视图，也就是说，如果对 subList 出来的集合进行修改或新增操作，那么原始集合也会发生同样的操作。
```
ArrayList<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");

List<String> subList = list.subList(0, 1);
for(String str : subList){
    System.out.print(str + " ");//a
}
subList.add("d");
System.out.println(subList.size());//2
System.out.println(list.size());//4,原始集合长度也增加了
```
　　想要独立出来一个集合，解决办法如下：
> List<String> subList = new ArrayList<>(list.subList(0, 1));

### 10、size()
```
    public int size() {
        return size;
    }
```
　　注意：返回集合的长度，而不是数组的长度，这里的 size 就是定义的全局变量。
### 11、isEmpty()
```
     public boolean isEmpty() {
         return size == 0;
     }
```
　　返回 size == 0 的结果。
### 12、trimToSize()
```
public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = Arrays.copyOf(elementData, size);
        }
    }
```
　　该方法用于回收多余的内存。也就是说一旦我们确定集合不在添加多余的元素之后，调用 trimToSize() 方法会将实现集合的数组大小刚好调整为集合元素的大小。

　　**注意**：该方法会花时间来复制数组元素，所以应该在确定不会添加元素之后在调用。

### 13、小结
　　好了，这就是JDK中java.util.ArrayList 类的介绍。

　　我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！