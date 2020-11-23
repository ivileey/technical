# Spring 

## 一、SpringIOC 

## (一）、启动流程

### refresh()方法：

1. prepareRefresh();准备工作，记录下容器的启动时间，标记“已启动”状态，处理配置文件中的占位符
2. obtainFreshBeanFactory();配置文件会解析成一个个Bean定义，注册到BeanFactory中，当然这里说的是Bean还没有初始化，只是配置信息提取出来了。注册将信息保存到注册中心（beanName ->beanDefinition的map）
3. prepareBeanFactory(beanFactory);设置BeanFactory的类加载器，添加几个BeanPostProcessor，手动注册几个特殊的bean
4. postProcessBeanFactory(beanFactory);Bean如果实现了BeanFactoryPostProcessor()这个接口，那么在容器初始化以后，Spring会负责调用里面的postProcessBeanFactory方法。
5. invokeBeanFactoryPostProcessors(beanFactory);调用BeanFactoryProcessor各个实现类的postProcessBeanFactory(factory)方法。
6. registerBeanPostProcessors(beanFactory);注册BeanPostProcessor的实现类。（此接口的两个方法：postProcessBeforeInitialization和postProcessAfterInitialization。两个方法分别在Bean初始化之前和初始化之后得到执行)。
7. initMessageSource();初始化当前ApplicationContext的MessageSource。国际化。
8. initApplicationEventMulticaster();初始化当前ApplicationContext的事件广播器。
9. onRefresh()；钩子方法，具体的子嘞可以在这里初始化一些特殊的Bean
10. registerListeners();注册事件监听器，监听器需要实现ApplicationListener接口。
11. **finishBeanFactoryInitialization(beanFactory)**；初始化所有的singleton beans
12. finishRefresh();ApplicationContext初始化完成

### obtainFreshBeanFactory()

> 这里会初始化BeanFactory、加载Bean、注册Bean

调用refreshBeanFactory();关闭旧的Beanfactory，创建新的BeanFactory，加载Bean定义、注册Bean等等。

**refreshBeanFactory()：**

1. createBeanFactory();初始化一个defaultListableBeanFactory。
2. cutomizeBeanFActory(beanFactory);设置BeanFactory的两个配置属性：是否循序Bean覆盖、是否允许循环引用
3. loadBeanDefinitions(beanFactory);加载Bean到BeanFactory中

**customizeBeanFactory(beanFactory：**

- 比较简单，就是配置是否允许BeanDefinition覆盖、是否允许循环引用。

**loadBeanDefinitions**:

这个方法将根据配置加载各个Bean，然后放到BeanFactory中，

1. 给这个BeanFactory实例化一个XmlBeanDefinitionReader
2. 用刚刚初始化的Reader来加载xml配置。
3. 返回从当前配置文件加载了多少数量的Bean
4. 一个配置文件转换为一颗DOM树
5. 核心解析方法parseBeanDefinitions

### **finishBeanFactoryInitialization(beanFactory)**；

> 初始化所有的singleton beans

1. 调用preInstantiateSingletons()方法

2. ```java
   // this.beanDefinitionNames 保存了所有的 beanNames
      List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
   ```

3. 遍历集合触发所有的非懒加载的singleton beans

### getBean()

```java
// 我们在剖析初始化 Bean 的过程，但是 getBean 方法我们经常是用来从容器中获取 Bean 用的，注意切换思路，
// 已经初始化过了就从容器中直接返回，否则就先初始化再返回
// 获取一个 “正统的” beanName，处理两种情况，一个是前面说的 FactoryBean(前面带 ‘&’)，
// 一个是别名问题，因为这个方法是 getBean，获取 Bean 用的，你要是传一个别名进来，是完全可以的
```

### BeanDefinition接口

1. 默认只提供sington和prototype两种
2. 设置父Bean
3. 获取父Bean

4. 设置Bean的类名称
5. 获取Bean的类名称
6. 设置bean的scope
7. 设置是否是懒加载
8. 设置该Bean依赖的所有Bean
9. 返回该Bean的所有依赖
10. 设置该Bean是否可以注入到其他Bean中
11. 该Bean是否可以注入到其他Bean中
12. 设置/获取是否是主要的
13. 设置/获取工厂名称/工厂方法名称
14. 构造器参数
15. Bean中的属性值
16. 是否是单例以及是否是多例
17. 是否被设置成Abstract。如果是不能被实例化

### （二）、IOC容器

#### Spring BeanFactory容器

- 这系列容器只实现了容器的最基本功能。

#### Spring ApplicationContext容器

- 作为容器的高级形态而存在。在简单容器的基础上，增加了许多面向框架的特性，同时对应用环境做了许多适配

### BeanFactory和ApplicationContext的区别

BeanFactory无法支持spring插件，例如：AOP、Web应用等功能。

<1>如果使用ApplicationContext，如果配置的bean是singleton，那么不管你有没有或想不想用它，它都会被实例化。好处是可以预先加载，坏处是浪费内存。
<2>BeanFactory，当使用BeanFactory实例化对象时，配置的bean不会马上被实例化，而是等到你使用该bean的时候（getBean）才会被实例化。好处是节约内存，坏处是速度比较慢。多用于移动设备的开发。

相比那些简单扩展BeanFactory的基本IoC容器，开发人员常用的ApplicationContext除了能够提供前面介绍的容器的基本功能外，还为用户提供了以下的附加服务：

- 支持不同的信息源
- 访问资源：体现在对ResourceLoader和Resource的支持上，可以从不同的地方得到Bean定义资源。
- 支持应用事件。继承了接口ApplicationEventPublisher。这些事件和Bean的生命周期的结合为Bean的管理提供了便利。
- 在ApplicationContext中提供的附加服务。对它使用的是一种面向框架的使用风格。

### BeanDefinition

- Spring通过定义BeanDefinition来管理基于Spring的应用中的各种对象以及它们之间的相互依赖关系。
- IoC容器是用来管理对象依赖关系的，对IoC容器来说，BeanDefinition就是对依赖反转模式中管理的对象依赖关系的数据抽象，也是容器实现依赖反转功能的核心数据结构，依赖反转功能都是围绕对这个BeanDefinition的处理来完成的。

#### Bean和BeanDefinition的区别

Bean相当于是BeanDefinition的实例。BeanDefinition 中保存了我们的 Bean 信息，比如这个 Bean 指向的是哪个类、是否是单例的、是否懒加载、这个 Bean 依赖了哪些 Bean 等等。再说一下Bean是如何变成的BeanDefinition的，在我们容器进行创建的时候调用refresh()方法，refresh()方法中第一步会BeanDefinition中

### BeanPostProcessor

有两个方法：

1. Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

2. Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

分别在Bean的初始化之前和初始化之后执行，执行这两个方法有什么用

Spring IoC容器允许BeanFactoryPostProcessor在容器实例化任何bean之前读取bean的定义(配置元数据)，并可以修改它。同时可以定义多个BeanFactoryPostProcessor，通过设置'order'属性来确定各个BeanFactoryPostProcessor执行顺序。

### 为什么使用SpringIOC

一开始创建我们必须要一步步手动的来创建对象，比如一个dao，service，controller。在使用springioc之后我们只需要向容器中直接要就可以了，可以使用注解的方式@Autowired或者使用xml配置然后使用getbean的方式。

### 使用了哪些设计模式



## 二、Spring AOP

 AOP(Aspect-Oriented Programming:⾯向切⾯编程)能够将那些与业务⽆关，却为业务模块所共同调⽤ 的逻辑或责任（例如事务处理、⽇志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring充分利用了IoC容器Proxy代理对象以及对AOP拦截器的功能特性，通过这些对AOP基本功能封装机制，为用户提供了AOP的实现框架。

 Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接⼝，那么Spring AOP会使⽤JDK Proxy，去创建代理对象，⽽对于没有实现接⼝的对象，就⽆法使⽤ JDK Proxy 去进⾏代理了，这时候 Spring AOP会使⽤Cglib ，这时候Spring AOP会使⽤ Cglib ⽣成⼀个被代理对象的⼦类来作为代理。



主要是两种，一种是JDK动态代理，一种是Cglib代理。

两者的区别：
 1.JDK动态代理只能代理实现了接口的类，通过newInstanse来创建，动态代理类的字节码在程序运行时由Java反射机制动态生成。
 2.Cglib是可以代理没有实现接口的类，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，所以不能对final修饰的类进行代理。底层采用ASM实现。



##  Spring 框架中用到了哪些设计模式？

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

## 1.springboot循环依赖怎么解决？

①构造器的循环依赖：这种依赖spring是处理不了的，直 接抛出BeanCurrentlylnCreationException异常。 

②单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖。

 ③非单例循环依赖：无法处理。

## SpringCloud

- Spring Cloud Config

  集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作。

- Spring Cloud Netflix
- Spring Cloud Bus
- Spring Cloud Consul
- Spring Cloud Security
- Spring Cloud Zookeeper

