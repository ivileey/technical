# Redis

## 1.Redis的应用场景

- String类型一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。
- List类型发布与订阅或者说消息队列、慢查询
- hash系统中对象数据的存储
- set需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景
- zset 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

## 2.Redis支持的数据类型

String，List，hash，set，zset

## 3.Zset的数据结构

Zset底层维护的是一种跳跃表，由多个有序链表组成插入速度非常快，从上层指针开始找，找到对应区间之后去下层开始找。不需要旋转，支持无锁操作。

## 4.Redis的数据过期策略

 volatile-lru： 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰
 volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰
 volatile-random：从已设置过期时间的数据集中任意选择数据淘汰
 allkeys-lru：从所有数据集中挑选最近最少使用的数据进行淘汰
 allkeys-random：从所有数据集中任意选择数据进行淘汰
 noeviction：禁止驱逐数据，新写入数据操作会报错

 4.0 版本后增加以下两种：

 volatile-lfu（least frequently used）：从已设置过期时间的数据集中挑选最不经常使用的数据淘汰
 allkeys-lfu（least frequently used）：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key

## 5.Redis的持久化机制

RDB持久化，将某个时间点的所有数据都存放到硬盘上。

AOF持久化，则是将Redis执行的每次写命令记录到单独的日志文件中，当重启Redis会重新将持久化的日志中文件恢复数据

## 6.Redis的LRU过期策略的具体实现

正常的lru：实现一个双向链表，每次有一个key被访问之后就将key放到链表的头部，当内存不够用时，从链表尾部淘汰key。

Redis中的LRU：

## 7.如何解决Redis缓存雪崩，缓存穿透问题

### 缓存雪崩

**针对 Redis 服务不可用的情况：**

1. 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用。
2. 限流，避免同时处理大量的请求。

**针对热点缓存失效的情况：**

1. 设置不同的失效时间比如随机设置缓存的失效时间。
2. 缓存永不失效。

### 缓存穿透

布隆过滤器，**布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在。**

## 8.Redis中的管道pipeline

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。
