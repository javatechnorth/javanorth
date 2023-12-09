
---
layout: post
title:  2023-11-08 
tagline: by 沉浮
categories: 
tags: 沉浮
---

## GraalVM

GraalVM是一个高性能的虚拟机，它以提高基于Java和JVM的应用的性能并简化Java云原生服务的构建和运行为目标。

GraalVM是一个高性能运行时环境，它基于OpenJDK, HotSpot和JRockit，并提供了在Java虚拟机上运行其他语言的能力，比如JavaScript，Python，Ruby等。

### GraalVM与JDK的对比

+ 性能  
GraalVM在性能方面相较于JDK有着显著的优势。GraalVM通过即时编译（JIT）技术实现了更低的启动时间和更高的执行速度。
通过JIT和AOT编译器，可以在运行时和部署时将字节码编译成本地机器代码，从而提高应用程序的性能。

+ 编译方式  
GraalVM提供了AOT编译器，可以在应用程序部署时将其编译成本地机器代码，从而进一步提高应用程序的性能和启动时间。而JDK在这方面的支持相对较弱。

+ 语言支持  
GraalVM支持多种编程语言，如JavaScript、Python、R等，这使得开发者可以使用熟悉的语言进行开发，降低了学习成本。而JDK主要支持Java语言。

+ 内存消耗  
GraalVM在内存消耗方面也优于JDK。GraalVM采用了低延迟垃圾回收（LLGC）技术，可以在不影响性能的情况下实现更高效的内存管理。

+ 兼容性  
GraalVM提供了与Java虚拟机（JVM）兼容的API，使得已经使用JVM的应用程序可以无缝迁移到GraalVM上。这使得GraalVM在兼容性方面具有优势。

+ 互操作性  
使用GraalVM Truffle，Java和其他支持的语言可以直接相互互操作，并在同一内存空间中来回传递数据。这种互操作性提高了不同语言之间的协作效率。

### GraalVM的使用场景

+ 微服务架构  
在微服务架构中，每个服务都需要独立部署和扩展。GraalVM可以作为服务的基础运行时环境，提供高性能和低内存消耗的支持。
同时服务之间的频繁调用，以及各个服务的部署，都依赖于服务本身的性能以及启动速度。

+ 云计算  
在云计算环境中，资源利用率是关键因素。GraalVM可以在有限的资源下提供更高的性能，降低云服务的运营成本。

+ 大数据处理  
大数据处理需要高性能和低延迟的计算能力。GraalVM可以有效地支持各种数据处理任务，提高数据处理速度。

### GraalVM的优缺点

+ 优点：

  - 高性能：GraalVM通过即时编译技术和多语言并行执行实现了高性能的运行时环境。
  - 低内存消耗：GraalVM采用低延迟垃圾回收技术实现了高效的内存管理。
  - 多语言支持：GraalVM支持多种编程语言，降低了开发者的学习成本。
  - 跨平台兼容性：GraalVM基于OpenJDK, HotSpot和JRockit，具有良好的跨平台兼容性。

+ 缺点：

  - 学习成本：虽然GraalVM支持多种编程语言，但对于Java开发者来说，需要学习新的编程模型和API。
  - 社区支持：相较于JDK，GraalVM的社区支持相对较弱。

### AOT与JIT

JIT（Just-in-Time，即时编译）和AOT（Ahead-of-Time，预编译）是两种主流的编译技术。

**JIT编译器**是在程序运行的时候进行编译，这个过程是动态的，
并且每次运行程序时都可能对代码进行重新编译。这样的编译方式能够支持更多的动态特性，峰值性能更高，更有利于调试。据说JIT编译可以拿到比AOT编译更多的运行时信息，从而做出更优化的决策。

**AOT编译器**则是在程序运行前就进行编译，这个过程是静态的。应用程序在安装的时候会通过dex2oat工具将dex文件预编译成ELF文件，这样在每次运行程序时，
因为代码已经被提前编译过，所以不需要再重新编译。这种方式使得应用的启动速度更快，资源占用也略微低一些。

值得一提的是，AOT和JIT也可以结合使用，以发挥各自的优势。例如在某些语言或框架中，可以使用静态AOT编译来提前将整个应用程序编译好，而在程序运行过程中则使用动态JIT编译来提升程序的运行效率。

### 准备

#### 下载GraalVM

直接进入官网，https://www.graalvm.org/downloads/，根据自己计算机系统类型选择对应版本即可。

我的电脑是WIN11，所以选择了Java 17 | Windows(x64),对应下载的版本为：graalvm-jdk-17.0.9+11.1

如果有安装SDKMAN，可以通过下面的命令安装：
```bash
sdk install java 21.0.1-graal
```

#### 配置GraalVM环境

从下载的graalvm文件名称也可以看出，其本身也是jdk，打开安装文件里的bin目录，可以看到也有java javac等等命令，可以将本地Java环境覆盖，当然这个不是必须的。
比如我使用IDEA时，选择项目对应的SDK即可。

```text
GRAALMVM_HOME= <your graalvm-jdk path>

PATH=%GRAALMVM_HOME%\bin
```

这样我们就可以在终端使用*native-image*命令了。

*通过native-image可以把Java代码编译为本地二进制可执行文件。本地可执行文件只包括运行时所需的代码，即应用程序类、标准库类、语言运行时和来自JDK的静态链接的本机码。*

#### 安装Visual Studio

可以通过Visual Studio Installer来安装，下载地址 https://visualstudio.microsoft.com/zh-hans/downloads/，
这里需要选择17.0以上的版本，否则后面会出现问题，我这里选择的是Visual Studio Community2022

![visual-studio](/assets/images/2023/sucls/11_08/visual-studio.png)

### 示例

+ 将一个Java文件生成一个可执行exe程序

1. 编写Java代码
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

2. 编译
```bash
javac  HelloWorld.java
```

3. 打包
```bash
native-image com.sucl.blog.vm.graalvm.HelloWorld
```

4. 测试
```bash
com.sucl.blog.vm.graalvm.HelloWorld.exe
```

---

+ 从Spring Boot 3.0就已经支持GraalVM原生镜像，这里通过一个基于Spring Boot的简单项目来看如何使用GraalVM

1.  引入依赖，这里需要使用native-maven-plugin将最终的jar打包成exe文件
```xml
<project>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.graalvm.buildtools</groupId>
                <artifactId>native-maven-plugin</artifactId>
                <version>0.9.25</version>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

2. 编写Java代码
```java
@RestController
@SpringBootApplication
public class GraalVMApplication {

    public static void main(String[] args) {
        SpringApplication.run(GraalVMApplication.class, args);
    }

    @GetMapping
    public String index(){
        return "hello GraalVM";
    }
}
```

3. 编译打包  
可以直接通过mvn命令：
```bash
mvn -Pnative clean native:compile -f pom.xml
```
或者通过右侧Maven选项，选择native插件，配置运行参数，直接运行即可

![compile](/assets/images/2023/sucls/11_08/compile.png)

整个打包过程比较长，由于需要将jar文件转换可直接运行的成二进制exe文件，整个编译的过程都会比较漫长。启动完成后可以看到这样的信息

![start-log](/assets/images/2023/sucls/11_08/start-log.png)

4. 测试

在项目target目录中，可以看到生成的exe文件，直接双击运行。以前几秒甚至几十秒才能启动的项目，现在实现了秒开...

通过GraalVM Native Image，可以将Java字节码直接编译成特定于平台的、自包含的本机可执行文件，从而实现更快的启动速度和更小的应用程序占用空间。java跨平台的特性好像也没了...

打开浏览器，输入http://localhost:8080/则可以看到与普通项目相同的效果

### 其他支持

通过官网可以看到，GraalVM Native除了支持上面的基于jar的构建，还有以下

![build](/assets/images/2023/sucls/11_08/build.png)

### 遇到的问题

+ 编译时出现如下问题

```text
Error: Error compiling query code (in C:\Users\Us\AppData\Local\Temp\SVM-9567697918431257239\AMD64LibCHelperDirectives.c). Compiler command ''C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\bin\Hostx64\x64\cl.exe' /WX /W4 /wd4201 /wd4244 /wd4245 /wd4800 /wd4804 /wd4214 '/FeC:\Users\Us\AppData\Local\Temp\SVM-9567697918431257239\AMD64LibCHelperDirectives.exe' 'C:\Users\Us\AppData\Local\Temp\SVM-9567697918431257239\AMD64LibCHelperDirectives.c'' output included error: [AMD64LibCHelperDirectives.c, C:\Users\Us\AppData\Local\Temp\SVM-9567697918431257239\AMD64LibCHelperDirectives.c(1): fatal error C1034: stdio.h: 不包括路径集]
Error: Use -H:+ReportExceptionStackTraces to print stacktrace of underlying exception
```
配置环境变量：
```text
INCLUDE:
C:\Program Files (x86)\Windows Kits\10\Include\10.0.16299.0\ucrt
C:\Program Files (x86)\Windows Kits\10\Include\10.0.16299.0\um
C:\Program Files (x86)\Windows Kits\10\Include\10.0.16299.0\shared
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\include

LIB:
C:\Program Files (x86)\Windows Kits\10\Lib\10.0.22000.0\um\x64
C:\Program Files (x86)\Windows Kits\10\Lib\10.0.22000.0\ucrt\x64
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\lib\x64

PATH:
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\bin\Hostx64\x64
```

### 扩展内容 

+ Spring Native

Spring Native是Spring Boot团队与GraalVM团队合作的产物，作为一个独立的子项目存在。它充分利用了GraalVM的特性，例如AOT编译，以提供更快的启动速度和更低的内存消耗。此外，Spring Native还支持将Spring Boot应用程序编译成本地可执行文件。
Spring Boot 3的Native则更侧重于与Spring Boot主体的整合，可能更深度地整合了Spring Boot的特性与功能。

https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#overview

+ Quarkus

传统的Java堆栈是为单体应用设计的，启动时间长，内存需求大，而当时还没有云、容器和Kubernetes的存在。Java框架需要发展以满足这个新世界的需求。

Quarkus的创建是为了使Java开发人员能够为现代的、云原生的世界创建应用程序。Quarkus是一个为GraalVM和HotSpot定制的Kubernetes
原生Java框架，由最佳的Java库和标准精心打造。其目标是使Java成为Kubernetes和无服务器环境的领先平台，同时为开发者提供一个框架，
以解决更广泛的分布式应用架构问题。

https://cn.quarkus.io/about/

