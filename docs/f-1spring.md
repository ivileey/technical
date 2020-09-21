# Spring概述
	Spring是一个基于IOC和AOP的结构J2EE系统的框架
## IOC 
	ioc反转控制是Spring的基础，Inversion Of Control
	简单说就是创建对象由以前的程序员自己new 构造方法来调用，变成了交由Spring创建对象
	DI 依赖注入 Dependency Inject. 简单地说就是拿到的对象的属性，已经被注入好相关值了，直接使用即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828154343599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pc2h1aWFlZQ==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828154408825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pc2h1aWFlZQ==,size_16,color_FFFFFF,t_70#pic_center)

## AOP
	AOP 即 Aspect Oriented Program 面向切面编程
	首先，在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。
	所谓的核心业务，比如登陆，增加数据，删除数据都叫核心业务
	所谓的周边功能，比如性能统计，日志，事务管理等等
	
	周边功能在Spring的面向切面编程AOP思想里，即被定义为切面
	
	在面向切面编程AOP的思想里面，核心业务功能和切面功能分别独立进行开发
	然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP

1. Spring bean加载，实例化的过程 
2. Spring的IOC/AOP的实现
3. 动态代理的实现方式
4. Spring如何解决循环依赖（三级缓存）
5. SpringBoot如何加载，源码
6. 为什么用mvc架构
7. Spring的后置处理器
8. Spring的@Transactional如何实现的
9.  Spring的事务传播级别 
10. BeanFactory和ApplicationContext的联系和区别 

## 参考链接：
1. [spring具体应用](https://how2j.cn/k/spring/spring-ioc-di/87.html)
2. [springIOC的好处](https://www.zhihu.com/question/23277575/answer/169698662)