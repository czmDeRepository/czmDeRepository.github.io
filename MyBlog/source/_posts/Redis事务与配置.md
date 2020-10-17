---
title: Redis事务与配置
date: 2020-09-24 21:10:11
tags: 
  - Redis
categories: 数据库
---

## 事务

### 通常情况下的事务

#### 事务的基本要素即四大特性（ACID）

- 原子性（Atomicity）：事务所有操作指令，要么全部完成，要么全部不做。

- 一致性（Consistency）：事务提交后，数据库从一个一致性状态变到另一个一致性状态。

    如事务（a-100，b+100）执行后a减100，b一定加100，不然a，b就都不变

- 隔离性（Isolation）：同一时间，只允许一个事务操作同一数据，不同的事务之间彼此没有任何干扰。

- 持久性（Durability）：事务一旦提交后，事务对数据库的所有更新将被保存到数据库，不能回滚。

### Redis事务

​	 本质：一组指令的集合，一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行！

#### 特性

1. **Redis事务没有隔离级别的概念！**

    一个事务的所有指令并没有被直接执行，而是存入队列中，只有发起执行命令（Exec）的时候才会顺序执行！

2.  **Redis单条命令保证原子性，但是事务不保证原子性！**

3. **单独的隔离操作**

    事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

#### 操作过程

1. 开启事务：multi
2. 输入命令入队列：set key value  ……
3. 提交事务：exec  或 撤销事务：discard

```bash
> multi
OK
> set key1 1
QUEUED
> set key2 2
QUEUED
> get key2
QUEUED
> exec
OK
OK
2
```

#### 异常

##### 编译时异常

​		命令有问题，事务所有的命令都不会被执行

```bash
> flushdb
OK
> multi
OK
> set key1 1
QUEUED
> set key2  #错误命令
QUEUED
> exec
EXECABORT Transaction discarded because of previous errors.  #执行事务报错
> get key2  
null
> get key1 #所有命令都不会被执行
null
```

##### 运行时异常

​	如果事务队列中存在语法性错误，那么执行命令的时候，其他命令是可以正常执行的（Redis事务不保证原子性）

```bash
> flushdb
OK
> set k1 abc
OK
> multi
OK
> set k2 2
QUEUED
> incr k2
QUEUED
> incr k1  #k1为字符串，无法加1
QUEUED
> get k1
QUEUED
> get k2
QUEUED
> exec
1) OK
2) (integer) 3
3) (error) ERR value is not an integer or out of range  # incr k1执行报错
4) "abc"
5) "3"
> get k2  #其他命令正常执行
"3"
```

## 监控watch

### 锁

#### 悲观锁

​	很悲观，总是认为最坏情况，拿数据的时候数据都会被别人修改，无论什么时候都加锁，效率低下

#### 乐观锁

​	很乐观，总是认为最好情况，拿数据的时候数据都不会被别人修改，所以不加锁，在更新数据的时候先判断下，拿数据这期间有没有人修改过此数据（mysql中可增加一版本字段version，更新前比较此字段）

### 测试

#### 正常测试

```bash
> set money 100
OK
> set usedMoney 0
OK
> watch money  #监视money对象
OK
> multi   
OK
> decrby money 20
QUEUED
> incrby usedMoney 20
QUEUED
> exec  #事务正常结束，数据期间没有发生变动，这时候事务正常执行
80
20
> get money
80
```

使用多客户端即多线程修改值，使用watch可以当作Redis的乐观锁操作

```bash
> set money 100
OK
> set usedMoney 0
OK
> watch money  #监视money
OK
> multi
OK
> decrby money 20
QUEUED
> incrby usedMoney 20
QUEUED
> exec  #执行之前，其他客户端执行 set money 10
null  #对比watch监控的money，发生变化，修改失败

> unwatch  #执行失败，就先解锁
OK
> watch money #获取最新值，再次监控（此时money=10，usedMoney=0）
OK
> multi
OK
> decrby money 5
QUEUED
> incrby usedMoney 5
QUEUED
> exec
5
5
> get money
5
> get usedMoney
5
```



## 配置文件Redis.conf

```bash
unit(单位)
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
只支持bytes，不支持bit，对大小写不敏感


INCLUDES模块
# include .\path\to\local.conf
# include c:\path\to\other.conf
可通过include 包含其他文件

网络
bind 127.0.0.1 #绑定ip
port 6379	#端口
protected-mode yes #保护模式，默认开启



GENERAL通用配置模块
daemonize yes #以守护进程执行（后台与运行）
pidfile /var/run/redis.pid #如果以守护进程方式运行，需指定一个pid文件


# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice #日志级别


logfile "" #日志文件位置
databases 16 #数据库数量

Tcp-keepalive #连接检测，单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60 
tcp-backlog
#设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。在高并发环境
#下你需要一个高backlog值来避免慢客户端连接问题。注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值，所以
#需要确认增大somaxconntcp_max_syn_backlog两个值来达到想要的效果


SNAPSHOTTING快照模块
#达到已下条件，就进行保存
save 900 1 #15分钟内改了1次。
save 300 10 #5分钟内改了10次，
save 60 10000 #1分钟内改了1万次，
#如果想禁用RDB持久化的策略，只要不设置任何save指令，或者给save传入一个空字符串参数也可以
save ""

stop-writes-on-bgsave-error yes #持久化操作出错，是否继续工作
rdbcompression yes  #对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能

rdbchecksum yes #存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
dir ./ #db文件的保存路径


SECURITY安全模块
# requirepass foobared
requirepass ""  #配置redis密码，默认空


LIMITS限制模块
# maxclients 10000
#设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。当你无法设置进程文件句柄限制时，redis会设置为当前的文件句柄限制值减去32，因为redis会为自身内部处理逻辑留一些句柄出来。如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。

# maxmemory <bytes>
#设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。
#如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信
#息，比如SET、LPUSH等。但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从#redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，
#才不用考虑这个因素


maxmemory-policy noeviction  #内存达到上限后处理策略
#（1）volatile-lru：使用LRU算法移除key，只对设置了过期时间的键
#（2）allkeys-lru：使用LRU算法移除key
#（3）volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
#（4）allkeys-random：移除随机的key
#（5）volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
#（6）noeviction：不进行移除。针对写操作，只是返回错误信息

maxmemory-samples 5 #设置检查的样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，
#redis默认会检查这么多个key并选择其中LRU的那个，LRU和最小TTL算法不是精确算法而是近似算法，该值越大越精准，但cpu成本高，
#3较快，默认5，10非常接近


APPEND ONLY MODE附加模块
appendonly no #指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时
#导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。
#默认为no
appendfsync everysec  #指定更新日志条件，共有3个可选值： 
#  no：表示等操作系统进行数据缓存同步到磁盘（快） 
#  always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
#  everysec：表示每秒同步一次（折衷，默认值）

appendfilename appendonly.aof #指定更新日志文件名，默认为appendonly.aof


No-appendfsync-on-rewrite no #重写时是否可以运用Appendfsync，用默认no即可，保证数据安全性。

Auto-aof-rewrite-min-size 64mb #设置重写的基准值 大小
Auto-aof-rewrite-percentage  100 # 设置重写的基准值 百分比

vm-enabled no #指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷
#数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中
   
vm-swap-file /tmp/redis.swap # 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
   
vm-max-memory 0 #将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
   
vm-page-size 32 # Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，
#vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很
#大大对象，则可以使用更大的page，如果不 确定，就使用默认值
   
vm-pages 134217728 # 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘
#上每8个pages将消耗1byte的内存。
   
vm-max-threads 4 # 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，
#可能会造成比较长时间的延迟。默认值为4
   
glueoutputbuf yes # 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
 
```

