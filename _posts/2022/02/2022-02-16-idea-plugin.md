---
layout: post
title:  从零开始开发idea 插件
tagline: by feng
categories: idea
tags: 
    - feng
---

大家好， 我是指北君。<br />
<br />idea 现在已经成为 Java 开发者最常用的开发工具。 idea 上有丰富的插件来帮助我们进行程序开发。<br />
<br />那作为 Java 开发者的各位同学，有没有想过要开发一个属于自己的插件呢？<br />
<br />最近指北君正好在看这一块内容，对 idea 插件开发的流程进行一个简单的介绍。<br />

### 准备工作

<br />我们这里使用 idea 社区版进行开发，社区版的源码都是开放的，方便我们开发调试。 如果你安装了付费版，开发插件当然也没问题，但是idea 插件开发过程中，还是会自动下载 idea 社区版的源码到本地的。<br />
<br />社区版下载地址 [https://www.jetbrains.com/idea/download/#section=windows](https://www.jetbrains.com/idea/download/#section=windows)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644980782937-4b9e927d-fa89-4cb4-b96e-01225cdeb385.png)<br />
<br />idea 官方建议我们使用最新版本的社区
>  You can use either IntelliJ IDEA Community Edition or IntelliJ IDEA Ultimate as your IDE (it is highly recommended to use the latest available version). Both include the complete set of plugin development tools.

所以我们自己在开发的过程中，尽量的使用最新的版本去进行开发。<br />​<br />
### 创建工程

<br />这里简单说一下， 目前创建 idea 插件工程有三种方式， 分别是 Github Template， Gradle ，DevKit 。DevKit 是最早期的开发方式，官方比较推荐 Github Template 和 Gradle。 <br />​

我们这里通过创建 Gradle 项目的方式来开发插件。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644981570106-a1fe7e08-d199-4c9f-9abd-319bd356cc78.png)<br />新建项目 -> 选中 Gradle -> 选择项目 SDK 。 注意这里需要选择 JDK 11 版本以上的 JDK 。 我在开发过程中使用 Microsoft 编译的 JDK11 的版本JDK，最后编译生成项目的时候编译报错了。换了 JDK 版本之后就好了，这个问题比较诡异。<br />​

如果你想使用kotlin 进行开发的话，记得选中画圈中间的 Kotlin/JVM 。 如果想让Gradle  也使用 kotlin ， 那可以选择 kotlin DSL 构建脚本。 <br />
<br />选择完毕之后，进入下一步。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644981951872-08bc967e-c1f6-45c0-93f3-a9105215d3f1.png)<br />输入项目名称，包名之后，这个可以创建Java项目一致的。<br />​

点击完成，等待 idea 下载开发插件所需的依赖。可以需要等待几分钟，具体视网络情况而定。<br />

### 项目结构
项目结构大致如下
```cpp
plugindemo
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   └── resources
    │       └── META-INF
    │           └── plugin.xml
    └── test
        ├── java
        └── resources

```
plugin.xml 是我们需要重点关注的文件，插件的所有配置项都在这里面。<br />​<br />
```xml
<idea-plugin>
    <id>org.example.plugindemo</id>
    <name>pplugin Demo</name>
    <vendor  url="http://www.xx.com">seiko</vendor>

    <description><![CDATA[
    test ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest ittest it

    ]]></description>

    <!-- please see https://plugins.jetbrains.com/docs/intellij/plugin-compatibility.html
         on how to target different products -->
    <depends>com.intellij.modules.platform</depends>

    <extensions defaultExtensionNs="com.intellij">
        <!-- Add your extensions here -->
    </extensions>

    <actions>
        <!-- Add your actions here -->
        <group id="MyPlugin.GreetingMenu" text="Greeting" description="xxx">
            <add-to-group group-id="MainMenu" anchor="last"/>
            <action class="org.example.plugindemo.HelloAction" id="MyPlugin.Hello" text="hello" description="hello"/>
        </group>
    </actions>
</idea-plugin>
```
plugin.xml 中的 actions 节点，就是我们开发插件所有的操作入口。我这里把我的插件配置到主菜单上面。主要操作就是一个 helloAction。<br />

### 创建 Action
我们新建一个 class 继承 AnAction就行。 我自己开发的过程中继承了 DumbAwareAction ， DumbAwareAction本身继承了 AnAction。<br />​<br />
```kotlin
class HelloAction : DumbAwareAction() {

    override fun actionPerformed(e: AnActionEvent) {
        val project = e.getData(PlatformDataKeys.PROJECT);
        Messages.showMessageDialog(project, "Hello Kotlin", "Greeting", Messages.getInformationIcon());
    }
}
```
继承实现了 actionPerformed 方法。 弹出一个 Hello Kotlin的提示框。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644982914616-073b39c1-57ea-4e6b-8f12-440daee664a0.png)<br />
<br />我们需要对 Action 注册到 plugin.xml ， 可以在 class 上进行操作，进行“注册操作”。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644983084366-630f266c-02a0-499d-8559-f4afde63c5d3.png)<br />这种方式比较友好， 如果直接在plugin.xml 编辑的话，容易出错。<br />
<br />注册完了之后，我们就可以通过 Gradle 的 runIde 进行运行。 注意这里会独立启动一个IDE来运行你开发的插件， 需要注意下电脑的内存情况。 <br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644983245633-86c2a1fe-e236-4b2a-b5ff-593cbbac5230.png)<br />运行启动之后，我们能够看到主菜单上 有个 Greeting 的菜单栏，这里和我们在 plugin.xml 中的actions 配置项是一致的。 <br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644983303845-c2697596-8ba0-46fa-b599-4de492e99908.png)<br />点击 Hello ， 弹出一个Hello Kotlin 的消息框， 这里就是我们 HelloAction 的输出结果。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/555731/1644983323803-a5481b0f-2296-405d-8462-c6ebc27b09f3.png)<br />到这里我们可以算是开发完一个简单的插件了。如果你还想了解更多插件开发的事项，记得继续关注我的文章哦。<br />​<br />
### 总结
从0到1 的演示了如何进行 idea 的插件开发， 如果还需更加深入的学习如何开发插件，可以继续关注哦。
