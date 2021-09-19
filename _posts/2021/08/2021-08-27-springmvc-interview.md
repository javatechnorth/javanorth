---
layout: post
title: springMVC 面试题
tagline: by feng
categories: spring
tags: 
    - feng
---

springMVC 面试题
<!--more-->

### 1、什么是Spring MVC？简单介绍下你对Spring MVC的理解？

Spring MVC是一个基于Java的实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把模型-视图-控制器分离，将web层进行职责解耦，把复杂的web应用分成逻辑清晰的几部分，简化开发，减少出错，方便组内开发人员之间的配合。

### 2、Spring MVC的优点

（1）可以支持各种视图技术,而不仅仅局限于JSP；

（2）与Spring框架集成（如IoC容器、AOP等）；

（3）清晰的角色分配：前端控制器(dispatcherServlet) , 请求到处理器映射（handlerMapping), 处理器适配器（HandlerAdapter), 视图解析器（ViewResolver）。

（4） 支持各种请求资源的映射策略。

### 3、Spring MVC的主要组件？

（1）前端控制器 DispatcherServlet（不需要程序员开发）

作用：接收请求、响应结果，相当于转发器，有了DispatcherServlet 就减少了其它组件之间的耦合度。

（2）处理器映射器HandlerMapping（不需要程序员开发）

作用：根据请求的URL来查找Handler

（3）处理器适配器HandlerAdapter

注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。

（4）处理器Handler（需要程序员开发）

（5）视图解析器 ViewResolver（不需要程序员开发）

作用：进行视图的解析，根据视图逻辑名解析成真正的视图（view）

（6）视图View（需要程序员开发jsp）

View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf等等）

### 4、什么是DispatcherServlet

Spring的MVC框架是围绕DispatcherServlet来设计的，它用来处理所有的HTTP请求和响应。

### 5、什么是Spring MVC框架的控制器？

控制器提供一个访问应用程序的行为，此行为通常通过服务接口实现。控制器解析用户输入并将其转换为一个由视图呈现给用户的模型。Spring用一个非常抽象的方式实现了一个控制层，允许用户创建多种用途的控制器。

### 6、Spring MVC的控制器是不是单例模式,如果是,有什么问题,怎么解决？

答：是单例模式,所以在多线程访问的时候有线程安全问题,不要用同步,会影响性能的,解决方案是在控制器里面不能写字段。

### 7、请描述Spring MVC的工作流程？描述一下 DispatcherServlet 的工作流程？

（1）用户发送请求至前端控制器DispatcherServlet；
（2） DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handle；
（3）处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet；
（4）DispatcherServlet 调用 HandlerAdapter处理器适配器；
（5）HandlerAdapter 经过适配调用 具体处理器(Handler，也叫后端控制器)；
（6）Handler执行完成返回ModelAndView；
（7）HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet；
（8）DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析；
（9）ViewResolver解析后返回具体View；
（10）DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）
（11）DispatcherServlet响应用户。

![](http://www.javanorth.cn/assets/images/2021/feng/springmvc1.jpg)

### 8、注解原理是什么
注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。

### 9、springmvc常用到的注解，作用是什么，原理。

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


### 10、 @Component, @Controller, @Repository, @Service 有何区别？

- @Component：这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。
- @Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。
- @Service：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。
- @Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

### 11、Springmvc controller方法中为什么不能定义局部变量？

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

### 12、SpingMvc中的控制器的注解一般用哪个,有没有别的注解可以替代？

一般用@Controller注解,也可以使用@RestController,@RestController注解相当于@ResponseBody ＋ @Controller,表示是表现层,除此之外，一般不用别的注解代替。

### 13、@Controller注解的作用

在Spring MVC 中，控制器Controller 负责处理由DispatcherServlet 分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。在Spring MVC 中提供了一个非常简便的定义Controller 的方法，你无需继承特定的类或实现特定的接口，只需使用@Controller 标记一个类是Controller ，然后使用@RequestMapping 和@RequestParam 等一些注解用以定义URL 请求和Controller 方法之间的映射，这样的Controller 就能被外界访问到。此外Controller 不会直接依赖于HttpServletRequest 和HttpServletResponse 等HttpServlet 对象，它们可以通过Controller 的方法参数灵活的获取到。

@Controller 用于标记在一个类上，使用它标记的类就是一个Spring MVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。单单使用@Controller 标记在一个类上还不能真正意义上的说它就是Spring MVC 的一个控制器类，因为这个时候Spring 还不认识它。那么要如何做Spring 才能认识它呢？这个时候就需要我们把这个控制器类交给Spring 来管理。有两种方式：

- 在Spring MVC 的配置文件中定义MyController 的bean 对象。
- 在Spring MVC 的配置文件中告诉Spring 该到哪里去找标记为@Controller 的Controller 控制器。

### 14、@RequestMapping注解的作用

RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

RequestMapping注解有六个属性，下面我们把她分成三类进行说明（下面有相应示例）。

value， method

value： 指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；

method： 指定请求的method类型， GET、POST、PUT、DELETE等；

consumes，produces

consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;

produces: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

params，headers

params： 指定request中必须包含某些参数值是，才让该方法处理。

headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。

### 15、@ResponseBody注解的作用

作用： 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。

使用时机：返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

### 16、 @PathVariable和@RequestParam的区别

请求路径上有个id的变量值，可以通过@PathVariable来获取 @RequestMapping(value = “/page/{id}”, method = RequestMethod.GET)

@RequestParam用来获得静态的URL请求入参 spring注解时action里用到。

### 17、如果在拦截请求中，我想拦截get方式提交的方法，怎么配置？

可以在@RequestMapping注解里面加上method=RequestMethod.GET。

### 18、怎样在方法里面得到Request，或者Session？

直接在方法的形参中声明request，SpringMvc就自动把request对象传入。

### 19、如果想在拦截的方法里面得到从前台传入的参数，怎么得到？

直接在形参里面声明这个参数就可以，但必须名字和传过来的参数一样。

### 20、如果前台有很多个参数传入，并且这些参数都是一个对象的，那么怎么样快速得到这个对象？

直接在方法中声明这个对象，SpringMvc就自动会把属性赋值到这个对象里面。

### 21、SpringMvc中函数的返回值是什么？

答：返回值可以有很多类型，有String，ModelAndView。ModelAndView类把视图和数据都合并的一起的，但一般用String比较好。

### 22、SpringMvc用什么对象从后台向前台传递数据的？

通过ModelMap对象，可以在这个对象里面调用put方法，把对象加到里面，前端就可以通过el表达式拿到。

### 23、怎么样把ModelMap里面的数据放入Session里面？

可以在类上面加上@SessionAttributes注解，里面包含的字符串就是要放入session里面的key。
