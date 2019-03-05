# Redis设计与实现

#### 疑问:
 - Redis五种数据类型分别使用什么数据结构实现的?
 - Redis字符串数据类型能存储字符串,整数,浮点数,二进制位,内部是怎么实现的?
 - Redis一部分命令只能针对特定的数据类型(append之于字符串,HSET之于哈希表),不同的命令怎么进行类型检查的?
 - Redis怎么存储不同类型的键值对,过期键怎么自动删除的?
 - 除了数据库之外， Redis 还拥有发布与订阅、脚本、事务等特性， 这些特性又是如何实现的？
 - Redis 使用什么模型或者模式来处理客户端的命令请求？ 一条命令请求从发送到返回需要经过什么步骤？

## 第一部分：数据结构与对象
### 1.1简单动态字符串
Redis 没有直接使用 C 语言传统的字符串表示（以空字符结尾的字符数组，以下简称 C 字符串），而是自己构建了一种名为简单动态字符串（simple dynamic string，SDS）的抽象类型， 并将 SDS 用作 Redis 的默认字符串表示.

#### 1.1.1 sds
1.Sds 在 Redis 中的主要作用有以下两个：
- 实现字符串对象（ StringObject）；
- 在 Redis 程序内部用作 char* 类型的替代品；


2.例子-实现字符串对象
a.如果客户端执行命令：
```
redis> SET msg "hello world"
OK
```
那么 Redis 将在数据库中创建了一个新的键值对， 其中：
键值对的键是**一个字符串对象**， 对象的底层实现是一个保存着字符串 "msg" 的 SDS 。
键值对的值也是**一个字符串对象**， 对象的底层实现是一个保存着字符串 "hello world" 的 SDS 。

b.如果客户端执行命令：
```
redis> RPUSH fruits "apple" "banana" "cherry"
(integer) 3
```
那么 Redis 将在数据库中创建一个新的键值对， 其中：
键值对的键是一个字符串对象， 对象的底层实现是一个保存了字符串 "fruits" 的 SDS 。
键值对的值是一个列表对象， 列表对象包含了**三个字符串对象**， 这三个字符串对象分别由三个 SDS 实现： 第一个 SDS 保存着字符串 "apple" ， 第二个 SDS 保存着字符串 "banana" ， 第三个 SDS 保存着字符串 "cherry" 。

c.以下命令创建了另一个键值对，它的键是字符串对象，而值则是一个集合对象：
```
redis> SADD nosql "Redis" "MongoDB" "Neo4j"
(integer) 3
redis> SMEMBERS nosql
1) "Neo4j"
2) "Redis"
3) "MongoDB"
```

3.Redis 数据库里面的每个键值对（key-value pair）都是由对象（object）组成的：
- 其中， 数据库键总是一个字符串对象（string object）；
- 而数据库键的值则可以是字符串对象、 列表对象（list object）、 哈希对象（hash object）、 集合对象（set object）、 有序集合对象（sorted set object）这五种对象中的其中一种。


#### 1.1.2 Redis 中的字符串
1.为何需要sds而不是使用c字符串
在 C 语言中，字符串可以用一个 \0 结尾的 char 数组来表示。
比如说，hello world 在 C 语言中就可以表示为 "hello world\0" 。 它并不能高效地支持**长度计算和追加（ append）**这两种操作;
而在Redis中,长度计算(Redis命令:STRLEN)和追加(Redis命令:APPEND)都很常用,不应成为Redis性能的瓶颈.

sds 既可以高效地实现追加和长度计算，并且它还是二进制安全的.

2.sds实现












































