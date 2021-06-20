---
layout: post
title:  不懂这个工具类，你还说你会Java?
tagline: by IT可乐
categories: JDK 源码解读
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。 
日常开发中，我们会使用各种工具类，利用封装好的轮子，能让我们的开发事半功倍。但是在JDK中，有一个特别的工具类——java.lang.Arrays.class，其源码实现还是挺精湛，接下来让我们来揭开它神秘的面纱。
<!--more-->
　　java.util.Arrays 类是 JDK 提供的一个工具类，用来处理数组的各种方法，而且每个方法基本上都是静态方法，能直接通过类名Arrays调用。
### 1、asList
```java
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```
　　作用是返回由指定数组支持的固定大小列表。

　　**注意**：这个方法返回的 ArrayList 不是我们常用的集合类 java.util.ArrayList。这里的 ArrayList 是 Arrays 的一个内部类 java.util.Arrays.ArrayList。这个内部类有如下属性和方法：  
　　
![](http://www.javanorth.cn/assets/images/2021/itcore/arrays-01-01.png)  

```java
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        private static final long serialVersionUID = -2764017481108945198L;
        private final E[] a;

        ArrayList(E[] array) {
            if (array==null)
                throw new NullPointerException();
            a = array;
        }

        public int size() {
            return a.length;
        }

        public Object[] toArray() {
            return a.clone();
        }

        public <T> T[] toArray(T[] a) {
            int size = size();
            if (a.length < size)
                return Arrays.copyOf(this.a, size,
                                     (Class<? extends T[]>) a.getClass());
            System.arraycopy(this.a, 0, a, 0, size);
            if (a.length > size)
                a[size] = null;
            return a;
        }

        public E get(int index) {
            return a[index];
        }

        public E set(int index, E element) {
            E oldValue = a[index];
            a[index] = element;
            return oldValue;
        }

        public int indexOf(Object o) {
            if (o==null) {
                for (int i=0; i<a.length; i++)
                    if (a[i]==null)
                        return i;
            } else {
                for (int i=0; i<a.length; i++)
                    if (o.equals(a[i]))
                        return i;
            }
            return -1;
        }

        public boolean contains(Object o) {
            return indexOf(o) != -1;
        }
}
```
　　**①、返回的 ArrayList 数组是一个定长列表，我们只能对其进行查看或者修改，但是不能进行添加或者删除操作**

　　通过源码我们发现该类是没有add()或者remove() 这样的方法的，如果对其进行增加或者删除操作，都会调用其父类 AbstractList 对应的方法，而追溯父类的方法最终会抛出 UnsupportedOperationException 异常。如下：
```
 String[] str = {"a","b","c"};
 List<String> listStr = Arrays.asList(str);
 listStr.set(1, "e");//可以进行修改
 System.out.println(listStr.toString());//[a, e, c]
 listStr.add("a");//添加元素会报错 java.lang.UnsupportedOperationException 
```

![](http://www.javanorth.cn/assets/images/2021/itcore/arrays-01-02.png)  

　　**②、引用类型的数组和基本类型的数组区别**
```
String[] str = {"a","b","c"};
List listStr = Arrays.asList(str);
System.out.println(listStr.size());//3

int[] i = {1,2,3};
List listI = Arrays.asList(i);
System.out.println(listI.size());//1
```
　　上面的结果第一个listStr.size()==3，而第二个 listI.size()==1。这是为什么呢？

　　我们看源码，在 Arrays.asList 中，方法声明为  <T> List<T> asList(T... a)。该方法接收一个可变参数，并且这个可变参数类型是作为泛型的参数。我们知道基本数据类型是不能作为泛型的参数的，但是数组是引用类型，所以数组是可以泛型化的，于是 int[] 作为了整个参数类型，而不是 int 作为参数类型。

　　所以将上面的方法泛型化补全应该是：
```java
String[] str = {"a","b","c"};
List<String> listStr = Arrays.asList(str);
System.out.println(listStr.size());//3

int[] i = {1,2,3};
List<int[]> listI = Arrays.asList(i);//注意这里List参数为 int[] ，而不是 int
System.out.println(listI.size());//1

Integer[] in = {1,2,3};
List<Integer> listIn = Arrays.asList(in);//这里参数为int的包装类Integer，所以集合长度为3
System.out.println(listIn.size());//3
```
　　**③、返回的列表ArrayList里面的元素都是引用，不是独立出来的对象**
```java
String[] str = {"a","b","c"};
List<String> listStr = Arrays.asList(str);
//执行更新操作前
System.out.println(Arrays.toString(str));//[a, b, c]
listStr.set(0, "d");//将第一个元素a改为d
//执行更新操作后
System.out.println(Arrays.toString(str));//[d, b, c]
```
　　这里的Arrays.toString()方法就是打印数组的内容，后面会介绍。我们看修改集合的内容，原数组的内容也变化了，所以这里传入的是引用类型。

　　**④、已知数组数据，如何快速获取一个可进行增删改查的列表List？**
```java
 String[] str = {"a","b","c"};
 List<String> listStr = new ArrayList<>(Arrays.asList(str));
 listStr.add("d");
 System.out.println(listStr.size());//4
```
　　这里的ArrayList 集合类后面我们会详细讲解，大家目前只需要知道有这种用法即可。

　　**⑤、Arrays.asList() 方法使用场景**

　　Arrays工具类提供了一个方法asList, 使用该方法可以将一个变长参数或者数组转换成List 。但是，生成的List的长度是固定的；能够进行修改操作（比如，修改某个位置的元素）；不能执行影响长度的操作（如add、remove等操作），否则会抛出UnsupportedOperationException异常。

　　所以 Arrays.asList 比较适合那些已经有数组数据或者一些元素，而需要快速构建一个List，只用于读取操作，而不进行添加或删除操作的场景。
### 2、sort
　　该方法是用于数组排序，在 Arrays 类中有该方法的一系列重载方法，能对7种基本数据类型，包括 byte,char,double,float,int,long,short 等都能进行排序，还有 Object 类型（实现了Comparable接口），以及比较器 Comparator 。
　　
![](http://www.javanorth.cn/assets/images/2021/itcore/arrays-01-03.png)    


　　**①、基本类型的数组**

　　这里我们以 int[ ] 为例看看：
```java
 int[] num = {1,3,8,5,2,4,6,7};
 Arrays.sort(num);
 System.out.println(Arrays.toString(num));//[1, 2, 3, 4, 5, 6, 7, 8]
```
　　通过调用 sort(int[] a) 方法，将原数组按照升序的顺序排列。下面我们通过源码看看是如何实现排序的：
```java
    public static void sort(int[] a) {
        DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
    }
```
　　在 Arrays.sort 方法内部调用 DualPivotQuicksort.sort 方法，这个方法的源码很长，分别对于数组的长度进行了各种算法的划分，包括快速排序，插入排序，冒泡排序都有使用。详细源码可以参考这篇博客。

　　**②、对象类型数组**

　　该类型的数组进行排序可以实现 Comparable 接口，重写 compareTo 方法进行排序。
```java
 String[] str = {"a","f","c","d"};
 Arrays.sort(str);
 System.out.println(Arrays.toString(str));//[a, c, d, f]
```
　　String 类型实现了 Comparable 接口，内部的 compareTo 方法是按照字典码进行比较的。

　　**③、没有实现Comparable接口的，可以通过Comparator实现排序**
```java
Person[] p = new Person[]{new Person("zhangsan",22),new Person("wangwu",11),new Person("lisi",33)};
Arrays.sort(p,new Comparator<Person>() {
    @Override
    public int compare(Person o1, Person o2) {
        if(o1 == null || o2 == null){
            return 0;
        }
        return o1.getPage()-o2.getPage();
    }
});    
System.out.println(Arrays.toString(p));
```
### 3、binarySearch
　　用二分法查找数组中的某个元素。该方法和 sort 方法一样，适用于各种基本数据类型以及对象。

　　注意：二分法是对以及有序的数组进行查找（比如先用Arrays.sort()进行排序，然后调用此方法进行查找）。找到元素返回下标，没有则返回 -1

　　实例：
```java
 int[] num = {1,3,8,5,2,4,6,7};
 Arrays.sort(num);
 System.out.println(Arrays.toString(num));//[1, 2, 3, 4, 5, 6, 7, 8]
 System.out.println(Arrays.binarySearch(num, 2));//返回元素的下标 1
```
　　具体源码实现：
```java
public static int binarySearch(int[] a, int key) {
        return binarySearch0(a, 0, a.length, key);
    }
    private static int binarySearch0(int[] a, int fromIndex, int toIndex,int key) {
        int low = fromIndex;
        int high = toIndex - 1;
        
        while (low <= high) {
            int mid = (low + high) >>> 1;//取中间值下标
            int midVal = a[mid];//取中间值
            
            if (midVal < key)
            low = mid + 1;
            else if (midVal > key)
            high = mid - 1;
            else
            return mid; 
        }
        return -(low + 1); 
}
```
### 4、copyOf
　　拷贝数组元素。底层采用 System.arraycopy() 实现，这是一个native方法。
```java
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```
　　src:源数组

　　srcPos:源数组要复制的起始位置

　　dest:目的数组

　　destPos:目的数组放置的起始位置

　　length:复制的长度

　　注意：src 和 dest都必须是同类型或者可以进行转换类型的数组。
```
int[] num1 = {1,2,3};
int[] num2 = new int[3];
System.arraycopy(num1, 0, num2, 0, num1.length);
System.out.println(Arrays.toString(num2));//[1, 2, 3]
```

```java
    /**
     * @param original 源数组
     * @param newLength //返回新数组的长度
     * @return
     */
    public static int[] copyOf(int[] original, int newLength) {
        int[] copy = new int[newLength];
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
### 5、equals 和 deepEquals
　　**①、equals**

　　equals 用来比较两个数组中对应位置的每个元素是否相等。  
　　
![](http://www.javanorth.cn/assets/images/2021/itcore/arrays-01-04.png)      

　　八种基本数据类型以及对象都能进行比较。

　　我们先看看 int类型的数组比较源码实现：
```java
public static boolean equals(int[] a, int[] a2) {
        if (a==a2)//数组引用相等，则里面的元素一定相等
            return true;
        if (a==null || a2==null)//两个数组其中一个为null，都返回false
            return false;

        int length = a.length;
        if (a2.length != length)//两个数组长度不等，返回false
            return false;

        for (int i=0; i<length; i++)//通过for循环依次比较数组中每个元素是否相等
            if (a[i] != a2[i])
                return false;

        return true;
}
```
　　在看对象数组的比较：
// todo
　　基本上也是通过 equals 来判断。

　　**②、deepEquals**

　　也是用来比较两个数组的元素是否相等，不过 deepEquals 能够进行比较多维数组，而且是任意层次的嵌套数组。
```java
         String[][] name1 = {{ "G","a","o" },{ "H","u","a","n"},{ "j","i","e"}};  
         String[][] name2 = {{ "G","a","o" },{ "H","u","a","n"},{ "j","i","e"}};
         System.out.println(Arrays.equals(name1,name2));// false  
         System.out.println(Arrays.deepEquals(name1,name2));// true
```
### 6、fill
　　该系列方法用于给数组赋值，并能指定某个范围赋值。
```java
    //给a数组所有元素赋值 val
    public static void fill(int[] a, int val) {
        for (int i = 0, len = a.length; i < len; i++)
            a[i] = val;
    }
    
    //给从 fromIndex 开始的下标，toIndex-1结尾的下标都赋值 val,左闭右开
    public static void fill(int[] a, int fromIndex, int toIndex, int val) {
        rangeCheck(a.length, fromIndex, toIndex);//判断范围是否合理
        for (int i = fromIndex; i < toIndex; i++)
            a[i] = val;
    }
```
### 7、toString 和 deepToString
　　toString 用来打印一维数组的元素，而 deepToString 用来打印多层次嵌套的数组元素。
```java
public static String toString(int[] a) {
        if (a == null)
            return "null";
        int iMax = a.length - 1;
        if (iMax == -1)
            return "[]";

        StringBuilder b = new StringBuilder();
        b.append('[');
        for (int i = 0; ; i++) {
            b.append(a[i]);
            if (i == iMax)
                return b.append(']').toString();
            b.append(", ");
        }
    }
```
### 小结
　　好了，这就是JDK中java.lang.Arrays 类的源码解析。
　　指北君后续的文章会给大家介绍JDK的各种源码，让大家吃透JDK，另外还有各种工作趣闻，面试宝典。  

　　我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！