---
layout: post
title:  Java开发人员应该知道的Oracle-未完成
tagline: by 揽月中人
categories: DataBase
tags:
- 揽月中人
---
Oracle数据库使用广泛，大多数Java开发者都会用到Oracle。这里为大家准备了一些Oracle的基础知识，有助于理解Oracle的一些设计思想，以及数据库调优有着非常大的帮助，简易详细浏览一遍，对基本的架构有一个理解。
<!--more-->

### 1. Oracle整体架构

Oracle整体架构包含Oracle数据库正常运行的必需组件等。主要有实例（Instance），数据库（Database）

#### 1.1 实例（Instance）

数据库实例是由服务器上的一组内存结构以及进程组成。用来支撑、完成数据库的正常运行以及操作。

实例是可以独立于数据库存在的。其中实例包含了以下组件：

##### 1.1.1 内存

即服务器OS为当前Instance分配的内存区域。主要用来完成数据库内存的移动和操作。内存主要分为SGA(System Global Area) , PGA(Program or Process Global Area). 

SGA是实例范围内共享的，包含共享池，数据还冲，Redo缓冲等。 共享池包含库缓冲和字典缓冲。

PGA为各个会话私有的，

##### 1.1.2 后台进程（Background Process）

​	实例创建和维护的一组后台进程，其作用是完成数据中的统一管理和监控任务。进程是共享的，不属于某个或某些会话

##### 1.1.3 服务进程（Server Process） 

实例为数据库会话创建或分配，完成会话任务的Serve端服务进程。 其中在专用服务器模式和共享服务器模式下又有不同。

- 专用服务器模式: 该模式下，用户和数据库服务器建立会话，Instance会为本次会话创建一个服务进程，用以完成此会话任务。
- 共享服务器模式下，Instance会维护一组服务进程，Instance调度进程会将会话放入共享任务的队列中。该模式下所有的会话是共享一组服务进程的，也是一种池化思想。

#### 1.2 数据库（DataBase）

数据库是有服务器上的一组磁盘文件组成，存储着数据库相关的管理信息和用户数据，保证数据库的正常运转和用户数据的不丢失。数据库及其文件可以独立于Instance存在。

数据库中包含了许多类型的文件，主要有参数文件（Parameter File）、控制文件（Control File）、数据文件（Data File）、回滚文件（Undo File）、临时文件（Temp File）、重做日志文件（Redo Log File）、归档日志文件（Archive Log File）、警告日志文件（Alert Log File）、跟踪文件（Trace File）等

![image-20211122005454917](E:\javaNorth\javanorth\assets\images\2021\lyj\dataBaseSystemFile1.png)



下面是一个比较完整的Oracle架构图

![image-20211122005649252](E:\javaNorth\javanorth\assets\images\2021\lyj\oracle picture 2.png)

### 2. Oracle内存架构

### 3. Oracle存储架构
### 总结

