---
layout: post
title:  Redis 的过期策略
tagline: by 某某白米饭
categories: redis
tags: 
    - 某某白米饭
---

Redis 中所有的键都可以设置过期策略，就像是所有的键都可以上"生死簿"，上了生死簿的键到时间后阎王就会叉掉这个键。同一时间大量的键过期，阎王就会忙不过来。同时因为 Redis 是单线程的，导致阎王的处理时间会变得很长，而且处理繁忙，Redis 就会出现卡顿现象。

<!--more-->

Redis 有三种策略删除过期 Key

### 相关命令

```
expire key seconds  # 过期时间为秒数，key 不存在时返回(integer) 0，key 存在的时返回(integer) 1

pexpire key milliseconds # 同 expire，设置的过期时间为毫秒数

setex key seconds value # 只能设置字符串的过期时间

ttl key # 查看 Key 的过期时间（秒数），用不过期返回(integer) -1，Key 不存在返回(integer) -2

pttl key # 同 ttl，返回毫秒数
```

### 过期 Key

Redis 的每个设置了过期时间的 Key 都会放在一个独立的字典中，用于遍历删除。

### 过期策略

#### 被动删除

Key 在被操作时，Redis 主动检查 Key 是否过期，过期则删除，返回 nil

1. 对 CPU 友好，只有 Key 在被操作时删除，不会浪费 CPU 时间
2. 对内存不友好，如果同时有大量的 Key 过期，这些 Key 在被使用之前不会被删除，就会浪费内存

#### 主动删除
Redis 会周期性的随机扫描一批设置了过期时间的 Key 并进行处理，Redis 每秒进行10次过期扫描会做的操作有：

1. 随机扫描100个设置了过期时间的 Key
2. 删除所有发现的过期 Key
3. 如果删除的 Key 超过1/4则重复步骤1

```
hz 10
```

Redis 除了设置每秒10次的扫描频率之外，还设置了每次扫描不会超过25ms 的上限，以防出现过度循环扫描，导致线程卡死。

#### maxmemory

```
# maxmemory <bytes>
```

当已用的内存超过 maxmemory 配置的内存时，会触发主动清除策略

```
# maxmemory-policy noeviction
```

1. noeviction 永不过期策略，当已用内存超过 maxmemory 配置时，写操作将返回错误，读操作和 del 操作可以继续服务。
2. volatile-lru 只删除设置了过期时间的 Key，使用频率越少的 Key 优先删除，不会对没有设置过期时间的 Key 删除
3. volatile-ttl 和上面一样，只删除设置过期时间的 Key，TTL 过期时间越少优先删除
4. volatile-random 随机删除快要过期的 Key
5. allkeys-lru 和 lru 一样，删除所有的 Key，没有设置过期时间的 Key 也会被删除
6. allkeys-random 和上面一样，删除掉随机的 Key

### Redis 采用的过期策略

被动删除+主动删除

### 结语

学好 java 需要的刚需知识越来越多，越来越多...，还大伙儿都下班学学学，就这样的卷呀卷呀卷
