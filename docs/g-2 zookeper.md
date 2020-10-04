# ZooKeeper

> 分布式，开源的程序协调服务。
>
> 提供的功能：配置管理、名字服务、分布式锁、集群管理

## Zookeeper的作用

### 配置管理

- 使用zab一致性协议。
- HBase中，客户端就是连接一个Zookeeper，获得必要的HBase集群的配置信息。
- Kafka使用zookeeper来维护broker的信息。
- SOA框架Dubbo中也广泛使用Zookeeper管理一些配置。

### 名字服务

- 提供统一的入口，保存需要访问的点的路径

### 分布式锁

- 管理集群，当一个服务器出现问题时锁释放（Leader Election （leader选举））
- HBase的Master就是采用这种机制

### 集群管理

- 有节点进进出出，集群中的机器需要感知到变化，根据变化做出对应的决策。
- SOA框架的Dubbo采用了Zookeeper作为服务发现的底层机制。还有开源的Kafka队列采用了Zookeeper作为Consumer的上下管理。

## Zookeeper的存储结构

## Znode

是一个跟Unix文件系统路径相似的节点，这个节点可以存储或获取数据。每一个znode默认能够存储1MB的数据

## Znode节点类型

- PERSISTENT 持久化节点：一直存在，直到删除操作到来。不会因为创建该节点的客户端会话失效而消失。
- PERSISTENT_SEQUENTIAL持久化顺序节点：每个夫节点为他的第一级子节点维护一份时许，会记录每一个子节点创建的先后顺序。
- EPHEMERAL临时节点：临时节点的生命周期和客户端会话绑定。
- EPHEMERAL_SEQUENTIAL临时自动编号节点：此节点是属于临时节点，带有顺序。

## Zookeeper目录结构

> Zookeeper 在启动时默认的去 conf 目录下查找一个名称为 zoo.cfg 的配置文件。 在 zookeeper 应用目录中有子目录 conf。其中有配置文件模板：zoo_sample.cfg cp zoo_sample.cfg zoo.cfg。zookeeper 应用中的配置文件为 conf/zoo.cfg。 修改配置文件 zoo.cfg - 设置数据缓存路径

### 单机版

1. bin：放置脚本和工具脚本。
2. conf：默认读取配置的目录，里面会有默认的配置文件
3. contrib：zookeeper的拓展功能
4. dist-maven：zookeeper的maven打包目录
5. docs：zookeeper的相关文档
6. lib：zookeeper的核心jar
7. recipes：zookeeper分布式相关的jar包
8. src：zookooper源码

### 集群版：

### Zookeeper集群中的角色

![](https://picb.zhimg.com/v2-9170718aec3c1143376f9bd69b0d3107_r.jpg)

设计目的：

1. 最终一致性：client不论连接到哪个Server，展示给它的都是同一个视图。
2. 可靠性：具有简单、健壮、良好的性能。
3. 实时性：Zookeeper 保证客户端将在一个时间间隔范围内获得服务器的更新信息，或 者服务器失效的信息。但由于网络延时等原因，Zookeeper 不能保证两个客户端能同时得到 刚更新的数据，如果需要最新数据，应该在读数据之前调用 sync()接口。
4. 等待无关：慢的或失效的client不得干预快的client的请求。
5. 原子性：更新只能成功或者失败
6. 顺序性：包括全局有序和偏序两种

### Zookeeper 怎么保证主从节点的状态同步？

Zookeeper 的核心是原子广播机制，这个机制保证了各个 server 之间的同步。实现这个机制的协议叫做 Zab 协议。Zab 协议有两种模式，它们分别是恢复模式和广播模式。

1. 恢复模式

   当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数 server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 leader 和 server 具有相同的系统状态。

2. 广播模式

   一旦 leader 已经和多数的 follower 进行了状态同步后，它就可以开始广播消息了，即进入广播状态。这时候当一个 server 加入 ZooKeeper 服务中，它会在恢复模式下启动，发现 leader，并和 leader 进行状态同步。待到同步结束，它也参与消息广播。ZooKeeper 服务一直维持在 Broadcast 状态，直到 leader 崩溃了或者 leader 失去了大部分的 followers 支持。