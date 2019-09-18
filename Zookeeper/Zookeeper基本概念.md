## Zookeeper 基础##
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


### ZooKeeper 一致性保证 ###
基于 ZooKeeper 的设计，允许使用“Leader”和“Follower”同时提供“读”服务，由此涉及“一致性”问题，ZooKeeper 保证：

- 顺序一致性（Sequential Consistency）：来自相同客户端提交的事务，ZooKeeper 将严格按照其提交顺序依次执行；
- 原子性（Atomicity）：于 ZooKeeper 集群中提交事务，事务将“全部完成”或“全部未完成”，不存在“部分完成”；
- 单一系统镜像（Single System Image）：客户端连接到 ZooKeeper 集群的任意节点，其获得的数据视图都是相同的；
- 可靠性（Reliability）：事务一旦完成，其产生的状态变化将永久保留，直到其他事务进行覆盖；
- 实时性（Timeliness）：事务一旦完成，客户端将于限定的时间段内，获得最新的数据。
- 综合而言，ZooKeeper 实现了“最终一致性”模型。



## ZooKeeper 使用方法 ##
### 服务部署 ###
ZooKeeper 允许进行单机模式（Standalone）和集群模式（Replicated）部署。

关于集群模式，需要说明以下两点。

- 使用集群模式进行 ZooKeeper 进行部署：集群中的全部 ZooKeeper 节点称为“quorum”，必须使用相同的配置文件。
- 根据 ZooKeeper 特性，ZooKeeper 集群通常以“奇数”作为节点数量（3、5……）。
ZooKeeper 配置文件（conf/zoo.cfg）涉及的配置项如下（以下仅列出 conf/zoo_sample.cfg 的内容 ，更多配置项请参阅 ZooKeeper Administrator's Guide）：
```
#
# ZooKeeper 基础时间单元，即 tick（毫秒）
#   1. 心跳时间间隔
#   2. 默认 session 超时时间：tickTime × 2
#
tickTime=2000

#
# 数据快照、事务日志、myid 文件的存储路径
#
dataDir=/data/service/zookeeper

#
# ZooKeeper 端口
#
clientPort=2181

#
# 适用于集群模式：集群节点与 Leader 建立初始化连接的时间限制（基于 tick 数计算）
#
initLimit=10

#
# 适用于集群模式：集群节点与 Leader 进行通信的时间限制（基于 tick 数计算）
#
syncLimit=5

#
# 适用于集群模式：集群的节点列表
#   2888：集群节点与 Leader 通信的端口
#   3888：集群进行 Leader 选举的端口
#
server.1=host1:2888:3888
server.2=host2:2888:3888
server.3=host3:2888:3888
# server.X=hostX:2888:3888  # X 即为 ZooKeeper Server Id
```

完成配置，于 dataDir 路径建立 myid 文件（内容即为 ZooKeeper Server Id）。使用以下脚本命令即可启动 / 停止 ZooKeeper：
```
# 启动
./bin/zkServer.sh start

# 停止
./bin/zkServer.sh stop
```
### 客户端 ### 
ZooKeeper 提供了非常简洁的编程接口，包括：

1.创建 ZNode
2.写入 ZNode 数据
3.读取 ZNode 数据
4.获取 ZNode 的子级 ZNode
5.删除 ZNode
6.判断 ZNode 是否存在

脚本
```
#
# 与 127.0.0.1:2181 的 ZooKeeper 建立连接，进入脚本客户端的控制台
#
./bin/zkCli.sh -server 127.0.0.1:2181
```

于 ZooKeeper 脚本客户端的控制台：
```
#
# 创建 ZNode /Y2018，2018 即为 ZNode 数据
#
create /Y2018 2018

#
# 创建 ZNode /Y2018/M02，Feb 即为 ZNode 数据（需要说明，ZooKeeper 不支持 ZNode 递归创建）
#   允许通过参数 -e 或 -s 创建 “临时” 或 “顺序” ZNode
#
create /Y2018/M02 Feb

#
# ZNode /Y2018/M02 写入 2018-Feb
#
set /Y2018/M02 2018-Feb

#
# ZNode /Y2018 写入 2018，基于 ZNode 版本 0
#   ZooKeeper 通过 set 命令的 “版本” 参数实现 CAS，“版本” 即来自 Stat - dataVersion
#
set /Y2018 2018 0

#
# 读取 ZNode /Y2018/M02 数据
#
get /Y2018/M02

#
# 获取 ZNode /Y2018 子级 ZNode
#
ls /Y2018

#
# 删除 ZNode /Y2018/M02
#   1. 与 set 相同，delete 命令支持 “版本” 参数
#   2. ZNode 删除时，必须确保其无子级 ZNode
#
#
delete /Y2018/M02

#
# zkCli.sh 未提供 “判断 ZNode 是否存在” 命令，使用 get 亦可
#
```

Java API，使用 Curator
Curator 使用“流式接口”风格封装了 ZooKeeper Java API，并实现了诸多工具，例如 Leader 选举。

使用 Curator，于 build.gradle 添加如下代码。
```
compile('org.apache.curator:curator-framework:4.0.0') {
    exclude group: 'org.apache.zookeeper', module: 'zookeeper'
}
compile('org.apache.zookeeper:zookeeper:3.4.11')
```

限于篇幅，本文仅以“判断 ZNode 是否存在”及其 Watch 作为示例，代码如下。
```Java
package com.gitchat.zk;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.CuratorWatcher;
import org.apache.curator.retry.RetryOneTime;
import org.apache.zookeeper.Watcher.Event.EventType;

import java.util.concurrent.CountDownLatch;

public class Main {

    static CountDownLatch countDownLatch = new CountDownLatch(1);

    public static void main(String args[]) throws Exception {
        //
        // 与 127.0.0.1:2181 连接，重试策略：仅重试 1 次
        //
        CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2181", new RetryOneTime(1000));
        client.start();

        //
        // 判断 ZNode /2018-02 是否存在，设置 Watch
        //
        client.checkExists().usingWatcher((CuratorWatcher) (event) -> {
            if (event.getType() == EventType.NodeCreated) {
                System.out.println(Thread.currentThread().getName() + ": " + "node Created.");
            } else if (event.getType() == EventType.NodeDeleted) {
                System.out.println(Thread.currentThread().getName() + ": " + "node Deleted.");
            }

            countDownLatch.countDown();
        }).inBackground((c, event) -> {
            //
            // 异步接口
            //
            System.out.println(Thread.currentThread().getName() + ": " + "checkExists == " + event.getResultCode() + ", event == " + event.getType());
        }).forPath("/2018-02");

        countDownLatch.await();
        System.out.println("terminate...");
    }
}
```

如代码所示：

- 使用 CuratorFrameworkFactory 的 newClient 方法创建 ZooKeeper Session；
- 使用 Curator 的异步接口，判断 ZNode 是否存在，并注册 CuratorWatcher。
在此做下说明：

- ZooKeeper 的异步响应、Watch 通知，全部由 EventThread 完成（ZooKeeper 客户端与服务端的 TCP 通信，由独立的 I/O 线程完成）；
- Curator 提供了 org.apache.curator.framework.recipes.cache.NodeCache，能够简化 Watch 设置，例如实现自动的 Watch 设置。

### ZooKeeper 实践场景 ### 
根据前文阐述，基于 ZooKeeper 的特性，能够用于构建分布式系统的核心组件，包括但不限于命名服务、Leader 选举、分布式同步、配置注册中心……

限于篇幅，本章节使用 ZooKeeper 构建分布式锁和分布式 Barrier，作为 ZooKeeper 实践的示例。

#### 分布式锁 ####
```Java
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.api.CuratorWatcher;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.Watcher.Event.EventType;
import org.apache.zookeeper.data.Stat;

public class Locker {

    private CuratorFramework zkClient;

    private final String LOCKER = "/lock";

    private final Integer conditionVariable = 0;

    private Boolean locked = false;

    public Locker(CuratorFramework zkClient) {
        this.zkClient = zkClient;
    }

    //
    // 获取 “互斥锁”（阻塞）
    //
    public synchronized void lock() throws Exception {
        assert !locked;

        while (true) {
            try {
                this.zkClient.create().withMode(CreateMode.EPHEMERAL).forPath(LOCKER, "lock".getBytes());

                //
                // lock acquired
                //
                locked = true;
                return;
            } catch (KeeperException exception) {
                if (exception.code() == KeeperException.Code.NODEEXISTS) {
                    //
                    // lock acquire by others
                    //
                } else {
                    throw exception;
                }
            }

            Stat stat = this.zkClient.checkExists().usingWatcher((CuratorWatcher) (event) -> {
                if (event.getType() == EventType.NodeDeleted) {
                    synchronized (conditionVariable) {
                        conditionVariable.notify();
                    }
                }
            }).forPath(LOCKER);

            if (stat != null) {
                //
                // lock NOT released, block ...
                //
                synchronized (conditionVariable) {
                    conditionVariable.wait();
                }
            }
        }
    }

    //
    // 释放 “锁”
    //
    public synchronized void unLock() throws Exception {
        assert locked;

        this.zkClient.delete().forPath(LOCKER);
        locked = false;
    }
}
```

如上面代码所示：

使用“ZNode 创建”获取“锁”，借助 ZooKeeper 的原子性，确保互斥：有且仅有唯一的 ZooKeeper 客户端能够获得“锁”；
将 ZNode 删除，即完成“锁”释放；
若未能获取“锁”，设置 ZooKeeper Watch 并阻塞，当 ZNode 被删除（“锁”释放）时，再次尝试获取“锁”。

代码中，代表“锁”的 ZNode 是“临时 ZNode”，实现：ZooKeeper 客户端异常中止时，“锁”自动释放。

#### 分布式 Barrier ####
```JAVA
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.api.CuratorWatcher;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.Watcher.Event.EventType;

import java.util.List;

public class Barrier {

    private CuratorFramework zkClient;

    private final String BARRIER = "/barrier";
    private final String BARRIER_NODE = BARRIER + "/node";
    private String zNode = null;

    private int barrierSize;
    private final Integer conditionVariableForEnter = 0;
    private final Integer conditionVariableForLeave = 0;

    public Barrier(CuratorFramework zkClient, int barrierSize) throws Exception {
        this.zkClient = zkClient;
        this.barrierSize = barrierSize;

        try {
            this.zkClient.create().withMode(CreateMode.PERSISTENT).forPath(BARRIER);
        } catch (KeeperException exception) {
            if (exception.code() != KeeperException.Code.NODEEXISTS) {
                throw exception;
            }
        }
    }

    //
    // 进入 Barrier
    //
    public synchronized void enter() throws Exception {
        assert zNode == null;

        zNode = zkClient.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(BARRIER_NODE, BARRIER_NODE.getBytes());
        while (true) {
            List<String> children = zkClient.getChildren().usingWatcher((CuratorWatcher) (event) -> {
                if (event.getType() == EventType.NodeChildrenChanged) {
                    synchronized (conditionVariableForEnter) {
                        conditionVariableForEnter.notify();
                    }
                }
            }).forPath(BARRIER);

            if (children.size() >= barrierSize) {
                return;
            }

            synchronized (conditionVariableForEnter) {
                conditionVariableForEnter.wait();
            }
        }
    }

    //
    // 离开 Barrier
    //
    public synchronized void leave() throws Exception {
        assert zNode != null;

        zkClient.delete().forPath(zNode);
        while (true) {
            List<String> children = zkClient.getChildren().usingWatcher((CuratorWatcher) (event) -> {
                if (event.getType() == EventType.NodeChildrenChanged) {
                    synchronized (conditionVariableForLeave) {
                        conditionVariableForLeave.notify();
                    }
                }
            }).forPath(BARRIER);

            if (children.size() == 0) {
                return;
            }

            synchronized (conditionVariableForLeave) {
                conditionVariableForLeave.wait();
            }
        }
    }
}
```

### ZooKeeper 应用示例 ###

前面介绍了 “ZooKeeper 实践场景”，以分布式锁和分布式 Barrier 作为示例进行了阐述，本节，我们以 Kafka 和 Dubbo 作为示例，阐述 ZooKeeper 于工业界的应用示例。

#### Kafka ####
Kafka 是开源的分布式消息系统（或称为：流式计算平台）。

开始了解 ZooKeeper 于 Kafka 的应用之前，我们简要阐述 Kafka 中各个角色，如图中所示：
![](http://images.gitbook.cn/02c4d720-1641-11e8-9675-c95ffd390962)

1.Producer：消息生产者；
2.Consumer：消息消费者；
3.Broker：消息存储的服务器；
4.Topic：Producer 与 Consumer 之间的订阅关系，Producer 发送消息到 Topic，Consumer 消费 Topic 的消息；
5.Partition：Topic 消息的分区；
6.Offset：Consumer 消费的消息，于 Broker 存储的偏移量；
7.Consumer Group：Consumer 分组，分组内的 Consumer 消费相同的 Topic，单个 Consumer 消费其中的部分消息。
ZooKeeper 于 Kafka，主要进行各个角色的信息注册和分布式协调，下面进行详细介绍。


1. 信息注册。
于 ZooKeeper 创建 ZNode，注册 Topic、Broker、Topic-Broker-Partition、Producer、Consumer、Consumer Group-Consumer、Offset 的信息。

2. Producer 协调。
通过 ZooKeeper Watch，Producer 能够动态获取 ZooKeeper 的 Broker、Topic-Broker-Partition，由此计算 Producer 进行消息生产的 Broker 及 Partition。

3. Consumer 协调。
通过 ZooKeeper Watch，Consumer 能够动态获取 Broker、Topic-Broker-Partition、Consumer Group-Consumer，由此计算 Consumer 进行消息消费的 Broker 及 Partition（基于 Kafka 实现机制，单个 Partition 仅允许唯一的 Consumer 进行消费）。

需要说明的是，Kafka 不使用 ZooKeeper Watch 构建 Producer 与 Consumer 之间的“发布/订阅”。

#### Dubbo #### 
Dubbo 是由阿里巴巴开源的 RPC 及微服务框架，我所在的公司，亦广泛使用。Dubbo 开源版本推荐使用 ZooKeeper 作为“服务注册中心”。
![](http://images.gitbook.cn/1c540030-15d3-11e8-8f0a-3d15dad568c1)

如上图中所展示的 Dubbo 架构，使用 ZooKeeper 作为服务注册中心（Registry）时，以 com.chat.Service 为例：

1. /dubbo/com.chat.Service：服务 ZNode；
2. /dubbo/com.chat.Service/providers：服务提供者（Provider）根 ZNode，子级 ZNode 即代表一个真实的服务提供者；
3. /dubbo/com.chat.Service/consumers：服务消费者（Consumer）根 ZNode，子级 ZNode 即代表一个真实的服务消费者。

图中涉及的流程如下。

1. 服务提供者：启动时，创建 /dubbo/com.chat.Service/providers 的子级 ZNode，写入 IP & port；
2. 服务消费者：启动时，获取并订阅（设置 Watch） /dubbo/com.chat.Service/providers 的全部子级 ZNode，获取服务提供者的 IP & port，即能够进行正常的 RPC 调用（同时创建 /dubbo/com.chat.Service/consumers 的子级 ZNode）。
服务提供者 & 消费者创建的 ZNode，全部为临时 ZNode：当服务提供者异常（机器故障、网络中断）时，其临时 ZNode 自动移除，消费者亦能够自动感知。










