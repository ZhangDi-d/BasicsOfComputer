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

### ZooKeeper Session ### 
ZooKeeper Session（会话），即为 ZooKeeper 客户端与服务端交互的通道。概述而言，客户端和服务端的 TCP 连接即为 Session，ZooKeeper 抽象了更多的会话状态。
Session 于 ZooKeeper 服务端，以 SessionId 作为唯一标示，同时，ZooKeeper 支持客户端使用 SessionId 进行“Session 复用”（需要同时提供 SessionPasswd）。

#### ZooKeeper Session 状态维护 ####
ZooKeeper 客户端对象创建时，Session 即进入 CONNECTING 状态，当客户端与服务端（集群的任意节点）完成连接，即进入 CONNECTED 状态。

客户端主动关闭 Session 前，通过“心跳”维护 Session 有效性，若连接中断，ZooKeeper 客户端将尝试重新连接（再次进入 CONNECTING ）：

若在“Session 超时时间”内，连接重新建立，Session 继续有效，再次进入 CONNECTED；
否则，服务端将标记 Session 过期（即使连接最终重新建立），进行清理（例如：临时 ZNode 删除），Session 最终进入 CLOSE 状态。
Session 是否过期，完全由 ZooKeeper 服务端维护。对于 ZooKeeper 客户端，仅当 Session 过期，才应当重新创建客户端对象。

多个客户端使用相同的 SessionId 与 SessionPasswd “连接复用”，可能出现“SessionMovedException”，请参阅：ZooKeeper Programmer's Guide。


### ZooKeeper Watch ### 
对于全部的“读”操作，ZooKeeper 允许客户端于 ZNode 设置 Watch，当 ZNode 变更时，Watch 将被触发并且通知到客户端（即 Watcher）。Watch 是 “一次性” 的，Watch 被触发时即被清除。

Watch“异步地”通知到客户端，“通知内容”不包含 ZNode 变更后的数据，需要由客户端读取。

由 ZooKeeper 确保，事件到客户端的通知，严格“按顺序”进行（事务于 ZooKeeper 中的顺序）。此外，当 Watch 被触发时，设置了 Watch 的客户端，接收到通知前，无法获取变更后的数据。


#### ZooKeeper Watch 事件类型 ####
我们假设判断 ZNode 是否存在、获取 ZNode 数据、获取 ZNode 子级 ZNode 的方法分别为 exists()、getData()、getChildren()。

- 创建事件：exists() 设置的 Watch 能够被触发；
- 删除事件：exists()、getData()、getChildren() 设置的 Watch 能够被触发；
- 变更事件：getData() 设置的 Watch 能够被触发；
- 子级 ZNode 事件： getChildren() 设置的 Watch 能够被触发。
对于单一的 Watch 对象（例如，回调函数），由单一变更引起的事件，Watch 对象将被调用仅仅被调用一次，即使由多个“读”进行了 Watch 设置。


### ZooKeeper ACL ###
ZooKeeper 通过 ACL 控制 ZNode 的访问权限（默认情况，ZNode 无访问权限控制），权限维度包括：

- CREATE：创建 ZNode；
- READ：获取 ZNode 数据及其子级 ZNode；
- WRITE：ZNode 数据写入；
- DELETE：删除 ZNode；
- ADMIN：权限设置。
- ZooKeeper 支持多种权限模式，最常见的 Digest 模式，类似于 username / password。

限于篇幅，关于 ACL，本文不予以展开，请参阅：ZooKeeper Programmer's Guide。

















































