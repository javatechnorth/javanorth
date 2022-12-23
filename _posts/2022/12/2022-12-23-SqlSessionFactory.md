---
layout: post
title:  Mybatis核心类SqlSessionFactory，看完我悟了
tagline: by IT可乐
categories: mybatis
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
<!--more-->
请大家搬好小板凳，指北君将会用最通俗易懂，图文并茂的方式，给大家深入剖析 Mybatis 的实现原理。

本篇文章我们首先解析 SqlSessionFactory 的创建过程。

### 1、实例代码

在实例代码中，我们在测试类中写了一个 init() 方法，里面包括了 SqlSessionFactory 的构建，分为两步。

第一步：读取配置文件 mybatis-config.xml 输入流

第二步：根据输入流构建 SqlSessionFactory；

```java
public void init() {
    //定义mybatis全局配置文件
    String resource = "mybatis-config.xml";
    //加载 mybatis 全局配置文件
    InputStream inputStream = null;
    try {
        inputStream = Resources.getResourceAsStream(resource);
    } catch (IOException e) {
        e.printStackTrace();
    }
    //构建sqlSession的工厂
    sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

}
```



没什么难的，去掉 try-catch，也就两行代码。

```java
InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

是的，那只是你以为的两行代码，其实......

![](http://www.javanorth.cn/assets/images/2022/itcoke/mybatisplus/冰山一角.jpeg)

话不多说，指北君就来给大家揭秘这冰山下面的东西。


### 2、代码剖析

根据上面的时序图，我们分析根据源码分析每个步骤。

①、获取配置文件输入流

```java
InputStream inputStream = Resources.getResourceAsStream("mybatis.config.xml");
```

这里没什么好说的，就是获取配置文件的输入流。



②、build(in)

这里的 in 就是上一步获取的输入流 inputStream。

```java
  public SqlSessionFactory build(InputStream inputStream) {
    return build(inputStream, null, null);
  }
```

在进入到 build 方法：

```java
  public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        inputStream.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```



③、XMLConfigBuilder(in)

这一段代码是为了解析我们的配置文件，配置文件是 XML形式 ，我在之前的博客介绍过解析 XML 的几种方式。

**一种是基于树的结构来解析的称为DOM；另一种是基于事件流的形式称为SAX和（StAX）**

两者各有优缺点，我这里不做详细说明，想了解的可以看我之前的文章。

而 Mybatis 使用的是 DOM 形式，并结合 XPath 来解析配置文件。



④、parse()

```java
    public Configuration parse() {
        if (this.parsed) {
            throw new BuilderException("Each XMLConfigBuilder can only be used once.");
        } else {
            this.parsed = true;
            this.parseConfiguration(this.parser.evalNode("/configuration"));
            return this.configuration;
        }
    }
```

从 /configuration 标签处开始解析。然后我们进入到 this.parseConfiguration() 方法中：

```java
    private void parseConfiguration(XNode root) {
        try {
            this.propertiesElement(root.evalNode("properties"));
            Properties settings = this.settingsAsProperties(root.evalNode("settings"));
            this.loadCustomVfs(settings);
            this.loadCustomLogImpl(settings);
            this.typeAliasesElement(root.evalNode("typeAliases"));
            this.pluginElement(root.evalNode("plugins"));
            this.objectFactoryElement(root.evalNode("objectFactory"));
            this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
            this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
            this.settingsElement(settings);
            this.environmentsElement(root.evalNode("environments"));
            this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
            this.typeHandlerElement(root.evalNode("typeHandlers"));
            this.mapperElement(root.evalNode("mappers"));
        } catch (Exception var3) {
            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
        }
    }
```

看到这是不是很熟悉了，这不就是mybatis-config.xml 配置文件里面的各个标签名嘛，是的，这就是解析该文件，然后全部放在 configuration 对象中。需要注意的是，这里的 configuration 对象不仅包括 mybatis-config.xml 文件内容，也包括 xxxMapper.xml 文件内容。

![](http://www.javanorth.cn/assets/images/2022/itcoke/mybatisplus/原来如此.png)

![](http://www.javanorth.cn/assets/images/2022/itcoke/mybatisplus/mybatis03-01.png)



⑤、build(configuration)

```java
  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }

```

就是去 new 了一个 DefaultSqlSessionFactory 对象，将 configuration 作为参数。



⑥、DefaultSqlSessionFactory(configuration)

```java
    public DefaultSqlSessionFactory(Configuration configuration) {
        this.configuration = configuration;
    }
```



### 3、总结

自此，SqlSessionFactory 的创建过程就讲完了，总的来说就是一个封装了配置文件的工厂类。那么得到了 SqlSessionFactory 这个工厂对象，接下来干嘛？生产 SqlSession，然后通过 SqlSession 进行数据库的增删改查操作

没错，接下来，指北君将给大家介绍 SqlSession 的交互过程，这也是 Mybatis 里面最重要的一个对象。