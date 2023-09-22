---
layout: post
title:  2023-09-01 SpringBoot启动参数
tagline: by 沉浮
categories: 
tags: 沉浮
---


<!--more-->
## Spring BOOT 启动参数

在Java Web的开发完成后，以前我们都会打包成war文件，然后放大web容器，比如tomcat、jetty这样的容器。
现在基于SpringBoot开发的项目，我们直接打包成jar文件，基于内嵌的tomcat来实现一样的效果。

而启动的方式变成了这样：
```bash
java [ options ] -jar *.jar [ arguments ]
```
### 常见配置

我们常见的配置有：

1. --server.port：指定应用程序的端口号。
2. --spring.profiles.active：设置应用程序使用的配置文件中的环境配置。
3. --spring.config.additional-location：指定额外的配置文件路径。
4. --Xms：设置JVM初始堆大小。
5. --Xmx：设置JVM最大堆大小。
6. --XX:PermSize：设置JVM永久代大小。
7. --XX:MaxPermSize：设置JVM最大永久代大小。
8. --Xdebug：开启远程JDWP调试。
9. -D：定义属性。

### options

在启动参数中，我们可以通过添加这样的配置，来覆盖系统属性中的值：
```bash
java -Dfile.encoding=UTF-8 -jar app.jar 
```
在代码中可以通过这样获取该值：
```
String fileEncoding = System.getProperties("file.encoding"); //UTF-8
```
在很多项目中，都会基于*System.getProperties()*来控制代码流程，这里要注意，通过启动参数配置的值优先级会大于系统中的配置。
同时注意改配置出现的位置，在上面使用了**options**位置来进行区分。

### arguments

在SpringBoot项目中，我们一般把配置都会写在application.yml文件中，随着项目一并打包到jar文件中，在生产环境中，
启动项目时通过添加*--spring.config.location=<path>/application.yml*来修改项目的配置文件指向，从而实现覆盖application的效果。

同样，我们可以通过配置启动参数来覆盖application中的某个配置项，比如：
```bash
java -Dfile.encoding=UTF-8 -jar app.jar --server.port=8080 
```
可以在main方法的参数中获取该值
```
log.info(">>>>> args: {}", Arrays.toString(args) );
```

参数的位置在上面对应**arguments**位置。

### 优先级

系统参数或环境变量：

 + 启动配置
 + set prop=value （export prop=value）
 + 系统中配置的参数或环境变量

Spring中的配置：

 + 启动参数
 + --spring.config.location=application.yml
 + classpath:application.yml

### EnvironmentAware

在Spring中，提供了一个Aware接口**EnvironmentAware**，通过该接口我们可以很方便地获取上面说的那些参数，不用关心是系统属性、环境变量还是main方法的args。

```java
public class MyService implements ApplicationContextAware, EnvironmentAware {
    
    @Override
    public void setEnvironment(Environment environment) {
        // 可以读取System properties|env 数据；系统参数
        log.info(">>>>> 从系统属性中取值: {}", environment.getProperty("file.encoding") );
    }
}
```

通过观察SpringBoot启动流程中，其中在SpringApplication的run方法中，可以看到系统环境属性加载过程
```
ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
```

基于StandardEnvironment的扩展。
```java
public class StandardEnvironment extends AbstractEnvironment {
    
	public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
    
	public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";
    
	@Override
	protected void customizePropertySources(MutablePropertySources propertySources) {
		propertySources.addLast(
				new PropertiesPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
		propertySources.addLast(
				new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
	}

}
```
我们看到的这样的写法其实就是基于SpEL对PropertySources的资源的解析：
```
    @Value("#{systemProperties['file.encoding']}")
    private String fileEncoding;

    @Value("#{systemEnvironment['JAVA_HOME']}")
    private String javaHome;
```

### 读取配置顺序

1. 默认配置：Spring Boot 默认提供了一些基本的配置，如应用程序的端口号、上下文路径等。这些配置位于 SpringBoot jar 包中的默认配置文件中。
2. 用户自定义配置：如果应用程序中有自定义的配置文件，Spring Boot 会首先加载这些文件。用户可以通过在应用程序的 classpath 下放置一个名为 application.properties 或 application.yml 的文件来提供自定义配置。
3. 命令行参数：在启动应用程序时，可以通过命令行参数来传递配置。这些参数会被加载并覆盖默认配置和用户自定义配置。
4. 环境变量：环境变量也可以提供配置信息。如果应用程序中定义了环境变量，它们将被加载并覆盖默认配置、用户自定义配置和命令行参数。
5. 系统属性：系统属性也可以提供配置信息。如果应用程序中定义了系统属性，它们将被加载并覆盖默认配置、用户自定义配置、命令行参数和环境变量。

Springboot会先加载PropertiesPropertySourceLoader，后加载YamlPropertySourceLoader，所以先读取的是properties文件。

### 结束语

了解SpringBoot配置加载相关知识，可以有效解决配置项不生效问题以及需要快速修改配置项的情况。

需要注意的是，在加载多个配置文件时，如果有冲突的配置项，后加载的配置文件中的配置项将覆盖先加载的配置文件中的相同配置项。因此，在应用程序中，应该避免使用相同的配置项名来定义不同的配置值。
