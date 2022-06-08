---
layout: post
title:  Redis 基础数据类型 -- 20220608
tagline: by 某某白米饭
categories: Redis
tags: 
    - 某某白米饭
---

大家好，我是指北君。


Redis 作为一个内存数据库已经被许许多多的公司使用，它的性能非常的优秀，读写速度支持非常快而且支持 10W 的 QPS 。今天我们就来学习下它的丰富的数据类型。
<!--more-->
### Redis 数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset（有序集合）。

#### string（字符串）

字符串类型是Redis的最基本数据结构。
字符串类型的值实际可以为字符串，数字，二进制，但是值最大不能超过512M。

key | value
-- | --
hello | world
counter | 1
bits | 10000100
json | {"id":1,"name":"xiaocai","age":18}

###### string命令

设置

set key value [EX seconds] [PX milliseconds] [NX|XX]

get key
```
> set name xiaocai
OK
> get name
"xiaocai"
```

过期

setex key seconds value
```
> set name1 redis1 ex 10 #10秒后过期
OK
> get name1
(nil)

> setex name2 10 redis2 #10秒后过期
OK
> get name2
"redis2"
> get name2
(nil)

> set name3 redis3 px 10 #10豪秒后过期
OK
> get name3
(nil)
```

不存在才能设置成功或者必须存在才能设置成功
```
> set hello world nx #不存在才能设置成功
OK
> set hello w nx #存在就设置失败
(nil)
> get hello
"world"

>set hello w xx  #存在才能设置成功
OK
> set world hello xx #不存在就设置失败
(nil)
> get hello
"w"
> get world
(nil)
```

批量设置
```
mset key value [key value ...]
mget key [key ...]


> mset name1 redis1 name2 redis2
OK

> mget name1 name2
1) "redis1"
2) "redis2"

> mget name1 name2 name3
1) "redis1"
2) "redis2"
3) (nil)
```

计数

incr key

incrby key increment

```
> set age 18  #value只能为整数
OK
> incr age
(integer) 19
> incrby age -5
(integer) 14
> incrby age 10
(integer) 24
```

删除

del key [key ...]

```
> del age
(integer) 1
> get age
(nil)
```

###### 内部编码
1. int 8个字节的长整型
2. embstr 小于等于39个字节的字符串
3. raw  大于39个字节的字符串

```
> set port 6379
OK
> object encoding port
"int"

> set hello world
OK
> object encoding hello 
"embstr"

> set longString abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz
OK
> object encoding hello
"raw"
> strlen longString
(integer) 52
```

#### list（列表）
list类型是用来存储多个有序的字符串。每列字符串称之为元素。一个list的最大存储为2^32-1个元素。可以对列表进行双端插入和弹出，也可以指定索引下标获取元素。

![](http://www.javanorth.cn/assets/images/2021/redis_base_type/0.png)

###### list命令

头部和尾部添加元素

lpush key value [value ...]

rpush key value [value ...]

lrange key start stop

```
> lpush letter "a"
(integer) 1
> lpush letter "b"
(integer) 2
> lpush letter "c"
(integer) 3
> lrange letter 0 -1
1) "c"
2) "b"
3) "a"

> rpush letter "a"
(integer) 4
> rpush letter "b"
(integer) 5
> rpush letter "c"
> lrange letter 0 -1
1) "c"
2) "b"
3) "a"
4) "a"
5) "b"
6) "c"
```

头部和尾部弹出元素

lpop key

rpop key

```
> lpop letter
"c"
> lpop letter
"b"
> lrange letter 0 -1
1) "a"
2) "a"
3) "b"
4) "c"

> rpop letter
"c"
> rpop letter
"b"
> lrange letter 0 -1
1) "a"
2) "a"
```

索引操作
索引需要对全部list进行遍历，性能会随着元素个数的增大而变差

lrange key start stop

lindex key index

ltrim key start stop

len key
```
> rpush letter b c
(integer) 4
> lrange letter 0 -1
1) "a"
2) "a"
3) "b"
4) "c"
> lindex letter 2
"b"
> ltrim letter 0 -2
OK
> lrange letter 0 -1
1) "a"
2) "a"
3) "b"
> llen letter
(integer) 3
```

插入

insert key BEFORE|AFTER pivot value

```
> linsert letter before b c
(integer) 4
> linsert letter after a d
(integer) 5
> lrange letter 0 -1
1) "a"
2) "d"
3) "a"
4) "c"
5) "b"
```

修改

set key index value

```
> lset letter 2 B
OK
> lrange letter 0 -1
1) "a"
2) "d"
3) "B"
4) "c"
5) "b"
```

###### 内部编码
1. ziplist（压缩列表）：小于3.2版本，当元素个数小于list-max-ziplist-entries配置（默认512个），同时每个元素的值长度都小于list-max-ziplist-value配置（默认64字节）
2. linkedlist（链表）：小于3.2版本，不满足ziplist的条件
3. quicklist：Redis 3.2版本，以一个ziplist为节点的linkedlist

```
> object encoding letter
"quicklist"
```


#### hash (哈希)
hash是一个string类型的field和value的映射表。 它适合用于存储对象，它是无序的，不能使用索引操作。

![](http://www.javanorth.cn/assets/images/2021/redis_base_type/1.png)

###### hash命令

设置

hset key field value

```
> hset user:1 name zhangSan
(integer) 1
> hset user:1 age 18
(integer) 1
```

获取和获取所有的field-value

hget key field

hgetall key

```
> hget user:1 name
"zhangSan"
> hgetall user:1
1) "name"
2) "zhangSan"
3) "age"
4) "18"
```

删除

hdel key field [field ...]

```
> hdel user:1 age
(integer) 1
> hdel user:2 age
(integer) 0
```

长度

hlen key

```
> hlen user:1
(integer) 1
```

批量设置

hmset key field value [field value ...]

```
> hmset user:2 name liSi age 23
OK
> hmget user:2 name age
1) "liSi"
2) "23"
> hlen user:1
(integer) 1
> hlen user:2
(integer) 2
```

是否存在

hexists key field

```
> hexists user:2 name
(integer) 1
> hexists user:2 city
(integer) 0
```

所有的field和所有的value

hkeys key

hvals key

```
> hkeys user:1
1) "name"
> hkeys user:2
1) "name"
2) "age"
> hvals user:1
1) "zhangSan"
> hvals user:2
1) "liSi"
```

###### 内部编码
1. ziplist（压缩列表）：当元素个数小于hash-max-ziplist-entries配置（默认512个）和每个元素大小小于hash-max-ziplist-value配置（默认64字节）时
2. hashtable（哈希表）：不满足ziplist条件时

```
> object encoding user:1
"ziplist"
```

修改配置文件hash-max-ziplist-entries为5
```
> hmset test t1 v1 t2 v2 t3 v3 t4 v4 t5 v5 t6 v6
OK
> object encoding test
"hashtable"
```

#### set（集合） 
用来保存多个字符串元素，无序的，不能有重复元素，不能使用索引下标获取元素。一个集合可以存储2^32-1个元素。

![](http://www.javanorth.cn/assets/images/2021/redis_base_type/2.png)

###### set命令

增加

add key member [member ...]

```
> sadd user:1:tag it music news
(integer) 3
> sadd user:1:tag bike news
(integer) 1
```

删除

srem key member [member ...]

```
> srem user:1:tag bike
(integer) 1
```

个数

scard key

```
> scard user:1:tag
(integer) 3
```

是否存在

sismember key member

```
> sismember user:1:tag bike
(integer) 0
> sismember user:1:tag it
(integer) 1
```

随机返回指定个数

srandmember key [count]

```
> srandmember user:1:tag
"news"
> srandmember user:1:tag 3
1) "news"
2) "music"
3) "it"
```

随机弹出

spop key [count]

```
> spop user:1:tag
"news"
> srandmember user:1:tag 3
1) "music"
2) "it"
```

所有个数

smembers key

```
> smembers user:1:tag
1) "music"
2) "it"
```

交集

sinter key [key ...]

```
> sinter user:1:tag user:2:tag
1) "music"
2) "it"
```

并集

sunion key [key ...]

```
> sunion user:1:tag user:2:tag
1) "music"
2) "java"
3) "run"
4) "it"
```

差集

sdiff key [key ...]

```
> sdiff user:1:tag user:2:tag
(empty list or set)
> sadd user:1:tag sleep
(integer) 1
> sdiff user:1:tag user:2:tag
1) "sleep"
> sdiff user:2:tag user:1:tag
1) "java"
2) "run"
```

###### 内部编码
1. intset（整数集合）：元素都是整数和元素个数小于set-max-intset-entries配置（默认512个）时
2. hashtable（哈希表）：不满足intset时

```
> object encoding user:1:tag
"hashtable"

> sadd numbers 1 2 3 4 5
(integer) 5
> object encoding numbers
"intset"
```

#### zset（有序集合）
zset保证了元素不能重复，每个元素都有一个分数（score）作为排序的依据。

![](http://www.javanorth.cn/assets/images/2021/redis_base_type/3.png)

###### zset命令

添加

zadd key [NX|XX] [CH] [INCR] score member [score member ...]

```
> zadd books 8.2 "Redis in Action"
(integer) 1
> zadd books 9.3 "Effective Java: Second Edition : Java"
(integer) 1
> zadd books 9.1 "Think in Java"
(integer) 1
> zadd books 9.3 "Python Cookbook" 9.0 "Effective Python"
(integer) 2
```

个数

zcard key

```
> zcard books
(integer) 5
```

升序返回范围的成员

zrange key start stop [WITHSCORES]

```
> zrange books 0 -1
1) "Redis in Action"
2) "Effective Python"
3) "Think in Java"
4) "Effective Java: Second Edition : Java"
5) "Python Cookbook"

> zrange books 2 5
1) "Think in Java"
2) "Effective Java: Second Edition : Java"
3) "Python Cookbook"
```

升序返回成员时带上分数
```
> zrange books 0 -1 withscores
 1) "Redis in Action"
 2) "8.1999999999999993" 
 3) "Effective Python"
 4) "9"
 5) "Think in Java"
 6) "9.0999999999999996"
 7) "Effective Java: Second Edition : Java"
 8) "9.3000000000000007"
 9) "Python Cookbook"
10) "9.3000000000000007"

> zrange books 2 5 withscores
1) "Think in Java"
2) "9.0999999999999996"
3) "Effective Java: Second Edition : Java"
4) "9.3000000000000007"
5) "Python Cookbook"
6) "9.3000000000000007"
```
1. 精度问题：内部 score 使用 double 类型进行存储，所以存在小数点精度问题
2. withscores:带上分数

降序

zrevrange key start stop [WITHSCORES]

```
> zrevrange books 0 -1 withscores
 1) "Python Cookbook"
 2) "9.3000000000000007"
 3) "Effective Java: Second Edition : Java"
 4) "9.3000000000000007"
 5) "Think in Java"
 6) "9.0999999999999996"
 7) "Effective Python"
 8) "9"
 9) "Redis in Action"
10) "8.1999999999999993"
```

指定value的score

zscore key member

```
> zscore books "Think in Java"
"9.0999999999999996"
```

根据score的数值区间升序

zrangebyscore key min max [WITHSCORES] [LIMIT offset count]

```
> zrangebyscore books 0 9.1 withscores
1) "Redis in Action"
2) "8.1999999999999993"
3) "Effective Python"
4) "9"
5) "Think in Java"
6) "9.0999999999999996"
```

根据score的数值区间降序

zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]

```
> zrevrangebyscore books 9.1 0  withscores
1) "Think in Java"
2) "9.0999999999999996"
3) "Effective Python"
4) "9"
5) "Redis in Action"
6) "8.1999999999999993"
```

根据score的数值降序输出所有元素
```
> zrevrangebyscore books +inf -inf  withscores
 1) "Python Cookbook"
 2) "9.3000000000000007"
 3) "Effective Java: Second Edition : Java"
 4) "9.3000000000000007"
 5) "Think in Java"
 6) "9.0999999999999996"
 7) "Effective Python"
 8) "9"
 9) "Redis in Action"
10) "8.1999999999999993"
```
1. +inf 正无穷
2. -inf 负无穷

删除

zrem key member [member ...]

```
> zrem books "Effective Java: Second Edition : Java"
(integer) 1
```

增加分数

zincrby key increment member

```
> zincrby books 2  "Redis in Action"
"10.199999999999999"
> zincrby books -1  "Redis in Action"
"9.1999999999999993"
```

交集

zinterstore destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE

```
> zadd textbooks 8 "chinese" 9 "english" 9.5 "mathematics"
(integer) 3
> zadd textbooks 9.2 "Think in Java"
(integer) 1

> zrange books 0 -1
1) "Redis in Action"
2) "Effective Python"
3) "Think in Java"
4) "Python Cookbook"
> zrange textbooks 0 -1
1) "chinese"
2) "english"
3) "Think in Java"
4) "mathematics"

> zinterstore newbooks 2 books textbooks
(integer) 1
> zrange newbooks 0 -1
1) "Think in Java"
```

并集

zunionstore destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE

```
> zunionstore newUnionbooks 2 books textbooks
(integer) 7
> zrange newUnionbooks 0 -1
1) "chinese"
2) "Redis in Action"
3) "Effective Python"
4) "english"
5) "Python Cookbook"
6) "mathematics"
7) "Think in Java"
```

###### 内部编码

1. ziplist（压缩列表）：元素个数小于zset-max-ziplist-entries配置（默认128个）和元素长度小于zset-max-ziplist-value配置（默认64B）时
2. skiplist（跳跃表）：不满足ziplist时

```
> object encoding books
"ziplist"
```

### 总结

这篇文章主要是学会基础的 Redis 数据类型和基本的 API 以及每个数据类型背后的原理。
