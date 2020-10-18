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



## 三、ArrayList

存储基本类型的时候要存储包装类，底层是数组，当删除元素的时候要从后往前删，否则会出现数组索引发生变化，跳跃删除。

ArrayList在内存中是一片连续的内存空间，LinkedList在内存是分散存储的。

### 1.ArrayList是线程安全的吗？

不是线程安全的，因为我们正常使用的场景中，都是用来查询，不会涉及太频繁的增删，如果涉及频繁的增删，可以使用LinkedList，如果你需要线程安全就使用Vector，这就是三者的区别了，实际开发过程中还是ArrayList使用最多的。Vector的实现很简单，就是把所有的方法统统加上synchronized就完事了。

之所以ArrayList线程不安全，会造成数组下标越界，因为在多线程场景下，比如一个多核cpu，拥有L1，L2，L3三级缓存，一个线程将ArrayList的size取到缓存中运算，另一个线程拿到的size还是之前的size，但是数组已经满了还向数组添加元素，此时就会造成数组下标越界。

### 2.ArrayList扩容为什么是1.5倍？

ArrayList底层调用grow方法，oldCapacity+（oldCapacity >> 1))的值约为oldCapacity的1.5倍。一次性扩容太大的话可能会浪费更多的内存。但是扩容太小需要多次对数组重新分配内存，对性能消耗比较严重。1.5倍是个经验值。

### 3.ArrayList增删一定比LinkedList慢吗？

取决于你删除的元素离数组末端有多远，ArrayList拿来作为堆栈来用还是挺合适的，push和pop操作完全不涉及数据移动操作。

### 4.增加和删除操作是如何进行的？如果删除了之后还会缩小数组的容量吗？

ArrayList增加操作先判断有没有达到数组容量，如果达到数组的容量的话就进行扩容，扩容为原来的1.5倍，然后将数组的地址向后移一位，这里使用的是System.arraycopy()。删除操作，也是使用System.arraycopy()。

删除之后不会缩小数组的容量，可以手动使用list.trimToSize();  *// 将容量设为当前元素的个数*。

**为什么删除了元素不缩小容量呢？**

我觉得是因为每次改变容量的时候，都需要新建一个数组，然后把旧数组里面的元素复制到新数组里面去，这种操作是损耗性能的，而往往ArrayList里面的元素会经常会在一定范围内变化，因此这种操作将带来很大的性能损耗，我们为什么不能损耗一点内存来提高服务的响应速度呢？从这个角度来看，也是一种空间换时间的体现。另外，在新建ArrayList的时候，设置一个合理的容量也会对我们系统的服务性能有一定的帮助。

### 5.ArrayList遍历和LinkedList遍历哪个更快？

ArrayList更快，因为ArrayList存储数据是一片连续的空间，而LinkedList是连续的内存片段，因此ArrayList更快。

### 6.ArrayList的默认数组大小为什么是10？

10这个长度的数组是最常用最有效率的。8或者12都可以。是一个经验值。

### 7.ArrayList用来做队列合适吗？

不合适。队列一般是FIFO的，如果用ArrayList做队列，就需要在数组尾部追加数据，数组头部删除数组，反过来也可以。但是无论如何总会有一个操作会涉及到数组的数据搬迁，这个是比较耗费性能的。

但是数组比较适合做队列，可以使用一个定长数组。记录数组的读位置和写位置，如果超过长度就折回到数组开头。





