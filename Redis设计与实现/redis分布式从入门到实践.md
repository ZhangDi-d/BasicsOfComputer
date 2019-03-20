
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


