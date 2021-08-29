### 使用 Spring 框架的好处是什么？

* 轻量：Spring 是轻量的，基本的版本大约 2MB。
* 控制反转：Spring 通过控制反转实现了松散耦合，对象们给出它们的依 赖，而不是创建或查找依赖的对象们。
* 面向切面的编程(AOP)：Spring 支持面向切面的编程，并且把应用业务 逻辑和系统服务分开。
* 容器：Spring 包含并管理应用中对象的生命周期和配置。
* MVC 框架：Spring 的 WEB 框架是个精心设计的框架，是 Web 框架的 一个很好的替代品。
* 事务管理：Spring 提供一个持续的事务管理接口，可以扩展到上至本地 事务下至全局事务（JTA）。
* 异常处理：Spring提供方便的 API 把具体技术相关的异常（比如由JDBC， HibernateorJDO 抛出的）转化为一致的 unchecked 异常。

### 2、Spring AOP 的实现原理。

Spring AOP 的面向切面编程，是面向对象编程的一种补充，用于处理系统中分布的各个模块的横切关注点，比如说事务管理、日志、缓存等。它是使用动态 代理实现的，在内存中临时为方法生成一个 AOP 对象，这个对象包含目标对象 的所有方法，在特定的切点做了增强处理，并回调原来的方法。

Spring AOP 的动态代理主要有两种方式实现，JDK 动态代理和 cglib 动态代理。 JDK 动态代理通过反射来接收被代理的类，但是被代理的类必须实现接口，核 心是 InvocationHandler 和 Proxy 类。cglib 动态代理的类一般是没有实现接口的类，cglib 是一个代码生成的类库，可以在运行时动态生成某个类的子类，所 以，CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 final， 那么它是无法使用 CGLIB 做动态代理的。

###  3、讲讲 Spring 事务的传播属性。

事务就是对一系列的数据库操作(比如插入多条数据)进行统一的提交或回滚 操作。如果插入成功，那么一起成功，如果中间有一条出现异常，那么回滚之前的所有操作。这样可以防止出现脏数据，防止数据库数据出现问题。 事务的传播行为，指的是当前带有事务配置的方法，需要怎么处理事务。

例如，方法可以继续在当前事务中运行，也可能开启一个新的事务，并在自己 的事务中运行。

有一点需要注意，事务的传播级别，并不是数据库事务规范中的名词，而是 Spring 自身所定义的。通过事务的传播级别，Spring 才知道如何处理事务，是 创建一个新的事务，还是继续使用当前的事务。
在 TransactionDefinition 接口中，定义了三类七种传播级别。如下:

```java
// TransactionDefinition.java
// ========== 支持当前事务的情况 ========== /**
* 如果当前存在事务，则使用该事务。
* 如果当前没有事务，则创建一个新的事务。
*/
int PROPAGATION_REQUIRED = 0;
/**
* 如果当前存在事务，则使用该事务。
* 如果当前没有事务，则以非事务的方式继续运行。
 
*/
int PROPAGATION_SUPPORTS = 1; /**
* 如果当前存在事务，则使用该事务。 * 如果当前没有事务，则抛出异常。
 */
int PROPAGATION_MANDATORY = 2;
// ========== 不支持当前事务的情况 ==========
 /**
* 创建一个新的事务。
* 如果当前存在事务，则把当前事务挂起。 */
 int PROPAGATION_REQUIRES_NEW = 3; /**
* 以非事务方式运行。
* 如果当前存在事务，则把当前事务挂起。
 */
int PROPAGATION_NOT_SUPPORTED = 4; /**
* 以非事务方式运行。
  * 如果当前存在事务，则抛出异常。
*/
int PROPAGATION_NEVER = 5;
// ========== 其他情况 ==========
/**
* 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行。 * 如果当前没有事务，则等价于 {@link
 TransactionDefinition#PROPAGATION_REQUIRED} */
int PROPAGATION_NESTED = 6;

```

当然，绝大数场景下，我们是只用到 PROPAGATION_REQUIRED 传播级别的。

需要注意的是，以 PROPAGATION_NESTED 启动的事务内嵌于外部事物(如果 存在外部事务的话)，此时，内嵌事务并不是一个独立的事务，它依赖于外部 事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子 事务不能单独提交。这点类似于 JDBC 中的保存点(SavePoint)概念，嵌套的子事务就相当于保存点的一个应用，一个事务中可以包括多个保存点，每一个 嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

### 4. Spring 如何管理事务的，怎么配置事务

所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”。 

Spring 提供两种类型的事务管理:
* 声明式事务，通过使用注解或基于 XML 的配置事务，从而事务管理与业务代码分离。
* 编程式事务，通过编码的方式实现事务管理，需要在代码中显示的调用事务的获得、提交、回滚。它提供了极大的灵活性，但维护起来非常困难。
 
实际场景下，我们一般使用 SpringBoot+注解(@Transactional)的声明式事务。Spring 的声明式事务管理建立在 AOP 基础上，其本质是在目标方法执行 前进行拦截，在方法开始前创建一个事务，在执行完方法后根据执行情况提交 或回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这 样就不用侵入业务代码，只需要在配置文件中做相关的事物声明就可将业务规则应用到业务逻辑中。和编程式事务相比，声明式事务唯一的不足是只能作用 到方法级别，无法做到像编程式事务那样到代码块级别。

### 5. 解释 Spring 支持的几种 bean 的作用域。

* singleton:bean 在每个 Springioc 容器中只有一个实例。
* prototype：一个 bean 的定义可以有多个实例。 
* request：每次 http 请求都会创建一个 bean，该作用域仅在基于 web 的SpringApplicationContext 情形下有效。
* session：在一个 HTTPSession 中，一个 bean 定义对应一个实例。该 作用域仅在基于 web 的 SpringApplicationContext 情形下有效。
* global-session：在一个全局的 HTTPSession 中，一个 bean 定义对应 一个实例。该作用域仅在基于 web 的 SpringApplicationContext 情形下 有效。

缺省的 Springbean 的作用域是 Singleton。

### 6. Spring 框架中用到了哪些设计模式?

* 工厂设计模式: Spring 使用工厂模式通过 BeanFactory、 ApplicationContext 创建 bean 对象。
* 代理设计模式: Spring AOP 功能的实现。
* 单例设计模式: Spring 中的 Bean 默认都是单例的。
* 模板方法模式: Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
* 包装器设计模式: 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据 客户的需求能够动态切换不同的数据源。
* 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
* 适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式。

### 7. 解释 Spring 框架中 bean 的生命周期。

* Spring 容器从 XML 文件中读取 bean 的定义，并实例化 bean。 
* Spring 根据 bean 的定义填充所有的属性。 
* 如果 bean 实现了 BeanNameAware 接口，Spring 传递 bean 的 ID 到 setBeanName 方法。 
* 如果 Bean 实现了 BeanFactoryAware 接口，Spring 传递 beanfactory 给 setBeanFactory 方法。 
* 如果有任何与 bean 相关联的 BeanPostProcessors，Spring 会在 postProcesserBeforeInitialization()方法内调用它们。 
* 如果 bean 实现 IntializingBean 了，调用它的 afterPropertySet 方法， 如果 bean 声明了初始化方法，调用此初始化方法。 
* 如果有 BeanPostProcessors 和 bean 关联，这些 bean 的 postProcessAfterInitialization()方法将被调用。 
* 如果 bean 实现了 DisposableBean，它将调用 destroy()方法。

### 8 .哪些是重要的 bean 生命周期方法？你能重载它们吗？

有两个重要的 bean 生命周期方法，第一个是 setup，它是在容器加载 bean 的 时候被调用。第二个方法是 teardown 它是在容器卸载类的时候被调用。

Thebean 标签有两个重要的属性（init-method 和 destroy-method）。用它 们你可以自己定制初始化和注销方法。它们也有相应的注解（@PostConstruct 和@PreDestroy）。

### 9.在 Spring 中如何注入一个 java 集合？

Spring 提供以下几种集合的配置元素： 
* <list>类型用于注入一列值，允许有相同的值。 
* <set>类型用于注入一组值，不允许有相同的值。 
* <map>类型用于注入一组键值对，键和值都可以为任意类型。 
* <props>类型用于注入一组键值对，键和值都只能为 String 类型。

### 10. 解释不同方式的自动装配

有 5 种自动装配的方式，可以用来指导 Spring 容器用自动装配方式来进行依赖注入。
* no：默认的方式是不进行自动装配，通过显式设置 ref 属性来进行装配。 
* byName：通过参数名自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被设置成byname，之后容器试图匹配、装配和该 bean 的属性具有相同名字的 bean。 
* byType:：通过参数类型自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被设置成byType，之后容器试图匹配、装配和该 bean 的属性具有相同类型的 bean。如果有多个 bean 符合条件，则抛出错误。 
* constructor：这个方式类似于 byType，但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。 
* autodetect：首先尝试使用 constructor 来自动装配，如果无法工作，则使用 byType 方式。

11. 什么是基于 Java 的 Spring 注解配置?

基于 Java 的配置，允许你在少量的 Java 注解的帮助下，进行你的大部分 Spring 配置而非通过 XML 文件。 

以 @Configuration 注解为例，它用来标记类可以当做一个 bean 的定义，被 SpringIOC 容器使用。另一个例子是@Bean 注解，它表示此方法将要返回一个 对象，作为一个 bean 注册进 Spring 应用上下文。

12. Spring 7种事务的传播⾏为

* PROPAGATION_REQUIRED：如果当前没有事务，就新建⼀个事务，如果已经存在⼀个事务中，加⼊到这个事务中。这是最常⻅的选择。
* PROPAGATION_SUPPORTS：⽀持当前事务，如果当前没有事务，就以⾮事务⽅式执⾏。
* PROPAGATION_MANDATORY：使⽤当前的事务，如果当前没有事务，就抛出异常。
* PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。
* PROPAGATION_NOT_SUPPORTED：以⾮事务⽅式执⾏操作，如果当前存在事务，就把当前事务挂起。
* PROPAGATION_NEVER：以⾮事务⽅式执⾏，如果当前存在事务，则抛出异常。
* PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执⾏。如果当前没有事务，则执⾏与PROPAGATION_REQUIRED类似的
操作。

13. BeanFactory 和 ApplicationContext 有什么区别

* BeanFactory 可以理解为含有 bean 集合的工厂类。BeanFactory 包含了种 bean 的定义，以便在接收到客户端请求时将对应的 bean 实例化。
* BeanFactory 还能在实例化对象的时生成协作类之间的关系。此举将 bean 自身与 bean 客户端的配置中解放出来。BeanFactory 还包含了 bean 生命周期的控制，调用客户端的初始化方法（initialization methods）和销毁方法（destruction methods）。
* 从表面上看，application context 如同 bean factory 一样具有 bean 定义、bean 关联关系的设置，根据请求分发 bean 的功能但 application context 在此基础上还提供了其他的功能。
* 提供了支持国际化的文本消息
* 统一的资源文件读取方式
* 已在监听器中注册的 bean 的事件

14. Spring IOC 如何实现

* Spring 中的 org.springframework.beans 包和 org.springframework.context 包构成了Spring 框架 IoC 容器的基础。
* BeanFactory 接口提供了一个先进的配置机制，使得任何类型的对象的配置成为可能。ApplicationContex 接口对 BeanFactory（是一个子接口）进行了扩展，在 BeanFactory 的基础上添加了其他功能，比如与 Spring 的 AOP 更容易集成，也提供了处理 messageresource的机制（用于国际化）、事件传播以及应用层的特别配置，比如针对 Web 应用的WebApplicationContext。 
* org.springframeworkbeans.factory.BeanFactory 是 Spring IoC 容器的具体实现，用来包装和管理前面提到的各种 bean。BeanFactory 接口是 Spring IoC 容器的核心接口。

15. 怎样开启注解装配？

注解装配在默认情况下是不开启的，为了使用注解装配，我们必须在 Spring 配 置文件中配置<context:annotation-config/>元素。

16. @Required 注解

这个注解表明 bean 的属性必须在配置的时候设置，通过一个 bean 定义的显式 的属性值或通过自动装配，若@Required 注解的 bean 属性未被设置，容器将 抛出 BeanInitializationException。

17. @Autowired 注解

@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。 它的用法和@Required 一样，修饰 setter 方法、构造器、属性或者具有任意名 称和/或多个参数的 PN 方法。

18. @Qualifier 注解

当有多个相同类型的 bean 却只有一个需要自动装配时，将@Qualifier 注解和 @Autowire 注解结合使用以消除这种混淆，指定需要装配的确切的 bean。

19. 如何给 Spring 容器提供配置元数据?

* XML 配置文件。
* 基于注解的配置。
* 基于 java 的配置。
