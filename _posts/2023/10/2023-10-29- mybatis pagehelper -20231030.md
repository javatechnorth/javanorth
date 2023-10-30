---
layout: post
title: Pagehelper原理及注意事项
tagline: by 无花
categories: 
tags:
- 无花
---

哈喽，大家好，我是了不起。

Mybatis插件Pagehelper 很多人都会使用吧，这里我们一起看看其原理以及日常使用注意事项。

<!--more-->

### 如何使用PageHelper

​		PageHelper是Mybatis-Plus中的一个插件，主要用于实现数据库的分页查询功能。其核心原理是将传入的页码和条数赋值给一个Page对象，并保存到本地线程ThreadLocal中。接下来，PageHelper会进入Mybatis的拦截器环节，在拦截器中获取并处理刚才保存在ThreadLocal中的分页参数。这些分页参数会与原本的SQL语句和内部已经定义好的SQL进行拼接，从而完成带有分页处理的SQL语句的构建。

Pagehelper 有很多种使用方式，下面列出官方给出的几种使用方式。

```java
   //第一种，RowBounds方式的调用
    List<Country> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));

    //第二种，Mapper接口方式的调用，推荐这种使用方式。
    PageHelper.startPage(1, 10);
    List<Country> list = countryMapper.selectIf(1);

    //第三种，Mapper接口方式的调用，推荐这种使用方式。
    PageHelper.offsetPage(1, 10);
    List<Country> list = countryMapper.selectIf(1);

    //第四种，参数方法调用
    //存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
    public interface CountryMapper {
        List<Country> selectByPageNumSize(
                @Param("user") User user,
                @Param("pageNum") int pageNum,
                @Param("pageSize") int pageSize);
    }
    //配置supportMethodsArguments=true
    //在代码中直接调用：
    List<Country> list = countryMapper.selectByPageNumSize(user, 1, 10);

    //第五种，参数对象
    //如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
    //有如下 User 对象
    public class User {
        //其他fields
        //下面两个参数名和 params 配置的名字一致
        private Integer pageNum;
        private Integer pageSize;
    }
    //存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
    public interface CountryMapper {
        List<Country> selectByPageNumSize(User user);
    }
    //当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
    List<Country> list = countryMapper.selectByPageNumSize(user);

    //第六种，ISelect 接口方式
    //jdk6,7用法，创建接口
    Page<Country> page = PageHelper.startPage(1, 10).doSelectPage(new ISelect() {
        @Override
        public void doSelect() {
            countryMapper.selectGroupBy();
        }
    });
    //jdk8 lambda用法
    Page<Country> page = PageHelper.startPage(1, 10).doSelectPage(()-> countryMapper.selectGroupBy());

    //也可以直接返回PageInfo，注意doSelectPageInfo方法和doSelectPage
    pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(new ISelect() {
        @Override
        public void doSelect() {
            countryMapper.selectGroupBy();
        }
    });
    //对应的lambda用法
    pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(() -> countryMapper.selectGroupBy());

    //count查询，返回一个查询语句的count数
    long total = PageHelper.count(new ISelect() {
        @Override
        public void doSelect() {
            countryMapper.selectLike(country);
        }
    });
    //lambda
    total = PageHelper.count(()->countryMapper.selectLike(country));
```





### PageHelper原理

 其核心原理是将传入的页码和条数赋值给一个Page对象，并保存到本地线程ThreadLocal中。

下面以常见的使用方式看一下：

```java
PageHelper.startPage(1, 10, orderBy);
```

经过一些列的循环俄罗斯套娃调用之后，来到了这里：

```java

public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
        Page<E> page = new Page<E>(pageNum, pageSize, count);
        page.setReasonable(reasonable);
        page.setPageSizeZero(pageSizeZero);
        //当已经执行过orderBy的时候
        Page<E> oldPage = getLocalPage();
        if (oldPage != null && oldPage.isOrderByOnly()) {
            page.setOrderBy(oldPage.getOrderBy());
        }
        setLocalPage(page);
        return page;
    }
```



1. 重点在setLocalPage(page) 这个方法，将page对象放到了静态变量ThreadLocal中。

![image-20231029212116015](http://www.javanorth.cn/assets/images/2023/Flowerless/10-29-01-pagehelper .png)



2. 查询接口进来的时候， PageHelper中的Threadlocal对象中就保存的该线程对应的分页参数，在调用查询的时候就会拿出来使用。

PageHelper 中有写了一个 com.github.pagehelper.PageInterceptor ， 这里是执行分页的地方。如果你想要些其他的拦截器，也可以自定义一个拦截器，在这里对sql进行处理。

![image-20231029213320273](http://www.javanorth.cn/assets/images/2023/Flowerless/10-29-02-pagehelper .png)



3. 在PageInterceptor  中有一个主要interceptor方法，在方法中需要判断是否需要分页，如果需要分页，则获取分页信息，查询数据量等。

![image-20231029214002441](http://www.javanorth.cn/assets/images/2023/Flowerless/10-29-03-pagehelper .png)





4. 接下来，PageHelper会进入Mybatis的拦截器环节，在拦截器中获取并处理刚才保存在ThreadLocal中的分页参数。这些分页参数会与原本的SQL语句和内部已经定义好的SQL进行拼接，从而完成带有分页处理的SQL语句的构建。需要注意一点的是在finally中remove掉 ThreadLocal对象中当前线程的page对象。

![image-20231029214349011](http://www.javanorth.cn/assets/images/2023/Flowerless/10-29-04-pagehelper .png)



5. 调用skip方法， 并获取分页参数 判断是否需要分页。

![image-20231029214551012](http://www.javanorth.cn/assets/images/2023/Flowerless/10-29-05-pagehelper .png)



6. 从静态ThreadLocal中获取page对象，

![image-20231029214726665](http://www.javanorth.cn/assets/images/2023/Flowerless/10-29-06-pagehelper .png)



PageHelper在执行这一过程时，会判断SQL的类型，只有当该SQL是查询操作时，才会进入分页逻辑。并且，在进入分页逻辑处理后，PageHelper会通过反射获取该方法的参数，判断是否存在IPage对象（这是Mybatis-Plus中的另一个分页对象）的实现类。如果存在这样的实现类，那么也会进行相应的分页处理。



### PageHelper注意事项

使用pagehelper进行分页的时候推荐使用  PageHelper.startPage(1, 10); 这种方式；

另外startPage语句最好紧挨着查询语句，避免中间抛出异常，没有办法清除ThreadLocal中当前线程的page对象。

下面看一下官方给出的不安全的用法 和推荐的例子：

不安全用法： 下面的方法如果不走查询的话，page对象就会保留在当前线程中，算是一个内存泄漏，如果下一个mybatis查询方法刚好是这个线程的时候，就会被动分页。

```java
PageHelper.startPage(1, 10);
List<Country> list;
if(param1 != null){
    list = countryMapper.selectIf(param1);
} else {
    list = new ArrayList<Country>();
}
```



推荐用法，主要是紧挨着mybatis查询方法，查询结束后，page对象就被清理，对线程后续的运行不受影响。

```java
List<Country> list;
if(param1 != null){
    PageHelper.startPage(1, 10);
    list = countryMapper.selectIf(param1);
} else {
    list = new ArrayList<Country>();
}
```



