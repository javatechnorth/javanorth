---
layout: post
title:  你知道Spring Boot的彩蛋怎么设置吗？--20210709
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

文稿原地址： https://www.yuque.com/wbf1013/mglhnr/ah83wl

大家好，我是指北君。

### 前言

今天，我给大家来讲讲在 Spring Boot 项目中，自定义 banner 的事情。有些新入门的朋友可能会不知道 banner 是什么？它在哪里？ 我在哪里见过它吗？ 这3连门是不是很有意思。 我们今天所说的 banner 如下图所示，想必大家在启动 Spring Boot 项目的时候，大家都见过吧。

<!--more-->
![spring boot banner](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-banner1.png)

大家可能都见过永不宕机的佛祖的 banner 图片吧。 下图，大家应该都很熟悉吧。

![spring boot banner](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-banner2.png)

今天我就来带大家来看看源码，看看这个 banner 到底是怎么实现的。

### Spring Boot 内置 3 种 Banner打印方式

从下图可以看出，Spring Boot 支持打印的Banner 方式有3种， SpringBootBanner 是Spring Boot 默认的 Banner 打印方式， ResouceBanner 是文本类型的 Banner 打印方式， ImageBanner 是图片类型的 Banner 打印方式。

![spring boot banner 3](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-banner3.png)

我们来看看 Banner 接口，从下方的代码，我们可以了解到，Banner 接口包含一个 printBanner 的方法 和 Mode 的枚举，Mode 包含三种状态，其中 OFF 表示不打印 Banner ， LOG 表示把 Banner 输出到日志文件中， CONSOLE 表示输出到控制台，也是我们比较常见的方式。

```java
@FunctionalInterface
public interface Banner {
    void printBanner(Environment environment, Class<?> sourceClass, PrintStream out);
    enum Mode {
        OFF,
        CONSOLE,
        LOG
    }
}
```

### Banner 打印流程

从 SpringApplication 类的 run 方法可以看出，printBanner 方法在 prepareEnvironment 之后，这是因为 application.properties 中有一些关于 Banner 的配置项。需要先解析 application.properties 的值，并将其绑定到对应的 bean 之后，再进行后续的操作。

```java
public ConfigurableApplicationContext run(String... args) {
    ...
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
    ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
    configureIgnoreBeanInfo(environment);
    // 打印 Banner
    Banner printedBanner = printBanner(environment);
    ...
}
private Banner printBanner(ConfigurableEnvironment environment) {
    if (this.bannerMode == Banner.Mode.OFF) {
        return null;
    }
    ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
        : new DefaultResourceLoader(null);
    SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
    if (this.bannerMode == Mode.LOG) {
        return bannerPrinter.print(environment, this.mainApplicationClass, logger);
    }
    return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
```

printBanner 具体的流程如下：

1. 判断 bannerMode，如果是 OFF，表示不打印。如果是 LOG，表示打印到文件，否则打印到控制台。
2. SpringApplicationBannerPrinter 依据指定位置是是否存在文件，判断 Banner 类型是文本还是图片，文本类型使用 ResourceBanner, 图片类型使用 ImageBanner， 如果都不是，使用 Spring Boot 默认的 SpringBootBanner。
3. SpringApplicationBannerPrinter#print 方法调用 Banner 对象的的 printBanner 方法。不同类型的 Banner 的 printBanner 方法实现不同。

如下是 Banner 对象的获的方法，可以看出，Spring Boot 首先获得 ImageBanner，然后是 ResourceBanner，需要注意的是，这两者可以同时存在，此时会一次性打印两中 Banner。如果都不满足，还会去获得 fallbackBanner，这个是用户自己设定的 Banner，但是我们基本很少使用，大部分情况我们使用了 Spring Boot 内置的 SpringBootBanner。

```
class SpringApplicationBannerPrinter {
    private Banner getBanner(Environment environment) {
        Banners banners = new Banners();
        banners.addIfNotNull(getImageBanner(environment));
        banners.addIfNotNull(getTextBanner(environment));
        if (banners.hasAtLeastOneBanner()) {
            return banners;
        }
        if (this.fallbackBanner != null) {
            return this.fallbackBanner;
        }
        return DEFAULT_BANNER;
    }
    private Banner getTextBanner(Environment environment) {
        String location = environment.getProperty(BANNER_LOCATION_PROPERTY, DEFAULT_BANNER_LOCATION);
        Resource resource = this.resourceLoader.getResource(location);
        try {
            if (resource.exists() && !resource.getURL().toExternalForm().contains("liquibase-core")) {
                return new ResourceBanner(resource);
            }
        }
        catch (IOException ex) {
            // Ignore
        }
        return null;
    }
    private Banner getImageBanner(Environment environment) {
        String location = environment.getProperty(BANNER_IMAGE_LOCATION_PROPERTY);
        if (StringUtils.hasLength(location)) {
            Resource resource = this.resourceLoader.getResource(location);
            return resource.exists() ? new ImageBanner(resource) : null;
        }
        for (String ext : IMAGE_EXTENSION) {
            Resource resource = this.resourceLoader.getResource("banner." + ext);
            if (resource.exists()) {
                return new ImageBanner(resource);
            }
        }
        return null;
    }
}
```

### 如何关闭 Spring Boot 的 Banner

从 SpringApplication#printBanner 方法可以看出。当我们将 bannerMode 设置为 Banner.Mode.OFF 的时候，该方法返回 null，也就是此时不会打印 Banner。所以只要设置 bannerMode 为 OFF ，我们就能关闭 Banner 功能。

```java
private Banner printBanner(ConfigurableEnvironment environment) {
  if (this.bannerMode == Banner.Mode.OFF) {
    return null;
  }
  ...
}
```

第一种就是在启动代码中设置。如下所示。

```java
public static void main(String[] args) {
        SpringApplication app
                = new SpringApplication(HelloApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
}
```

第二种是在 application.properties 中配置 spring.main.banner-mode

```java
spring.main.banner-mode=off
```

这两种方式都可以关闭 Banner，那当它们同时存在的时候，哪个生效呢？我们可以这样分析，启动代码中调用 setBannerMode 方法，改变了 bannerMode 的值，之后 SpringApplication 对象执行 run 方法，在 run 方法中会解析 application.properties 的值，并将其绑定到对应的 bean，后者覆盖前者，所以 application.properties 中的配置优先级更高。

### 自定义文本类型的 Banner

Spring Boot 中自定义文本类型的 Banner 很简单，我们只要在 resources 下增加一个 banner.txt 就可以了。比如我想让 Banner 显示 永不宕机的佛祖雕像。那我就可以在 banner.txt 中增加以下文本。

```java
////////////////////////////////////////////////////////////////////  
//                          _ooOoo_                               //  
//                         o8888888o                              //  
//                         88" . "88                              //  
//                         (| ^_^ |)                              //  
//                         O\  =  /O                              //  
//                      ____/`---'\____                           //  
//                    .'  \\|     |//  `.                         //  
//                   /  \\|||  :  |||//  \                        //  
//                  /  _||||| -:- |||||-  \                       //  
//                  |   | \\\  -  /// |   |                       //  
//                  | \_|  ''\---/''  |   |                       //  
//                  \  .-\__  `-`  ___/-. /                       //  
//                ___`. .'  /--.--\  `. . ___                     //  
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //  
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //  
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //  
//      ========`-.____`-.___\_____/___.-`____.-'========         //  
//                           `=---='                              //  
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //  
//            佛祖保佑       永不宕机      永无BUG                　　//
////////////////////////////////////////////////////////////////////
```

打印结果是这样的

![spring boot banner 4](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-banner4.png)

如果我们想打印彩色的 banner 出来呢？在Spring Boot 中我们可以利用 AnsiColor 类来实现，我们可以在 banner.txt 中使用 AnsiColor 指定后续的文本的颜色。

```java
${AnsiColor.YELLOW}
////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//            佛祖保佑       永不宕机      永无BUG                    //
////////////////////////////////////////////////////////////////////
```

如下是最终的结果

![spring boot banner5](http://www.javanorth.cn/assets/images/2021/feng/spring-boot-banner5.png)

```java
public class ResourceBanner implements Banner {
    @Override
    public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
        try {
            String banner = StreamUtils.copyToString(this.resource.getInputStream(),
                    environment.getProperty("spring.banner.charset", Charset.class, StandardCharsets.UTF_8));
            for (PropertyResolver resolver : getPropertyResolvers(environment, sourceClass)) {
                banner = resolver.resolvePlaceholders(banner);
            }
            out.println(banner);
        }
        catch (Exception ex) {
            logger.warn(LogMessage.format("Banner not printable: %s (%s: '%s')", this.resource, ex.getClass(),
                    ex.getMessage()), ex);
        }
    }
    protected List<PropertyResolver> getPropertyResolvers(Environment environment, Class<?> sourceClass) {
        List<PropertyResolver> resolvers = new ArrayList<>();
        resolvers.add(environment);
        resolvers.add(getVersionResolver(sourceClass));
        resolvers.add(getAnsiResolver());
        resolvers.add(getTitleResolver(sourceClass));
        return resolvers;
    }
}
```
我们通过上述代码来看看，Spring Boot 为什么能打印出带颜色的 banner， ResourceBanner 将读取到的文本流转换为一串字符串。再通过4个解析器，来处理待输出的 Banner 内容。

四个解析器分别是：
- environment 对应 application.properties 中的配置;
- VersionResolver 解析 spring boot 版本，
- AnsiResolver 解析颜色或字体等样式配置，
- TitleResolver 解析当前应用的版本，名称等。

### Banner图在线生成

在线生成 banner 的地址：

- https://www.bootschool.net/ascii
- http://www.network-science.de/ascii/
- http://patorjk.com/software/taag//
- http://www.degraeve.com/img2txt.php

### 总结

本文主要讲述了Banner的输出打印过程，如何打印个性化的文本 Banner。ImageBanner 没在本文详细说明，有兴趣的朋友可以查看阅读源码，也比较简单，也有助于你了解 Java 如何实现图片转文本的知识点。通过本文的知识点，你也可以根据自己的要求来实现 Banner， 设置为 fallbackBanner, 从而达到完全个性化的自定义 Banner 的效果。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。

[EOF]