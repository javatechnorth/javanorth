---
layout: post
title:  手把手教你如何在IDEA中配置Gradle
tagline: by IT可乐
categories: Gradle
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
之前的文章，我们介绍了Gradle的安装配置，既然准备工作已经完成了，那么接下来我们就来体验体验在IDEA如何用Gradle创建项目。
<!--more-->
### 1、谈谈开发工具

本篇博客我们将会以Intellij IDEA 开发工具为例，所以我这里简单谈谈我们所使用的开发工具。

目前大家做Java开发的，无外乎就两种，eclipse或者Intellij IDEA。eclipse是老牌的开发工具，可以说算是我们的初恋了，熟悉的界面，熟悉的快捷键，刚入坑时，不知道陪伴了我们多少个寂寞的夜晚。但是 Idtellij IDEA 的横空出世，也让很多人抛弃了初恋，转投新欢。我使用IDEA开发也有几年了，刚转过来时，也是各种不适应，但是过了这个适应期后，嗯，真香。

至于为什么真香，我这里说几点：

①、强大的整合能力，包括对spring，maven，gradle，kotlin....等等

②、丰富的插件，插件社区很繁华，你想要的都有

③、很多智能化的提示功能

④、搜索很强大

⑤、界面也比较美观，而且有各种皮肤设置

可能还有很多优点，我这里也没说全。当然也有缺点，相对于eclipse，更占内存，另外最重要的是，IDEA是收费的，所以，你懂得。

说那么多，还在用eclipse，可以试着用用IDEA，相信我，用了之后，你不会在换回eclipse的。

### 2、IDEA 创建Gradle项目

收，回到我们的正题，如何用 IDEA 创建我们的 Gradle 项目呢？

#### 2.1 new project

![](http://www.javanorth.cn/assets/images/2022/itcoke/gradle/gradle-02-01.png)

项目选择 Gradle，JDK 选择我们自己安装的，项目类型我们以创建Java工程为例。然后点击 Next

![](http://www.javanorth.cn/assets/images/2022/itcoke/gradle/gradle-02-02.png)

输入项目名称，然后点击 Finish 即可。

#### 2.2 gradle配置

需要在 IDEA 中配置gradle为我们本地安装的。

![](http://www.javanorth.cn/assets/images/2022/itcoke/gradle/gradle-02-03.png)

User Gradle from：选择 Specified location，然后后面的路径选择我们本地的gradle目录。

至此，一个gradle的Java项目就创建完毕了，然后只需要等待 gradle自己加载文件完成即可。

### 3、gradle项目结构

上一步创建完成后，项目目录如下：

![](http://www.javanorth.cn/assets/images/2022/itcoke/gradle/gradle-02-04.png)

左边是我们创建的 gradle01 项目目录，大家看一下，是不是和maven很像，没错，gradle和maven一样，也是基于【约定优于配置】的理念。

src/main/java：放置正式代码文件目录

src/main/resources：放置正式资源配置文件目录

src/test/java：放置测试文件目录

src/test/resources：放置测试配置文件目录

只有这种配置目录，gradle才能正确的帮助我们进行打包，编译，测试。

然后我们看看右边打开的build.gradle文件，如下：

```groovy
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

```

这其实是 groovy 语言的写法，这里关于这门语言的简单介绍，我们会在后面进行讲解

