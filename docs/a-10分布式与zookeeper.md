# 分布式与Zookpeer

## 分布式

## Zookeper

### CAP定理？

解决分布式系统中各个节点状态如何同步。指的是在一个分布式系统中，[一致性](https://baike.baidu.com/item/一致性/9840083)（Consistency）、[可用性](https://baike.baidu.com/item/可用性/109628)（Availability）、分区容错性（Partition tolerance）。CAP 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。一致性和可用性不能同时成立

### ZAB协议

Zookeeper基于ZAB（Zookeeper Atomic Broadcast），实现了主备模式下的系统架构，保持集群中各个副本之间的数据一致性。

ZAB协议定义了选举（election）、发现（discovery）、同步（sync）、广播(Broadcast)四个阶段。

### leader选举算法和流程

当Leader崩溃或者Leader失去大多数的Follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的Leader，让所有的Server都恢复到一个正确的状态。Zookeeper中Leader的选举采用了三种算法：

- LeaderElection
- FastLeaderElection
- AuthFastLeaderElection
- 选举算法流程如下：
  1. 选举线程首先向所有Server发起一次询问(包括自己)；
  2. 选举线程收到回复后，验证是否是自己发起的询问(验证xid是否一致)，然后获取对方的id(myid)，并存储到当前询问对象列表中，最后获取对方提议的leader相关信息(id,zxid)，并将这些信息存储到当次选举的投票记录表中；
  3. 收到所有Server回复以后，就计算出zxid最大的那个Server，并将这个Server相关信息设置成下一次要投票的Server；
  4. 线程将当前zxid最大的Server设置为当前Server要推荐的Leader，如果此时获胜的Server获得多数Server票数， 设置当前推荐的leader为获胜的Server，将根据获胜的Server相关信息设置自己的状态，否则，继续这个过程，直到leader被选举出来。

## 参考链接

1.[CAP定理](https://www.ruanyifeng.com/blog/2018/07/cap.html)

2.[Zookeeper选举算法](https://juejin.im/post/6844903672824987661)