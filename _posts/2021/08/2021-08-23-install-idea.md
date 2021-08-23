---
layout: post
title: 手把手教大家安装 Intellij IDEA
tagline: by feng
categories: JDK
tags: 
    - feng
---

大家好，我是指北君。

### 前言
今天指北君将手把手教大家怎么安装Java最强集成开发环境--Intellij IDEA。 Intellij IDEA 也被大家简称为IDEA。IDEA 是目前业界评价最好的Java集成开发环境，尤其表现在代码自动提示、代码重构、代码版本管理、代码分析等方面。

IDEA 分为社区版和付费版两个版本，我们现在是处于学习Java的阶段，社区版已经足够我们使用了。
<!--more-->
指北君以前都是用eclipse的，期间也和IDEA换来换去使用了好一阵子，最终IDEA胜利了。我甚至还为此购买了JetBrain 全家桶。前几天刚过期了，还没续上，打算等新的高级特性再续了。

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea1.png)

说了这么多，我们还是上学直接下载安装吧，这次我们也分Windows 和 Mac 版本的安装。

### Windows 用户安装

#### 下载IDEA

我们直接从官网下载软件，下载地址：https://www.jetbrains.com/idea/download/#section=windows

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea2.png)

Ultimate 版本就是付费版，可以免费试用30天，适用于web和企业开发用户。Community 是免费版本，适用于Java初学者和Android开发用户。

功能上大致差别如图所示：

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea3.png)

我们这里就下载免费版就好了。直接点击 Community那边的Download 按钮 即可下载。 文件包可能比较大，需要花个几分钟等一下。

#### 安装IDEA
双击运行 IDEA 安装程序，一步步傻瓜式的下一步就行了。

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea4.png)

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea5.png)

点击 Install 后，可能需要花个几秒到几分钟的时间等待他安装完成。

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea6.png)

就这样，我们顺利安装完成。

### Mac 用户安装

#### 下载IDEA

下载地址：https://www.jetbrains.com/idea/download/#section=mac 下载过程不再赘述，直接参考Windows的下载即可。

有一点可能要注意一下，Mac 有 Intel 和M1 两个版本，你需要选择正确的版本才能正常安装运行。

#### 安装IDEA

双击运行 IDEA 安装程序。

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea7.png)

鼠标按住左边的Intellij IDEA CE的图标，直接拖到 Applications 的文件夹，这样就安装完成了。

Mac安装软件可真方便啊。

### 来个hello world

注意： 我这里演示的是**Mac版本**，Windows 的用户可能有些差别，但是大同小异。放心直接参考使用就行。

我们打开 Intellij IDEA , 点击 New Project。

![](http://www.javanorth.cn/assets/images/2021/feng/install-idea8.png)

左边栏选择 Java， Project SDK 我这里选择的 Java 1.8， 这里反正你安装了什么就选择什么就行。点击Next 。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea9.png)
这里我们可以不用管，直接点击 Next。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea10.png)
输入Project name，我这里是输入hello。Project location 可以自己修改到想要保存的地方。点击Finish 进入下一步。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea11.png)
等待几秒钟之后我们可以看到下面这个界面。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea12.png)
我们在 src 上右键 选择 New -> Java Class
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea13.png)
输入Java的名字。 我这里输入的是 Hello，然后按键盘Enter键创建 Hello 类。注意 首字母大写，这是Java的规范，后期会讲到。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea14.png)
我们可以看到IDEA 已经帮我们创建好 Hello 类了。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea15.png)
输入下方代码，这个阶段先照抄就行啊。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea16.png)

在class 文件内，鼠标右键 选择 Run 'Hello.main()' 执行代码
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea17.png)
输出结果。
![](http://www.javanorth.cn/assets/images/2021/feng/install-idea18.png)

好了，到此，我们的hello world 也结束了。

### 总结
今天主要讲了Windows 和Mac 两个平台中 Intellij IDEA的安装，和做了一个hello world的演示。 指北君期待和大家一起成长。

本文的所有示例源代码都已上传到了 Github：
> https://github.com/javatechnorth/java-study-note
欢迎大家 Star 关注，后续会不断更新。
