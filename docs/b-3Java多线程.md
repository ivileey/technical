# Java多线程
## 可见性和原子性
- volatile可以确保变量的可见性，当一个线程读一个线程写时，当两个线程同时对读取的数据进行写操作可见性就失效了
- AtomicInteger
## java多线程中一般使用哪些锁？
#ReetrantLock和Synchronized关键字和实现原理
synchronized关键字可以加在static方法上吗？锁的是什么？
synchronized关键字加在非static方法上锁的是什么？
ReentrantLock的特点和使用方法？
“可重入”的概念及实现方式以及Lock，tryLock，unLock等基本方法，以及使用
这些方法实现一个简单的自旋锁的原理？
ConcurrentHashMap在Java7前使用的是可重入锁吗？可重入锁如何实现线程安全？
线程之间共享哪些内容？
不共享jvm调用栈，本地调用栈和程序计数器。
如何访问这些共享内存？
voltile和synchronized的区别
对java.util.concurrent包的了解
## 线程池，有哪几个参数，有哪几种，有哪些拒绝策略 （线程池所有参数讲一讲）
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



