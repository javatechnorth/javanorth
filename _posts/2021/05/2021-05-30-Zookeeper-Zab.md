---
layout: post
title:  Zookeeper实现Zab协议详解
tagline: by 程序员七哥
categories: 分布式系统
tags: 
    - zookeepr
---




大家好，我是指北君。

之前说了立志不当最菜程序员，所以上次我们不是决定要学习下这个分布式都会用到的这个技术：Zookeeper嘛，也给大家聊了被称为是最难理解的 Paxos 算法。

不懂 Paxos 算法的那得看下我们之前的这篇文章：[Zookeeper基础Paxos算法详解](http://www.javanorth.cn/2021/05/30/Zookeeper-Paxos/)

今天咱指北君就更进一步，学习下这个 Zookeeper 实现的基础协议--**Zab协议（Zookeeper Atomic Broadcast）**，也称为zk原子广播协议。

首先大家可能会有疑问，**Zab和我们上一篇讲解的Paxos之间有什么区别和联系** 别着急，带着这个问题接着往下看吧，后面指北君会揭晓这个问题的答案的。

<!--more-->


## Zookeeper简介

那聊 Zab 协议，先得看下 Zookeeper 是个什么玩意。

Zookeeper 是一个分布式数据一致性的解决方案，分布式应用可以基于它实现诸如数据发布/订阅，负载均衡，命名服务，分布式协调/通知，集群管理，Master选举，分布式锁和分布式队列等功能。**Zookeeper 致力于提供一个高性能、高可用、且具有严格的顺序访问控制能力的分布式协调系统**。

好像确实挺牛逼的，能干这么多事呢，看来搞它是必须的呀！😮

考虑到 Zookeeper 主要操作数据的状态，为了保证状态的一致性，Zookeeper提出了两个安全属性:

1. 全序（Total order）：如果 消息a 在 消息b 之前发送，则所有 Server 应该看到相同的结果；
2. 因果顺序（Causal order）：如果 消息a 在 消息b 之前发生（a导致了b），并被一起发送，则 a 始终在 b 之前被执行；

为了保证上述两个安全属性，Zookeeper使用了 **TCP协议 和 Leader**：

1. 通过使用TCP协议保证了消息的全序特性（先发先到）；
2. 通过Leader解决了因果顺序问题：先到Leader的先执行，但是这样的话Leader有可能出现出现网络中断、崩溃退出与重启等异常情况，这就有必要引入Leader选举算法；

而 ZAB(Zookeeper Atomic Broadcast 即 Zookeeper 原子消息广播协议) 正是作为其数据一致性的核心算法，下面介绍一下ZAB协议。

## 二、什么是Zab协议

ZAB ，Zookeeper Atomic Broadcast，zk 原子消息广播协议，是专为 ZooKeeper 设计的一 种支持崩溃恢复的原子广播协议。在 Zookeeper 中，基于该协议，ZooKeeper 实现了一种主从模式的系统架构来保持集群中各个副本之间的数据一致性。



Zookeeper 使用一个单一主进程来接收并处理客户端的所有事务请求，即写请求。当服 务器数据的状态发生变更后，集群采用 ZAB 原子广播协议，以事务提案 Proposal 的形式广 播到所有的副本进程上。ZAB 协议能够保证一个全局的变更序列，即可以为每一个事务分配 一个全局的递增编号 xid。



当 Zookeeper 客户端连接到 Zookeeper 集群的一个节点后，若客户端提交的是读请求， 那么当前节点就直接根据自己保存的数据对其进行响应；如果是写请求且当前节点不是 Leader，那么节点就会将该写请求转发给 Leader，Leader 会以提案的方式广播该写操作，只 要有超过半数节点同意该写操作，则该写操作请求就会被提交。然后 Leader 会再次广播给 所有订阅者，即 Learner，通知它们同步数据。



![](/assets/images/2021/sevenluo/zookeeper服务工作原理.jpg)



## 三、Zab协议原理



Zab协议要求每个 Leader 都要经历三个阶段：**发现，同步，广播**。

1. **发现**：要求 zookeeper 集群必须选举出一个 Leader 进程，同时 Leader 会维护一个 Follower 可用客户端列表。将来客户端可以和这些 Follower 节点进行通信。
2. **同步**：Leader 要负责将本身的数据与 Follower 完成同步，做到多副本存储。这样也是提现了CAP中的高可用和分区容错。Follower将队列中未处理完的请求消费完成后，写入本地事务日志中
3. **广播**：Leader 可以接受客户端新的事务 Proposal 请求，将新的 Proposal 请求广播给所有的 Follower。



## 四、Zab协议核心

Zab协议的核心：**定义了事务请求的处理方式**

1. 所有的事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被叫做 **Leader服务器**。其他剩余的服务器则是 **Follower服务器**。
2. Leader服务器 负责将一个客户端事务请求，转换成一个 **事务Proposal**，并将该 Proposal 分发给集群中所有的 Follower 服务器，也就是向所有 Follower 节点发送数据广播请求（或数据复制）
3. 分发之后Leader服务器需要等待所有 Follower 服务器的反馈（Ack请求），**在Zab协议中，只要超过半数的Follower服务器进行了正确的反馈**后（也就是收到半数以上的Follower的Ack请求），那么 Leader 就会再次向所有的 Follower服务器发送 Commit 消息，要求其将上一个 事务proposal 进行提交。



![](/assets/images/2021/sevenluo/zk事务提交过程.jpg)



## 五、Zab协议内容

Zab 协议包括两种基本的模式：**崩溃恢复** 和 **消息广播**

### 1.协议过程

当整个集群启动过程中，或者当 Leader 服务器出现网络中弄断、崩溃退出或重启等异常时，Zab协议就会 **进入崩溃恢复模式**，选举产生新的Leader。

当选举产生了新的 Leader，同时集群中有过半的机器与该 Leader 服务器完成了状态同步（即数据同步）之后，Zab协议就会退出崩溃恢复模式，**进入消息广播模式**。

这时，如果有一台遵守 Zab 协议的服务器加入集群，那么因为此时集群中已经存在一个 Leader 服务器在广播消息，那么该新加入的服务器自动进入恢复模式：找到 Leader 服务器，并且完成数据同步。同步完成后，作为新的 Follower 一起参与到消息广播流程中。

### 2.协议状态切换

当 Leader 出现崩溃退出或者机器重启，亦或是集群中不存在超过半数的服务器与Leader保存正常通信，Zab 就会再一次进入崩溃恢复，发起新一轮Leader选举并实现数据同步。同步完成后又会进入消息广播模式，接收事务请求。

### 3.保证消息有序

在整个消息广播中，Leader 会将每一个事务请求转换成对应的 proposal 来进行广播，并且在广播 事务 Proposal 之前，Leader 服务器会首先为这个事务 Proposal 分配一个全局单递增的唯一ID，称之为事务ID（即zxid），由于 Zab 协议需要保证每一个消息的严格的顺序关系，因此必须将每一个 proposal 按照其 zxid 的先后顺序进行排序和处理。

## 六、崩溃恢复



**一旦 Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与过半 Follower 的联系，那么就会进入崩溃恢复模式。**

前面我们说过，崩溃恢复具有两个阶段：**Leader 选举 与 初始化同步**。当完成 Leader 选 举后，此时的 Leader 还是一个准 Leader，其要经过初始化同步后才能变为真正的 Leader。

###  初始化同步



![](/assets/images/2021/sevenluo/zab崩溃恢复.jpg)



具体过程如下：

1. 为了保证 Leader 向 Learner 发送提案的有序，Leader 会为每一个 Learner 服务器准备一 个队列；
2. Leader 将那些没有被各个 Learner 同步的事务封装为 Proposal；
3. Leader 将这些 Proposal 逐条发给各个 Learner，并在每一个 Proposal 后都紧跟一个 COMMIT 消息，表示该事务已经被提交，Learner 可以直接接收并执行 ；
4. Learner 接收来自于 Leader 的 Proposal，并将其更新到本地；
5. 当 Learner 更新成功后，会向准 Leader 发送 ACK 信息；
6. Leader 服务器在收到来自 Learner 的 ACK 后就会将该 Learner 加入到真正可用的 Follower 列表或 Observer 列表。没有反馈 ACK，或反馈了但 Leader 没有收到的 Learner，Leader 不会将其加入到相应列表。



## 七、恢复模式的两个原则



当集群正在启动过程中，或 Leader 与超过半数的主机断连后，集群就进入了恢复模式。 对于要恢复的数据状态需要遵循两个原则。

### 1. 已被处理过的消息不能丢

当 Leader 收到超过半数 Follower 的 ACKs 后，就向各个 Follower 广播 COMMIT 消息， 批准各个 Server 执行该写操作事务。当各个 Server 在接收到 Leader 的 COMMIT 消息后就会在本地执行该写操作，然后会向客户端响应写操作成功。

但是如果在非全部 Follower 收到 COMMIT 消息之前 Leader 就挂了，这将导致一种后 果：**部分 Server 已经执行了该事务，而部分 Server 尚未收到 COMMIT 消息**，所以其并没有执行该事务。当新的 Leader 被选举出，集群经过恢复模式后需要保证所有 Server 上都执行 了那些已经被部分 Server 执行过的事务。



### 2. 被丢弃的消息不能再现

当新事务在 Leader 上已经通过，其已经将该事务更新到了本地，但所有 Follower 还都没有收到 COMMIT 之前，Leader 宕机了（比前面叙述的宕机更早），此时，所有 Follower 根本 就不知道该 Proposal 的存在。当新的 Leader 选举出来，整个集群进入正常服务状态后，之 前挂了的 Leader 主机重新启动并注册成为了 Follower。若那个别人根本不知道的 Proposal 还保留在那个主机，那么其数据就会比其它主机多出了内容，导致整个系统状态的不一致。 所以，该 Proposa 应该被丢弃。类似这样应该被丢弃的事务，是不能再次出现在集群中的， 应该被清除。



## 八、消息广播

当集群中的 Learner 完成了初始化状态同步，那么整个 zk 集群就进入到了正常工作模式 了。

![](/assets/images/2021/sevenluo/zab广播.jpg)



如果集群中的 Learner 节点收到客户端的事务请求，那么这些 Learner 会将请求转发给 Leader 服务器。然后再执行如下的具体过程：

1. Leader 接收到事务请求后，为事务赋予一个全局唯一的 64 位自增 id，即 zxid，通过 zxid 的大小比较即可实现事务的有序性管理，然后将事务封装为一个 Proposal。
2. Leader 根据 Follower 列表获取到所有 Follower，然后再将 Proposal 通过这些 Follower 的 队列将提案发送给各个 Follower。
3. 当 Follower 接收到提案后，会先将提案的 zxid 与本地记录的事务日志中的最大的 zxid 进行比较。若当前提案的 zxid 大于最大 zxid，则将当前提案记录到本地事务日志中，并 向 Leader 返回一个 ACK。
4. 当 Leader 接收到过半的 ACKs 后，Leader 就会向所有 Follower 的队列发送 COMMIT 消息，向所有 Observer 的队列发送 Proposal。
5. 当 Follower 收到 COMMIT 消息后，就会将日志中的事务正式更新到本地。当 Observer 收到 Proposal 后，会直接将事务更新到本地。
6. 无论是 Follower 还是 Observer，在同步完成后都需要向 Leader 发送成功 ACK。







## 九、实现原理

### 1.三类角色

为了避免 Zookeeper 的单点问题，zk 也是以集群的形式出现的。zk 集群中的角色主要有 以下三类：

- Leader：接收和处理客户端的读请求；zk 集群中事务请求的唯一处理者，并负责发起决议和投票，然后将通过的事务请求在本地进行处理后，将处理结果同步给集群中的其它主机。
- Follower：接收和处理客户端的读请求; 将事务请求转给 Leader；同步 Leader 中的数据； 当 Leader 挂了，参与 Leader 的选举（具有选举权与被选举权）；
- Observer：就是没有选举权与被选举权，且没有投票权的 Follower（临时工）。若 zk 集 群中的读压力很大，则需要增加 Observer，最好不要增加 Follower。因为增加 Follower 将会增大投票与统计选票的压力，降低写操作效率，及 Leader 选举的效率。

这三类角色在不同的情况下又有一些不同的名称，在zookeeper源码中的定义，可以了解下，看源码可能会少点疑惑的。

- Learner = Follower + Observer
- QuorumServer = Follower + Leader

### 2.三个数据

在 ZAB 中有三个很重要的数据：

- zxid：是一个 64 位长度的 Long 类型。其中高 32 位表示 epoch，低 32 表示 xid。
- epoch：每个 Leader 都会具有一个不同的 epoch，用于区分不同的时期（可以理解为朝代的年号）
- xid：事务 id，是一个流水号，（每次朝代更替，即 leader 更换），从0开始递增。

每当选举产生一个新的 Leader ，就会从这个 Leader 服务器上取出本地事务日志中最大编号 Proposal 的 zxid，并从 zxid 中解析得到对应的 epoch 编号，然后再对其加1，之后该编号就作为新的 epoch 值，并将低32位数字归零，由0开始重新生成zxid。

### 3.三种状态

zk 集群中的每一台主机，在不同的阶段会处于不同的状态。每一台主机具有四种状态。

- LOOKING：选举状态
- FOLLOWING：Follower 的正常工作状态，从 Leader 同步数据的状态
- LEADING：Leader 的正常工作状态，Leader 广播数据更新的状态

> 代码实现中，多了一种状态：Observing 状态这是 Zookeeper 引入 Observer 之后加入的，Observer 不参与选举，是只读节点，实际上跟 Zab 协议没有关系。这里为了阅读源码加上此概念。

- OBSERVING：Observer 的正常工作状态，从 Leader 同步数据的状态



### 4.Zab 的四个阶段

- **myid**:这是 zk 集群中服务器的唯一标识，称为 myid。例如，有三个 zk 服务器，那么编号分别 是 1,2,3。
- **逻辑时钟**:逻辑时钟，Logicalclock，是一个整型数，该概念在选举时称为 logicalclock，而在选举结 束后称为 epoch。即 epoch 与 logicalclock 是同一个值，在不同情况下的不同名称。

#### 1).选举阶段（Leader Election）

节点在一开始都处于选举节点，只要有一个节点得到超过半数节点的票数，它就可以当选准 Leader，只有到达第三个阶段（也就是同步阶段），这个准 Leader 才会成为真正的 Leader。

**Zookeeper 规定所有有效的投票都必须在同一个 轮次 中，每个服务器在开始新一轮投票时，都会对自己维护的 logicalClock 进行自增操作**。

每个服务器在广播自己的选票前，会将自己的投票箱（recvset）清空。该投票箱记录了所收到的选票。

例如：Server_2 投票给 Server_3，Server_3 投票给 Server_1，则Server_1的投票箱为(2,3)、(3,1)、(1,1)。（每个服务器都会默认给自己投票）

前一个数字表示投票者，后一个数字表示被选举者。票箱中只会记录每一个投票者的最后一次投票记录，如果投票者更新自己的选票，则其他服务器收到该新选票后会在自己的票箱中更新该服务器的选票。

![](/assets/images/2021/sevenluo/zab-leader选举.jpg)



**这一阶段的目的就是为了选出一个准 Leader ，然后进入下一个阶段。**

#### 2). 发现阶段（Descovery）

在这个阶段，Followers 和上一轮选举出的准 Leader 进行通信，同步 Followers 最近接收的事务 Proposal 。

**这个阶段的主要目的是发现当前大多数节点接收的最新 Proposal，并且准 Leader 生成新的 epoch ，让 Followers 接收，更新它们的 acceptedEpoch**。

#### 3). 同步阶段（Synchronization)

**同步阶段主要是利用 Leader 前一阶段获得的最新 Proposal 历史，同步集群中所有的副本**。

只有当 quorum（超过半数的节点） 都同步完成，准 Leader 才会成为真正的 Leader。Follower 只会接收 zxid 比自己 lastZxid 大的 Proposal。

#### 4). 广播阶段（Broadcast）

到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 Leader 可以进行消息广播。同时，如果有新的节点加入，还需要对新节点进行同步。
需要注意的是，Zab 提交事务并不像 2PC 一样需要全部 Follower 都 Ack，只需要得到 quorum（超过半数的节点）的Ack 就可以。



## 十、Zab与Paxos

上面已经针对Zab协议涉及流程作了详细的描述，那么它和Paxos是什么关系呢？

Zab的作者认为Zab与paxos并不相同，之所以没有采用Paxos是因为Paxos保证不了全序顺序：

> Because multiple leaders can propose a value for a given instance two problems arise.
> First, proposals can conflict. Paxos uses ballots to detect and resolve conflicting proposals.
> Second, it is not enough to know that a given instance number has been committed, processes must also be able to fi gure out which value has been committed.



Paxos算法的确是不关心请求之间的逻辑顺序，而只考虑数据之间的全序，但很少有人直接使用paxos算法，都会经过一定的简化、优化。

Google的粗粒度锁服务Chubby的设计开发者Burrows曾经说过：“**所有一致性协议本质上要么是Paxos要么是其变体**”。这句话还是有一定道理的，ZAB本质上就是Paxos的一种简化形式。



**总结**:

本文主要讲解了Zab上述一系列巧妙的设计，比如：为了加快收敛速度避免活锁引发的竞争引入了Leader角色，在正常情况下最多只有一个参与者扮演Leader角色，其他参与者扮演Acceptor；在这种优化算法中，只有Leader可以提出议案，从而避免了竞争使得算法能够快速地收敛而趋于一致；而为了保证Leader的健壮性，又引入了Leader选举，再考虑到同步的阶段，提出了消息广播和崩溃初始化同步以及恢复模式的两个原则。

看完了Zab协议，就可以准备盘源码了，关注指北君，下期更精彩。


