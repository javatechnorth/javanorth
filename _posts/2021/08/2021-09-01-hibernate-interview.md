---
layout: post
title:  hibernate 面试题
tagline: by IT可乐
categories: hibernate 面试题
tags: 
    - IT可乐
---

### 1、什么是Hibernate，好处是什么？
 1）Hibernate是一个操作数据库的框架，实现了对JDBC的封装； 

 2）Hibernate是一个ORM（对象关系映射）框架，我们在写程序时 ，用的时面向对象的方法，但是在关系型数据库里，存的是一条条的数据，为了用纯面向对象的思想解决问题，所有需要将程序中的对象和数据库的记录建立起映射关系，ORM就是这样的技术，而Hibernate就是这样一个框架，以操作对象的方式操作数据库。 

 3）Hibernate简化了代码的编写，原生态的JDBC需要很对代码来实现操作数据库，在这里用Hibernate只需要很少的代码就可以实现。

 4）使用Hibernate的基本流程是：配置实现类与数据库表的映射关系，产生sessionFactory，打开session通道拿到session对象，开启事务，完成操作，关闭session。 

 5）Hibernate屏蔽了数据库的差异，增强了对数据库的可移植性。 

 6）使用Hibernate时，先要配置hibernate.cfg.xml文件，其中配置数据库连接信息和方言等，还要为每个实体配置相应的hbm.xml文件（Hibernate的映射文件），当然，也可以采用注解编程实现映射关系，hibernate.cfg.xml文件中需要登记每个hbm.xml文件

### 2、Hibernate执行流程
第一步:加载 hibernate 的配置文件，读取配置文件的参数(jdbc 连接参数，数据 库方言，hbm 表与对象关系映射文件)

第二步:创建 SessionFactory 会话工厂(内部有连接池)

第三步:打开 session 获取连接，构造 session 对象(一次会话维持一个数据连接， 也是一级缓存)

第四步:开启事务

第五步:进行操作

第六步:提交事务

第七步:关闭 session(会话)将连接释放

第八步:关闭连接池
 
### 3、为什么使用 hibernate？
 1）对JDBC访问数据库的代码做了封装，大大简化了数据访问层繁琐的重复性代码。 

 2）Hibernate是一个基于JDBC的主流持久化框架，是一个优秀的ORM实现。他很大程度的简化DAO层的编码工作 

 3）hibernate使用Java反射机制，而不是字节码增强程序来实现透明性。 

 4）hibernate的性能非常好，因为它是个轻量级框架。映射的灵活性很出色。它支持各种关系数据库，从一对一到多对多的各种复杂关系。

### 4、Hibernate 和 Mybatis 的区别？
两者相同点：

1）Hibernate 与 MyBatis 都可以是通过 SessionFactoryBuider 由 XML 配置文件生成 SessionFactory，然后由

SessionFactory 生成 Session，最后由 Session 来开启执行事务和 SQL 语句。其中 SessionFactoryBuider，

SessionFactory，Session 的生命周期都是差不多的。

2）Hibernate 和 MyBatis 都支持 JDBC 和 JTA 事务处理。

Mybatis 优势：

1）MyBatis 可以进行更为细致的 SQL 优化，可以减少查询字段。

2）MyBatis 容易掌握，而 Hibernate 门槛较高。

Hibernate 优势：

1）Hibernate 的 DAO 层开发比 MyBatis 简单，Mybatis 需要维护 SQL 和结果映射。

2）Hibernate 对对象的维护和缓存要比 MyBatis 好，对增删改查的对象的维护要方便。

3）Hibernate 数据库移植性很好，MyBatis 的数据库移植性不好，不同的数据库需要写不同 SQL。

4）Hibernate 有更好的二级缓存机制，可以使用第三方缓存。MyBatis 本身提供的缓存机制不佳。


### 5、Hibernate 和 JDBC 优缺点对比
相同点：

1）两者都是 java 数据库操作的中间件、

2）两者对数据库进行直接操作的对象都是线程不安全的，都需要及时关闭。

3）两者都可对数据库的更新操作进行显式的事务处理。

不同点：

JDBC 是 SUN 公司提供一套操作数据库的规范，使用 java 代码操作数据库。Hibernate 是一个基于 jdbc 的主流持久化框架，对 JDBC 访问数据库的代码做了封装。

使用的 SQL 语言不同：JDBC 使用的是基于关系型数据库的标准 SQL 语言，Hibernate 使用的是 HQL(Hibernatequery language)语言。

操作的对象不同：JDBC 操作的是数据，将数据通过 SQL 语句直接传送到数据库中执行，Hibernate 操作的是持久化对象，由底层持久化对象的数据更新到数据库中。

数据状态不同：JDBC 操作的数据是“瞬时”的，变量的值无法与数据库中的值保持一致，而 Hibernate 操作的数据是可持久的，即持久化对象的数据属性的值是可以跟数据库中的值保持一致的。

### 6、Hibernate是如何延迟加载?
当Hibernate在查询数据的时候，数据并没有存在与内存中，当程序真正对数据的操作时，对象才存在与内存中，就实现了延迟加载，他节省了服务器的内存开销，从而提高了服务器的性能。

### 7、hibernate 中对象的三种状态
①、瞬时态(临时态、自由态):不存在持久化标识OID，尚未与 Hibernate Session 关联对象， 被认为处于瞬时态，失去引用将被 JVM 回收  

②、持久态：存在持久化标识 OID，与当前 session 有关联，并且相关联的 session 没有关闭 ， 并且事务未提交  

③、脱管态(离线态、游离态):存在持久化标识 OID，但没有与当前 session 关联，脱管状态 改变 hibernate 不能检测到

### 8、Hibernate中怎样实现类之间的关系?(如：一对多、多对多的关系)
类与类之间的关系主要体现在表与表之间的关系进行操作，它们都是对对象进行操作，我们程序中把所有的表与类都映射在一起，它们通过配置文件中的many-to-one、one-to-many、many-to-many

### 9、get 和 load 的区别？
（1）get 是立即加载，load 是延时加载。

（2）get 会先查一级缓存，再查二级缓存，然后查数据库;load 会先查一级缓存，如果没有找到，就创建代理对象，等需要的时候去查询二级缓存和数据库。(这里就体现 load 的延迟加载的特性。)

（3）get 如果没有找到会返回 null，load 如果没有找到会抛出异常。

（4）当我们使用 session.load()方法来加载一个对象时，此时并不会发出 sql 语句，当前得到的这个对象其实是一个代理对象，这个代理对象只保存了实体对象的 id 值，只有当我们要使用这个对象，得到其它属性时，这个时候才会发出 sql 语句，从数据库中去查询我们的对象；相对于 load 的延迟加载方式，get 就直接的多，当我们使用session.get()方法来得到一个对象时，不管我们使不使用这个对象，此时都会发出 sql 语句去从数据库中查询出来。

### 10、Hibernate的缓存机制
一级缓存：

Hibenate中一级缓存，也叫做session的缓存，它可以在session范围内减少数据库的访问次数！ 只在session范围有效！ Session关闭，一级缓存失效！
只要是持久化对象状态的，都受Session管理，也就是说，都会在Session缓存中！
Session的缓存由hibernate维护，用户不能操作缓存内容； 如果想操作缓存内容，必须通过hibernate提供的evit/clear方法操作。
二级缓存：

二级缓存是基于应用程序的缓存，所有的Session都可以使用
Hibernate提供的二级缓存有默认的实现，且是一种可插配的缓存框架！如果用户想用二级缓存，只需要在hibernate.cfg.xml中配置即可； 不想用，直接移除，不影响代码。
如果用户觉得hibernate提供的框架框架不好用，自己可以换其他的缓存框架或自己实现缓存框架都可以。
Hibernate二级缓存：存储的是常用的类

### 11、Hibernate有哪几种查询数据的方式？
3种：hql、条件查询QBC(QueryBy Criteria)、原生sql （通过createSQLQuery建立）

### 12、什么是SessionFactory,她是线程安全么？
SessionFactory 是Hibrenate单例数据存储和线程安全的，以至于可以多线程同时访问。一个SessionFactory 在启动的时候只能建立一次。SessionFactory应该包装各种单例以至于它能很简单的在一个应用代码中储存

### 13、如何优化Hibernate？
 1）使用双向一对多关联，不使用单向一对多 

 2）灵活使用单向一对多关联 

 3）不用一对一，用多对一取代 

 4）配置对象缓存，不使用集合缓存 

 5）一对多集合使用Bag,多对多集合使用Set 

 6）继承类使用显式多态 

 7）表字段要少，表关联不要怕多，有二级缓存撑腰
 

### 14、在数据库中条件查询速度很慢的时候,如何优化?
 1）建索引 

 2）减少表之间的关联

 3）优化sql，尽量让sql很快定位数据，不要让sql做全表查询，应该走索引,把数据量大的表排在前面 

 4）简化查询字段，没用的字段不要，已经对返回结果的控制，尽量返回少量数据
 
### 15、谈谈Hibernate中inverse的作用
inverse属性默认是false,就是说关系的两端都来维护关系。

比如Student和Teacher是多对多关系，用一个中间表TeacherStudent维护。  
如果Student这边inverse=”true”, 那么关系由另一端Teacher维护，就是说当插入Student时，不会操作TeacherStudent表（中间表）。只有Teacher插入或删除时才会触发对中间表的操作。所以两边都inverse=”true”是不对的，会导致任何操作都不触发对中间表的影响；当两边都inverse=”false”或默认时，会导致在中间表中插入两次关系。
如果表之间的关联关系是“一对多”的话，那么inverse只能在“一”的一方来配置！

### 16、比较 Hibernate 三种检索策略的优缺点
1、立即检索

优点：对应用程序完全透明，不管对象处于持久化状态，还是游离状态，应用程序都可以方便的从一个对象导航到与它关联的对象；

缺点：1.select 语句太多；2.可能会加载应用程序不需要访问的对象白白浪费许多内存空间；

2、延迟检索

优点：由应用程序决定需要加载哪些对象，可以避免可执行多余的 select 语句，以及避免加载应用程序不需要访问的对象。因此能提高检索性能，并且能节省内存空间；

缺点：应用程序如果希望访问游离状态代理类实例，必须保证他在持久化状态时已经被初始化；

3、 迫切左外连接检索

优点：1、对应用程序完全透明，不管对象处于持久化状态，还是游离状态，应用程序都可以方便地冲一个对象导航到与它关联的对象。2、使用了外连接，select 语句数目少；

缺点：1、可能会加载应用程序不需要访问的对象，白白浪费许多内存空间；2、复杂的数据库表连接也会影响检索性能；

