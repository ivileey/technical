#  Java集合

## 1.HashMap和ConcurrentHashMap的区别？

- HashMap是线程不安全的 ，ConcurrentHashMap是线程安全的

## 2. ConcurrentHashMap的数据结构？

1.7ConcurrentHashMap采用了分段锁的技术。使用segment数组存放数据。每个segment维护了若干个HashEntry的桶构成链表。1.8舍弃了segment，大量使用了synchronized，以及无锁操作CAS以保证ConcurrentHashMap的线程安全性。synchronized做了很多优化，包括偏向锁，轻量级锁，重量级锁。底层数据结构改为采用数组+链表+红黑树的结构。ConcurrentHashMap在进行put操作时根据 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。都不满足的情况按链表或红黑树的方式插入则用synchronized进行put。

## 3.高并发HashMap的环是如何产生的？

线程不安全的HashMap, HashMap在并发执行put操作时会引起死循环，是因为多线程会导致HashMap的Entry链表形成环形数据结构，查找时会陷入死循环。hashmap成环的原因的代码出现在transfer代码中，在扩容之后的数据迁移部分。



