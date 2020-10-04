# JAVA集合
建议本文结合java源码来阅读，看了之后就什么都懂了，还有参考文献。

- 散列(Hash) 是一种按关键字编址的存储和检索方法
- 散列表(HashTable)根据元素的关键字确定元素的位置
- 散列函数(Hash Function)建立数据元素的关键字到该元素的存储位置的一种映射关系
（具体如何计算百度一下很简单，Hash算法的难处在如何确定散列函数和解决冲突)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731114555575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pc2h1aWFlZQ==,size_16,color_FFFFFF,t_70)

集合框架中的接口。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731114430855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pc2h1aWFlZQ==,size_16,color_FFFFFF,t_70)
哪些集合可以存放null？
除了TreeSet和TreeMap还有HashTable不能存放null，会报空指针异常。


- List的主要特点就是有序性和元素的可空性。 ArrayList(数组实现)和LinkedList(双向链表实现)
- Set主要特性是唯一性。自动排序。HashSet,LinkedHashSet,TreeSet
- Queue 特性是队列
- Map健值对 继承自Object
[List,Set,Queue,Map参考链接](https://www.jianshu.com/p/0fedfd888857)

---
#### Set (无序，唯一)
##### HashSet
- 底层数据结构是哈希表
- 哈希表依赖两个方法 hashCode()和equals()
- 执行顺序: 首先判断hashCode()值是否相同
	是：继续执行equals()，看其返回值
	&emsp; true：说明元素重复不添加
	&emsp; false：直接添加到集合
	否： 直接添加到集合
###### LinkedHashSet
- 底层数据结构由链表和哈希表组成
- 由链表保证元素有序
- 由哈希表保证元素唯一
###### TreeSet
- 底层数据结构是红黑树
- 唯一性是根据比较返回值是否是0来决定
- 排序两种方式
	&emsp; 自然排序：让元素所属的类实现Comparable接口
	&emsp; 比较器排序：让集合接受一个Comparator的实现类对象
##### Map
- Map集合的数据结构仅仅对键有效与值无关
- 存储的是健值对的形式的元素，键唯一，值可重复
###### HashMap
- 底层数据结构是哈希表，线程不安全，效率高

###### LinkedHashMap
- 底层数据结构由链表和哈希表组成

###### HashTable
- 底层数据结构是哈希表，线程安全，效率低

###### TreeMap
- 底层数据结构是红黑树

##### 集合选取原则
是否是健值对象形式：
&emsp; 是：Map
&emsp;&emsp; 键是否需要排序：
&emsp;&emsp;&emsp;是：TreeMap
&emsp;&emsp;&emsp;否：HashMap
&emsp;&emsp;不知道就用HashMap
&emsp;否：Collection
&emsp;&emsp;元素是否唯一
&emsp;&emsp;&emsp;是：Set
&emsp;&emsp;&emsp;&emsp;元素是否需要排序
&emsp;&emsp;&emsp;&emsp;&emsp;是：TreeSet
&emsp;&emsp;&emsp;&emsp;&emsp;否：HashSet
&emsp;&emsp;&emsp;&emsp;不知道就用HashSet
&emsp;&emsp;&emsp;否：List
&emsp;&emsp;&emsp;&emsp;要安全吗：
&emsp;&emsp;&emsp;&emsp;&emsp;是：Vector
&emsp;&emsp;&emsp;&emsp;&emsp;否：ArrayList或者LinkedList
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;增删多：LinedList
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;查询多：ArrayList
&emsp;&emsp;&emsp;&emsp;&emsp;不知道就是用ArrayList

[参考链接](https://www.cnblogs.com/wangyu19900123/p/11799627.html)

## 常见的散列函数计算方法
- 除留余数法
```
int hash(int key)
{
	return key%prime;
}
```
如果key的类型是T 要返回一个 int hashCode() 返回散列码，约定对象到int的一对一映射，每个对象的散列码必须不同
```
template<T>
int HashSet<T>::set(T key)
{
	return key.hashCode() %prime;
}
```
除留余数法的关键在于如何选取prime的值。通常选取小于散列表长度的最大素数。
-  平方取中法 例如，k=4731，k2=22382361，若表长为100，取中间两位。hash(k)=82
- 全域散列法：在所有的散列函数中随机的选取一个散列函数，以防止n个关键字全部散列到同一个槽中

## 处理冲突
- 链地址法。将同义词冲突的元素存储在一条同义词单链表中。

## 下面来谈一下java中的hash集合
### HashMap
#### 取hashMap的方法

```
System.out.println("里面的键值对是：");  
Set<Map.Entry<String, String>> set = map.entrySet();  
Iterator<Map.Entry<String, String>> iterator2 = set.iterator();  
while (iterator2.hasNext()) {  
    Map.Entry<String, String> entry = iterator2.next();  
    System.out.println("键是：" + entry.getKey() + "值是：" + entry.getValue());  
}  
```

#### 谈谈你理解的 HashMap，讲讲其中的 get put 过程。
 - 是由数组加链表组成，
 - put过程会根据key的hashcode做hash，如果没碰撞直接放到桶里，如果碰撞了，则执行equals()函数判断key是否相等，如果相等进行覆盖，不相等，以链表的形式存放，如果长度于TREEIFY_THRESHOLD，就把链表转换成红黑树，桶满了要resize() 超过了load factor * current capacity
 - get 如果桶中只有一个节点则命中，如果有冲突则通过key.equals去查找要查找的值，有红黑树和链表两种。
#### 1.8 做了什么优化？
- 在1.8之前是用链表解决冲突的，1.8采用的红黑树解决冲突时间复杂度编程logn
####  你知道hash的实现吗？为什么要这样实现？
- 在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。
#### 是线程安全的嘛？
- 不是线程安全的，线程安全的有HashTable和ConcurrentHashMap.
#### 不安全会导致哪些问题？
- 原子性，有序性，可见性
#### ConcurrentHashMap 是如何实现的？ 1.7、1.8 实现有何不同？为什么这么做？
原理上来说：ConcurrentHashMap 采用了分段锁技术。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel 的线程并发。jdk 1.7使用segment存放数据，segment继承了ReentrantLock充当锁的角色，为每一个segment提供了线程安全的保障，segment维护了哈希的若干个桶，每个桶由HashEntry构成链表。1.8舍弃了segment，大量使用了synchronized，以及CAS无锁操作以保证ConcurrentHashMap操作的线程安全性。至于为什么不用 ReentrantLock 而是 Synchronzied 呢？实际上，synchronzied 做了很多的优化，包括偏向锁，轻量级锁，重量级锁，可以依次向上升级锁状态，但不能降级，因此，使用 synchronized 相较于 ReentrantLock 的性能会持平甚至在某些情况更优。另外，底层数据结构改变为采用数组+链表+红黑树的数据形式。

#### HashTable和HashMap的区别
- HashMap不是线程安全的，但是效率较高，HashTable是线程安全的但是效率较低，因为put和get的操作都加上了同步锁，解决这个问题就是用ConcurrentHashMap

#### HashTable和ConcurrentHashMap的区别
- HashTable的put和get操作都用到了同步锁是一种悲观锁效率比较低。
- ConcurrentHashMap用的是同步锁加CAS无锁操作，通过加volatile关键字来实现。
- put操作。
	- 如果没有初始化就先调用initTable（）方法来进行初始化过程
	- 如果没有hash冲突就直接CAS插入
	- 如果还在进行扩容操作就先进行扩容如果存在hash冲突，就加锁来保证线程安全，这里有两种情况，一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入，最后一个如果该链表的数量大于阈值8，就要先转换成黑红树的结构，break再一次进入循环如果添加成功就调用addCount（）方法统计size，并且检查是否需要扩容
- get操作
	-  计算hash值，定位到该table索引位置，如果是首节点符合就返回
	- 如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方		法，查找该节点，匹配就返回
	- 以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null

### HashSet
- HashSet是基于HashMap来实现的，操作很简单，更像是对HashMap做了一次“封装”，而且只使用了HashMap的key来实现各种特性。

### LinkedHashMap和LinkedHashSet
- LinkedHashSet就是继承了HashSet维护一个双链表和LinkedHashMap一样。、
-  LinkedHashMap 是 HashMap 的一个子类，它保留插入的顺序，如果需要输出的顺序和输入时的相同，那么就选用 LinkedHashMap。

## 参考文献
1.java源码
2.《数据结构(C++版)》 叶核亚编著
[3.ConcurrentHashMap](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)
[4.Java并发编程:volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)
[5.java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)
[6.java中Cloneable和Serializable接口详情](https://blog.csdn.net/xiaomingdetianxia/article/details/74453033)
[7.Java transient关键字使用小记](https://www.cnblogs.com/lanxuezaipiao/p/3369962.html)
[8.无锁算法——CAS原理](https://blog.csdn.net/Roy_70/article/details/69799845)
[9.HashTable，ConcurrentHashMap](https://juejin.im/post/6844903597373669389)
[10.LinkedHashMap](https://wiki.jikexueyuan.com/project/java-collection/linkedhashmap.html)

