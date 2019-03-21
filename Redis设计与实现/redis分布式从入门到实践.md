
### Redis 的高级功能

##### 4.1 慢查询
慢查询的默认值
我们在使用 Redis 时，可以设置慢查询的默认值。有以下两种方式。

方式一，请见下面代码。

// 不推荐
```
config get slowlog-max-len = 128 // 设置 slowlog-max-len 为 128，保存数据最多 128 条
config get slowlog-log-slower-than = 10000// 表示超时到 10000 微秒会被记录到日志上
```

方式二，动态配置。配置方式，请见下面两条命令。推荐使用该方式。
```
// 推荐使用动态配置
config set slowlog-max-len 1000 // 设置 slowlog-max-len
 为 1000，保存数据最多 1000
 条，不能太小，也不能太大
config set slowlog-log-slower-than 10000 // 表示超时到 10000 微秒会被记录到日志上
```

#### 4.2 PipeLine 流水线

#### 4.3 BitMap 位图
使用 BitMap，有如下五个命令。

SETBIT
```
setbit key offset value //给位图指定索引设置值
```
例如，setbit book1:books:2017-01-10 1 1，即将 book1 的 2017 年 1 月 10 日的第一个位图值改为 1，返回结果是之前的值。

GETBIT
```
getbit key offset //获取位图对应的值
```
例如，getbit book1:books:2017-01-10 1 8，和 SET 类似。

BITCOUNT
```
bitcount key [start end] // 获取位图指定范围值为 1 的个数
```
例如，bitcount book1：books：2017-01-10 1 3，返回 3。

BITOP
```
bitop op destkey key [key...]
```
上面代码表示，进行多个 Bitmap 的 and、or、not、xor 操作并保存到 destkey 中。


#### 4.4 HyperLogLog
这种算法又叫：极小空间完成独立数量统计。我们日常是不会使用 HyperLogLog 算法的，只有当统计数据量非常大的时候，才会使用它。很多小伙伴在研究 HyperLogLog 的时候，就会发现 HyperLogLog 本质上还是字符串 String。你不信可以使用 type 命令，就知道 HyperLogLog 返回的是字符串。

常用的 HyperLogLog 命令有以下三个。

pfadd key element ...：向HyperLogLog 添加元素；
pfcount key ...：计算HyperLogLog 的独立总数；
pfmerge key1 key2 ...：合并多个 HyperLogLog。


#### 4.5 发布订阅

当发布者发送一条消息到 Redis Server 后，只要订阅者订阅了该频道就可以接收到这样的信息。同时，订阅者可以订阅不同的频道。

使用发布订阅，有如下四个命令。

PUBLISH，发布。
```
publish channel message //发布命令
publish youku:tv "welcome back!" // 返回有几个订阅者。
```

SUBSCRIBE，订阅。
```
subcribe [channel] // 订阅一个或者多个
subcribe youku:tv // 订阅优酷并返回信息。
```

UNSUBSCRIBE，取消订阅。
```
unsubcribe [channel] // 取消订阅一个或者多个
unsubcribe youku:tv // 取消订阅优酷并返回信息。
```

其他命令

如 PSUBSCRIBE（订阅模式）、PUNSUBSCRIBE（退订指定模式）、PUBSUB CHANNELS（列出至少一个订阅者频道）




### 05：正确认识 Redis 持久化和开发运维问题
5.1 持久化的概念及其作用
首先我们来看下什么是 Redis 的持久化。Redis 的所有数据都保持在内存中，对数据的更新将异步保存到磁盘上。如果 Redis 需要恢复时，就是从硬盘再到内存的过程。

这就是 Redis 持久化的用处。持久化是为了让硬盘成为备份，以保证数据库数据的完整。

持久化有哪些方式呢？你会看到网上和书上有很多分类，但是其实大致只分成两种。
```
快照 RDB
日志 AOF
```

#### 5.2 快照 RDB
顾名思义，快照就是拍个照片，做个备份，而这种备份是 Redis 自动完成的。

这个功能是 Redis 内置的一个持久化方式，即使你不设置，它也会通过配置文件自动做快照持久化的操作。你可以通过配置信息，设置每过多长时间超过多少条数据做一次快照。具体的操作如下：
```
save 500 100 // 每过 500 秒超过 100 个 key 被修改就执行快照
```

我们再来看看它们的运行状况。

**Redis 调用 fork() 进程，同时拥有父进程和子进程**；

子进程将数据都写到一个临时 RDB 文件之中；

当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换旧的 RDB 文件。

这个过程使 Redis 可以从写时复制中得到备份。

接下来我们讲解三种 RDB 的触发机制：save（ 同步 ）、bgsave（ 异步 ）和自动触发 RDB 文件。

1. save（同步）
即其他的命令都要排队。

![在这里插入图片描述](http://images.gitbook.cn/0548abb0-f9cd-11e7-a366-e3aeb8ba8d5e)
如果有 1000 万条数据，执行 save 命令，Redis 就会对 1000 万条数据打包，而这个时候是同步的，Redis 就会进入阻塞状态。这也是 save 的缺点，接下来我们讲异步命令 bgsave。

2. bgsave（异步命令）
即返回 OK，后台会新开一个线程去执行。
![在这里插入图片描述](http://images.gitbook.cn/60ed2100-f9cb-11e7-b881-8fdce9c8497b)
上图可知，客户端会在 Redis 发出 bgsave 命令，另外开一个进程调用 fork() 方法，这个进程同时会创建 RDB 文件，也就是我们上面提到的自动触发 RDB 文件机制。这样，你在父进程里操作别的命令，就不会受影响了。

3. 自动触发 RDB 文件
快照方式虽然是 Redis 自动的，但是如果 Redis 服务器挂掉后，那么最近写入的，是不会被拷入快照中的。所以 RDB 存在两方面的缺点。

耗时耗性能：Redis 写 RDB 文件是 dump 操作，所以需要的时间是 O（n），需要所有命令都执行一遍，非常耗时耗性能。

容易丢失数据和不可控制：如果我们在某个时间 T1 内写多个命令，这个时候 T2 时间执行 RDB 文件操作，T3 时间又执行多个命令，T4 时间就会出现宕机了。那么 T3 到 T4 的数据就会丢失。

![在这里插入图片描述](http://images.gitbook.cn/381c4270-fa60-11e7-9f83-d55f581a6c37)

虽然 RDB 有缺陷，但是依然在生产环境中会使用，**RDB 适合冷备，就是当用户数据不高的时候，比如在午夜时分就可使用 RDB 备份**。其他时候我们通常使用 AOF 日志备份。

#### 5.3 日志 AOF
AOF（Append-only File）是用日志方式，通俗点讲就是当写一条命令的时候，如 Set 某个值，Redis 就会去日志里写一条 Set 某个值的语句。如下图：
![在这里插入图片描述](http://images.gitbook.cn/ea14c190-fa61-11e7-acb1-c520391eb94c)

当服务器宕机后，Redis 就会调用 AOF 日志文件，并且这个过程一般是实时的，不需要时间消耗。

![在这里插入图片描述](http://images.gitbook.cn/c4eba2a0-fa64-11e7-acb1-c520391eb94c)

上图表示客户端向 AOF 文件写入的时候，是会通过缓冲的，缓冲是系统机制，是为了提高文件的写入效率。

AOF 三种策略分别是 always 、everysec 和 no。

always
客户端是不会直接把命令写入 AOF 文件的，Liunx 系统会有一个缓冲机制，把一部分命令打包再同步到 AOF 文件，从而提高效率。

但是如果你使用的是 always 命令，就表示每条命令都写入 AOF 文件中，这样是为了保证每条命令都不丢失。

everysec
即每秒策略，简而言之，就是说每一秒的缓冲区的数据都会刷新到硬盘当中。但是它不像 always 那样，每条数据都会写入硬盘中，如果硬盘发生故障有可能丢失 1 秒的数据。

no
这个 no 的配置相当于把控制权给了操作系统，操作系统来决定什么时候刷新数据到硬盘，以及不需要我们考虑哪种情况。


关于 AOF，我们补充一点。对于 AOF 操作，Redis 在写入的时候，会压缩命令。它既可以减少硬盘的占用量，同时可以提高恢复硬盘的速度。例如如下表格。



|原生 AOF|	AOF 复写|
|--|--|
|set hello a1|	set hello a3|
|set hello a2	|
|set hello a3	|
|incr counter	|set counter 2|
|incr counter	||
|rpush hello a	|rpush hello a b c|
|rpush hello b	||
|rpush hello c	||

从上表可以看到，set hello 有三个值，但是 a1 和 a2 是无效的，最终 AOF 会自动 set 最后一个 a3 的值；incr counter 两次，AOF 自动识别 2 次；rpush 三个值，rpush 会自动简写为一条数据。

针对上面的 Redis 的 AOF 复写。Redis 提供了两种命令。这两种命令是 bgrewriteaof 和 AOF 重写配置。bgrewriteaof 类似 RDB 中的 bgsave 命令，它还是 fork() 子进程，然后完成 AOF 的过程。

AOF 重写配置包含两个配置命令，如下。
```
auto-aof-rewrite-min-size	AOF 文件重写最小的尺寸
auto-aof-rewrite-percentage	AOF 文件增长率
```
auto-aof-rewrite-min-size 表示配置最小尺寸，超过这个尺寸就进行重写。

auto-aof-rewrite-percentage ，这里说的是 AOF 文件增长比例，指当前 AOF 文件比上次重写的增长比例大小。AOF 重写即 AOF 文件在一定大小之后，重新将整个内存写到 AOF 文件当中，以反映最新的状态（相当于 bgsave）。这样就避免了 AOF 文件过大而实际内存数据小的问题（频繁修改数据问题）。



接下来看下统计配置，如下表所示，有了它，就可以对上面的配置命令进行控制。

|配置名	|含义|
|--|--|
|aof-current-size	|AOF 当前尺寸（字节）|
|aof-base-size	|AOF 上一次启动和重写的尺寸（字节）|

![在这里插入图片描述](http://images.gitbook.cn/fbf5bcb0-fb2f-11e7-8c30-4fe386329305)

由上图可知，bgrewriteaof 命令发出后，Redis 会在父进程中 fork 一个子进程，同时父进程会分别对旧的 AOF 文件和新的 AOF 文件发出 aof_buf 和 aof_rewrite_buf 命令，同时子进程写入新的 AOF 文件，并通知父进程，最后 Redis 使用 aof_rewrite_buf 命令写入新的 AOF 文件。

实际配置过程如下。
```
appendonly yes // appendonly 
 默认是 no
appendfilename "append only - ${port}.aof" //设置 AOF 名字
appendfsync everysec // 每秒同步
dir /diskpath // 新建一个目录
no-appendfsync-on-rewrite yes // 为了减少磁盘压力，AOF 性能上需要权衡。默认是 no，不会丢
```


#### 5.4 Redis 持久化开发运维时遇到的问题
问题可总结为四种，即 fork 操作、进程外的开销和优化、AOF 追加阻塞和单机多实例部署。

1. fork 操作
fork 操作包括以下三种：

- 同步操作，即 bgsave 时是否进行同步；
- 与内存量息息相关：内存越大，耗时越长；
- info：lastest_fork_usec，持久化操作。
改善 fork 的方式有以下四种：

- 使用物理机或支持高效 fork 的虚拟技术；
- 控制 Redis 实例最大可用内存 maxmemory；
- 合理配置 Liunx 系统内存分配策略：vm.overcommit_memory=1；
- 降低频率，如延长 AOF 重写 RDB，不必要的全量复制。

2. 子进程的开销以及优化
这里主要指 CPU、内存、硬盘三者的开销与优化。

- CPU

	开销：AOF 和 RDB 生成，属于 CPU 密集型，对 CPU 是巨大开销；

	优化：不做 CPU 绑定，不与 CPU 密集型部署。

- 内存

	开销：需要通过 fork 来消耗内存的，如 copy-on-write。

	优化：echo never > /sys/kernel/mm/transparent_hugepage/enabled，有时启动的时候会出现警告的情况，这个时候需要配置这个命令。

- 硬盘

   开销：由于大量的 AOF 和 RDB 文件写入，导致硬盘开销大，建议使用 iostat、iotop 分析硬盘状态。

  优化：

 不要和高硬盘负载部署到一起，比如存储服务、消息队列等等；

 配置文件中的 no-appendfsync-on-rewrite 设置成 yes；

 当写入量很大的时候，建议更换 SSD 硬盘；

 单机多实例持久化文件考虑硬盘分配分盘。

3. AOF 追加阻塞
我们如果使用 AOF 策略，通常就会使用每秒刷盘的策略（everysec），主线程会阻塞，直到同步完成。首先我们知道主线程是非常宝贵的资源，其次我们每秒刷盘实际上未必是 1 秒，可能是 2 秒的数据。
![在这里插入图片描述](http://images.gitbook.cn/47dd7580-fc18-11e7-b435-c9c42b4c17e4)

我们如何定位 AOF 阻塞？

- 通过 Redis 日志
![在这里插入图片描述](http://images.gitbook.cn/776f3c10-fc19-11e7-b435-c9c42b4c17e4)

上图可以看到， Redis 日志会出现上述的语句，告诉你异步 IO 同步时间太长，你的硬盘是否有问题，同时会拖慢 Redis。

当然除了上述的问题，你还可以用 Redis 的 info 方式来确定问题。

...


### 第06课：高可用运维必学的 Redis 复制

要实现分布式数据库的更大的存储容量和高并发访问量，我们会将原来集中式数据库的数据分别存储到其他多个网络节点上。Redis 为了解决这个单一节点的问题，也会把数据复制多个副本部署到其他节点上进行复制，实现 Redis的高可用，实现对数据的冗余备份，从而保证数据和服务的高可用。

下面我们将从以下五个方面全面认识 Redis 复制。

- 什么是主从复制
- 复制的配置
- 全量复制和部分复制
- 故障如何处理
- 开发和运维遇到的常见问题

在此之前，我们需要明确将 Redis 应用到工程项目中时，只用一台 Redis 显然是万万不能的，为什么不能呢？主要概括为以下三个原因。
- 第一，机器故障
- 第二，容量瓶颈。
- 第三，QPS 瓶颈。

#### 6.1 什么是主从复制

![](http://images.gitbook.cn/f143c910-00a7-11e8-b469-398f4f0730cb)
如上图所示，我们将 Redis 服务器作为 Master 主库，另外一台作为 Slave 从库，主库只是负责写数据，每次有数据更新的时候，Redis 会将数据同步到其他的从库中，而从库只是负责读数据。

当然你还可以根据业务需求，增加更多的从库，如下图所示，红色的 Redis 为主库，蓝色的是三台从库。
![](http://images.gitbook.cn/6c96c2a0-00aa-11e8-967f-f3c9b2437329)


很多小伙伴都觉得这没什么好说的，但在这里我还是想说这么做的两大好处。

- 实现了读写分离，读写分离不仅可以提高服务器的负载能力，同时可根据需求的变化，改变从库的数量，第一张图中只有一个从库，你还可以像第二张图那样增加至两个、三个……你觉得这个优点怎么样？

- 数据备份了多份，如果一台机器宕机，你可以从其他机器上快速恢复。但需要注意的是一台主库可以拥有多个从库，但一个从库却只能隶属于一个主库。


#### 6.2 复制的配置
接下来，我们讲一下主从复制的作用。首先请看下面这张图。
![](http://images.gitbook.cn/e14efa90-00aa-11e8-b469-398f4f0730cb)


6.3 全量复制和部分复制
在讲解全量复制和部分复制之前，我们先来讲一下，runid 和偏移量的概念。

什么是 runid，每次 Redis 启动的时候，Redis 就会有一个运行的 ID，这个 ID 只在 Redis 运行的时候才有，如果关闭 runid 就不存在了。runid 的作用是一个标识，如果主库去复制从库的数据，就需要根据这个 runid 去复制。

6.5 开发和运维中的问题
我将从下面四点来说明：读写分离、主从配置不一致、规避全量复制、规避复制风暴。
1. 读写分离
读流量分摊到从节点。这是个非常好的特性，如果一个业务只需要读数据，那么我们只需要连一台 Slave 从机读数据。

2. 配置不一致
主机和从机不同，经常导致主机和从机的配置不同，并带来下列两种问题。

- 数据丢失：主机和从机有时候会发生配置不一致的情况，例如 maxmemory 不一致，如果主机配置 maxmemory 为 8G，从机 Slave 设置为 4G，这个时候是可以用的，而且还不会报错。但是如果要做高可用，让从节点变成主节点的时候，就会发现数据已经丢失了，而且无法挽回。

- 数据结构优化参数导致不一致：hash-max-ziplist-enties 参数，如果主机对这些优化参数设置了，从机 Slave 却没有优化，就会发生数据不一致的情况。


3. 规避全量复制
全量复制指的是当 Slave 从机断掉并重启后，runid 产生变化而导致需要在 Master 主机里拷贝全部数据。这种拷贝全部数据的过程非常耗资源。

全量复制是不可避免的，例如第一次的全量复制是不可避免的，这时我们需要选择小主节点，且maxmemory 值不要过大，这样就会比较快。同时选择在低峰值的时候做全量复制。
造成全量复制的原因之一是主从机的运行 runid 不匹配。
造成全量复制的第二个原因是复制缓冲区空间不足，比如默认值 1M，可以部分复制。

4. 规避复制风暴
当一个主机下面挂了很多个 Slave 从机的时候，主机 Master 挂了。这时 Master 主机重启后，因为 runid 发生了变化，所有的 Slave 从机都要做一次全量复制。这将引起单节点和单机器的复制风暴，开销会非常大。
![](http://images.gitbook.cn/581fbbc0-017b-11e8-b469-398f4f0730cb)

- 单节点复制风暴。当主节点重启，多从节点会复制。这个时候需要更换复制拓扑。上图就是改变拓扑结构的问题，通过在 Slave 下再分从机，可以有效的减少主机 Master 的压力。
![](http://images.gitbook.cn/1a2ef3d0-017b-11e8-b469-398f4f0730cb)
- 单机器的复制风暴。如上图，如果每个 Master 主机只有一台 Slave 从机，那么当机器宕机以后，会产生大量全量复制。这是非常危险的情况，带宽马上会被占用，会导致不可用。这个问题在实际运维中必须注意。在这种情况下，建议将**单机器改成 Redis Sentinel。这样可以自动将从机变成主机 Master。**




### 07：Redis Sentinel 部署和运维
上一篇，我们讲解了 Redis 复制的主要内容，但 Redis 复制有一个缺点，当主机 Master 宕机以后，我们需要人工解决。那么能不能自动解决主机宕机的问题呢？

Redis Sentinel 正是为了解决这样的问题而被开发的。Redis Sentinel 是一个分布式的架构，每一个 Sentinel 节点会对数据节点和其余 Sentinel 节点进行监控，当发现某个节点无法到达的时候，会自动标识该节点。如果这个节点是主节点，那么它会和其他 Sentinel 节点“协商”，大部分节点都认为主节点无法到达的时候，它们会选举一个 Sentinel 节点来完成自动故障转移，同时会告知 Redis 的应用方。

由于这个过程是自动化的，不需要人工参与，大大提高了 Redis 的高可用性。

接下来，我们将从实现流程、安装配置、客户端连接、实现原理、常见开发运维问题这五个方面来探讨。

#### 7.1 实现流程
如下图所示，Sentinel 集群会监控每一个 Slave 和 Master。客户端不再直接从 Redis 获取信息，而是通过 Sentinel 集群来获取信息。

![](http://images.gitbook.cn/d93125c0-0cb9-11e8-a370-e181c30bd776)

再看下面这张图，当 Master 宕机了，Sentinel 监控到 Master 有问题，就会和其他 Sentinel 进行选举，选举出一个 Sentinel 作为领导，然后选出一个 Slave 作为 Master，并通知其他 Slave。上图 Slave1 变成了 Master，如果原来的 Master 又连上了，也会变成 Slave 从机。

![](http://images.gitbook.cn/0b10c960-0cba-11e8-9706-9106925a3925)

#### 7.2 安装与配置
我们将从以下两个方面讲解如何安装和配置主从节点和 Sentinel 节点。

- 如何配置开启主从节点；
- 如何开启 Sentinel 监控主节点。

1. 开启主从节点
Sentinel 对主节点和从节点的配置是不同的，需要分别配置，我们分开来讲解。

- 主节点配置
我们在命令行使用下面的命令进行主节点的启动。
```
redis-server redis-7000.conf
```
启动完成以后，我们参考下面的配置进行参数的设置。
```
port 7000
daemonize yes // 守护进程
pidfile /var/run/redis-7000.pid // 给出 pid
logfile “7000.log” // 日志查询
dir "/opt/redis/data" // 工作目录
```


- 从节点配置
我们在命令行使用下面的命令进行从节点的启动。
```
redis-server redis-7001.conf
redis-server redis-7002.conf
```
启动完成以后，和主节点配置一样，配置下面的参数。这个时候要注意，我们需要分别对 Slave 节点的每台机器进行配置。
Slave1 的配置如下。
```
port 7001
daemonize yes // 守护进程
pidfile /var/run/redis-7001.pid // 给出 pid
logfile “7001.log” // 日志查询
dir "/opt/redis/data" // 工作目录
slaveof 127.0.0.1 7000
```
Slave2 的配置如下。
```
port 7002
daemonize yes // 守护进程
pidfile /var/run/redis-7002.pid // 给出 pid
logfile “7002.log” // 日志查询
dir "/opt/redis/data" // 工作目录
slaveof 127.0.0.1 7000
```

2. Sentinel 监控主要配置
开启了主从节点以后，我们需要对 Sentinel 进行监控上的配置，见下面的配置参数。
```
port 端口号
dir "/opt/redis/data/"
logfile "端口号.log"
sentinel monitor mymaster 127.0.0.1 7000 2
sentinel down-after-millseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

由于需要配置多台 Sentinel，从上面配置信息可以看到，除了修改端口号，其他配置都是相同的。重点看最后四个配置，这四个配置是 Sentinel 的核心配置。我们分别来解释一下这四个配置，斜杠后面的文字解释了该参数的意义。
```
sentinel monitor mymaster 127.0.0.1 7000 2 // 监控的主节点的名字、IP 和端口，最后一个 2 表示有 2 台 Sentinel 发现有问题时，就会发生故障转移；

sentinel down-after-millseconds mymaster 30000 // 这个是超时的时间。打个比方，当你去 ping 一个机器的时候，多长时间后仍 ping 不通，那么就认为它是有问题；

sentinel parallel-syncs mymaster 1 // 指出 Sentinel 属于并发还是串行。1 代表每次只能复制一个，可以减轻 Master 的压力；

sentinel failover-timeout mymaster 180000 // 表示故障转移的时间。
```

7.3 Sentinel 客户端原理
我们配置高可用的时候，如果只是配置服务端的高可用是不够的。如果客户端感知不到服务端的高可用，是不会起作用的。所以，我们不但要让服务端高可用，还要让客户端也是高可用的。

我们先来看下客户端基本原理。

第一步，客户端 Client 需要遍历 Sentinel 节点集合，找到一个可用的 Sentinel 节点，同时需要获取 Master 主机的 masterName。如下图所示。

![](http://images.gitbook.cn/1ff79610-0cba-11e8-bdd7-1d78e572a792)

第二步，当客户端找到 Sentinel-2 节点的时候，Client 会通过 get-master-addr-by-name 命令获取 masterName，这个时候，Sentinel-2 会获取真正的名称和地址。如下图所示。

![](http://images.gitbook.cn/73df8940-0cba-11e8-b141-a5ea9a5a91f4)

第三步，Client 获取到 Master 节点的时候，还会发出 role 或 role replication 命令，验证是不是 Master 节点，Sentinel-2 会返回这个节点信息加以验证。如下图所示。

![](http://images.gitbook.cn/55760b50-0cba-11e8-a370-e181c30bd776)

第四步，如果 Sentinel 感知到 Master 宕机了，这时 Sentinel 集群应该是最先知道的。客户端和 Sentinel 集群之间其实是发布订阅，客户端 Client 去订阅某个 Sentinel 的频道，如果哪个 Sentinel 发现 Master 发生了变化，Client 是会被通知到这个信息的，如下图所示。但是要注意这个不是代理模式。

![](http://images.gitbook.cn/86744fa0-0cba-11e8-b141-a5ea9a5a91f4)

总结一下，以上四步就是客户端和 Sentinel 集群的基本原理，任何客户端原理都是按照这个流程做的，只是内部的封装不同而已。



1. Jedis
我们先通过使用率最高的 Java 的客户端 Jedis 讲起。

如何通过代码实现 Sentinel 的访问，让我们来看看代码如何连接 Sentinel 的资源池。
```
JedisSentinelPool sentinelPool = new JedisSentinelPool(masterName,sentinelSet,poolConfig,timeout); //内部的本质还是去连接 Master 主机，参数 masterName 表示 Master 名称，sentinelSet
 表示 Sentinel 集合，后面依次是 poolConfig 配置和超时时间
Jedis jedis = null;
try{
    //获得
 redisSentinelPool 资源
    jedis = redisSentinelPool.getResource();
    //Jedis 相关的命令
}catch (Exception e){
    logger.error(e.getMessage(),e);
}finally{
    if(jedis!=null){
        jedis.close(); // Jedis 归还
    }
}
```

2. redis-py
接下来我们再来看下如何使用 Python 连 Redis 客户端的 Sentinel，和 Jedis 一样，我们将直接给出连接 Sentinel 的代码。
```
from redis.sentinel import Sentinel
sentinel = Sentinel([('localhost',26379),('localhost',26380),('localhost',26381)],socket_time=0.1) // 获取可用的 Sentinel，并设置超时时间。

sentinel.discover_master('mymaster') // 获取 Master 地址
>>> ('127.0.0.1',7000)

sentinel.discover_slaves('mymaster') //获取 Slave 地址
>>> [('127.0.0.1',7001),('127.0.0.1',7002)]
```


sentinal 原理...



### 08：Redis Cluster——分布式解决方案

#### 8.1 为什么要有集群
首先是并发量，一般 QPS 到10万每秒已经非常牛了，随着公司业务的发展，或者当需要离散计算的时候，需要用到中间件缓存的时候，业务需要100万每秒。这个时候，我们就需要使用分布式了。

其次是数据量，一般一个 Redis 的内存大约是16G~256G，假设我们在一台主从机器上配置了200G内存，但是业务需求是需要500G的时候，我们首先不会通过升级硬件，而是通过分布式。

我们对并发量大和数据量剧增的时候，采取的最常用的手段就是加机器，对数据进行分区。做一个形象的比喻，我们的数据量相当于货物，当货物只有很少一部分的时候，我们可以使用驴来拉货；当货物多起来了的时候，已经超出驴能拉的范围，我们可以使用大象来拉货；当货物更多的时候，已经没有更强壮的动物了，这个时候我们可以考虑使用多只大象来拉货。

分布式就是一种采用某种规则对多台机器管理的方式。采用分布式我们就是为了节省费用。

#### 8.2 如何进行数据分布
什么是数据分布？数据分布有两种方式，顺序分区和哈希分区。

1. 顺序分布
顺序分布就是把一整块数据分散到很多机器中，如下图所示。
![](http://images.gitbook.cn/ca6bd2e0-1067-11e8-82c7-73b6ab65f44f)


2. 哈希分区
如下图所示，1~100这整块数字，通过 hash 的函数，取余产生的数。这样可以保证这串数字充分的打散，也保证了均匀的分配到各台机器上。
![](http://images.gitbook.cn/29f9b000-1069-11e8-ab94-d981756e188b)



### 09：缓存设计与优化


#### 9.1 缓存收益与成本的问题

1. 收益
主要有以下两大收益。

- 加速读写：通过缓存加速读写，如 CPU L1/L2/L3 的缓存、Linux Page Cache 的读写、游览器缓存、Ehchache 缓存数据库结果。
- 降低后端负载：后端服务器通过前端缓存来降低负载，业务端使用 Redis 来降低后端 MySQL 等数据库的负载。

2. 成本
产生的成本主要有以下三项。

- 数据不一致：这是因为缓存层和数据层有时间窗口是不一致的，这和更新策略有关的。
- 代码维护成本：这里多了一层缓存逻辑，顾会增加成本。
- 运维费用的成本：如 Redis Cluster，甚至是现在最流行的各种云，都是费用的成本了。


3. 使用场景
使用场景主要有以下三种。

- 降低后端负载：这是对高消耗的 SQL，join 结果集和分组统计结果缓存。
- 加速请求响应：这是利用 Redis 或者 Memcache 优化 IO 响应时间。
- 大量写合并为批量写：比如一些计数器先 Redis 累加以后再批量写入数据库。


#### 9.2 缓存的更新策略
主要有以下三种策略。
- LRU、LFU、FIFO 算法策略。例如 maxmemory-policy，这是最大内存的策略，当 maxmemory 最大时，会优先删除过期数据。我们在控制最大内存，让它帮我们去删除数据。
- 过期时间剔除，例如 expire。设置过期时间可以保证其性能，如果用户更新了重要信息，应该怎么办。所以这个时候就不适用了。
- 主动更新，例如开发控制生命周期。

这三个策略中，**一致性最好的就是主动更新。能够根据代码实时的更新数据，但是维护成本也是最高的**；算法剔除和超时剔除一致性都做的不够好，但是维护成本却非常低。

根据缓存的使用场景，我们会采用不同的更新策略。

实际开发中我给大家以下两个建议。

- 低一致性：最大内存和淘汰策略，数据库有些数据是不需要马上更新的，这个时候就可以用低一致性来操作。
- 高一致性：超时剔除和主动更新的结合，最大内存和淘汰策略兜底。你没办法确保内存不会增加，从而使服务不可用了。

#### 9.3 缓存粒度问题
我们知道，用户第一次访问客户端，客户端访问 Redis 肯定是没有的，这个时候只能从数据库 DB 那里获取信息，代码如下。
```
select * from t_teacher where id= {id}
```
在 Redis 设置用户信息缓存，代码如下。
```
set teacher:{id} select * from t_teacher where id= {id}
```

这个时候我们来看看缓存粒度问题。

因为我们要更新全部属性。到底我们是采用 select * 还是仅仅只是更新你需要更新的那些字段呢？如下两段代码。

```
set key1 = ? from select * from t_teacher
set key1 = ? from select key1 from t_teacher
```

缓存粒度控制可以从以下三个角度来观察，通过这三点来决定如何选择。

- 通用性：全量属性更好。上面一个对比 * 和某个字段的查询，最好是通过全量属性，这样的话，select * 具有很好的通用性，因为如果你 select 某个字段的话，未来如果一旦业务改变，代码也要随之改变才可以。
- 占用空间：部分属性会更好。因为这样占用的空间是最小的。
- 代码维护上：表面上全量属性会更好。我们真的需要全量吗？其实我们在使用缓存的时候，优先考虑的是内存而不单单只是保证代码的扩展性。

#### 9.4 缓存穿透问题
首先大家看下下面这张图。
![](http://images.gitbook.cn/fceb3e50-17cd-11e8-8087-9b6b3b447adf)

当请求发送给服务器的时候，缓存找不到，然后都堆到数据库里。这个时候，缓存相当于穿透了，不起作用了。

原因有两点：

- 业务代码自身的问题。很多实际开发的时候，如果是一个不熟练的程序员，由于缺乏必要的大数据的意识，很多代码在第一次写的时候是 OK 的，但是当需要修改业务代码的时候，常常会出现问题。
- 恶意攻击和爬虫问题。网络上充斥着各种攻击和各种爬虫模仿着人为请求来访问你的数据。如果恶意访问穿透你的数据库，将会导致你的服务器瞬间产生大量的请求导致服务中止。

那我们去如何发现这些问题呢？

- 业务的相应时间：一般请求的时间都是稳定的，但是如果出现类似穿透现象，必然在短时间内有一个体现。
- 业务本身的问题。产品的功能出现问题。
- 对缓存层命中数、存储层的命中数这些值的采集。

1. 解决方案一：**缓存空对象**
当缓存中不存在，访问数据库的时候，又找不到数据，需要设置给 cache 的值为 null，这样下次再次访问该 id 的时候，就会直接访问缓存中的 null 了。

但是可能存在的两个问题。首先**是需要更多的键**，但是如果这个量非常大的话，对业务也是有影响的，所以需要设置过期时间；其次是缓存层和存储层数据“短期”不一致。当缓存层过期时间到了以后，可能会产生和存储层数据不一致的情况。这个时候需要使用一些消息队列等方式，来确保这个值的一致性。

下面的代码用 Java 来实现简单的缓存空对象。
```
public String getCacheThrough(String key){
    String cacheValue = cache.get(key);
    if(StringUtils.isBlank(cacheValue)){ // 如存储数据为空
        String storageValue = storage.get(key);
        cache.set(key,storageValue);//需要设置一个过期时间
        if(StringUtils.isBlank(strageValue){
            cache.expire(key.60*10);
}    
    return storageValue;
    }else{
    return cacheValue;
 }
}
```


2. 解决方案二：布隆过滤器拦截
布隆过滤器，实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

类似于一个字典，你查词典的时候不需要把所有单词都翻一遍，而是通过目录的方式，用户通过检索的形式在极小内存中可以找到对应的内容。

虽然布隆过滤器可以通过极小的内存来存储，但是免不了需要一部分代码来维护这个布隆过滤器，并且经常需要根据规则来调整，在选取是否使用布隆过滤器，还需要通过场景来选取。


#### 9.5 无底洞问题优化
无底洞问题就是即使加机器，性能却没有提升，反而降低了。到底这是怎么回事呢？先看下面的图。

![](http://images.gitbook.cn/de91d5e0-17e1-11e8-8087-9b6b3b447adf)


当客户端增加一个缓存的时候，只需要 mget 一次，但是如果增加到三台缓存，这个时候则需要 mget 三次了，每增加一台，客户端都需要做一次新的 mget，给服务器造成性能上的压力。

同时，mget 需要等待最慢的一台机器操作完成才能算是完成了 mget 操作。这还是并行的设计，如果是串行的设计就更加慢了。

通过上面这个实例可以总结出：更多的机器！=更高的性能
但是并不是没办法，一般在优化 IO 的时候可以采用以下几个方法。

- 命令的优化。例如慢查下 keys、hgetall bigkey。
- 我们需要减少网络通讯的次数。这个优化在实际应用中使用次数是最多的，我们尽量减少通讯次数。
- 降低接入成本。比如使用客户端长连接或者连接池、NIO 等等。


1. 四种批量优化的方法
四种方法主要是：串行 mget、串行 IO、并行 IO、hash_tag。

 - 串行 mget
如下图所示，串行 mget 就是根据 Redis 增加的台数，来 mget 多次网络时间。
![](http://images.gitbook.cn/b6217b30-17e4-11e8-8087-9b6b3b447adf)

- 串行 IO
如下图所示，根据 key 的增加，先在客户端组装成各种 subkeys，然后一次性根据 pipeline 方式进行传输，这样能有效的减少网络时间。
![](http://images.gitbook.cn/55542e50-17e5-11e8-8087-9b6b3b447adf)

- 并行 IO
如下图所示，在串行 IO 的基础上，再根据并行打包，把请求一次性的传给 Redis 集群。
![](http://images.gitbook.cn/814bd3a0-17e5-11e8-8087-9b6b3b447adf)

- hash_tag
如下图所示，用最极端的方式进行哈希传送给 Redis 集群。
![](http://images.gitbook.cn/0d460650-17e6-11e8-8087-9b6b3b447adf)


总之，实际使用过程中，我们根据特定的业务场景，选定对应的批量优化方式，可以有效的优化。

#### 9.7 热点 Key 重建优化
我们知道，使用缓存，如果获取不到，才会去数据库里获取。但是如果是热点 key，访问量非常的大，数据库在重建缓存的时候，会出现很多线程同时重建的情况。

![](http://images.gitbook.cn/08e464d0-1831-11e8-ab6f-c92a5ff63d5e)
如上图，就是因为高并发导致的大量热点的 key 在重建还没完成的时候，不断被重建缓存的过程，由于大量线程都去做重建缓存工作，导致服务器拖慢的情况。只有最后一个是重建完成，命中缓存。

为了解决以上的问题，我们着重研究了三个目标和两个解决方案。

三个目标为：

- 减少重建缓存的次数；
- 数据尽可能保持一致；
- 减少潜在的风险。


两个解决方案为

- 互斥锁
- 永不过期
我们根据三个目标，解释一下两个解决方案。


1. 互斥锁（mutex key）
由下图所示，第一次获取缓存的时候，加一个锁，然后查询数据库，接着是重建缓存。这个时候，另外一个请求又过来获取缓存，发现有个锁，这个时候就去等待，之后都是一次等待的过程，直到重建完成以后，锁解除后再次获取缓存命中。
![](http://images.gitbook.cn/0a435bc0-1835-11e8-ac0a-03d2406b28d7)

那么这个过程是怎么做到的呢？请见下面代码演示。
```
public String getKey(String key){
    String value = redis.get(key);
    if(value == null){
        String mutexKey = "mutex:key:"+key; //设置互斥锁的key
        if(redis.set(mutexKey,"1","ex 180","nx")){ //给这个key上一把锁，ex表示只有一个线程能执行，过期时间为180秒
          value = db.get(key);
          redis.set(key,value);
          redis.delete(mutexKety);
		  }else{
				// 其他的线程休息100毫秒后重试
				Thread.sleep(100);
				getKey(key);
		  }
	}
 return value;
}
```

但是互斥锁也有一定的问题，就是大量线程在等待的问题。下面我们就来讲一下永远不过期。

2. 永远不过期


首先在缓存层面，并没有设置过期时间（过期时间使用 expire 命令）。但是功能层面，我们为每个 value 添加逻辑过期时间，当发现超过逻辑过期时间后，会使用单独的线程去构建缓存。这样的好处就是不需要线程的等待过程。见下图。
![](http://images.gitbook.cn/4e52cc20-1866-11e8-ac0a-03d2406b28d7)

如上图所示，T1 时间无需等待，直接输出，到 T2 的时候，发现 value 已经到了过期时间，于是就开始构建缓存，还是输出旧值。到了 T3 已经是旧值，直到 T4 时间，构建缓存已经完成，直接输出新值。

这样就避免了上面互斥锁大量线程等待的问题。具体实现伪代码如下：
```
public String getKey(final String key){
    V v = redis.get(key);
    String value = v.getValue();
    long logicTimeout = v.getLogicTimeout();
    if(logicTimeout >= System.currentTimeMillis()){
      String mutexKey = "mutex:key:"+key; //设置互斥锁的key
      if(redis.set(mutexKey,"1","ex 180","nx")){ //给这个key上一把锁，ex表示只有一个线程能执行，过期时间为180秒
        threadPool.execute(new Runable(){
            public void run(){
            String dbValue = db.getKey(key);
            redis.set(key,(dbValue,newLogicTimeout));//缓存重建，需要一个新的过期时间
            redis.delete(keyMutex); //删除互斥锁
     }
   };
  }
 }
}
```
互斥锁的优点是思路非常简单，具有一致性，其缺点是代码复杂度高，存在死锁的可能性。

永不过期的优点是基本杜绝 key 的重建问题，但缺点是不保证一致性，逻辑过期时间增加了维护成本和内存成本。










