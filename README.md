# Java-JUC  
## 1.什么是JUC?  
**在 Java 5.0 提供了 `java.util.concurrent`(简称JUC)包,在此包中增加了在并发编程中很常用的工具类,
用于定义类似于线程的自定义子系统,包括线程池,异步 IO 和轻量级任务框架;还提供了设计用于多线程上下文中
的 Collection 实现等;**  
**java.util.concurrent包总览**  
![3](https://github.com/chenkuifang/java-juc/blob/master/3.png)  
![2](https://github.com/chenkuifang/java-juc/blob/master/22.png)  
![1](https://github.com/chenkuifang/java-juc/blob/master/1.png)  
![4](https://github.com/chenkuifang/java-juc/blob/master/4.png)  
## 2.volatile关键字   
- 所有共享的资源都会放在主存中，线程操作的时候会从主存中先复制一份到线程的内存中，之后的操作都是在线程自己的内存中操作，然后再同步到主存中
- 这就会导致存在并发的线程，获取到初始值一样，各自操作后，同步到主存中值也是一样的，存在并发的问题
- 如果共享资源使用volatile修饰，可以保证内存中的数据是可见的。可以理解成所有的线程操作共享数据就是在主存中操作，不存在同步这个操作的时间差。
- volatile 相较于synchronized 是一种较为轻量级的同步策略
- volatile与synchronizde不同点
  1. volatile不具备"互斥性"
  2. volatile 不能保证变量的"原子性"
