### Zookeeper 基本概念 ###
ZooKeeper 是面向分布式应用的协调服务，其实现了树形结构的数据模型（与文件系统类似），并且提供了简洁的编程原语。ZooKeeper 能够作为基础，用于构建更高层级的分布式服务。
ZooKeeper 是分布式的，具备高性能、高可用的特点。
![](http://images.gitbook.cn/3955e590-15d8-11e8-a2e2-d39d0a116722)
如上架构图所示，ZooKeeper 集群中包括：

Leader：提供 “读” & “写” 服务（Leader 由集群全部机器通过“Leader 选举”产生）。
Follower：集群中非 “Leader” 的其他节点。
集群的机器相互通信，基于 ZooKeeper 的实现机制，只要超过一半数量的机器正常，整个集群即能够正常提供服务。

ZooKeeper 数据全部于内存中存储，于硬盘中落地数据快照、事务日志。

### ZooKeeper 数据模型 ###
#### ZNode ####
ZooKeeper 实现树形结构的数据模型，即为 ZooKeeper 提供的 “命名空间”，与文件系统类型。“命名空间”由“路径”唯一标示，由“/”间隔的每一层级，“树形结构” 的任意节点，即为 ZNode。
ZooKeeper 的 ZNode，全部使用“绝对路径”，与文件系统所不同的是，ZNode 既允许存储数据，亦能够建立子级 ZNode。
需要说明，ZNode 存储的数据，限定为 1M。

#### ZNode 类型 ####
通常情况，ZNode 永久存储于 ZooKeeper，直到被主动删除。此外，ZooKeeper 允许客户端创建 “临时”（Ephemeral）ZNode，若客户端与 ZooKeeper 的连接中断，临时 ZNode 将自动删除。并且，临时 ZNode 不允许建立子级 ZNode。
ZooKeeper 支持“顺序”（Sequence）ZNode，顺序 ZNode 创建时，ZooKeeper 将自动添加“自增数字”作为后缀（“自增数字”由父级 ZNode 通过 4 字节有符号整数维护）。


#### ZNode Stat ####
除了存储的数据，ZNode 包含了称为 Stat 的数据结构，用于存储 ZNode 的属性信息，主要包括：

- cZxid / mZxid：ZNode 创建 / 最后更新的 Zxid
- ctime / mtime：ZNode 创建 / 最后更新的时间（Unix 时间，毫秒）
- dataVersion ：ZNode 数据版本
- dataLength ：ZNode 存储的数据长度
- numChildren ：子级 ZNode 的数量
- 其他关于 ACL、子级 ZNode 的信息
关于 Zxid：所有提交到 ZooKeeper 的事务，都会被标记唯一的 ZooKeeper Transaction Id。

























