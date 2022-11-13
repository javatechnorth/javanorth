---
title: 代码简洁之道：对象转换神器MapStruct -- 2022-09-02
date: 2022-09-02 08:15:00
author: gotanks广楠
categories: gotanks广楠
tags: ["MapStruct", "规范", "gotanks广楠"]
---

> 其实很多时候, 写代码更像是在创造艺术。 —— 指北君

今天指北君再分享一个让代码变得更简洁的小技巧。

---

<!--more-->

### 0. 前言
在我们日常开发的程序中，为了各层之间解耦，一般会定义不同的对象用来在不同层之间传递数据，比如xxxDTO、xxxVO、xxxQO，当在不同层之间传输数据时，不可避免地经常需要将这些对象进行相互转换。

今天给大家介绍一个对象转换工具MapStruct，代码简洁安全、性能高，强烈推荐。


### MapStruct简介

MapStruct是一个代码生成器，它基于约定优于配置，极大地简化了Java Bean类型之间映射的实现。特点如下：

1. 基于注解
2. 在编译期自动生成映射转换代码
3. 类型安全、高性能、无依赖性、易于理解阅读


### MapStruct入门

##### 1. 引入依赖

这里使用Gradle构建

```java
dependencies {
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
}
```

##### 2. 需要转换的对象

创建两个示例对象(eg.将Demo对象转换为DemoDto对象)

```java
/**
 * 源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private Integer id;
    private String name;
}
```

##### 3. 创建转换器

只需要创建一个转换器接口类，并在类上添加 @Mapper 注解即可（官方示例推荐以 xxxMapper 格式命名转换器名称）

```java
@Mapper
public interface DemoMapper {
    //使用Mappers工厂获取DemoMapper实现类
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);
    //定义接口方法，参数为来源对象，返回值为目标对象
    DemoDto toDemoDto(Demo demo);
}
```


##### 4. 验证

```java
public static void main(String[] args) {
    Demo demo = new Demo();
    demo.setId(111);
    demo.setName("hello");

    DemoDto demoDto = DemoMapper.INSTANCE.toDemoDto(demo);

    System.out.println("目标对象demoDto为：" + demoDto);
    //输出结果：目标对象demoDto为：DemoDto(id=111, name=hello)
}
```
测试结果如下：
> 目标对象demoDto为：DemoDto(id=111, name=hello)

达到了我们的预期结果。

##### 5. 自动生成的实现类

为什么声明一个接口就可以转换对象呢？我们看一下MapStruct在编译期间自动生成的实现类：

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2022-09-01T17:54:38+0800",
    comments = "version: 1.4.2.Final, compiler: IncrementalProcessingEnvironment from gradle-language-java-7.3.jar, environment: Java 1.8.0_231 (Oracle Corporation)"
)
public class DemoMapperImpl implements DemoMapper {

    @Override
    public DemoDto toDemoDto(Demo demo) {
        if ( demo == null ) {
            return null;
        }

        DemoDto demoDto = new DemoDto();

        demoDto.setId( demo.getId() );
        demoDto.setName( demo.getName() );

        return demoDto;
    }
}
```

可以看到，MapStruct帮我们将繁杂的代码自动生成了，而且实现类中用的都是最基本的get、set方法，易于阅读理解，转换速度非常快。



### MapStruct进阶

上面的例子只是小试牛刀，下面开始展示MapStruct的强大之处。

（限于篇幅，这里不展示自动生成的实现类和验证结果，大家可自行测试）

##### 场景1：属性名称不同、（基本）类型不同

- **属性名称不同：** 在方法上加上 **@Mapping** 注解，用来映射属性
- **属性基本类型不同：** 基本类型和String等类型会自动转换

关键字：@Mapping注解

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String id;
    private String fullname;
}

/**
 * 转换器
 */
@Mapper
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    @Mapping(target = "fullname", source = "name")
    DemoDto toDemoDto(Demo demo);
}
```

##### 场景2：统一映射不同类型

下面例子中，time1、time2、time3都会被转换，具体说明看下面的注释：

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
    /**
     * time1、time2名称相同，time3转为time33
     * 这里的time1、time2、time33都是Date类型
     */
    private Date time1;
    private Date time2;
    private Date time3;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String id;
    private String name;
    /**
     * 这里的time1、time2、time33都是String类型
     */
    private String time1;
    private String time2;
    private String time33;
}

/**
 * 转换器
 */
@Mapper
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    @Mapping(target = "time33", source = "time3")
    DemoDto toDemoDto(Demo demo);
    
    //MapStruct会将所有匹配到的：
    //源类型为Date、目标类型为String的属性，
    //按以下方法进行转换
    static String date2String(Date date) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String strDate = simpleDateFormat.format(date);
        return strDate;
    }
}
```

##### 场景3：固定值、忽略某个属性、时间转字符串格式

一个例子演示三种用法，具体说明看注释，很容易理解：

关键字：ignore、constant、dateFormat

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
    private Date time;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String id;
    private String name;
    private String time;
}

/**
 * 转换器
 */
@Mapper
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    //id属性不赋值
    @Mapping(target = "id", ignore = true)
    //name属性固定赋值为“hello”
    @Mapping(target = "name", constant = "hello")
    //time属性转为yyyy-MM-dd HH:mm:ss格式的字符串
    @Mapping(target = "time", dateFormat = "yyyy-MM-dd HH:mm:ss")
    DemoDto toDemoDto(Demo demo);
}
```

##### 场景4：为某个属性指定转换方法

场景2中，我们是按照某个转换方法，统一将一种类型转换为另外一种类型；
而下面这个例子，是为某个属性指定方法：

关键字：@Named注解、qualifiedByName

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String id;
    private String name;
}

/**
 * 转换器
 */
@Mapper
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    //为name属性指定@Named为convertName的方法进行转换
    @Mapping(target = "name", qualifiedByName = "convertName")
    DemoDto toDemoDto(Demo demo);

    @Named("convertName")
    static String aaa(String name) {
        return "姓名为：" + name;
    }
}
```


##### 场景5：多个参数合并为一个对象

如果参数为多个的话，@Mapping注解中的source就要指定是哪个参数了，用点分隔：

关键字：点（.）

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String fullname;
    private String timestamp;
}

/**
 * 转换器
 */
@Mapper
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    //fullname属性赋值demo对象的name属性(注意这里.的用法)
    //timestamp属性赋值为传入的time参数
    @Mapping(target = "fullname", source = "demo.name")
    @Mapping(target = "timestamp", source = "time")
    DemoDto toDemoDto(Demo demo, String time);
}
```


##### 场景6：已有目标对象，将源对象属性覆盖到目标对象

覆盖目标对象属性时，一般null值不覆盖，所以需要在类上的@Mapper注解中添加属性： **nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE** 代表null值不进行赋值。

关键字：@MappingTarget注解、nullValuePropertyMappingStrategy

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String name;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String id;
    private String name;
}

/**
 * 转换器
 */
@Mapper(unmappedTargetPolicy = ReportingPolicy.IGNORE,
        nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    //将已有的目标对象当作一个参数传进来
    DemoDto toDemoDto(Demo demo, @MappingTarget DemoDto dto);
}
```


##### 场景7：源对象两个属性合并为一个属性

这种情况可以使用@AfterMapping注解。

关键字：@AfterMapping注解、@MappingTarget注解

```java
/**
 * 来源对象
 */
@Data
public class Demo {
    private Integer id;
    private String firstName;
    private String lastName;
}

/**
 * 目标对象
 */
@Data
public class DemoDto {
    private String id;
    private String name;
}

/**
 * 转换器
 */
@Mapper
public interface DemoMapper {
    DemoMapper INSTANCE = Mappers.getMapper(DemoMapper.class);

    DemoDto toDemoDto(Demo demo);

    //在转换完成后执行的方法，一般用到源对象两个属性合并为一个属性的场景
    //需要将源对象、目标对象（@MappingTarget）都作为参数传进来，
    @AfterMapping
    static void afterToDemoDto(Demo demo, @MappingTarget DemoDto demoDto) {
        String name = demo.getFirstName() + demo.getLastName();
        demoDto.setName(name);
    }
}
```

### 小结
本文介绍了对象转换工具 MapStruct 库，以安全、简洁、优雅的方式来优化我们的转换代码。

从文中的示例场景中可以看出，MapStruct 提供了大量的功能和配置，使我们可以快捷的创建出各种或简单或复杂的映射器。
而这些，也只是 MapStruct 库的冰山一角，还有很多强大的功能文中没有提到，感兴趣的朋友可以自行查看官方文档。

### 写在最后
欢迎加入**Java技术指北读者交流群**，聊天学习摸鱼为主，不定时会分享一些技术要点和优质学习资源，有一群有趣有料的小伙伴在等你哦！进群方式：公众号后台回复**888**，按提示操作即可进群。
