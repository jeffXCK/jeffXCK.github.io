---
title: Redis持久化——AOF（Append Only File）
comments: false
date: 2019-08-24 08:02:34
categories: Redis
tags: Redis
---

### 一、是什么？
> - **以日志的形式来记录每个写操作**，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
#### 优势
> - 每次修改同步：appendfsync always   同步持久化 每次发生数据变更会被立即记录到磁盘 ，性能较差但数据完整性比较好。
> - 每秒同步：appendfsync everysec  异步操作，每秒记录   如果一秒内宕机，有数据丢失。
> - 不同步：appendfsync no   从不同步。
#### 劣势
> - 相同数据集的数据而言AOF文件要远大于RDB文件，恢复速度慢于RDB。
> - AOF运行效率要慢于RDB，每秒同步策略效率较好，不同步效率和RDB相同。
### 二、AOF介绍
> - 可以同时启用AOF和RDB持久化，不会出现任何问题。
> - 在启动Redis服务时， 如果启用了AOF，则Redis将加载AOF文件进行数据恢复，这是是具有更好持久性保证的文件。
### 三、AOF启动配置
```
# 开启AOF持久化，默认：no，开启改为 yes
appendonly no
# AOF文件的名字，建议使用默认值。
appendfilename "appendonly.aof"

# Redis支持三种不同的备份模式:
# always：同步持久化 每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好
# everysec：出厂默认推荐，异步操作，每秒记录   如果一秒内宕机，有数据丢失
# no：不要fsync，只要让操作系统在需要的时候刷新数据即可。得更快。
# appendfsync always
appendfsync everysec
# appendfsync no

# 重写时是否可以运用Appendfsync，用默认no即可，保证数据安全性。
no-appendfsync-on-rewrite no

# 设置重写的基准值
# 指定百分比为0，以禁用自动AOF重写特性。
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```
### 四、AOF恢复 (RDB同理)
#### 正常恢复
> - 将有数据的AOF文件复制一份保存到对应目录(config get dir)
> - 恢复：重启redis然后重新加载
#### 异常恢复
> - 备份被写坏的AOF文件
> - 修复：执行 redis-check-aof --fix appendonly.aof  进行修复。
> - 重启redis然后重新加载
### AOF重写
#### 是什么？
> - AOF采用文件追加的方式，文件会越来越大。为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof。
> ####重写原理：
> - AOF文件因持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的set语句。重写AOF文件的操作，并没有读取旧的AOF文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。
> ####触发机制：
> - Redis会记录上次重写时的AOF大小。
> - 默认配置是：*当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发*。
### 五、总结
![AOF.png](https://upload-images.jianshu.io/upload_images/18660770-905eefe75818061f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> - AOF文件是一个只进行追加的日志文件。
> - Redis 可以在AOF文件体积变得过大时。自动地在后台对AOF进行重写。
> - AOF文件有序的保存了对数据库执行的所有写入操作，这些写入操作以Redis协议的格式保存，因此AOF文件的内容非常容易被人读懂，对文件进行分析也很轻松。
> - 对于相同的数据集来说，AOF文件的体积通常大于RBD文件的体积。
> - 根据所使用的的fsync策略，AOF的速度可能慢于RDB。