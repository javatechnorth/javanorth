---
layout: post
title: List 转 Map， 齐活了
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

在我们平时的工作中，充满了各种类型之间的转换。今天指北君带大家上手 List 转 Map 的各种操作。

我们将假设 List 中的每个元素都有一个标识符，该标识符将在生成的 Map 中作为一个键使用。

<!--more-->

### 定义一个类型

我们在转换之前，我们先暂定一个类来用于各种转换demo的演示。

```java
public class Animal {
    private int id;
    private String name;

    //  构造函数 、 get 、 set
}
```

我们假定 id 字段 是唯一的， 所以我们把 id 作为 Map 的key。

### 使用 Java 8 之前的方法

在使用Java 8 之前，就只能使用比较传统的for 循环来转换。

```java
public Map<Integer, Animal> convertListBeforeJava8(List<Animal> list) {
    Map<Integer, Animal> map = new HashMap<>();
    for (Animal animal : list) {
        map.put(animal.getId(), animal);
    }
    return map;
}
```

我们需要写一个测试代码，测试下是否正常运行了。

```java
@Test
public void testConvertListBeforeJava8() {
    Map<Integer, Animal> map = convertListService
      .convertListBeforeJava8(list);
    
    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```

### 使用Java 8 stream

在Java 8 之后，我们可以通过新增的 Stream API 来进行转换操作

```java
public Map<Integer, Animal> convertListAfterJava8(List<Animal> list) {
    Map<Integer, Animal> map = list.stream()
      .collect(Collectors.toMap(Animal::getId, Function.identity()));
    return map;
}
```

测试代码

```java
@Test
public void testConvertListAfterJava8() {
    Map<Integer, Animal> map = convertListService.convertListAfterJava8(list);
    
    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```

### 使用Guava库

除了使用核心的Java API ，我们还能通过第三方库来实现这些操作。

使用Guava 库， 我们需要先引入依赖， 我们先在maven 中引入进来。

```xml
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

接下来使用 Maps.uniqueIndex() 进行转换

```java
public Map<Integer, Animal> convertListWithGuava(List<Animal> list) {
    Map<Integer, Animal> map = Maps
      .uniqueIndex(list, Animal::getId);
    return map;
}
```

测试代码

```java
@Test
public void testConvertListWithGuava() {
    Map<Integer, Animal> map = convertListService
      .convertListWithGuava(list);
    
    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```
### 使用 Apache Commons 库

除了 Guava ，我们还可以使用常用的 Apache Commons 库来进行转换。

我们现在Maven 中引入 commons 的依赖库

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-collections4 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

接下来我们使用 MapUtils.populateMap() 方法进行转换。

```java
public Map<Integer, Animal> convertListWithApacheCommons2(List<Animal> list) {
    Map<Integer, Animal> map = new HashMap<>();
    MapUtils.populateMap(map, list, Animal::getId);
    return map;
}
```

测试代码

```java
@Test
public void testConvertListWithApacheCommons2() {
    Map<Integer, Animal> map = convertListService
      .convertListWithApacheCommons2(list);
    
    assertThat(
      map.values(), 
      containsInAnyOrder(list.toArray()));
}
```

### Map Key 的冲突问题
由于List中可以存在多个相同的实例， 但是map却不行， 那我们来看看Map要怎么处理呢？

首先，我们初始化一个有重复对象的 List

```java
@Before
public void init() {

    this.duplicatedIdList = new ArrayList<>();

    Animal cat = new Animal(1, "Cat");
    duplicatedIdList.add(cat);
    Animal dog = new Animal(2, "Dog");
    duplicatedIdList.add(dog);
    Animal pig = new Animal(3, "Pig");
    duplicatedIdList.add(pig);
    Animal cow = new Animal(4, "牛");
    duplicatedIdList.add(cow);
    Animal goat= new Animal(4, "羊");
    duplicatedIdList.add(goat);
}
```

从代码中可以看到， 牛 和 羊 对象的id 都是 4 。

Apache Commons 和 Java 8 之前的代码是一样的，相同id的Map 在put 的时候会进行覆盖。

```java
@Test
public void testConvertBeforeJava8() {

    Map<Integer, Animal> map = convertListService
      .convertListBeforeJava8(duplicatedIdList);

    assertThat(map.values(), hasSize(4));
    assertThat(map.values(), hasItem(duplicatedIdList.get(4)));
}

@Test
public void testConvertWithApacheCommons() {

    Map<Integer, Animal> map = convertListService
      .convertListWithApacheCommons(duplicatedIdList);

    assertThat(map.values(), hasSize(4));
    assertThat(map.values(), hasItem(duplicatedIdList.get(4)));
}
```

而 Java 8 的 Collectors.toMap() 和 Guava 的 MapUtils.populateMap() 分别抛出 IllegalStateException 和 IllegalArgumentException。

```java
@Test(expected = IllegalStateException.class)
public void testGivenADupIdListConvertAfterJava8() {

    convertListService.convertListAfterJava8(duplicatedIdList);
}

@Test(expected = IllegalArgumentException.class)
public void testGivenADupIdListConvertWithGuava() {

    convertListService.convertListWithGuava(duplicatedIdList);
}
```

### 总结

在这篇文章中，指北君给大家分享了各种List 转 Map 的方法， 给出了使用 Java 原生API 以及一些流行的第三方库的例子。