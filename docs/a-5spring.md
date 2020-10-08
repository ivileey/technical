# Spring 常见面试题

## 1.springboot循环依赖怎么解决？

①构造器的循环依赖：这种依赖spring是处理不了的，直 接抛出BeanCurrentlylnCreationException异常。 

②单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖。

 ③非单例循环依赖：无法处理。