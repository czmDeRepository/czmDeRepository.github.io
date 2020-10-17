---
title: Redis高级
date: 2020-09-27 22:30:30
tags: 
  - Redis
categories: 数据库
---



## Redis持久化

### 为什么持久化

​	Redis是内存数据库，数据都保存在内存中，因此数据读取速度快，效率高，但也容易发生丢失（如Redis宕机，突然断电等）。所以必须要有持久化的机制保存数据到硬盘，防止数据丢失。

### RDB（Redis DataBase）

#### 简介

​		RDB(默认的持久化方式)指在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照。这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb,它恢复时是将快照文件直接读到内存里。

#### 持久化过程

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

#### 触发RDB快照条件

1. 配置文件save配置
2. 执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义
3. 执行命令save或者bgsave

注：save：save时只管保存，其它不管，全部阻塞

bgsave：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。

可以通过lastsave命令获取最后一次成功执行快照的时间。

#### 停用

1. 动态所有停止RDB保存规则的方法：redis-cli config set save ""
2. 修改配置文件save ""

#### 优势

1. 适合大规模的数据恢复
2. 对数据完整性和一致性要求不高

#### 劣势

1. 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改
2. Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

### AOF（Append Only File）

#### 简介

​	以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。AOF默认存储文件名appendonly.aof

#### 启用

修改配置appendonly no改为yes

#### 修复

修复受损appendonly.aof文件命令： Redis-check-aof --fix  appendonly.aof

#### Rewrite

##### 简介：

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof

##### 原理

AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似

##### 触发机制

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

#### 劣势

1. 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
2. Aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同

#### 优势

1. 每修改同步：appendfsync always   同步持久化 每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好
2. 每秒同步：appendfsync everysec    异步操作，每秒记录   如果一秒内宕机，有数据丢失
3. 不同步：appendfsync no   从不同步，性能最好

### 小结

1. RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。
2. AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大。
3. 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.

#### 同时开启两种持久化方式

1. 在这种情况下,当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.
2. RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)，快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段。

#### 性能建议

1. 
    因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。
2. 如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。
3. 如果不Enable AOF ，仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。新浪微博就选用了这种架构

## 发布订阅

### 简介

​	进程间的一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。复杂情况下可使用专门的消息中间件，如RabbitMQ。

### 操作命令

PSUBSCRIBE pattern [pattern ...] 	订阅一个或多个符合给定模式的频道。
PUBSUB subcommand [argument [argument ...]]  	查看订阅与发布系统状态。
PUBLISH channel message	 将信息发送到指定的频道。
PUNSUBSCRIBE [pattern [pattern ...]] 	退订所有给定模式的频道。
SUBSCRIBE channel [channel ...] 	订阅给定的一个或多个频道的信息。
UNSUBSCRIBE [channel [channel ...]]	指退订给定的频道。

#### 示例

```bash
#窗口1
127.0.0.1:6379> SUBSCRIBE channel  #订阅channel频道
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel"
#当下面发布消息到此频道时，显示下面信息
3) (integer) 1 #数量 
1) "message" #消息
2) "channel" #频道
3) "hello word" #消息内容

#与上面同一时刻的另一个客户端窗口2
127.0.0.1:6379> SUBSCRIBE channel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel"
3) (integer) 1
1) "message"
2) "channel"
3) "hello word"


#窗口3
#将信息发送到指定的频道
127.0.0.1:6379> PUBLISH channel "hello word"
(integer) 2 #推送给两个订阅者
#获取当前系统所有频道
127.0.0.1:6379> PUBSUB CHANNELS
1) "channel2"
2) "channel"
```



## Redis集群

### 简介

也就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主，实现读写分离，Slave无法写入数据

### 使用

```bash
127.0.0.1:6379> Info replication #查看
# Replication
role:master #角色
connected_slaves:0 #从机数
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379>
```

1. 拷贝多个redis.conf文件
2. 开启daemonize yes
3. 修改port，pidfile，logfile，dbfilename
4. 执行指令：redis-server 配置文件路径
5. 在从机上运行SLAVEOF masterip  端口

### 常用方式

#### 一主多仆：

​		一个master多个slaver

#### 薪火相传：

- 上一个Slave可以是下一个slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master,可以有效减轻master的写压力。
- 中途变更转向:会清除之前的数据，重新建立拷贝最新的。
- Slaveof 新主库IP 新主库端口

#### 反客为主：

​		执行指令SLAVEOF no one，使当前数据库停止与其他数据库的同步，转成主数据库

### 复制原理

- Slave启动成功连接到master后会发送一个sync命令
- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
- 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行

### 哨兵模式(sentinel)（实际开发中常用）

#### 简介

反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

##### 使用

1. 新建sentinel.conf文件
2.  写入基本配置：sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1 （最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机）
3. 在Redis根目录下执行命令启动哨兵：Redis-sentinel  sentinel.conf 文件路径

### 复制延时

由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。

## END