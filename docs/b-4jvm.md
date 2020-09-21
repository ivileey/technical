# Jvm常见问题

## java内存模型

- Java 内存模型，描述的是多线程允许的行为
- JVM 内存结构，描述的是线程运行所设计的内存空间

## jvm内存结构(java运行时数据区）

- 局部变量是线程安全的
- 递归会导致栈帧过多栈内存溢出
- 可以使用jstack查看进程中线程的栈状态

1. 堆（Heap） 
   - 堆是 JVM 内存中最大的一块内存空间，该内存被所有线程共享，几乎所有对象和数组都被分配到了堆内存中。
   - 堆被划分为新生代和老年代，新生代又被进一步划分为 Eden 区和 Survivor 区，最后 Survivor 由 From Survivor 和 To Survivor 组成。
2. 方法区（method Area）
   - 方法区主要是用来存放已被虚拟机加载的类相关信息，包括类信息、常量池(字符串常量池以及所有基本类型都有其相应的常量池)、运行时常量池。这其中，类信息又包括了类的版本、字段、方法、接口和父类等信息。
   - 涉及到的Error
     OutOfMemoryError出现在方法区无法满足内存分配需求的时候，比如一直往常量池中加入数据，运行时常量池就会溢出，从而报错。
3. 程序计数器（Program Counter Register）
   - 程序计数器是一块很小的内存空间，主要用来记录各个线程执行的字节码的地址，例如，分支、循环、跳转、异常、线程恢复等都依赖于计数器。
4. 虚拟机栈（VM Stack）
   - 虚拟机栈是线程私有的内存空间，它和 Java 线程一起创建。
     当创建一个线程时，会在虚拟机栈中申请一个线程栈，用来保存方法的局部变量、操作数栈、动态链接方法和返回地址等信息，并参与方法的调用和返回。
     每一个方法的调用都伴随着栈帧的入栈操作，方法的返回则是栈帧的出栈操作。
   - 涉及到的Error
     StackOverflowError出现在栈内存设置成固定值的时候，当程序执行需要的栈内存超过设定的固定值时会抛出这个错误。
     OutOfMemoryError出现在栈内存设置成动态增长的时候，当JVM尝试申请的内存大小超过了其可用内存时会抛出这个错误。

5. 本地方法栈（Native Method Stack）
   - 本地方法栈跟虚拟机栈的功能类似，虚拟机栈用于管理 Java 方法的调用，而本地方法栈则用于管理本地方法的调用。

## 垃圾回收机制

- 对于程序计数器，虚拟机栈，本地方法栈来说，由于是跟随当前线程的生命周期，当线程被销毁时其占用的内存自然回收
- 而java堆和方法区则不一定。

### 判断对象是否存活

1. 引用计数算法（主流的java虚拟机不用这个方法）会出现循环引用的问题
2. 可达性分析算法 和GC Roots直接或间接关联的对象是有效对象，反之则是无效对象。
   可作为GCRoots的对象
   - 虚拟机栈中引用的对象
   - 方法区中类静态属性引用的对象
   - 方法区中常量引用的对象
   - 本地方法栈中JNI（Native方法引用的对象）
   - ·Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象(比如
     NullPointExcepiton、OutOfMemoryError)等，还有系统类加载器。
   - 所有被同步锁(synchronized关键字)持有的对象。
   - 反映Java虚拟机内部情况的JM XBean、JVM TI中注册的回调、本地代码缓存等。

### 什么时候回收

当 JVM 经过可达性分析法筛选出实效对象时，并不是马上清除，而是进行标记并判断是否回收：

1. 判断对象是否覆盖了finalize()方法
2. 执行F-Queue队列中的finalize()方法
   - 由虚拟机自动建立一个优先级较低的线程去执行 F-Queue 中的 finalize() 方法，这里的执行只是触发这些方法并不保证会等待它执行完毕。如果 finalize() 方法作了耗时操作，虚拟机会停止执行并将该对象清除。
3. 对象销毁或重生
   - 在 finalize() 方法中，将 this 赋值给某一个引用，那么该对象就重生了。如果没有引用，该对象会被回收 

### 方法区的内存回收

- 废弃的常量：
  当前系统中没有任何对象引用常量池中的该常量，则是废弃常量
- 废弃的类判断规则：
  该类所有实例都被回收；
  加载该类的 ClassLoader 已经被回收；
  该类对应的 Class 对象没有引用，也无法通过反射访问该类的方法。

### 回收算法

- 标记 - 清除算法
- 标记-复制算法 回收新生代，HotSpot虚拟机默认Eden和Survivor的大小比例是8∶1，也即每次新生代中可用内存空间为整个新 生代容量的90%(Eden的80%加上一个Survivor的10%)，只有一个Survivor空间，即10%的新生代是会 被“浪费”的。
- 标记整理算法
- 分代收集算法 ：“Minor GC”“Major GC”“Full GC”这样的回收类型的划

### 引用

- 强引用：类似 "Object obj = new Object()" 属于强引用，只有引用还在，垃圾收集器永远不会回收掉被引用对象。
- 软引用：用来描述一些还有用但不是必须的对象。对于软引用相关的对象，在系统将要发生 OOM（内存溢出）时，将会把软引用对象列进回收范围并进行二次回收。如果这次回收后还是没有足够内存才会抛出 OOM 异常。JDK 1.2 后提供 SoftReference 类来实现。
- 弱引用：也是用来描述非必须对象，但它的强度比软引用更弱，被弱引用的对象只会生存到下一次垃圾回收之前。当进行 GC 时，无论当前内存是否足够，都会回收掉弱引用的对象。弱引用对应的类为 WeakReference。
- 虚引用：又称幽灵引用或幻影引用，最弱的引用关系。无法通过虚引用获取对象的实例，为对象设置虚引用唯一的目的就是能在该对象被垃圾收集器回收时收到一个系统通知。对应 PhantomReference 类。

### 垃圾收集

#### 标记清除算法

- 容易造成内存碎片，快

#### 标记整理算法

- 慢 ，有碎片

#### 复制算法

- 没碎片，占用内存空间大

#### 分代垃圾回收机制

- 新生代
- 老年代
  -> Minor GC-> Full GC


## GC算法

### GC中Stop the world（STW）

当GC线程启动时（即进行垃圾收集），应用程序都要暂停（Stop The World）

### Minor GC和Full GC触发条件

Minor GC触发条件：当Eden区满时，触发Minor GC。

Full GC触发条件：

（1）调用System.gc时，系统建议执行Full GC，但是不必然执行

（2）老年代空间不足

（3）方法区空间不足

（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存

（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

### 分代收集算法

#### 新生代，老年代，和永久代都指的什么？

 1、夭折对象(新生代)：朝生夕灭的对象，通俗点讲就是活不了多久就得死的对象。
例子：某一个方法的局域变量、循环内的临时变量等等。
2、老不死对象（老年代）：这类对象一般活的比较久，岁数很大还不死，但归根结底，老不死对象也几乎早晚要死的，但也只是几乎而已。
例子：缓存对象、数据库连接对象、单例对象（单例模式）等等。
3、不灭对象（永久代）：此类对象一般一旦出生就几乎不死了，它们几乎会一直永生不灭，记得，只是几乎不灭而已。
例子：String池中的对象（享元模式）、加载过的类信息等等

#### 分代收集算法

1. 新生代分为Eden区和Survivor From和To区，用复制算法，老年代用标记-整理、标记-清除，
2. 复制算法：开辟两个空间，一块用，一块不用，清除的时候把一块用的，存活的放入另外一个空的里面
3. 标记-清除：标记，然后清除，缺点：碎片化
4. 标记-整理：解决碎片化问题，把内存整合连续

#### 什么参数能够调整新生代的比例？

        * -XX:SurvivorRatio：Eden和Survivor的比值，默认8:1
        * -XX:NewRatio：年轻代和老年代内存大小的比例

#### 一个程序频繁的发生Full GC，有什么办法改善这个情况？

   - 用一个指令去参考Full GC的次数，新生代老年代的比例，调整各比例
   - 产生Full GC的原因可能是：新生代到老年代的对象，老年代的空间不足，才产生Full GC

### 标记-清除和标记整理区别？

  是当我们对对象进行完标记之后，触发垃圾回收的动作时，回收完垃圾是否将剩余的对象整理到连续的内存区域中，如果用的是标记清除会导致内存碎片比较多，但是不需要整理的动作，效率比较高。

### 程序发生内存泄漏，你会怎么去查这个问题？CPU运行到达100%怎么找到问题？

产生内存泄漏的原因，可能是哪个引用没有用了，但是没有被回收

### 发现频繁Full GC怎么去排查和调整

## 垃圾回收器

### CMS垃圾收集器

### 各垃圾回收器的特点及区别 

- Serial收集器：但线程收集器，运行在客户端虚拟机，对于新生代的垃圾收集
- ParNew收集器：Serial收集器的多线程版本，用于服务端垃圾收集，可与CMS收集器配合使用
- Parallel Scavenge收集器：新生代垃圾收集器，Parallel Scavenge收集器的目标则是达到一个可控制的吞吐 量(Throughput)
- Serial Old收集器：Serial Old是Serial收集器的老年代版本，它同样是一个单线程收集器，使用标记-整理算法，供客户端模式下的HotSpot虚拟机使用
- Parallel Old收集器：是Parallel Scavenge收集器的老年代版本，支持多线程并发收集，基于标记-整理算法实现
- CMS(Concurrent Mark Sweep)收集器：是一种以获取最短回收停顿时间为目标的收集器
  1)初始标记(CMS initial mark)：需要stop the world，时间很短
   2)并发标记(CM S concurrent mark) 
   3)重新标记(CM S remark) stop the world，时间短
   4)并发清除(CM S concurrent sweep)：使用标记-清除算法
- G1是一款主要面向服务端应用的垃圾收集器
  - G1不再坚持固定大小以及固定数量的 分代区域划分，而是把连续的Java堆划分为多个大小相等的独立区域(Region)，每一个Region都可以根据需要，扮演新生代的Eden空间、Survivor空间，或者老年代空间。收集器能够对扮演不同角色的 Region采用不同的策略去处理，这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果。
  - 初始标记(Initial M arking):仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAM S 指针的值，让下一阶段用户线程并发运行时，能正确地在可用的Region中分配新对象。这个阶段需要 停顿线程，但耗时很短，而且是借用进行Minor GC的时候同步完成的，所以G1收集器在这个阶段实际 并没有额外的停顿。
  - 并发标记(Concurrent Marking):从GC Root开始对堆中对象进行可达性分析，递归扫描整个堆 里的对象图，找出要回收的对象，这阶段耗时较长，但可与用户程序并发执行。当对象图扫描完成以 后，还要重新处理SAT B记录下的在并发时有引用变动的对象。
  - 最终标记(Final M arking):对用户线程做另一个短暂的暂停，用于处理并发阶段结束后仍遗留 下来的最后那少量的SAT B记录。
  - 筛选回收(Live Data Counting and Evacuation):负责更新Region的统计数据，对各个Region的回 收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，可以自由选择任意多个Region 构成回收集，然后把决定回收的那一部分Region的存活对象复制到空的Region中，再清理掉整个旧 Region的全部空间。这里的操作涉及存活对象的移动，是必须暂停用户线程，由多条收集器线程并行 完成的。

### CMS的优缺点

优点：GC过程短暂停，适合对时延要求较高的服务，用户线程不允许长时间的停顿。
缺点：服务长时间运行，造成严重的内存碎片化。 另外，算法实现比较复杂（如果也算缺点的话），在并发清理过程中用户线程可能会产生大对象发生full gc，导致csm回收失败

### CMS回收几个阶段是只有自己的线程吗？还是多个线程并行

正确应该是
    1. 初始标记 自己线程
    2. 并发标记：并发标记线程+用户线程
    3. 重新标记：重新标记线程
    4. 并发清理：并发清理线程+用户线程
    5. 并发重置：重置线程+用户线程

### 众所周期，G1跟其他的垃圾回收算法差别很大，你了解G1的垃圾回收架构吗？为什么G1可以做到回收时间用户可以设定?

- 可以指定最大停顿时间、分Region的内存布局、按收益动 态确定回收集这些创新性设计带来的红利
- G1的另一个显著特点他能够让用户设置应用的暂停时间，为什么G1能做到这一点呢？也许你已经注意到了，G1回收的第4步，它是“选择一些内存块”，而不是整代内存来回收，这是G1跟其它GC非常不同的一点，其它GC每次回收都会回收整个Generation的内存(Eden, Old), 而回收内存所需的时间就取决于内存的大小，以及实际垃圾的多少，所以垃圾回收时间是不可控的；而G1每次并不会回收整代内存，到底回收多少内存就看用户配置的暂停时间，配置的时间短就少回收点，配置的时间长就多回收点，伸缩自如

# JAVA类加载机制（加载验证准备解析初始化，双亲委派模型）

双亲委派模型

8. JDBC和双亲委派模型关系 

# jvm常用命令

### cpu 占用过多

- top定位进程
- ps命令查看哪个线程占用过多
- jstack进程id查看线程具体信息（线程id要转换为16进制）找到源码位置

### 程序很久没有结束

- 用jstack查看进程中线程运行状态 

### 堆内存诊断

- jps 查看当前系统中有哪些java进程
- jmap 查看堆内存占用情况
- jconsole

### 方法区

- javap -v 反编译class文件
- 字符串intern方法，将这个字符串对象放入字符串常量池

### 字符串常量池

- 当字符串常量比较多时将StringTableSize调的更大一些，使它有更好的HashSet分布

### 直接内存 direct memory

- 常用于 nio操作
- 属于操作系统的内存，读写操作效率非常高

```
	    Unsafe unsafe = getUnsafe();
        // 分配内存
        long base = unsafe.allocateMemory(_1Gb);
        unsafe.setMemory(base, _1Gb, (byte) 0);
        System.in.read();

        // 释放内存
        unsafe.freeMemory(base);
        System.in.read();
```

- System.GC() 是FullGC 比较占用时间，会回收新生代和老年代

### Java 垃圾判断

- eclipse MAT 工具 可视化的Java堆分析工具

#### 可达性分析算法四种引用

- 软引用 当没有强引用内存不足时回收

```
public static void soft() {
        // list --> SoftReference --> byte[]

        List<SoftReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB]);
            System.out.println(ref.get());
            list.add(ref);
            System.out.println(list.size());

        }
        System.out.println("循环结束：" + list.size());
        for (SoftReference<byte[]> ref : list) {
            System.out.println(ref.get());
        }
    }
```

- 弱引用 不管内存足不足都回收

```
		 //  list --> WeakReference --> byte[]
        List<WeakReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            WeakReference<byte[]> ref = new WeakReference<>(new byte[_4MB]);
            list.add(ref);
            for (WeakReference<byte[]> w : list) {
                System.out.print(w.get()+" ");
            }
            System.out.println();

        }
        System.out.println("循环结束：" + list.size());
```

- 虚引用Cleaner直接内存地址  必须配合引用队列 
- 终结器引用 必须配合引用队列 重写finalize()方法



#### 相关 VM参数

- [Java VM参数表](https://blog.csdn.net/rj042/article/details/6870897)

### 垃圾回收器

#### 串行

 - 单线程 
 - 堆内存较小
 - -XX:+UseSerialGC = Serial +SerialOld

 #### 吞吐量优先

 - 多线程
 - 堆内存较大，多核cpu
 - -XX:+UseParallelGC~-XX:+UseParallerlOldGC
 - -XX:+UseAdaptiveSizePolicy
 - -XX:GCTimeRatio=ratio
 - -XX:MaxGCPauseMills=ms
 - -XX:ParallerGCThreads=n

 #### 响应时间优先

 - 多线程
 - 堆内存较大，多核cpu
 - XX:+UseConcMarkSweepGC ~ XX:+UseParNewGC ~ SerialOld
 - XX:ParallelGCThreads=n ~ XX:ConcGCThreads=threads
 - XX:CMSInitiatingOccupancyFraction=percent
 - XX:+CMSScavengeBeforeRemark

#### G1

- -XX:UseG1GC
- -XX:G1HeapRegionSize=size
- XX:MaxGCPauseMillis=time
- 使用的标记整理和复制算法


- Young Collection
- Young Collection + CM
- Mixed Collection
- Full GC
  - SerialGC 
    - 新生代内存不足发生的垃圾收集 - minor gc 
    - 老年代内存不足发生的垃圾收集 - fullgc
  - ParallelGC
    - 新生代内存不足发生的垃圾收集 - minor gc 
    - 老年代内存不足发生的垃圾收集 - fullgc
  - CMS
    - 新生代内存不足发生的垃圾收集 - minor gc 
    - 老年代内存不足
  - G1
    - 新生代内存不足发生的垃圾收集 - minor gc 
    - 老年代内存不足

# 参考链接

1. 深入理解java虚拟机第三版 周志明
2. [jvm内存模型](https://juejin.im/post/6844903975850868743)
3. [java内存模型](https://juejin.im/post/6844903902018535438)
4. [频繁full gc解决办法](https://blog.csdn.net/n8765/article/details/50911742)
5. [G1垃圾收集器](https://www.cnblogs.com/aspirant/p/8663872.html)
6. [stop the world](