---
layout: post
title:  Zookeeper 源码分析：Leader选举
tagline: by 七哥
categories: 分布式系统
tags: 
    - 七哥
    - zookeeper
    - 源码分析

---

大家好，我是指北君。

Zookeeper作为一个开源的分布式应用协调系统，简称 ZK。已经用到了许多分布式项目中，用来完成统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等工作，使用场景也是非常多。

面试中如何你能够对原理讲解清楚，甚至从源码的角度谈一谈，那征服面试官想必也是水到渠成的事情。

<!--more-->

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-0.gif)


如果你对 Zookeeper是什么,能做什么,以及 Paxos 算法和底层使用的 Zab 协议还不了解的话，那么建议先看下这两篇文章，先懂原理再看源码才是正确的路线。

[Zookeeper基础Paxos算法详解](http://www.javanorth.cn/2021/05/30/Zookeeper-Paxos/)
[Zookeeper实现Zab协议详解](http://www.javanorth.cn/2021/05/30/Zookeeper-Zab/)

接下来本文就开始真正走进Zookeeper底层源码的学习。俗话说授人以鱼不如授人以渔，今天我们就来分析下Zookeeper非常重要的一个环节：**Leader选举**。



### 前言

首先将一些基本的概念做一个解释，这样就不会看起来云里雾里了。

#### 基本概念

**SID**：服务器ID，用来标示ZooKeeper集群中的机器，每台机器不能重复，和myid的值一致

**ZXID**：事务ID

**Vote**: 选票，具体的数据结构后面有

**Quorum**：过半机器数

**logicalclock**：逻辑时钟（选举轮次），zk服务器Leader选举的轮次

**服务器类型**

在zk中，引入了Leader、Follwer和Observer三种角色。zk集群中的所有机器通过一个Leader选举过程来选定一台被称为Leader的机器，Leader服务器为客户端提供读和写服务。Follower和Observer都能够提供读服务，唯一的区别在于，Observer机器不参与Leader选举过程，也不参与写操作的过半写成功策略。因此，Observer存在的意义是：在不影响写性能的情况下提升集群的读性能。

**服务器状态**

**LOOKING**：Leader选举阶段；

**FOLLOWING**：Follower服务器和Leader保持同步状态；

**LEADING**：Leader服务器作为主进程领导状态；

**OBSERVING**：观察者状态，表明当前服务器是Observer，不参与投票；

选举的目的就是选择出合适的Leader机器，由Leader机器决定事务性的Proposal处理过程，实现类两阶段提交协议（就是ZAB协议）。



### 启动选举主流程


> 在zk服务器集群启动过程中，经 QuorumPeerMain 中，不光会创建 ZooKeeperServer 对象，同时会生成 QuorumPeer 对象，代表了 ZooKeeper 集群中的一台机器。在整个机器运行期间，负责维护该机器的运行状态，同时会根据情况发起Leader选举。

QuorumPeer 是一个独立的线程，维护着zk机器的状态。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-1.jpg)


本次主要介绍的是选举相关的内容，之后的行文都是 从startLeaderElection中衍生出来的。

#### QuorumPeer维护集群机器状态


QuorumPeer的职责就是不断地检测当前的zk机器的状态，执行对应的逻辑，简单来说，就是根据服务所处的不同状态执行不同的逻辑。为了避免篇幅过长，影响阅读体验，删除了一部分逻辑后，代码如下：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-2.jpg)

当机器处于 LOOKING 状态时，QuorumPeer会进行选举，但是具体的逻辑并不是由QuorumPeer来负责的，整体的投票过程独立出来了，从逻辑执行的角度看，整个过程设计到两个主要的环节：

- 与其他的zk集群机器通信的过程

- 实现具体的选举算法

QuorumPeer 中默认使用的选举算法是 FastLeaderElection。

### 选举过程中的整体架构


zk提拱多种选举算法 不过之前版本的都废弃掉了，一般默认使用FastLeaderElection 也就是在配置文件中设置 electorArg=3。在集群启动的过程中，QuorumPeer会根据配置实现不同的选举策略：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-3.jpg)


#### QuorumCnxManager

如果ClientCnxn是zk客户端中处理IO请求的管理器，QuorumCnxManager是zk集群间负责选举过程中网络IO的管理器，在每台服务器启动的时候，都会启动一个QuorumCnxManager，用来维持各台服务器之间的网络通信。

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-4.jpg)


QuorumCnxManager、 Listener、 SendWorker、 RecvWorker 的分工很明确。准确的说 QuorumCnxManager 这个类的职责也很明确，就是负责监听端口、发消息、读消息 其中：

- Listener 监听连接，维护与其他服务器的连接；

- SendWorker 负责根据Listener保存的连接信息 向对应的server发送（投票）信息；

- RecvWorker 获取其他server的（投票）信息 并存入队列；



对于每一台zk机器，都需要建立一个TCP的端口监听，在 QuorumCnxManager 中交给 Listener 来处理，使用的是 Socket 的阻塞式IO（默认监听的端口是3888，是在config文件里面设置的）。

在两两相互连接的过程中，**为了避免两台机器之间重复地创建TCP连接**，zk制定了连接的规则：**只允许SID打的服务器主动和其他服务器建立连接**。

实现的方式也比较简单，在 receiveConnection 中，服务器会对比与自己建立连接的服务器的SID，判断是否接受请求，如果自己的 SID 更大，那么会断开连接，然后自己主动去和远程服务器建立连接。这段逻辑是由 Listener 来做的，且 Listener 独立线程。核心代码如下：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-5.jpg)

QuorumCnxManager 这里只负责与其他 server 的信息交换 但不负责信息的生成与处理 数据的处理就要交给对应的选举算法进行处理了。

以上内容主要是建立各台zk服务器之间的连接通信过程，具体的选举策略zk抽象成了Election，主要分析的是 **FastLeaderElection** 方式（选举算法的核心部分）：

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-6.jpg)


### 正式选举（FastLeaderElection选举算法）


上面QuorumPeer在一直循环的检测当前主机的状态，如果是Looking状态，就会进行新一轮的选举，通过：
```java
setCurrentVote(makeLEStrategy().lookForLeader());
```
也就是FastLeaderElection的lookForLeader来进行leader选择,实现代码不多，不过有些地方还是不好理解的。。。


> talk is cheap, show me code!



接下来，我们将 `org.apache.zookeeper.server.quorum.FastLeaderElection#lookForLeader` 方法中的逻辑分为5个步骤来理解，这样我认为比直接看一整段代码效果好，如果你觉得这样看比较碎片，可以打开源码对照我们这里梳理的流程，在整体梳理几遍（看源码一定得多啃几遍，一遍基本上搞下不来）。



#### 1. 创建选举对象，做选举前的初始化工作


![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-7.jpg)


2. 将自己作为新的Leader投出去（我选我）


![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-8.jpg)



这里需要注意的是更新选票时获取的epoch是当前Server的epech，即上一轮leader的epoch；

着重看一下sendNotifications方法，这里是将当前机器的选票发送给所有参与投票的机器，不包括Observer：



![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-9.jpg)



#### 3. 验证当前自己的选票与大家的选票谁更适合做Leader


![](https://files.mdnice.com/user/3825/54731876-b7c8-44d7-9ac6-61672cb81187.jpg)


这里有一些点还是比较难以理解的，比如当从票箱取出的选票为null时需要判断当前服务器是集群其他服务器是否保持连接状态，不过都已经标注了详细的注释，相信仔细看两遍一定可以理解的。

这里的 recvqueue 就是所有收到其它服务器投票后的票箱（带头结点的单向链表），recvqueue.poll 即取出第一票，这里我们看下poll方法做的操作：


![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-10.jpg)

一目了然，就是将链表头的next指针指向自己，即删除头节点，然后将head 指向之前头节点的next，也就是下一个元素，返回第一个元素的值，将当前第一个元素置为null，也就是新的头节点。



#### 4. 判断本轮选举中否应该结束了


![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-11.jpg)

到了这一步，开始遍历当前服务器收到的选票中是否已经有过半的参与者选择了当前服务器的选票（经过上面的步骤，当前服务器选票已经修改为最合适的），我们一起看下 `org.apache.zookeeper.server.quorum.FastLeaderElection#termPredicate` 方法:


![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-12.jpg)

如果当前选票没有过半，直接break继续取下一票进行判断，这个很好理解。

可是问题来了，如果已经过半了，后面的这个步骤为什么还要取下一票在和当前选票比看谁更适合呢？

我们一起来看下面的代码：


![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-13.jpg)


我初次看的时候也是难以理解，为什么取出下一票后判断比当前选票更合适后要在将选票放回去，然后break呢？

上面的代码我已经写了注释，这个while 循环的目的是要遍历完票箱防止有比当前更合适的选票， 如果 n==null 则说明没有找到任何比当前“过半选票更合适的选票”，进行收尾工作，修改当前主机状态：

```java
 proposedLeader == self.getId()) ?
         ServerState.LEADING: learningState()
```

然后清空队列，返回最终选票。



如果剩下的选票中有比自己更合适的则将其放回票箱，重新走一遍前面的流程，修改当前选票广播。

说明：票箱也就是当前接收选票的容器 recvset，本质是一个 HashMap，key 为投票者的 serverId ，所以收到多次投票也只是更新选票而已，设计很是巧妙呀！



#### 5. 无需选举的情况

![](http://www.javanorth.cn/assets/images/2021/sevenluo/zk-source-14.jpg)

最后这块的代码虽然不多，可是却是最难理解的，上面的注释中分析了为什么选举过程中可以收到通知发送者状态为 FOLLOWING, LEADING, OBSERVING 的情况，结合注释还得仔细的看几遍，其实就是为了处理下面这三种情况下的选举状态：

1. 新的Server(非Observer)加入到正常运行的集群

2. 当Leader挂了，并不是所有follower都同时能够感知到leader挂了，先感知到的server会发送通知给其它server，但由于其它server还未感知到，所以它们发送给这个server的通知状态就是FOLLOWING

3. 本轮选举中其它Server已经选举出了新的leader，但还没有通知到当前server，这些已经知道leader选举完毕的server向该server发送的通知就是LEADING或FOLLOWING



### 总结


以上就是zk的默认选举流程，按照ZAB协议的两种状态分析：

1. 初始化的时候，处于同一轮次进行投票直到投票选择出一个Leader

2. 崩溃恢复阶段：

- Leader服务器挂了，那么经历的和初始化流程类似的过程，选择Leader

- Follower服务器挂了，那么自己在执行选举的过程中，会收到其他服务器给的Leader选票信息（对应上文无需选举情况中的分支代码），也可以确定Leader所属


文章主要介绍了 Zk 的 leader 选举过程中的代码逻辑，包括机器宕机重启以及集群初始化时 QuorumPeer 都会检测到机器的状态为 LOOKING，然后调用 FastLeaderElection 的 lookForLeader 方法进行 leader 选举。 这块的代码虽然不多，可是理解起来还是有一定的难读的，建议大家结合本文多度几遍，加深印象。


欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。
