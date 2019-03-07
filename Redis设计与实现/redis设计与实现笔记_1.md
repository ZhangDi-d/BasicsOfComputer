# Redis设计与实现

#### 疑问:
 - Redis五种数据类型分别使用什么数据结构实现的?
 - Redis字符串数据类型能存储字符串,整数,浮点数,二进制位,内部是怎么实现的?
 - Redis一部分命令只能针对特定的数据类型(append之于字符串,HSET之于哈希表),不同的命令怎么进行类型检查的?
 - Redis怎么存储不同类型的键值对,过期键怎么自动删除的?
 - 除了数据库之外， Redis 还拥有发布与订阅、脚本、事务等特性， 这些特性又是如何实现的？
 - Redis 使用什么模型或者模式来处理客户端的命令请求？ 一条命令请求从发送到返回需要经过什么步骤？

## 第一部分：数据结构与对象
###  1.1简单动态字符串
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

2.sds定义
```
struct sdshdr {

    // 记录 buf 数组中已使用字节的数量
    // 等于 SDS 所保存字符串的长度
    int len;

    // 记录 buf 数组中未使用字节的数量
    int free;

    // 字节数组，用于保存字符串
    char buf[];

};
```
3.一个SDS 示例：
free 属性的值为 0 ， 表示这个 SDS 没有分配任何未使用空间。
len 属性的值为 5 ， 表示这个 SDS 保存了一个五字节长的字符串。
buf 属性是一个 char 类型的数组， 数组的前五个字节分别保存了 'R' 、 'e' 、 'd' 、 'i' 、 's' 五个字符， 而最后一个字节则保存了空字符 '\0' 。(遵循空字符结尾这一惯例的好处是， SDS 可以直接重用一部分 C 字符串函数库里面的函数)
![在这里插入图片描述](http://redisbook.com/_images/graphviz-72760f6945c3742eca0df91a91cc379168eda82d.png)


4.带有未使用空间的SDS示例
![在这里插入图片描述](http://redisbook.com/_images/graphviz-5fccf03155ec72c7fb2573bed9d53bf8f8fb7878.png)

#### 1.1.3 SDS API
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190305115922240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv,size_16,color_FFFFFF,t_70)

#### 1.1.4 重点回顾
Redis 只会使用 C 字符串作为字面量， 在大多数情况下， Redis 使用 SDS （Simple Dynamic String，简单动态字符串）作为字符串表示。
比起 C 字符串， SDS 具有以下优点：
- 常数复杂度获取字符串长度。
- 杜绝缓冲区溢出。
- 减少修改字符串长度时所需的内存重分配次数。
- 二进制安全。
- 兼容部分 C 字符串函数。



###  1.2 链表

Redis 使用的 C 语言并没有内置这种数据结构， 所以 Redis 构建了自己的链表实现。

链表在 Redis 中的应用非常广泛， 比如列表键的底层实现之一就是链表： 当一个列表键包含了数量比较多的元素， 又或者列表中包含的元素都是比较长的字符串时， Redis 就会**使用链表作为列表键的底层实现**。

#### 1.2.1 链表和链表节点的 API


#### 1.2.2 重点回顾
- 链表被广泛用于实现 Redis 的各种功能， 比如列表键， 发布与订阅， 慢查询， 监视器， 等等。
- 每个链表节点由一个 listNode 结构来表示， 每个节点都有一个指向前置节点和后置节点的指针， 所以 Redis 的链表实现是双端链表。
- 每个链表使用一个 list 结构来表示， 这个结构带有表头节点指针、表尾节点指针、以及链表长度等信息。
- 因为链表表头节点的前置节点和表尾节点的后置节点都指向 NULL ， 所以 Redis 的链表实现是无环链表。
- 通过为链表设置不同的类型特定函数， Redis 的链表可以用于保存各种不同类型的值。


### 1.3 字典

字典， 又称符号表（symbol table）、关联数组（associative array）或者映射（map）， 是一种用于保存键值对（key-value pair）的抽象数据结构。
在字典中， 一个键（key）可以和一个值（value）进行关联（或者说将键映射为值）， 这些关联的键和值就被称为键值对。
**字典中的每个键都是独一无二的， 程序可以在字典中根据键查找与之关联的值**， 或者通过键来更新值， 又或者根据键来删除整个键值对， 等等。
字典经常作为一种数据结构内置在很多高级编程语言里面， 但 Redis 所使用的 C 语言并没有内置这种数据结构， 因此** Redis 构建了自己的字典实现。**
字典在 Redis 中的应用相当广泛， 比如** Redis 的数据库就是使用字典来作为底层实现的， 对数据库的增、删、查、改操作也是构建在对字典的操作之上的**。

举个例子， 当我们执行命令：
```
redis> SET msg "hello world"
OK
```
在数据库中创建一个键为 "msg" ， 值为 "hello world" 的键值对时， 这个键值对就是保存在代表数据库的字典里面的。

除了用来表示数据库之外， 字典还是哈希键的底层实现之一： 当一个哈希键包含的键值对比较多， 又或者键值对中的元素都是比较长的字符串时， Redis 就会使用字典作为哈希键的底层实现。

举个例子， website 是一个包含 10086 个键值对的哈希键， 这个**哈希键的键都是一些数据库的名字， 而键的值就是数据库的主页网址**：
```
redis> HLEN website
(integer) 10086

redis> HGETALL website
1) "Redis"
2) "Redis.io"
3) "MariaDB"
4) "MariaDB.org"
5) "MongoDB"
6) "MongoDB.org"
# ...
```
website 键的底层实现就是一个字典， 字典中包含了 10086 个键值对：
- 其中一个键值对的键为 "Redis" ， 值为 "Redis.io" 。
- 另一个键值对的键为 "MariaDB" ， 值为 "MariaDB.org" ；
- 还有一个键值对的键为 "MongoDB" ， 值为 "MongoDB.org" ；


#### 1.3.1 哈希算法



















































































