---
layout: post
title: spring 面试题
tagline: by feng
categories: spring
tags: 
    - feng
---

### 1、什么是 Spring 框架?Spring 框架有哪些主要模块?

Spring 框架是一个为 Java 应用程序的开发提供了综合、广泛的基础性支持的 Java 平台。 Spring 帮助开发者解决了开发中基础性的问题，使得开发人员可以专注于应用程序的开发。

Spring 框架本身亦是按照设计模式精心打造，这使得我们可以在开发环境中安心的集成 Spring 框架，不必担心 Spring 是如何在后台进行工作的。

Spring 框架至今已集成了 20 多个模块。这些模块主要被分如下图所示的核心容器、数据访问/集成,、Web、AOP(面向切面编程)、工具、消息和测试模块。
<!--more-->
### 2、使用 Spring 框架能带来哪些好处?

**轻量**:Spring 是轻量的，基本的版本大约 2MB

**控制反转**:Spring 通过控制反转实现了松散耦合，对象们给出它们的依赖，而不是 创建或查找依赖的对象们

**面向切面的编程(AOP)**:Spring 支持面向切面的编程，并且把应用业务逻辑和系统服 务分开

**容器**:Spring 包含并管理应用中对象的生命周期和配置

**MVC 框架**:Spring 的 WEB 框架是个精心设计的框架，是 Web 框架的一个很好的 替代品

**事务管理**:Spring 提供一个持续的事务管理接口，可以扩展到上至本地事务下至全 局事务(JTA)

**异常处理**:Spring 提供方便的 API 把具体技术相关的异常(比如由 JDBC， Hibernate or JDO 抛出的)转化为一致的 unchecked 异常

### 3、什么是 Spring 的依赖注入?

依赖注入，是 IOC 的一个方面，是个通常的概念，它有多种解释。这概念是说你不 用创建对象，而只需要描述它如何被创建。你不在代码里直接组装你的组件和服务， 但是要在配置文件里描述哪些组件需要哪些服务，之后一个容器(IOC 容器)负责把 他们组装起来。

### 4、有哪些不同类型的 IOC(依赖注入)方式?

构造器依赖注入:构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

Setter 方法注入:Setter 方法注入是容器通过调用无参构造器或无参 static 工厂方法实例化 bean 之后，调用该 bean 的 setter 方法，即实现了基于 setter 的依赖注入。

### 5、哪种依赖注入方式你建议使用，构造器注入，还是 Setter 方法注入?

你两种依赖方式都可以使用，构造器注入和 Setter 方法注入。最好的解决方案是用 构造器参数实现强制依赖，setter 方法实现可选依赖。

### 6、什么是 Spring beans?

Spring beans 是那些形成 Spring 应用的主干的 java 对象。它们被 Spring IOC 容 器初始化，装配，和管理。这些 beans 通过容器中配置的元数据创建。比如，以 XML 文件中 `<bean/>` 的形式定义。

Spring 框架定义的beans都是单件beans。在beantag中有个属性”singleton”， 如果它被赋为 TRUE，bean 就是单件，否则就是一个 prototype bean。默认是 TRUE，所以所有在 Spring 框架中的 beans 缺省都是单件。点击这里一图 Spring Bean 的生命周期。

### 7、一个 Spring Bean 定义 包含什么?

一个 Spring Bean 的定义包含容器必知的所有配置元数据，包括如何创建一个 bean，它的生命周期详情及它的依赖。

### 8、如何给 Spring 容器提供配置元数据?

这里有三种重要的方法给 Spring 容器提供配置元数据。

- XML 配置文件。
- 基于注解的配置。
- 基于 java 的配置。

### 9、解释 Spring 支持的几种 bean 的作用域

Spring 框架支持以下五种 bean 的作用域:

- singleton : bean 在每个 Spring ioc 容器中只有一个实例。
- prototype:一个 bean 的定义可以有多个实例。
- request:每次 http 请求都会创建一个 bean，该作用域仅在基于 web 的 Spring ApplicationContext 情形下有效。
- session:在一个 HTTP Session 中，一个 bean 定义对应一个实例。该作用域 仅在基于 web 的 Spring ApplicationContext 情形下有效。
- global-session:在一个全局的 HTTP Session 中，一个 bean 定义对应一个实 例。该作用域仅在基于 web 的 Spring ApplicationContext 情形下有效。
- 缺省的 Spring bean 的作用域是 Singleton。

### 10、Spring 框架中的单例 bean 是线程安全的吗?

肯定不是线程安全的，当多用户同时请求一个服务时，容器会给每一个请求分配一个线程，这是多个线程会并发执行该请求多对应的业务逻辑（成员方法），此时就要注意了，如果该处理逻辑中有对该单列状态的修改（体现为该单列的成员属性），则必须考虑线程同步问题.

Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。

### 11、解释 Spring 框架中 bean 的生命周期

Bean在Spring中的生命周期如下：

实例化。Spring通过new关键字将一个Bean进行实例化，JavaBean都有默认的构造函数，因此不需要提供构造参数。

填入属性。Spring根据xml文件中的配置通过调用Bean中的setXXX方法填入对应的属性。 事件通知。Spring依次检查Bean是否实现了BeanNameAware、BeanFactoryAware、ApplicationContextAware、BeanPostProcessor、InitializingBean接口，如果有的话，依次调用这些接口。

使用。应用程序可以正常使用这个Bean了。

销毁。如果Bean实现了DisposableBean接口，就调用其destroy方法。

---加载过程--- 

1. 容器寻找Bean的定义信息并且将其实例化。
2. 如果允许提前暴露工厂，则提前暴露这个bean的工厂，这个工厂主要是返回该未完全处理的bean．主要是用于避免单例属性循环依赖问题．
3. 受用依赖注入，Spring按照Bean定义信息配置Bean的所有属性。
4. 如果Bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID。
5. 如果Bean实现了BeanFactoryAware接口，工厂调用setBeanFactory()方法传入工厂自身。
6. 如果BeanPostProcessor和Bean关联，那么它们的postProcessBeforeInitialzation()方法将被调用。
7. 如果Bean指定了init-method方法，它将被调用。
8. 如果有BeanPostProcessor和Bean关联，那么它们的postProcessAfterInitialization()方法将被调用
9. 最后如果配置了destroy-method方法则注册DisposableBean.

到这个时候，Bean已经可以被应用系统使用了，并且将被保留在Bean Factory中知道它不再需要。 有两种方法可以把它从Bean Factory中删除掉。

1. 如果Bean实现了DisposableBean接口，destory()方法被调用。
2. 如果指定了订制的销毁方法，就调用这个方法。

### 12、请解释自动装配模式的区别

no：这是Spring框架的默认设置，在该设置下自动装配是关闭的，开发者需要自行在bean定义中用标签明确的设置依赖关系。

byName**：该选项可以根据bean名称设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的名称自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

byType：该选项可以根据bean类型设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的类型自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

constructor：造器的自动装配和byType模式类似，但是仅仅适用于与有构造器相同参数的bean，如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。

autodetect：该模式自动探测使用构造器自动装配或者byType自动装配。首先，首先会尝试找合适的带参数的构造器，如果找到的话就是用构造器自动装配，如果在bean内部没有找到相应的构造器或者是无参构造器，容器就会自动选择byTpe的自动装配方式。

### 13、Spring 框架中都用到了哪些设计模式

代理模式—在AOP和remoting中被用的比较多。

单例模式—在spring配置文件中定义的bean默认为单例模式。

模板方法—用来解决代码重复的问题 比如. RestTemplate, JmsTemplate, JpaTemplate。 前端控制器—Srping提供了DispatcherServlet来对请求进行分发。 视图帮助(View Helper )—Spring提供了一系列的JSP标签，高效宏来辅助将分散的代码整合在视图里。 依赖注入—贯穿于BeanFactory / ApplicationContext接口的核心理念。

工厂模式—BeanFactory用来创建对象的实例。

Builder模式- 自定义配置文件的解析bean是时采用builder模式，一步一步地构建一个beanDefinition

策略模式：Spring 中策略模式使用有多个地方，如 Bean 定义对象的创建以及代理对象的创建等。这里主要看一下代理对象创建的策略模式的实现。 前面已经了解 Spring 的代理方式有两个 Jdk 动态代理和 CGLIB 代理。这两个代理方式的使用正是使用了策略模式。

### 14、什么是基于 Java 的 Spring 注解配置?给一些注解的例子.

基于 Java 的配置，允许你在少量的 Java 注解的帮助下，进行你的大部分 Spring 配置而非通过 XML 文件。

以@Configuration 注解为例，它用来标记类可以当做一个 bean 的定义，被 SpringIOC 容器使用。另一个例子是@Bean 注解，它表示此方法将要返回一个 对象，作为一个 bean 注册进 Spring 应用上下文。

### 15、什么是基于注解的容器配置?

相对于 XML 文件，注解型的配置依赖于通过字节码元数据装配组件，而非尖括号的声明。

开发者通过在相应的类，方法或属性上使用注解的方式，直接组件类中进行配置， 而不是使用 xml 表述 bean 的装配关系。

### 16、Spring 支持的事务管理类型

Spring 支持两种类型的事务管理:

- 编程式事务管理:这意味你通过编程的方式管理事务，给你带来极大的灵活 性，但是难维护。
- 声明式事务管理:这意味着你可以将业务代码和事务管理分离，你只需用注 解和 XML 配置来管理事务。

### 17、什么是AOP， 什么是 Aspect 切面?

面向切面的编程，或 AOP，是一种编程技术，允许程序模块化横向切割关注点， 或横切典型的责任划分，如日志和事务管理。

AOP 核心就是切面，它将多个类的通用行为封装成可重用的模块，该模块含有 一组 API 提供横切功能。比如，一个日志模块可以被称作日志的 AOP 切面。根据需求的不同，一个应用程序可以有若干切面。在 SpringAOP 中，切面通过带 有@Aspect 注解的类实现。

### 18、在 SpringAOP 中，关注点和横切关注的区别是什么?

关注点是应用中一个模块的行为，一个关注点可能会被定义成一个我们想实现的 一个功能。 

横切关注点是一个关注点，此关注点是整个应用都会使用的功能，并影响整个应 用，比如日志，安全和数据传输，几乎应用的每个模块都需要的功能。因此这些 都属于横切关注点。

### 19、AOP 中的通知

通知是个在方法执行前或执行后要做的动作，实际上是程序执行时要通过 SpringAOP 框架触发的代码段。

Spring 切面可以应用五种类型的通知:

- before:前置通知，在一个方法执行前被调用。
- after:在方法执行之后调用的通知，无论方法执行是否成功。
- after-returning:仅当方法成功完成后执行的通知。
- after-throwing:在方法抛出异常退出时执行的通知。
- around:在方法执行之前和之后调用的通知。

### 20、什么是 Spring 的 MVC 框架?

Spring 配备构建 Web 应用的全功能 MVC 框架。Spring 可以很便捷地和其他 MVC 框架集成，如 Struts，Spring 的 MVC 框架用控制反转把业务对象和控制 逻辑清晰地隔离。它也允许以声明的方式把请求参数和业务对象绑定。

### 21、springmvc常用到的注解，作用是什么，原理。

**@Controller注解** 

是在Spring的org.springframework.stereotype包下，org.springframework.stereotype.Controller注解类型用于指示Spring类的实例是一个控制器

使用@Controller注解的类不需要继承特定的父类或者实现特定的接口，相对之前的版本实现Controller接口变的更加简单。

而Controller接口的实现类只能处理一个单一的请求动作，而@Controller注解注解的控制器可以同时支持处理多个请求动作，使程序开发变的更加灵活。 @Controller用户标记一个类，使用它标记的类就是一个Spring MVC Controller对象，即：一个控制器类。Spring使用扫描机制查找应用程序中所有基于注解的控制器类，分发处理器会扫描使用了该注解的方法，并检测该方法是否使用了@RequestMapping注解，而使用@RequestMapping注解的方法才是真正处理请求的处理器。为了保证Spring能找到控制器，我们需要完成两件事：



**@RequestParam注解**

下面来说org.springframework.web.bind.annotation包下的第三个注解，即：@RequestParam注解，该注解类型用于将指定的请求参数赋值给方法中的形参。那么@RequestParam注解有什么属性呢？它有4种属性，下面将逐一介绍这四种属性：

1、name属性该属性的类型是String类型，它可以指定请求头绑定的名称； 

2、value属性该属性的类型是String类型，它可以设置是name属性的别名； 

3、required属性该属性的类型是boolean类型，它可以设置指定参数是否必须绑定； 

4、defalutValue属性该属性的类型是String类型，它可以设置如果没有传递参数可以使用默认值。



**@PathVaribale注解** 

下面来说org.springframework.web.bind.annotation包下的第四个注解，即：@PathVaribale注解，该注解类型可以非常方便的获得请求url中的动态参数。@PathVaribale注解只支持一个属性value，类型String，表示绑定的名称，如果省略则默认绑定同名参数。


### 22、 @Component, @Controller, @Repository, @Service 有何区别？

- @Component：这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。
- @Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。
- @Service：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。
- @Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

### 23、Springmvc controller方法中为什么不能定义局部变量？

因为controller是默认单例模式，高并发下全局变量会出现线程安全问题

现这种问题如何解决呢？

1、 既然是全局变量惹的祸，那就将全局变量都编程局部变量，通过方法参数来传递。

2、 jdk提供了java.lang.ThreadLocal,它为多线程并发提供了新思路。

3、 使用@Scope("session")，会话级别

```java
    @Controller  
    //把这个bean 的范围设置成session，表示这bean是会话级别的，  
    @Scope("session")  
    public class XxxController{  
        private List<String> list ;  
 
      //@PostConstruct当bean加载完之后，就会执行init方法，并且将list实例化；  
        @PostConstruct  
        public void init(){  
            list = new ArrayList<String>();  
        }  
    } 
```

4、 将控制器的作用域从单例改为原型，即在spring配置文件Controller中声明 scope="prototype"，每次都创建新的controller

### 24、Spring 事务支持的隔离级别

Spring 事务上提供以下的隔离级别:

- ISOLATION_DEFAULT: 使用后端数据库默认的隔离级别
- ISOLATION_READ_UNCOMMITTED : 允许读取未提交的数据变更， 可能会导致脏读，幻读或不可重复读
- ISOLATION_READ_COMMITTD : 允许读取为提交数据,可以阻止脏 读，当时幻读或不可重复读仍可能发生
- ISOLATION_REPEATABLE_READ: 对统一字段多次读取结果是一致 的，除非数据是被本事务自己修改.可以阻止脏读，不可重复读，但幻读可 能发生
- ISOLATION_SERIALIZABLE : 完全服从 ACID