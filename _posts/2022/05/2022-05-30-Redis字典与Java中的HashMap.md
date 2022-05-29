---
layout: post
title: Redis中的哈希表 VS Javaz中HaspMap
tagline: by 揽月中人
categories: Redis
tags:
- 揽月中人
---

哈喽，大家好，我是指北君。  

之前给大家介绍了Redis的基本数据结构，本篇介绍一下Redis 字典的rehash 过程。并对比Java中HashMap的一些异同。



<!--more-->

### 1.前言

我们回顾一下之前讲到的Redis的字典结构，示意图如下：

![image-20220528013858288](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-01.png)

Redis的字典本质上来说也是数组+链表的数据结构，这与Java中HashMap的数据结构很类似啦。

由上述结构示意图也能看出，字典**dict**中维护了一个**ht**数组，而且只有两个元素，这两个元素是其扩容的关键点，这个我们后面会讲到。



Redis中的哈希对象在以下条件时，使用ziplist编码，

- 哈希对象保存的所有键值的字符串长度都小于64字节
- 哈希对象保存的键值对数量小于512个。

否则哈希对象会使用hashtable编码， 而hashtable则时使用了字典作为底层实现的。

如下redis 哈希对象编码由ziplist 变成hashtable

![image-20220529231639915](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-02.png)





### 2.增加元素与键冲突

当不同的键值经过哈希算法与散列算法之后被分配到了同一个哈希表数组的同一个索引上，那么这之后就会有键冲突。

Redis 哈希表解决哈希冲突同样是使用了链表地址法。使用哈希节点的next指针来链接同一个哈希表数组索引上的元素。不过Redis会将新添加的哈希节点加入到链表的表头位置。

如下所示： 如果程序要将键值对 （k2 , v2 ） 添加到如下的哈希表中，而且计算的书的索引为1，那么和 （k1 v1） 将产生冲突。 解决冲突时，会将两个节点使用next指针链接起来。而且会将新节点添加到链表表头的位置。

![哈希表1](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-03.png)





![链表解决hash冲突之后的哈希表](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-04.png)



### 3.rehash 扩容过程

哈希表不断的增加元素，其元素数量达到一定的比例之后，程序会对哈希表进行相应的扩展。通过执行rehash （重新散列）操作完成操作。其步骤如下：

- 执行扩展操作时会将字典中的ht[1] 哈希表大小设置成 **第一个大于等于** ht[0] 的 **ht[0].used * 2** 的 2^n (2的n次幂)
- 将保存早ht[0] 中的所有的键值对 rehash到ht[1] 上， rehash过程中会重新计算哈希值和索引值。
- 当ht[0]中所有的键值对都迁移到ht[1]上时，释放ht[0], 并将ht[1] 设置成 ht[0]， 并在ht[1]上建一个空的哈希表。



将下图中的字典做rehash操作：

![image-20220530000804106](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-05.png)





1. ht[0].used 是4，4*2 = 8 ，2的3次方8 是第一个大于**4**   的 2的n次幂。即程序会将ht[1] 的大小设置成8 ，并分配空间，结构示意如下： 

![image-20220530001049654](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-06.png)





2. 将ht[0] 上的几个键值对全部都rehash到ht[1] 上面，如下图：

![image-20220530001601222](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-07.png)





3. 释放ht[0]，并将ht[1] 设置成 ht[0] , 然后为ht[1]分配一个空白的哈希表 如下图：

![image-20220530002105147](https://www.javanorth.cn/assets/images/2022/lyj/redisHashmap-08.png)



以上是一个rehash的过程示意。



### 4.渐进式rehash

上面讲的是一个rehash的理论过程，redis实际操作时并不会一次将所有的迁移一次性完成。

如果键值对数量非常庞大，那么迁移过程必然需要花费一点时间。由此可知，服务器也不可能一次将所有的键值对迁移，需要分多次，逐渐将ht[0] 里面的键值对迁移到ht[1]中，

其步骤如下：

- 首先会给ht[1]分配内存空间，此时redis字典拥有两个哈希表
- 字典中维护一个rehashidx的计数器，将其值设置为0，表示rehash工作开始
- 在rehash期间，程序依然可以进行增删改查的操作，除此之外还会顺带将ht[0]上 rehashidx索引上所有的键值对rehash到ht[1]上，rehash的工作完成后会将rehashidx的值加1
- 随着字典的操作，ht[0]上的所有键值全部都rehash到ht[1]上时，程序会将rehashidx的值设为-1 ，表示rehash操作已经完成



在渐进式rehash的过程中，redis字典依然是可以进行增删改查的操作， 其中增加元素的时候会将元素直接保存到ht[1]中， 而删除，查找，更新的操作会在两个哈希表中进行， 查找时会现在ht[0]中进行查找，然后会在ht[1]中进行查找。 以上措施可以宝成ht[0]中的元素只会减少，最终变成空表。



### 总结

1. Redis 字典使用的时哈希表作为底层，并且每个字典维护了两个哈希表，ht[0] 时主要使用的哈希表，而ht[1] 是在rehash过程是才会使用到的表。
2. 哈希表的底层同样是使用了数组 + 链表的结构， 与Java 中HashMap 相似，只不过Java8 以后增加了红黑树，在特定情况下会替换链表。
3. 哈希表增加元素遇到哈希冲突是会将新添加的元素放到链表头，而Java HashMap会将其放到链表尾，
4. 扩容过程中redis的字典是渐进式扩容，扩容期间还是可以进行操作的，而Java的HashMap扩容需要一次性完成。

