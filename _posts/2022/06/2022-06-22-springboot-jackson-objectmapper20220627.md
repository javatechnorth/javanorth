---
layout: post
title:  Spring Boot 自定义Jackson ObjectMapper --20220627
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在Spring Boot Web 项目中，当使用JSON格式接收数据和返回数据的时候，Spring Boot 默认使用一个ObjectMapper实例来序列化响应和反序列化请求。

在本文中，我们将看看配置序列化和反序列化选项的最常用方法。

<!--more-->

### 默认配置

默认情况下，Spring Boot的配置将禁用以下配置项。

- MapperFeature.DEFAULT_VIEW_INCLUSION 
- DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES 
- SerializationFeature.WRITE_DATES_AS_TIMESTAMPS。

举个例子

- 客户端将发送一个GET请求到我们的/coffee?name=Javazzz。
- 控制器将返回一个新的Coffee对象。
- Spring将使用ObjectMapper将我们的POJO序列化为JSON。

我们将通过使用String和LocalDateTime对象来示范定制选项。

```java
public class Coffee {

    private String name;
    private String brand;
    private LocalDateTime date;

   //getter and setter
}
```

我们还将定义一个简单的REST控制器来演示序列化。

```java
@GetMapping("/coffee")
public Coffee getCoffee(
        @RequestParam(required = false) String brand,
        @RequestParam(required = false) String name) {
    return new Coffee()
      .setBrand(brand)
      .setDate(FIXED_DATE)
      .setName(name);
}
```

默认情况下，这将是调用GET http://lolcahost:8080/coffee?brand=javazzz 时的响应。

```json
{
  "name": null,
  "brand": "javazzz",
  "date": "2022-06-16T10:21:35.974"
}
```

我们希望排除空值，并有一个自定义的日期格式（dd-MM-yyyy HH:mm）。这就是我们的响应结果。

```json
{
  "brand": "javazzz",
  "date": "06-11-2022 10:34"
}
```

在使用Spring Boot时，我们可以选择定制默认的ObjectMapper或覆盖它。我们将在接下来的章节中介绍这两个选项。

### 自定义默认的ObjectMapper

在本节中，我们将看到如何定制Spring Boot使用的默认ObjectMapper。

#### 1.应用程序属性和自定义 Jackson 模块

配置映射器的最简单方法是通过应用程序属性。

下面是配置的一般结构。

```xml
spring.jackson.<category_name>.<feature_name>=true,false
```

作为一个例子，下面是我们要添加的内容，以禁用SerializationFeature.WRITE_DATES_AS_TIMESTAMPS。

```java
spring.jackson.serialization.write-dates-as-timestamps=false
```

除了提到的特征类别，我们还可以配置属性的包含。

```xml
spring.jackson.default-property-inclusion=always, non_null, non_absent, non_default, non_empty
```

配置环境变量是最简单的方法。这种方法的缺点是，我们不能定制高级选项，比如为LocalDateTime定制日期格式。

在这一点上，我们会得到这样的结果。

```json
{
  "brand": "javazzz",
  "date": "2022-06-16T10:35:34.593"
}
```

为了实现我们的目标，我们将注册一个新的JavaTimeModule，用我们的自定义日期格式。

```java
@Configuration
@PropertySource("classpath:coffee.properties")
public class CoffeeRegisterModuleConfig {

    @Bean
    public Module javaTimeModule() {
        JavaTimeModule module = new JavaTimeModule();
        module.addSerializer(LOCAL_DATETIME_SERIALIZER);
        return module;
    }
}
```

另外，配置属性文件coffee.properties将包含以下内容。

```yaml
spring.jackson.default-property-inclusion=non_null
```

Spring Boot会自动注册任何类型为com.fastxml.jackson.databind.Module的bean。下面是我们的最终结果。

```json
{
  "brand": "Javazzz",
  "date": "16-06-2022 10:43"
}
```

#### 2.Jackson2ObjectMapperBuilderCustomizer

这个功能接口的目的是让我们创建配置豆。

它们将被应用于通过Jackson2ObjectMapperBuilder创建的默认ObjectMapper。

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer jsonCustomizer() {
    return builder -> builder.serializationInclusion(JsonInclude.Include.NON_NULL)
      .serializers(LOCAL_DATETIME_SERIALIZER);
}
```

配置豆以特定的顺序应用，我们可以使用@Orderannotation来控制。如果我们想从不同的配置或模块来配置ObjectMapper，这种优雅的方法是适合的。

### 重写默认配置

如果我们想完全控制配置，有几个选项可以禁用自动配置，只允许应用我们的自定义配置。

让我们仔细研究一下这些选项。

#### 1.ObjectMapper

覆盖默认配置的最简单方法是定义一个ObjectMapper Bean并将其标记为@Primary。

```java
@Bean
@Primary
public ObjectMapper objectMapper() {
    JavaTimeModule module = new JavaTimeModule();
    module.addSerializer(LOCAL_DATETIME_SERIALIZER);
    return new ObjectMapper()
      .setSerializationInclusion(JsonInclude.Include.NON_NULL)
      .registerModule(module);
}
```

当我们想完全控制序列化过程而不想允许外部配置时，我们应该使用这种方法。

#### 2.Jackson2ObjectMapperBuilder 

另一种干净的方法是定义一个Jackson2ObjectMapperBuilderbean。

实际上，Spring Boot在构建ObjectMapper时默认使用这个构建器，并会自动拾取定义的构建器。

```java
@Bean
public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
    return new Jackson2ObjectMapperBuilder().serializers(LOCAL_DATETIME_SERIALIZER)
      .serializationInclusion(JsonInclude.Include.NON_NULL);
}
```

它将默认配置两个选项。

- 禁用MapperFeature.DEFAULT_VIEW_INCLUSION 
- 禁用DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES。

根据[Jackson2ObjectMapperBuilder 文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)，如果classpath上有一些模块，它也会注册这些模块。

- jackson-datatype-jdk8：支持其他Java 8类型，如Optional  
- jackson-datatype-jsr310：支持Java 8 Date and Time API类型  
- jackson-datatype-joda：支持Joda-Time类型  
- jackson-module-kotlin：支持Kotlin类和数据类

这种方法的优点是，Jackson2ObjectMapperBuilder提供了一种简单而直观的方法来构建ObjectMapper。

#### 3.MappingJackson2HttpMessageConverter的方法

我们可以直接定义一个类型为MappingJackson2HttpMessageConverter的bean，Spring Boot会自动使用它。

```java
@Bean
public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
    Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder().serializers(LOCAL_DATETIME_SERIALIZER)
      .serializationInclusion(JsonInclude.Include.NON_NULL);
    return new MappingJackson2HttpMessageConverter(builder.build());
}
```

### 测试配置

为了测试我们的配置，我们将使用TestRestTemplate并将对象序列化为String。

通过这种方式，我们可以验证我们的Coffee对象在序列化时没有null值，并且具有自定义的日期格式。

```java
@Test
public void whenGetCoffee_thenSerializedWithDateAndNonNull() {
    String formattedDate = DateTimeFormatter.ofPattern(CoffeeConstants.dateTimeFormat).format(FIXED_DATE);
    String brand = "Javazza";
    String url = "/coffee?brand=" + brand;
    
    String response = restTemplate.getForObject(url, String.class);
    
    assertThat(response).isEqualTo("{\"brand\":\"" + brand + "\",\"date\":\"" + formattedDate + "\"}");
}
```

### 总结

在这篇文章中，我们看了使用Spring Boot时配置JSON序列化选项的几种方法。

我们看到了两种不同的方法：配置默认选项或重写默认配置。
