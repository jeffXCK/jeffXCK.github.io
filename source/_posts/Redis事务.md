---
title: Redis持久化——RDB（Redis DataBase）
comments: false
date: 2019-08-25 11:24:56
categories: Redis
tags: Redis
---

### 一、Redis 事务命令
> 1. MULTI：标记一个事务块的开始。
> 2. EXEC：执行所有事务块内的命令。
> 3. DISCARD：取消事务，放弃执行事务块内的所有命令。
> 4. WATCH key [key ....] ：监视一个（或多个）key，如果在事务执行之前这个（或这些）key被其他命令所改动，那么事务将被打断。
> 5. UNWATCH：取消WATCH 命令对所有key 的监视。
### 二、Redis 官网介绍
> MULTI，EXEC，DISCARD和WATCH是Redis事务的基础命令。它们允许在一个步骤中执行一组命令，并且具有两个重要保证：
> - 事务中的所有命令都被序列化并按顺序执行。在执行Redis事务的过程中，永远不会被其他客户端发送的命令请求打断，保证命令作为单个隔离操作执行。
> - 要么处理所有命令，要么不处理任何命令，因此Redis事务也是原子的。EXEC 命令触发事务中的所有命令的执行，因此，如果客户在失去服务器连接之前调用了MULTI 命令开启事务并进行了相关操作，而没有使用 EXEC 命令执行所有操作，当使用 append only file 时，Redis确保使用单个write系统调用来在磁盘上写入事务。但是，如果Redis服务器以某种困难的方式崩溃或被系统管理员杀死，则可能只注册了部分操作。Redis将在重新启动时检测到此情况，并将退出并显示错误。可以使用`redis-check-aof`工具修复仅删除部分事务的附加文件，以便服务器可以重新启动。

### 三、用法
> 使用 MULTI 命令输入Redis事务。该命令总是回复`OK`。此时，用户可以发出多个命令。Redis不会执行这些命令，而是将它们加入队列。等到调用 EXEC 命令后才执行队列中的所有命令。而如果调用 DISCARD 命令将刷新事务队列并退出事务。
#### 1.正常执行：
```
127.0.0.1:6379>MULTI
OK
127.0.0.1:6379>set k1 v1
QUEUED
127.0.0.1:6379>set k2 v2
QUEUED
127.0.0.1:6379>get k2
QUEUED
127.0.0.1:6379>set k3 v3
QUEUED
127.0.0.1:6379>EXEC
1) OK
2) OK
3) "V2"
4) OK
127.0.0.1:6379>
```
#### 2.放弃事务：
```
127.0.0.1:6379>MULTI
OK
127.0.0.1:6379>set k1 11
QUEUED
127.0.0.1:6379>set k2 22
QUEUED
127.0.0.1:6379>set k3 33
QUEUED
127.0.0.1:6379>DISCARD
OK
127.0.0.1:6379>get k2
"v2"
```
#### 3.全体连坐（执行过程中出错）：
```
127.0.0.1:6379>MULTI
OK
127.0.0.1:6379>set k1 11
QUEUED
127.0.0.1:6379>set k2 22
QUEUED
127.0.0.1:6379>set k3 33
QUEUED
127.0.0.1:6379>getset k3
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379>set k4 v4
QUEUED
127.0.0.1:6379>set k5 v5
QUEUED
127.0.0.1:6379>EXEC
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379>mget k1 k2 k3 k4 k5
1) "v1"
2) "v2"
3) "v3"
4) (nil)
5) (nil)
```
#### 4.冤有头，债有主（执行EXEC命令后出错）：
```
127.0.0.1:6379>set k1 aa
OK
127.0.0.1:6379>MULTI
OK
127.0.0.1:6379>incr k1
QUEUED
127.0.0.1:6379>set k2 22
QUEUED
127.0.0.1:6379>set k3 33
QUEUED
127.0.0.1:6379>set k4 v4
QUEUED
127.0.0.1:6379>get k4 
QUEUED
127.0.0.1:6379>EXEC
1) （error）ERR value is not an integer or out of range
2) OK
3) OK
4) OK
5) "v4"
127.0.0.1:6379>get K4
"v4"
```
> **注意：** Redis支持事务，但是是仅部分支持。
#### 5.watch监控
#####介绍几种锁：
> - 悲观锁：
> > 悲观锁(Pessimistic Lock), 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁
> - 乐观锁：
> > 乐观锁(Optimistic Lock), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。
> > 乐观锁策略：提交版本号必须大于记录当前版本才能执行更新
##### 银行信用卡支付的例子
> 1. 初始化信用卡可用余额和欠额。
```
127.0.0.1:6379>set balance 100
ok
127.0.0.1:6379>set debt 0
ok
127.0.0.1:6379>keys * 
1）"balance"
2）"debt"
127.0.0.1:6379>
```
> 2. 无加塞篡改：先监控key再开启MULTI，保证两笔金额变动在同一个事务内。
```
127.0.0.1:6379>WATCH balance
ok
127.0.0.1:6379>MULTI
ok
127.0.0.1:6379>decrby balance 20
QUEUED
127.0.0.1:6379>incrby debt 20
QUEUED
127.0.0.1:6379>EXEC
1）（integer）80
2）（integer）20
127.0.0.1:6379>
```
> 3. 有加塞篡改：客户端1先监控了key，如果key被客户端2修改了，那么客户端1的事务执行将失效。
- 客户端1：
```
127.0.0.1:6379>WATCH balance
ok
127.0.0.1:6379>MULTI
ok
127.0.0.1:6379>decrvy balance 20
QUEUED
127.0.0.1:6379>incrby debt 20
QUEUED
127.0.0.1:6379>EXEC
(nil)
127.0.0.1:6379>get balance
"800"
127.0.0.1:6379>
```
- 客户端2：
```
127.0.0.1:6379>get balance
"80"
127.0.0.1:6379>set balance 800
```
> **注意：**一旦执行了EXEC命令，则之前加的监控锁都会被取消掉。当然也可以执行UNWATCH命令取消WATCH对所有key的监视。
##### 小结
> - Watch指令，类似乐观锁，事务提交时，如果Key的值已被别的客户端改变，比如某个list已被别的客户端push/pop过了，整个事务队列都不会被执行。
> - 通过 WATCH 命令在事务执行之前监控了多个 keys，倘若在 WATCH 之后有任何Key的值发生了变化，EXEC命令执行的事务都将被放弃，同时返回 Nullmulti-bulk 应答以通知调用者事务执行失败。
### 四、总结
#### Redis 事务三阶段：
> 1. 开启：以MULTI开始一个事务。
> 2. 入队：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面。
> 3. 执行：由EXEC命令触发事务。
#### Redis 事务三特性：
> 1. 单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
> 2. 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在”事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题。
> 3. 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。