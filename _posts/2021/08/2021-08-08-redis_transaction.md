---
layout: post
title:  一篇文章就能学会的 Redis 的事务
tagline: by 某某白米饭
categories: Redis
tags:
- 某某白米饭
---

大家好，我是指北君。

Redis 作为内存的存储中间件，已经是面试的面试题必问之一了，今天一起来看看 Redis 的事务吧。

事务提供了一种"将多个命令打包，一次性提交并按顺序执行"的机制，提交后在事务执行中不会中断。只有在执行完所有命令后才会继续执行来自其他客户的消息。

<!--more-->

### Redis 中的使用

Redis 通过 multi，exec，discard，watch 实现事务功能。

1. multi：开始事务
2. exec：提交事务并执行
3. discard：取消事务
4. watch：事务开始之前监视任意数量的键

```
> multi
OK
> set bookName "Redis"
QUEUED
> get bookName
QUEUED
> sadd tag "Redis" "New Book"
QUEUED
> smembers tag
QUEUED
> exec
1) OK
2) "Redis"
3) (integer) 2
4) 1) "Redis"
   2) "New Book"
```

#### 开始事务

```
> multi
OK
```
这个命令将 Redis_multi 选项打开，让客户端从非事务状态变为事务状态

![](http://www.javanorth.cn/assets/images/2021/transaction/0.png)

#### 命令入队

```
> set bookName "Redis"
QUEUED
> get bookName
QUEUED
> sadd tag "Redis" "New Book"
QUEUED
> smembers tag
QUEUED
```

在事务状态中，Redis 命令并不是立即执行的，而是进入一个先进先出的事务队列。QUEUED 表示这个命令已经入了事务队列。

#### 执行事务

```
> exec
1) OK
2) "Redis"
3) (integer) 2
4) 1) "Redis"
   2) "New Book"
```
当执行 exec 命令时，Redis 根据客户端所保存的事务队列， 以先进先出的方式执行事务队列中的命令： 最先入队的命令最先执行， 而最后入队的命令最后执行。
当 exec 命令执行完毕时，Redis 会将结果保存到一个回复队列，并将回复队列返回给客户端。客户端从事务状态退出，一个事务执行完毕。

#### discard 命令

```
> multi
OK
> set author "lisi"
QUEUED
> discard
OK
> get author
(nil)
```
discard 取消一个事务的命令，表示这个事务被取消。客户端从事务状态退出，回到非事务状态，Redis_multi 选项关闭。

![](http://www.javanorth.cn/assets/images/2021/transaction/1.png)

#### watch 命令

```
# Redis 客户端1
> watch letter
OK
> multi
OK
> set letter a
QUEUED
> exec
(nil)


# Redis 客户端2
> set letter b
OK

# Redis 客户端1
> get letter
"b"
```

设置监控 letter 键，客户端1进入事务，设置 letter 的 value 为 a，未提交事务。客户端2设置 letter 的 value 为 b。回到客户端1提交事务返回的结果为 nil，调用 get 命令得到 letter 为 b。这说明当 letter 键在其他客户端改变后，事务被取消了，不会被执行，返回失败。

watch 命令在事务开始之前监视任意数量的键：当调用 exce 命令执行事务时，如果任意一个被监视的键已经被其他客户端修改了，那么整个事务不再执行，直接返回失败。

![](http://www.javanorth.cn/assets/images/2021/transaction/2.png)


### 事务异常

#### 命令错误

```
> set letter ac
QUEUED
> get letter ac
(error) ERR wrong number of arguments for 'get' command
> exec
(error) EXECABORT Transaction discarded because of previous errors.
```
事务中命令异常属于语法错误，将导致事务无法执行。

#### 运行时异常

```
> multi
OK
> lpush books "Redis"
QUEUED
> incr books
QUEUED
> lpush books "Python"
QUEUED
> lrange books 0 -1
QUEUED
> exec
1) (integer) 1
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) (integer) 2
4) 1) "Python"
   2) "Redis"
```
上面的例子是事务执行到中间遇到失败了，因为不能对一个字符串进行 incr 命令，事务在遇到命令执行失败后，后续的命令还继续执行，所以 books 的值能继续得到设置。这种异常只有程序员在代码中避免。

### 事务的 ACID

#### 原子性

原子意味着要么一起成功执行，要么一起失败回滚。Redis 提供的所有 API 都是原子操作。那么 Redis 事务只要保证在一批操作中保证原子性，但是在运行时异常中，在一个事务中一个命令出现异常，其他命令还是会继续执行，事务没有回滚机制，所以 Redis 事务是不保证原子性的。

#### 一致性

事务异常

如果命令错误事务无法执行，如果是运行时异常，Redis 会将错误包含在返回结果中，并不影响后续执行，所以事务是一致性的。

Redis 进程被终结

在纯内存模式下，Redis 没有做持久化，重启之后数据库是空白的，所以是事务一致性的。

在 RDB 模式下，事务并不会在中途执行保存 RDB 文件的工作，只有在事务执行完后，RDB 工作才可能会开始。所以在事务执行过程中 Redis 进程被杀死，不管成功多少都不会保存到 RDB 文件中，所以是一致性的。

在 AOF 模式下，事务部分语句被写入 AOF 文件并保存成功，不完整的事务被保存到了 AOF 文件，当重启 Redis 时，检查 AOF 文件不完整，Redis 退出并报错。需要把这段不完整的事务删除后才能重启成功，所以是一致性的。

在 AOF 模式下，事务并未被写入 AOF 文件，所以重启后 Redis 数据库是最近一次成功保存到 AOF 文件中的数据。并没有这次事务的数据，所以是以一致性的。



#### 隔离性

Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。所以事务是带有隔离性的。

##### 持久

在纯内存模式下，事务肯定不是持续性的。

在 RDB 模式下，服务器可能在事务执行之后、RDB 文件更新之前的这段时间失败，所以 RDB 模式下的事务也是不持久的。

在 AOF 模式下，将命令添加到 AOF 文件中，但是对文件进行写入并不会马上写到磁盘上，而是先存储到缓冲区。所以数据保存到磁盘上有一段非常小的时间间隔。这种模式下事务也不是持久的。

### 结语

本文介绍了 Redis 的事务的 multi，exec，discard，watch  命令用法和 它的 ACID。

#### 参考资料

- [1] [Redis 设计与实现]
- [2] [Redis 开发与运维]
- [3] [Redis 深度历险：核心原理与应用实践]
