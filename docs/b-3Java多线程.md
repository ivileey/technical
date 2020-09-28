# Java多线程
## 基本概念
### 并发编程的三个重要特征(可见性和原子性)

1. **原子性** : 一个的操作或者多次操作，要么所有的操作全部都得到执行并且不会收到任何因素的干扰而中断，要么所有的操作都执行，要么都不执行。`synchronized` 可以保证代码片段的原子性。
2. **可见性** ：当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。`volatile` 关键字可以保证共享变量的可见性。
3. **有序性** ：代码在执行的过程中的先后顺序，Java 在编译器以及运行期间的优化，代码的执行顺序未必就是编写代码时候的顺序。`volatile` 关键字可以禁止指令进行重排序优化。

### ReetrantLock和Synchronized关键字的实现原理

1. 两者都是可重入锁

   **“可重入锁”** 指的是自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增 1，所以要等到锁的计数器下降为 0 时才能释放锁。

2. synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API

   `synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

3. ReentrantLock 比 synchronized 增加了一些高级功能

   相比`synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：	

- **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来制定是否是公平的。
- **可实现选择性通知（锁可以绑定多个条件）**: `synchronized`关键字与`wait()`和`notify()`/`notifyAll()`方法相结合可以实现等待/通知机制。`ReentrantLock`类当然也可以实现，但是需要借助于`Condition`接口与`newCondition()`方法。

> `Condition`是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个`Lock`对象中可以创建多个`Condition`实例（即对象监视器），**线程对象可以注册在指定的`Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用`notify()/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，用`ReentrantLock`类结合`Condition`实例可以实现“选择性通知”** ，这个功能非常重要，而且是 Condition 接口默认提供的。而`synchronized`关键字就相当于整个 Lock 对象中只有一个`Condition`实例，所有的线程都注册在它一个身上。如果执行`notifyAll()`方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而`Condition`实例的`signalAll()`方法 只会唤醒注册在该`Condition`实例中的所有等待线程。

### synchronized关键字可以加在static方法上吗？锁的是什么？

可以加在static方法上，锁的是该类对象，俗称“类锁”。

#### **情况1**：用类直接在两个线程中调用两个不同的同步方法

**结果**：会产生互斥。

#### **情况2**：用一个类的静态对象在两个线程中调用静态方法或非静态方法

**结果**：会产生互斥。

### synchronized关键字加在非static方法上锁的是什么？

**Synchronized修饰非静态方法，实际上是对调用该方法的对象加锁，俗称“对象锁”。**

#### 情况1：同一个对象在两个线程中分别访问该对象的两个同步方法

**结果**：会产生互斥。

#### **情况2**：不同对象在两个线程中调用同一个同步方法

**结果**：不会产生互斥。

### ReentrantLock的特点和使用方法？

由于`ReentrantLock`是重入锁，所以可以反复得到相同的一把锁，它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放(重入锁)。

### “可重入”的概念及实现方式以及Lock，tryLock，unLock等基本方法，以及使用

- 广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。
-  Lock是JAVA5.0出现的，它的出现并不是为了替代synchronized，而是在synchronized不适用的时候使用。是公平和非公平锁

- tryLock:拿到锁返回true，否则false；带有时间限制的tryLock(long time, TimeUnit timeUnit),拿不到锁，就等待一段时间，超时返回false

- lockInterruptibly ：调用后如果没有获取到锁会一直阻塞，阻塞过程中会接受中断信号。

这些方法实现一个简单的自旋锁的原理？
### ConcurrentHashMap在Java7前使用的是可重入锁吗？可重入锁如何实现线程安全？

k 1.7使用segment存放数据，segment继承了ReentrantLock，是一种可重入锁，segment维护了哈希的若干个桶，每个桶由HashEntry构成链表

### 线程之间共享哪些内容？

堆和方法区。不共享jvm调用栈，本地调用栈和程序计数器。

### 如何访问这些共享内存？

### volatile和synchronized的区别

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- **volatile 关键字**是线程同步的**轻量级实现**，所以**volatile 性能肯定比 synchronized 关键字要好**。但是**volatile 关键字只能用于变量而 synchronized 关键字可以修饰方法以及代码块**。
- **volatile 关键字能保证数据的可见性，但不能保证数据的原子性。synchronized 关键字两者都能保证。**
- **volatile 关键字主要用于解决变量在多个线程之间的可见性，而 synchronized 关键字解决的是多个线程之间访问资源的同步性。**

### 对java.util.concurrent包的了解

java.util.concurrent 包含许多线程安全、测试良好、高性能的并发构建块

此包包含locks,concurrent,atomic 三个包。

Atomic：原子数据的构建。

Locks：基本的锁的实现，最重要的AQS框架和lockSupport

Concurrent：构建的一些高级的工具，如线程池，并发队列等。

## 线程池
### 线程池，有哪几个参数，有哪几种，有哪些拒绝策略 （线程池所有参数讲一讲）
帮我们管理线程，避免增加创建线程和销毁线程的资源损耗。提高响应速度。 重复利用。
### 线程池函数
- Executors.newCachedThreadPool()：无限线程池。
	
	- 用于并发执行大量短期的小任务
- Executors.newFixedThreadPool(nThreads)：创建固定大小的线程池。
	- 提交任务
	- 如果线程数少于核心线程，创建核心线程执行任务
	- 如果线程数等于核心线程，把任务添加到LinkedBlockingQueue阻塞队列
	- 如果线程执行完任务，去阻塞队列取任务，继续执行。
	- FixedThreadPool适用于处理CPU密集型的任务，确保CPU在长期被工作线程使用的情况下，尽可能的少分配线程，适用执行长期的任务。
	#### 使用无界队列的线程池会导致内存飙升吗？
	- 会的，newFixedThreadPool使用了无界的阻塞队列LinkedBlockingQueue，如果线程获取一个任务后，任务的执行时间比较长(比如，上面demo设置了10秒)，会导致队列的任务越积越多，导致机器内存使用不停飙升， 最终导致OOM。
- Executors.newSingleThreadExecutor()：创建单个线程的线程池。
	
	- 用于串行执行任务
- newScheduledThreadPool(定时及周期执行的线程池)
	
	- 周期性执行任务的场景，需要限制线程数量的场景 
### 参数
- ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)
	- corePoolSize 为线程池的基本大小。
	- maximumPoolSize 为线程池最大线程大小。
	- keepAliveTime 和 unit 则是线程空闲后的存活时间。
	- workQueue 用于存放任务的阻塞队列。
	- handler 当队列和最大线程池都满了之后的饱和策略。
### 拒绝策略 
- AbortPolicy(抛出一个异常，默认的)
- DiscardPolicy(直接丢弃任务)
- DiscardOldestPolicy（丢弃队列里最老的任务，将当前这个任务继续提交给线程池）
- CallerRunsPolicy（交给线程池调用所在的线程进行处理)

### 线程池异常处理 ？？？
- try-catch捕获异常
- submit执行的任务，可以通过Future对象的get方法接收抛出的异常，再进行处理。
- 实例化是，传入自己的ThreadFactory，为工作者线程设置UncaughtExceptionHandler处理未检测的异常
- 重写ThreadPoolExecutor的afterExecute方法处理传递的异常引用

### 线程池的工作队列
- ArrayBlockingQueue 有界阻塞队列
- LinkedBlockingQueue 阻塞链表 newFixedThreadPool和newSingleThreadPool使用了这个队列
- DelayQueue 延迟队列 newScheduledThreadPool线程池使用了这个队列
- PriorityBlockingQueue 优先级队列（具有优先级的无界阻塞队列）
- SynchronousQueue 同步队列 一个不存储元素的阻塞队列，吞吐量高，newCachedThreadPool线程池使用了这个队列
### 如果核心线程数量满了，阻塞队列也满了那么再来个任务是直接创建非核心线程还是进入队再出队再创建 
- 直接创建非核心线程，如果线程数量已经达到maximumPoolSize则抛出RejectedExecutionHandler异常
乐观锁，悲观锁 
CAS是硬件实现还是软件实现 
volatile是锁吗？ 
除了wait和notifyall，还有什么办法实现类似的功能 
线程间通信有哪些方式（加锁，内存屏障）
NIO，说一下 

## Java锁

### java多线程中一般使用哪些锁？

### 乐观锁，悲观锁
#### 悲观锁
- synchronized，接口Lock的实现类
#### 乐观锁
- CAS算法，AtomicInteger
### 读锁（共享锁），写锁（排它锁）
### 自旋锁，非自旋锁
### 无锁->偏向锁->轻量级锁->重量级锁
- 1.6之后引入，锁可以升级，但是不可以降级
### 分布式锁
### 区间锁（分段锁）java.util.concurrent concurrentHashMap
### 重入锁，非重入锁
### 公平锁，非公平锁 

减少上下文切换的方法有无锁并发编程、CAS算法、使用最少线程和使用协程。

## 参考链接
1. [线程池](https://crossoverjie.top/2018/07/29/java-senior/ThreadPool/)
2. [线程池ThreadPoolExecutor详解](https://juejin.im/post/6844904146856837128)
3. [java线程池解析](https://juejin.im/post/6844903889678893063)
4. [volatile关键字](https://juejin.im/post/6844904149536997384)
5. [java并发常见面试题总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/JavaConcurrencyAdvancedCommonInterviewQuestions.md#2-volatile-%E5%85%B3%E9%94%AE%E5%AD%97)
6. [可重入锁](https://blog.csdn.net/rickiyeat/article/details/78314451)

7. [Synchronized同步静态方法和非静态方法](https://blog.csdn.net/u010842515/article/details/65443084)
8. [concurrent包详细分析](https://blog.csdn.net/windsunmoon/article/details/36903901)

