# IO多路复用	

说起javanio基础就是要先理解同步，异步，阻塞，非阻塞这四种含义，然后就涉及到了底层调用select，poll，epoll，函数的作用。这些全部理解下来可能会花个时间了，而且要从实战经验中理解，比如一个服务器什么时候用多路io复用，什么时候用多线程。javanio就是使用了多路io复用，基础就是同步非阻塞的io模型。redis和nginx也是用到了多路复用，epoll函数()。

<!--more-->

## 同步/异步/阻塞/非阻塞

当你同步执行某项任务时，你需要等待其完成才能继续执行其他任务。当你异步执行某些操作时，你可以在完成另一个任务之前继续进行

- **同步** ：两个同步任务相互依赖，并且一个任务必须以依赖于另一任务的某种方式执行。 比如在`A->B`事件模型中，你需要先完成 A 才能执行B。 再换句话说，同步调用中被调用者未处理完请求之前，调用不返回，调用者会一直等待结果的返回。
- **异步**： 两个异步的任务完全独立的，一方的执行不需要等待另外一方的执行。再换句话说，异步调用种一调用就返回结果不需要等待结果返回，当结果返回的时候通过回调函数或者其他方式拿着结果再做相关事情.
- **阻塞：** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
- **非阻塞：** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

同步/异步是从行为角度描述事物的，而阻塞和非阻塞描述的当前事物的状态（等待调用结果时的状态）。

## select/poll/epoll

​	判断是否有文件描述符准备好

- select

  fd_set 使用数组实现
  1.单个进程可监视的fd数量被限制，即能监听端口的大小有限。

     一般来说这个数目和系统内存关系很大。32位机默认是1024个。64位机默认是2048.

  2.rset不可重用，新的fd进来，重新创建
  3.用户态rset拷贝到内核态rset进行判断哪个文件描述符有数据，是一种阻塞函数，用户态内核态切换有开销
  4.O(n)时间复杂度的轮询去看哪个文件描述符有数据到达

  ```
    while(1){
  	FD_ZERO(&rset);
    	for (i = 0; i< 5; i++ ) {
    		FD_SET(fds[i],&rset);
    	}
   
     	puts("round again");
  	select(max+1, &rset, NULL, NULL, NULL);
   
  	for(i=0;i<5;i++) {
  		if (FD_ISSET(fds[i], &rset)){
  			memset(buffer,0,MAXBUF);
  			read(fds[i], buffer, MAXBUF);
  			puts(buffer);
  		}
  	}	
    }
  ```

  

- poll
  基于链表存储fd
  struct pollfd{
  int fd;
  short events; 
  short revents; //可重用
  }

置位revents字段，说明有数据进来，解决了size的限制，和不可重用问题，每次使用完revents置0

1、大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。

2、poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。 

- epoll

  1、没有最大并发连接的限制，能打开的FD的上限远大于1024（1G的内存上能监听约10万个端口）
  2、效率提升，不是轮询的方式，不会随着FD数目的增加效率下降。只有活跃可用的FD才会调用callback函数；

  3 epoll用红黑树存放要操作的fd

  4 epoll没有使用mmap，别再看那些说用了mmap的人了，直接看源码

## NIO

​	NIO（Non-blocking I/O，在Java领域，也称为New I/O）

​	BIO中等待就绪的阻塞是不使用CPU的，是在“空等”；而真正的读写操作的阻塞是使用CPU的，真正在”干活”，而且这个过程非常快，属于memory copy，带宽通常在1GB/s级别以上，可以理解为基本不耗时。

​	NIO的主要事件有几个：读就绪、写就绪、有新连接到来。

​	其次，用一个死循环选择就绪的事件，会执行系统调用（Linux 2.6之前是select、poll，2.6之后是epoll，Windows是IOCP），还会阻塞的等待新事件的到来。新事件到来的时候，会在selector上注册标记位，标示可读、可写或者有连接到来。



## Redis

1. 为什么 Redis 一开始选择单线程模型（单线程的好处）？

   - 使用 IO多路复用技术，同时监听多个文件描述符FD，由于大多是操作是基于内存的所以每次操作都非常快。
   - 可维护性高，由于是单线程的不用考虑并发读写问题
   - 基于内存，单线程状态下效率还是很高（一秒10万个用户请求），还可以使用redis分片技术。

2. 为什么 Redis 在 6.0 之后加入了多线程（在某些情况下，单线程出现了缺点，多线程可以解决）？

   因为读写网络的read/write系统调用在Redis执行期间占用了大部分CPU时间，如果把网络读写做成多线程的方式对性能会有很大提升。

   Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想 Redis 因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。

   Redis 在最新的几个版本中加入了一些可以被其他线程异步处理的删除操作，也就是我们在上面提到的 UNLINK、FLUSHALL ASYNC 和 FLUSHDB ASYNC。

## AIO

​	AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。



- 同步和异步仅仅是关注的消息如何通知的机制，而阻塞与非阻塞关注的是等待消息通知时的状态。也就是说，同步的情况下，是由处理消息者自己去等待消息是否被触发，而异步的情况下是由触发机制来通知处理消息者，所以在异步机制中，处理消息者和触发机制之间就需要一个连接的桥梁：

 - redis和nginx用的是epoll，javanio在linux下也是用的epoll
## BIO有什么缺陷？
- BIO中的“B”表示的是阻塞,作为服务器开发的话，我们使用ServerSocket绑定完端口号之后,我们会进行监听该端口,等待accept时间,accept会阻塞当前主线程
## 针对C10K这样的需求,NIO靠什么解决的问题？
## 多路复用操作系统函数 select(...)工作原理
## 多路复用操作系统函数select(...)默认监听socket数量为什么是1024
## 多路复用操作系统函数select(...)第一遍O(N)未发现就绪socket, 后续在某个socket就绪后,select(...)如何感知的？是不停的轮询吗？
## 多路复用操作系统函数poll(..)和select(..)主要区别是什么?
## 为什么会有epoll这个技术, 它的产生的背景是什么呢?
## epoll函数的工作原理是什么呢?
## eventpoll对象的就绪列表数据是如何维护的呢?
## eventpoll对象中存放需要检查的socket信息是采用什么数据结构？为什么？
## 参考链接

1. [JAVANIO](https://tech.meituan.com/2016/11/04/nio.html)
2. [redis为什么又引入了多线程](https://juejin.im/post/6844904131023339534)
3. [BIO,NIO,AIO总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/BIO-NIO-AIO.md)
4. [Select,poll,epoll](https://devarea.com/linux-io-multiplexing-select-vs-poll-vs-epoll/#.X0N-5NMzZTa)
5. [epoll](https://www.cnblogs.com/aspirant/p/9166944.html)
6. [同步，异步，阻塞，非阻塞](https://www.jianshu.com/p/aed6067eeac9)
2. [ServerSocket与Socket详解](https://blog.csdn.net/J080624/article/details/78468396)
3. [IO多路复用技术是什么](https://www.zhihu.com/question/28594409)

