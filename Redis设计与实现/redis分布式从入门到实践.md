
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
