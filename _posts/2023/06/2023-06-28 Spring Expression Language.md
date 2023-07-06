---
layout: post
title:  2023-06-28 Spring Expression Language
tagline: by 沉浮
categories: 
tags: 沉浮
---


<!--more-->

## Spring Expression Language

### 概念

Spring Expression Language(简称SpEL)是一种强大的表达式语言，支持在运行时查询和操作对象图。该语言的语法类似于Unified EL，但提供了额外的特性，
最显著的是方法调用和基本的字符串模板功能。

虽然还有其他几种Java表达式语言可用——OGNL、MVEL和JBoss EL等，SpEL是为了向Spring社区提供一种受良好支持的表达式语言，可以跨Spring产品组合中的所有产品使用。
SpEL基于一种与技术无关的API，在需要时可以集成其他表达式语言实现。

### 作用

基于SpEL我们可以实现下面这些功能，当然是基于表达式，SpEL提供了表达式运行环境，而且不依赖于Spring，这样我可以基于其强大的执行器实现各种扩展。

+ 支持功能
  + 字面表达式
  + 布尔和关系运算符
  + 正则表达式
  + 类表达式
  + 访问属性、数组、列表和映射
  + 方法调用
  + 关系运算符
  + 申明
  + 调用构造函数
  + bean引用
  + 数组构造
  + 内联的list
  + 内联的map
  + 三元运算符
  + 变量
  + 用户自定义函数
  + 集合投影
  + 集合选择
  + 模板化表达式

### 关键接口

将表达式字符串解析为可求值的编译表达式。支持解析模板以及标准表达式字符串。
```java
public interface ExpressionParser {

  /**
   * 解析字符串表达式为Expression对象
   */
  Expression parseExpression(String expressionString) throws ParseException;

  /**
   * 解析字符串表达式为Expression对象，基于ParserContext解析字符串，比如常见的#{exrp}
   */
  Expression parseExpression(String expressionString, ParserContext context) throws ParseException;

}
```

表达式在求值上下文中执行。正是在这个上下文中，表达式求值期间遇到的引用才会被解析。
EvaluationContext接口有一个默认的实现StandardBeanExpressionResolver，可以通过继承该类进行扩展。
```java
public interface EvaluationContext {

	/**
	 * 获取Root上下文对象
	 */
	TypedValue getRootObject();

	/**
	 * 返回属性读写访问器
	 */
	List<PropertyAccessor> getPropertyAccessors();

	/**
	 * 返回构造器解析器
	 */
	List<ConstructorResolver> getConstructorResolvers();

	/**
	 * 返回方法解析器
	 */
	List<MethodResolver> getMethodResolvers();

	/**
	 * 返回Bean解析器，用于Bean的查找
	 */
	@Nullable
	BeanResolver getBeanResolver();

	/**
	 * 根据类名（一般为全限名）返回一个类型定位器，比如T(java.lang.Math)
	 */
	TypeLocator getTypeLocator();

	/**
	 * 返回可以将值从一种类型转换(或强制转换)为另一种类型的类型转换器。
	 */
	TypeConverter getTypeConverter();

	/**
	 * 返回一个类型比较器，用于比较对象对是否相等。
	 */
	TypeComparator getTypeComparator();

	/**
	 * 返回一个运算符重载器，该重载器可以支持多个标准类型集之间的数学运算。
	 */
	OperatorOverloader getOperatorOverloader();

	/**
     * 为变量设值
	 */
	void setVariable(String name, @Nullable Object value);

	/**
	 * 从变量取值
	 */
	@Nullable
	Object lookupVariable(String name);

}
```

### 适用场景

+ 扩展变量  
  正如Spring中使用的那样，我们可以基于SpEL实现基于Spring容器、运行环境上下文等动态取值。

+ 数据审计
  通过从复杂的数据结构中进行数据运算，一般在数据汇总或者对复杂的结构化数据，记忆定于的表达式来实现数据运算。

### Spring的使用

在Spring中有着大量使用SpEL的场景，在平时的开发中，可能会看到如下的这些配置，比如通过${}、#{}这些包裹的表达式，主要都是基于SpEL实现。

1. @Value("#{systemProperties['pop3.port'] ?: 25}")  
简单的看下源吗，可以看到针对@Value标记的类属性，是如何为其注入属性：

AutowiredAnnotationBeanPostProcessor -> AutowiredFieldElement#inject
`
value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter)
`
DefaultListableBeanFactory#doResolveDependency
```java
public class DefaultListableBeanFactory {
    
  public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
                                    @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

    InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
    try {
      Class<?> type = descriptor.getDependencyType();
      Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
      if (value != null) {
        if (value instanceof String) {
          // 处理 ${} 占位替换
          String strVal = resolveEmbeddedValue((String) value);
          BeanDefinition bd = (beanName != null && containsBean(beanName) ?
                  getMergedBeanDefinition(beanName) : null);
          // SpEL处理
          value = evaluateBeanDefinitionString(strVal, bd);
        }
      }
      // 省略...
      return result;
    }catch (Exception e){
        // ...
    }
  }
}
```

2. @Cacheable(value="users", key="#p0")  
   具体实现可以阅读源码：
`org.springframework.cache.interceptor.CacheOperationExpressionEvaluator`

3. @KafkaListener(topics = "#{'${topics}'.split(',')}")  
   具体实现可以阅读源码：
`org.springframework.kafka.annotation.KafkaListenerAnnotationBeanPostProcessor`

### 示例

下面通过一些测试示例了解SpEL的基本用法，对应上面的一些功能实现，通过这些简单的例子，可以大概了解其语法与使用方式：

```java
public class Tests {
  @Test
  public void runParser() throws NoSuchMethodException {
    SpelExpressionParser parser = new SpelExpressionParser();
    Map map = new HashMap();
    map.put("name", "SpEL");
    StandardEvaluationContext context = new StandardEvaluationContext(map);
    context.setVariable("var1", 1);
    context.setVariable("func1", StringUtils.class.getMethod("hasText", String.class));
    context.setVariable("func2", new Root());
    context.setBeanResolver(new MyBeanResolver());

    log.info("字面量：{}", parser.parseExpression("'hello'").getValue(context));
    log.info("对象属性：{}", parser.parseExpression("'hello'.bytes").getValue(context));
    log.info("变量：{}", parser.parseExpression("#var1").getValue(context));
    log.info("调用方法：{}", parser.parseExpression("'hello'.concat(' world')").getValue(context));
    log.info("静态方法：{}", parser.parseExpression("T(java.lang.System).currentTimeMillis()").getValue(context));
    log.info("方法：{}", parser.parseExpression("#func1('1')").getValue(context));
    log.info("实例方法：{}", parser.parseExpression("#func2.print('2')").getValue(context));

    Root root = new Root();
    root.list.add("0");
    parser.parseExpression("list[0]").setValue(context, root, "1");
    log.info("设值：{}", root);

    log.info("ROOT: {}", parser.parseExpression("#root").getValue(context));
    log.info("ROOT 取值: {}", parser.parseExpression("#root[name]").getValue(context));
    log.info("ROOT 取值: {}", parser.parseExpression("[name]").getValue(context));

    log.info("THIS: {}", parser.parseExpression("#this").getValue(context));

    log.info("运算符: {}", parser.parseExpression("1+1").getValue(context));
    log.info("操作符: {}", parser.parseExpression("1==1").getValue(context));
    log.info("逻辑运算: {}", parser.parseExpression("true && false").getValue(context));

    ParserContext parserContext = new ParserContext() {
      @Override
      public boolean isTemplate() {
        return true;
      }

      @Override
      public String getExpressionPrefix() {
        return "#{";
      }

      @Override
      public String getExpressionSuffix() {
        return "}";
      }
    };
    log.info("#{表达式}: {}", parser.parseExpression("#{1+1}", parserContext).getValue(context));

    log.info("Map: {}", parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context));
    log.info("List: {}", parser.parseExpression("{1,2,3,4}").getValue(context));
    log.info("Array: {}", parser.parseExpression("new int[]{1,2,3}").getValue(context));

    log.info("instanceof: {}", parser.parseExpression("'hello' instanceof T(Integer)").getValue(context, Boolean.class));
    log.info("regex: {}", parser.parseExpression("'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(context, Boolean.class));
    log.info("三目运算: {}", parser.parseExpression("false ? 'trueExp' : 'falseExp'").getValue(context, String.class));

    log.info("Bean: {}", parser.parseExpression("@bean1").getValue(context));
  }
}
```

### 特殊处理

+ T  
  通过T(CLASS)指定类型，可以用来类型判断或者调用类静态方法
`
sec.setTypeLocator(new StandardTypeLocator(evalContext.getBeanFactory().getBeanClassLoader()));
`

+ @  
  通过@Name获取bean
`
  sec.setBeanResolver(new BeanFactoryResolver(evalContext.getBeanFactory()));
`

+ &  
  通过&Name获取beanFactory

`
sec.setBeanResolver(new BeanFactoryResolver(evalContext.getBeanFactory()));
`

+ \#  
  通过#{}标识表达式，主要在spring
`
  BeanExpressionResolver resolver = beanFactory.getBeanExpressionResolver();
  BeanExpressionContext context = new BeanExpressionContext(beanFactory, null);
`
---

从root取值： #root.name 或者 name(#root可以忽略)
从variables取值或方法：#var

### 扩展

下面以一个示例看下SpEL在我们项目中的具体应用：
```java
public class Tests{
  /**
   * 执行测试
   */
  @Test
    public void runCalc(){
        SpelExpressionParser parser = new SpelExpressionParser();
        StandardEvaluationContext context = new StandardEvaluationContext();
        context.setVariable("helper",new StudentHelper());
        context.setVariable("students", genStudents(100));

        log.info("max score: {}", parser.parseExpression("#helper.max(#students)").getValue(context, Double.class));
        log.info("min score: {}", parser.parseExpression("#helper.min(#students)").getValue(context, Double.class));
        log.info("avg score: {}", parser.parseExpression("#helper.avg(#students)").getValue(context, Double.class));
    }

  /**
   * 生成测试数据
   * @param count
   * @return
   */
  private List<Student> genStudents(int count){
        List<Student> students = new ArrayList<>();
        Faker faker = new Faker(Locale.CHINA);
        Name name = faker.name();
        Number number = faker.number();
        IntStream.range(0, count).forEach(i->{
            students.add(new Student(name.name(), number.randomDouble(3, 60, 100)));
        });
        return students;
    }

    @Data
    @AllArgsConstructor
    class Student{
        private String name;
        private double score;
    }

  /**
   * 工具类
   */
  class StudentHelper{

        public double max(List<Student> students){
            if(CollectionUtils.isEmpty(students)){
                return 0;
            }
            return students.stream().mapToDouble(Student::getScore).max().getAsDouble();
        }

        public double min(List<Student> students){
            if(CollectionUtils.isEmpty(students)){
                return 0;
            }
            return students.stream().mapToDouble(Student::getScore).min().getAsDouble();
        }

        public double avg(List<Student> students){
            if(CollectionUtils.isEmpty(students)){
                return 0;
            }
            return students.stream().mapToDouble(Student::getScore).average().getAsDouble();
        }
    }
}
```

在这个示例中，我们主要通过SpEL获取获取学生分值的最大值、最小值、平均值等方式，在实际的项目中使用时，绝非如此简单，比如我们在做数据统计时，对各项指标数值的计算就是通过SpEL实现，
因为具体功能的实现是由我们自己定义，因此在业务扩展上会非常的方便。

### 结束语

SpEL是一个功能非常强大的基于Java的解释型语言解析器，如果你想基于表达式的形式，对复杂结构数据计算或审计的需求时，不妨试试这个轻量级工具。