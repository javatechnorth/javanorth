---
layout: post
title:  Java开发人员应该知道的Oracle -2021-11-26
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

![image-20211123234322831](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-instance.png)

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

![](https://www.javanorth.cn/assets/images/2021/lyj/dataBaseSystemFile1.png)



下面是一个比较完整的Oracle架构图

![image-20211122005649252](https://www.javanorth.cn/assets/images/2021/lyj/oracle picture 2.png)

### 2. Oracle内存架构

内存架构主要是说Oracle实例内存管理和使用相关的逻辑设计与实现等。这里我们概略地说一下SGA和PGA，让大家有一个粗略的概念。

#### 2.1 SGA（System Global Area）系统全局区

数据库实例启动时创建的一个共享内存区域。主要由共享池、数据缓冲、重做日志缓冲等诸多区域组成。

![image-20211123234123424](https://www.javanorth.cn/assets/images/2021/lyj/ORACLE-sga.png)

共享池是数据库实例中最重要、最复杂的共享内存区域，里面存储着数据库最重要的结构和信息。

数据缓冲做为用户数据的缓存区，在系统共享内存中暂存数据库的数据块，其实这块的设计是为了提高数据库的读写性能。

重做日志缓冲作为日志数据的缓冲区，在系统共享内存中暂存数据库重做日志数据，可提高日志数据的读写性能。

#### 2.2 PGA (Program  Global Area) 程序全局区

服务进程存储数据以及控制信息，以及完成相关任务的内存区域。 相对与其他区域来说，该区域为私有区域。程序全局区域分为包含Stack Space、HashArea、UGA等。

![PGA 20c](https://www.javanorth.cn/assets/images/2021/lyj/ORACLE-PGA.png)

- 共享服务器模式下，多个客户端用户共享服务进程，UGA被挪到了Large pool，PGA中只有stack space、hash area、bitmap merge area等。

- 专用服务器模式下，PGA包括 SQL工作区，Session memory，Private SQL Area 等



会话区（User Global Area UGA），为会话分配的内存区域，用于存储各种会话变量，例如会话登录信息以及会话需要的其他各种信息等。

SQL 工作区是为服务进程进行各种内存操作分配的PGA私有内存。比如Sort Area（排序区）用于数据排序功能（ORDER BY ,  GROUP BY 等）



### 3. Oracle存储架构

Oracle数据库最终还是使用磁盘作为存储媒介，针对Oracle数据库的存储组织、分配、管理等，我们介绍一下（块）block、（区间）extent、（段）segment、（表空间）tableSpace.

下图为个存储单元的关系示意图。

![image-20211124004114994](https://www.javanorth.cn/assets/images/2021/lyj/oracle-block-extent-segment.png)

#### 3.1 Block

Block是Oracle数据库读写的最小单元，Block size是系统层面块大小整数倍。2KB、4KB.....



![block 示意图](https://www.javanorth.cn/assets/images/2021/lyj/oracle-data-block.png)

- Header中包含块的一些通用信息，block的地址，segment类型等
- Table dictionary 记录了这个块里面含有那些rows
- Row dictionary 包含了rows（数据行）的一些信息



#### 3.2 Extent（区间）

区间是关于存储空间的一个逻辑单位，由多个连续的快组成，也是Oracle存储空间分配的最小单元，若某个数据库对象需要存储空间时，Oracle至少要为其分配一个区间。

区间在段（Segment）被创建或段空间扩展是被分配。

当段被清除（drop）时，区间所占用的存储空间会被释放，会被系统中其他对象所使用



#### 3.3 Segment（段）

段是由一组区间组成，包含了表空建内特定逻辑存储结构的所有数据。针对每个表，Oracle分配一个或者多个区间形成该表的数据段（data segment），对于每一个索引，Oracle分配一个或者给多个区间组成索引段（index segment）.

非分区表和非分区索引分别对应一个段，分区表和分区索引的每个分区或子分区对应一个段。

段是存储数据库对象数据的实体，是存放数据的真正逻辑结构和单元。

段可分为数据段（Data segment）、索引段（Index segment）、临时段（Temporary segment）、回滚段（Rollback segment）等。



#### 3.4 TableSpace（表空间）

Oracle数据库中最大的存储空间相关的逻辑概念和容器，存储系统和用户数据的段都是在表空间中分配的。 表空间是共享资源，不同用户或段可以存储在同一个表空间，也可以存储在不同的表空间中。

Oracle将数据逻辑存储在表空间中，物理存储则在与表空间对应关联的数据文件中。

Oracle数据库有一个或者多个表空间的逻辑存储单元组成，这些表空间共同存储所有的数据。

Oracle中的每一个表空间有一个或者多个数据文件（data file）组成，这些数据文件与运行Oracle的系统的屋里存储结构相匹配。

表空间分为数据表空间（Data Tablespace）、临时表空间（Temporary Tablespace）、回滚表空间（Undo Tablespace）。



