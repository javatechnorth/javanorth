---
layout: post
title: 听说你还没学Spring就被源码编译劝退了？手把手带你编译Spring框架源码，让你的学习事半功倍
tagline: by 程序员七哥
categories: Java
tags:
- 程序员七哥
- Spring
---

大家好，我是指北君。


最近呀，有小伙伴提出 **自己在学习 Spring 的时候，这个源码环境有些搞不定**。 那这怎么能行，不能因为这点小困难就让小伙伴放弃呀。

这里咱就不在赘述读Spring源码的好处了吧，想干这行的应该都懂。

今天就是要带那些想要学习 Spring 源码的小伙伴，手把手带大家把这个源码编译好，这活尽量呀，给大家得整的漂亮点。

<!--more-->

这里说明下，这个操作过程不需要科学上网，但是，注意了，这里有个但是，你总得保证你的电脑能上网吧。

考虑到有的小伙伴可能因为网络原因可能访问不了 `github` 或者下载非常慢，那这种情况的小伙伴也可以选择看我那期一招搞定 `github` 访问慢的文章。

当然不搞也没关系，文章中涉及的源码和工具包指北君也都会打包下载好最后给到大家，那来吧，直接开搞。

### 下载Spring源码

这里有的小伙伴可能不知道源码在哪下载，没关系！这里的思路一般是我们要搞某个技术，第一反应就是先上它的官网。直接在搜索栏输入 Spring，就能看到它的官网了。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-1.png)

打开 Spring 的官网：[https://spring.io/](https://spring.io/)

Spring 全家桶包含的项目有很多，我们选择我们今天要搞得 Spring 框架：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-2.png)

这里你就可以看到最新的正式发布版本已经是 5.3.8 的，真的是更新的好快，反正咱们是学习（实际项目千万别用最新版本），这里我就带大家来编译最新的版本，这样大家也可以学习到最新的功能特性。

**当然了这个版本不重要，你也可以选择其他版本，咱今天是授人以渔，不是授人以鱼，重要的是学习这个方法，你看完后想编译啥版本就编译啥版本。**

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-3.png)

我们点击右上角的 github 图标就会跳转到 github 上面的源码项目了。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-4.png)

下载方式可以直接通过 git clone 或者 下载 zip 包自己解压下，我这里是直接下载 zip 包了。

下载地址：[https://github.com/spring-projects/spring-framework/releases](https://github.com/spring-projects/spring-framework/releases)

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-5.png)


### 阅读Spring的官方编译文档

其实呀，很多搞不定 Spring 源码编译的小伙伴，肯定很多都是去网上随便找个教程，结果各种踩坑，因为搜到的资料参差不齐，这点估计大家都深有体会了。

所以我们就要养成一个习惯，必须要看官方文档。这里也不是要求大家其它资料不看，毕竟也有很多优秀的博客，大家都是成年人，肯定是全都要喽。

官方文档一般都是英文，但是别害怕，我这个大学四级都考了三回的英语渣，都能勉强看，大家肯定都没问题了，况且咱可以借助翻译插件嘛。


如何编译 Spring 源码其实 Spring 官方提供了详细的文档，有离线版也有在线版，也就是开源项目都有的 `REDAME` 文件。

我这里选择离线版来演示吧。我们解压进入到下载好的 Spring 框架源码项目下，查看 `REDAME` 。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-6.png)

就会发现有一个 **build from source**  的章节。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-7.png)

点击这个链接，就会看到详细的编译构建 Spring 框架源码的文档了：[https://github.com/spring-projects/spring-framework/wiki/Build-from-Source](https://github.com/spring-projects/spring-framework/wiki/Build-from-Source)

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-8.png)

接下来，咱们就按照文档来编译 `Spring-Framework5.3.8` 版本的源码。 根据文档的说明，首先我们要有一个 `JDk8` 或者更高的版本环境，我电脑安装的是 `JDk11` 所以没啥问题。 然后要安装 Git，用来拉取源代码的，因为我们已经下载了源代码，就不管它了。 

### 修改配置

先别着急，在编译构建之前，教大家一绝招。这也是我在编译过程中实践出来滴心得。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-9.png)


#### 从本地下载gradle

因为第一次运行就会去下载 `gradle` 到本地，然后通过 `gradle` 来编译 `Spring` 源码。 这个就很慢了，外网服务器下载，你懂得，有被墙的风险，并且本身也贼慢。

那为什么开始编译构建就会自动下载 `gradle` 呢？从哪里下载？下载的版本是多少？

我们打开 `Spring` 源码的根目录：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-10.png)

配置文件的地址就是源码根目录的二级目录下的 `gradle-wrapper.properties` 文件。

打开文件包含下列内容：

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-6.8.3-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

- **distributionUrl**：表示的就是 gradle 的下载地址，默认配置的是一个远程 URL。

- **distributionBase**：下载后存放的目录，默认就是用户目录下的 `.gradle` 目录；
- **zipStoreBase**：解压后存放的目录


我在构建的过程中发现配置的下载地址下载非常慢，经常超时，甚至有时候被墙。

所以我们倒不如先下载到本地，然后修改配置文件从本地获取 `gradle`，这样一来编译构建就会快很多了。

我们只需要修改 `gradle` 的下载地址就好了，其他的配置项建议保持不变。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-11.png)


上述配置的意思就是如果我们运行 `gradlew` 会自动去 `file\:///Users/sevenluo/IdeaProjects/spring-simple/gradle-6.8.3-bin.zip` 这个目录下载一个 gradle 到当前用户的 `.gradle\wrapper\dists`目录 ,然后解压到当前用户的 `.gradle\wrapper\dists` 目录。

但是实际目录不是这个，这个命令还会自己生成一些目录。下图是指北君电脑实际 `gradle` 的下载解压目录：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-12.png)



我本地使用的是 `gradle-6.8.3-bin.zip` ,如果你本地网络没问题可以自行去官网下载，当然指北君也下载好了，最后也都会给大家。


#### 修改远程仓库

改完上面的配置也先别着急运行编译，现在解决了下载 `gradle` 慢的问题，但是编译开始会使用 gradle 从远程仓库下载大量相关依赖的包。

如果你采用默认配置，那么我可以负责任的告诉你，你绝对得等呀等呀等，等的黄花菜都凉了。而且还有很大可能会失败！原因也是因为默认仓库是国外的服务器，速度真的是没谁了，想屎，不烦躁都不行。

所以呀，这里我们要把远程仓库改为咱们国内的阿里云，这样就舒服多了，怎么改?

很简单，找到 `Spring` 源码根目录下的 `build.gradle`，打开编辑添加阿里云的仓库。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-13.png)



```java
maven { url 'https://maven.aliyun.com/repository/public' } // 阿里云
maven { url 'https://maven.aliyun.com/repository/gradle-plugin' } // 插件
```


那到这里，可以运行了？ 


**no！no！no！，hold on, hold on**


![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-14.png)

再打开 `Spring` 源码根目录下的 `settings.gradle` 文件，添加阿里云仓库。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-15.png)

```java
maven { url 'https://maven.aliyun.com/repository/public' } //阿里云
maven { url 'https://maven.aliyun.com/repository/gradle-plugin' } //插件
```


至此万事具备只欠东风了，就等我们开始编译。

当然上面这些配置优化是指北君在实践中自己总结出来的，如果你的网速够快你完全可以不用优化，直接运行下面的命令。

### 开始编译构建

我们编译构建 `Spring` 源码，一般都是要导入到 IDEA 里面进行测试或者阅读的。Spring 对于如何导入也提供了文档，是不是很贴心。当然也有导入 `Eclipse` 的文档，大家可以根据自己的需求来操作。我这里是用 IDEA 的，你如果导入 Eclipse 操作也都是基本上一样的。

导入 idea 的文档地址：[https://github.com/spring-projects/spring-framework/blob/main/import-into-idea.md](https://github.com/spring-projects/spring-framework/blob/main/import-into-idea.md)
 
![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-16.png)


根据文档的提示在导入 `idea` 之前需要去 Precompile `spring-oxm` ，也就是预编译 `spring-oxm` 这个项目。运行 `./gradlew :spring-oxm:compileTestJava`.

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-17.png)

预编译起来也很快，我电脑上一共耗时 23 毫秒。

然后我们还要预编译下 `spring-core` 这个项目，因为后面我们运行测试模块的时候需要用到里面的类。

执行 `./gradlew :spring-core:compileTestJava`.

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-18.png)



### 导入IDEA

1. 开始使用idea导入Spring源码，File -> New -> Project from Existing Souces...

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-19.png)

2. 选择 Spring 源码项目的目录，进而选择根目录当中的 `build.gradle` 文件导入;

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-20.png)


3. 导入 `idea` 之后会开始自动构建、建立索引，这个过程也是很漫长，你只能等，我这边用了不到7分钟，看电脑性能；

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-21.png)

4. 接下来我们对 `idea` 进行一些设置，不然每次 `idea` 运行都会通过 `gradle` 去编译运行。`gradle` 运行编译特别慢，需要改成idea自己编译运行.

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-22.png)

说明一下，这里不是一定要改，但是如果你不改用默认的则会特别慢，改成idea快的不止一点点。


到这，我们的 spring 框架源码就编译完了，为了检验我们的劳动成果，下面建一个 moudle 来测试一下。


### 验证测试

1. 在我们的 spring 源码项目下新建我们自己的测试module，如下图所示；

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-23.png)

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-24.png)

输入测试的 moudle 名：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-25.png)

2. 建好项目之后再 `gradle` 的配置文件中添加 `spring` 的依赖,相当于你建了一个maven项目，在pom文件中添加spring的依赖；


```java
implementation(project(":spring-context"))
```
 修改后的配置：
 
![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-26.png)
 
3. 写测试代码；

- 配置类的代码，用于指定扫描的bean；

```java
package com.sevenluo.spring.config;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("com.sevenluo.spring")
public class AppConfig {
//扫描ccom.sevenluo.spring包下面的所有bean
}
```

- 写一个sevice，被扫描的bean；

```java
package com.sevenluo.spring.services;

import org.springframework.stereotype.Service;

@Service
public class HelloService {
	public void hello() {
		System.out.println("Hello,sevenluo!");
	}
}
```
- 测试类代码；

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-27.png)


4. 右键运行——可能你会出现一些错误；比如博主这里就出现了`ava: 程序包jdk.jfr不存在` 错误；

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-28.png)

这个问题是因为 `jdk.jfr` 是 java9 以后才有的模块,而 spring 源码 5.3.8 版本已经使用这个包，所以我们需要设置我们项目配置的 jdk 版本高于9。

排查后，是因为 idea 配置的 java 编译的版本是8，改为11即可。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-29.png)

最后再次运行，就成功了。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/spring-source-30.png)

### 总结

今天主要分享了如何编译 spring-framework 源码如何编译构建，以及怎么导入到 IDEA 中，方便我们测试和阅读。

其中，在编译前我结合自己的实践经验，提出了一些需要优化的配置，极大的提高了编译构建的速度以及成功率。比如本地下载 gradle，配置阿里云仓库等。

正如开头的对话，我希望可以帮助想深入学习Spring，但是却苦于基础环境配置搞不太定的小伙伴。

在实践中如果遇到解决不了的问题，也欢迎留言告诉我，我看到后会第一时间和大家一起交流。

那本文如果对你有帮助，就点个赞吧，那是对我最大的鼓励！感谢大家的阅读。