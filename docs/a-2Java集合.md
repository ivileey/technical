#  Java集合

## 一、HashMap

### 1.谈谈我理解的HashMap，讲讲其中get、put的过程

> 两个重要的参数容量(Capacity)和负载因子(Load factor)
>
> 简单的说，Capacity就是buckets的数目，Load factor就是buckets填满程度的最大比例。如果对迭代性能要求很高的话不要把`capacity`设置过大，也不要把`load factor`设置过小。当bucket填充的数目（即hashmap中元素的个数）大于`capacity*load factor`时就需要调整buckets的数目为当前的2倍。

- 是由数组加链表组成
- put过程会根据key的hashcode做hash，如果没碰撞直接放到桶里，如果碰撞了则执行equals()函数判断key是否相等，如果相等进行覆盖，不相等则以链表的形式存放，如果长度大于TREEIFY_THRESHOLD(默认值是8）,就把链表转换成红黑树，桶满了要resize()超过了load factor * current capacity;
- get如果桶中只有一个节点则命中，如果有冲突则通过key.equals去查找要查找的值。

### 2.**你知道hash的实现吗？为什么要这样实现？**
在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：`(h = k.hashCode()) ^ (h >>> 16)`，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

### 3.为什么设置Capctiy和Load factor

当entry的数量达到桶的数量的0.75时，哈希冲突已经比较严重，就会成倍扩容桶数组，并重新分配原来的Entry。

### 4.HashMap和ConcurrentHashMap的区别？

- HashMap是线程不安全的 ，ConcurrentHashMap是线程安全的

### 5.高并发HashMap的环是如何产生的？

线程不安全的HashMap, HashMap在并发执行put操作时会引起死循环，是因为多线程会导致HashMap的Entry链表形成环形数据结构，查找时会陷入死循环。hashmap成环的原因的代码出现在transfer代码中，在扩容之后的数据迁移部分。

## 二、ConcurrentHashMap 

1.7ConcurrentHashMap采用了分段锁的技术。使用segment数组存放数据。每个segment维护了若干个HashEntry的桶构成链表。1.8舍弃了segment，大量使用了synchronized，以及无锁操作CAS以保证ConcurrentHashMap的线程安全性。synchronized做了很多优化，包括偏向锁，轻量级锁，重量级锁。底层数据结构改为采用数组+链表+红黑树的结构。ConcurrentHashMap在进行put操作时根据 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。都不满足的情况按链表或红黑树的方式插入则用synchronized进行put。

### 1.HashTable和ConcurrentHashMap的区别

- HashTable的put和get操作都用到了同步锁是一种悲观锁效率比较低。
- ConcurrentHashMap用的是同步锁加CAS无锁操作，通过加volatile关键字来实现。
- put操作。
  - 如果没有初始化就先调用initTable（）方法来进行初始化过程
  - 如果没有hash冲突就直接CAS插入
  - 如果还在进行扩容操作就先进行扩容如果存在hash冲突，就加锁来保证线程安全，这里有两种情况，一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入，最后一个如果该链表的数量大于阈值8，就要先转换成黑红树的结构，break再一次进入循环如果添加成功就调用addCount（）方法统计size，并且检查是否需要扩容
- get操作
  -  计算hash值，定位到该table索引位置，如果是首节点符合就返回
  -  如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方		法，查找该节点，匹配就返回
  -  以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null