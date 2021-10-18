---
layout: post
title: 软软猿妹问我JDK中眼花缭乱的Function/Consumer/Supplier/Predicate？
tagline: by 揽月中人
categories: Algorithm
tags:
- 揽月中人
---

JDK中有许多函数式接口，也会有许多方法会使用函数式接口作为参数，同时在各种源码中也大量使用了这些方法，那么我们在实际工作中应该如何使用！我们就来盘一盘，这样也有助于写出优雅的代码，使我们在阅读源码时事半功倍。

<!--more-->

### 1 JDK中常见的Lamada表达式

Java中可以使用Lamada表达式的接口都有@FunctionalInterface注解。

先来看看util.function包下面含有FunctionalInterface注解的接口。一屏显示不全，可见功能非常齐全。

鉴于常用的一些函数式接口有Function/Consumer/Supplier/Predicate以及Runnable等。本篇介绍这几类接口。

![image-20211010224457887](E:\javaNorth\javanorth\assets\images\2021\lyj\LamadaFunctionClass1.png)

![image-20211010224559121](E:\javaNorth\javanorth\assets\images\2021\lyj\LamadaFunctionClass2.png)

####  1.1 Runnable

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

 Runnable 使用Lamada方式书写时，无参数，无返回值。最终执行的是run方法

使用Demo

```java
new Thread(() -> {
    System.out.println("JavaNorth Runnable");
}).start();
```

#### 1.2 Function 

Function 表示接受一个参数并产生结果的函数。

```java
@FunctionalInterface
public interface Function<T, R> {
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

  Function<T,R>接受一个参数T，并且有返回值 R, 其实现也主要是实现此方法 R apply(T t);

Function 的一个示例:

```java
List<String> list = new ArrayList<String>();
List<String> collect = list.stream().map((x) -> {
    return "Java North Function" + x;
}).collect(Collectors.toList());
```

上述示例中是一个stream的map方法。其中x为输入参数，『"Java North and" + x』为输出。



#### 1.3 Consumer

  Consumer表示接受一个参数，没有返回值的操作，主要方法为 void accept(T t);

```java
@FunctionalInterface
public interface Consumer<T> {
    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

##### 1.3.1 Consumer 在Java中的应用

常见的有List的forEach等。

```java
list.forEach(x -> System.out.println( "Java North Consumer test " + x));
```

x为参数，输出语句直接执行。

下面的map的forEach参数为BiConsumer，其入参有两个。

```java
Map<String,String> map = new HashMap<>();
map.forEach((K,V) -> {
    System.out.println("Java North Big Consumer MAP key : " +K + " value: "+V );
});
```

##### 1.3.2 自动义带有Consumer的方法

```java
public class ConsumerBiConsumerDemo {
    public static void main(String[] args) {
        Consumer<String> conString = (x) -> System.out.println(x.toUpperCase());
        conString.accept("i love java north ");

        BiConsumer<String, String> biCon = (x,y) -> System.out.println (x + y);
        biCon.accept("i love ","java");

        List<Person> plist = Arrays.asList(new Person("Java"),new Person("North"));
        acceptAllEmployee(plist,p -> System.out.println(p.name));
        acceptAllEmployee(plist,person -> {person.name = "unknow";});
        acceptAllEmployee(plist,person -> System.out.println(person.name));
    }

    public static void acceptAllEmployee(List<Person> plist, Consumer<Person> con){
        plist.forEach(person -> {con.accept(person);});
    }
}
class Person{
    public String name;
    public Person (String name){
        this.name = name;
    }
}
```



#### 1.4 Supplier 

  Supplier没有参数，有一个返回值。

```
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

Supplier 通常会用在

```java
public class SupplierDemo {

    public static void main(String[] args) {
        SupplierDemo sdemo = new SupplierDemo();
        Supplier<LocalDateTime> s = () -> LocalDateTime.now();
        LocalDateTime localDateTime = s.get();
        System.out.println(s.get());

        Supplier<List> listSupplier = sdemo.listSupplier();
        List list = listSupplier.get();

        Person person = personFactory(Person::new);
        System.out.println(person);

        Person javaNorth = personFactory(() -> new Person("JavaNorth"));
        System.out.println(javaNorth);
    }

    public Supplier<List> listSupplier(){
        return ArrayList::new;
    }


    public static Person personFactory(Supplier<? extends Person> s){
        Person person = s.get();
        if(person.getName() == null)  person.setName("default");
        person.setAge(18);
        return person;
    }


    static class Person {
        String name;
        int age;
        public Person() {   }
        public Person(String name) {
            this.name = name;
        }
       。。。
    }
}
```



#### 1.5 Predicate

主要方法为test，其主要是传入一个参数，返回一个boolean类型的值。

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
	......
}
```

Predicate简单示例：

```java
public class PredicateDemo {
    public static void main(String[] args) {
        Predicate<Integer> predicate = (s) -> s > 5;
        Predicate<Integer> predicate2 = (s) -> s > 8;
        System.out.println(" 3 大于5 ？ " + predicate.test(3));
        System.out.println(" 6 大于5 ？ " + predicate.test(6));

        System.out.println("7 大于5 and 大于8 " + predicate.and(predicate2).test(7));
        System.out.println("7 大于5 or 大于8 " + predicate.or(predicate2).test(7));

        List<Integer> list = Arrays.asList(3,5,6,2,8,4,7,9);
        List<Integer> collect = list.stream().filter(predicate).collect(Collectors.toList());

        System.out.println(list);
        System.out.println(collect);
    }
}
```

上述代码运行结果

```javascript
 3 大于5 ？ false
 6 大于5 ？ true
7 大于5 and 大于8 false
7 大于5 or 大于8 true
[3, 5, 6, 2, 8, 4, 7, 9]
[6, 8, 7, 9]
```



### 2 常用的Lamada参数特征

Lamada 的一些表达式将方法的一些执行逻辑放到了参数中，使得方法的返回值根据传入的参数的逻辑而变化。从而实现了在一定的方法不变的情况下，使代码执行传入参数相关的逻辑。

常用的一些Lamada使用如下：



Runnable **无入参，无返回值**。 

```java
() -> { System.out.println(strF.apply("javaNorth Runnable"));}
```



Function **有入参，有返回值**

```java
Function strF = (s) -> { return s + "javaNorth Function"; };
System.out.println(strF.apply("TEST "));
```



Consumer有入参，无返回值。

```java
Consumer<String> srtC = (s) -> {System.out.println(s + "javaNorth TEST ");};
srtC.accept("Hello World");
```



Supplier **无入参，有返回值。**

```java
Supplier<String> srtP = () -> {return "I love avaNorth ";};
System.out.println(srtP.get());
```



Predicate **有入参，返回一个boolean类型的值**

```java
Predicate<Integer> predicate = (s) -> s > 5;
System.out.println(" 3 大于5 ？ " + predicate.test(3));
```



### 3 自定义Lamada函数式接口






### 总结