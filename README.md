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
 ``` java
  public class VolatileTest {
    private volatile int i = 0;

    @Test
    public void test1() {
        //10个线程操作add方法
        for (int i = 0; i < 10; i++) {
            new Thread(() -> System.err.println(Thread.currentThread().getName() + ":" + add()))
                    .start();
        }
    }
    private int add() {
        return i++;
    }
 }
}
```  
**以上代码是启动10个线程操作add()方法，同时对共享资源i进行操作，如果不使用同步策略或volatile关键字修饰i的话，那么得到的结果存在并发问题**  

**尚硅谷例子**  
```
// 使用 volatile 之前
public class TestVolatile{
 
    public static void main(String[] args){
        ThreadDemo td = new ThreadDemo();
        new Thread(td).start();
 
        while(true){
            if(td.isFlag()){
                System.out.println("########");
                break;
            }
        }
    }
}
 
class ThreadDemo implements Runnable{
    private boolean flag = false;
 
    public void run(){
        try{
            // 该线程 sleep(200), 导致了程序无法执行成功
            Thread.sleep(200);
        }catch(InterruptedException e){
            e.printStackTrace();
        }
 
        flag = true;
 
        Sytem.out.println("flag="+isFlag());
    }
 
    public boolean isFlag(){
        return flag;
    }
 
    public void setFlag(boolean flag){
        this.flag = flag;
    }
}
```
**以上代码结果：flag=true,然后主线程会一直不停的执行却不会打印########。因为while是jdk的底层封装好的代码，执行效率非常高，它的执行比下边的线程执行要早，所以当主线程while执行的时候，读取td中的flag到自己的内存中，一直都是false,所以主线程会一直的执行。**  

**修改一下private volatile boolean flag = false;使得多个线程操作共享数据时，彼此可见。结果为：########### flag=true**  
## 3.原子变量  
- i++的原子性问题
    1. i++的操作实际上分为三个步骤: "读-改-写";  
    如：i++;   
    实际可分为： int tmp = i; i = i + 1; i = tmp;
    2. 原子性: 就是"i++"的"读-改-写"是不可分割的三个步骤;
    3. 原子变量: JDK1.5 以后, java.util.concurrent.atomic包下,提供了常用的原子变量;
         1. 原子变量中的值,使用 volatile 修饰,保证了内存可见性;
         2. CAS(Compare-And-Swap) 算法保证数据的原子性;
            1. CAS算法：硬件对于并发操作共享资源的支持
               1. CAS 包含三个操作数：内存值 V .预估值 A. 更新值 B.
               2. 特性：当且仅当V==A时，才会执行V = B，否则不做任何操作
```               
public class AtomicTest {

    // 如果使用volatile 也不能保证数据操作的原子性，问题依然存在。
    private int i = 0;

    @Test
    public void test1() {
        for (int j = 0; j < 100; j++) {
            new Thread(() -> System.err.println(getI())).start();
        }
    }

    public int getI() {
        return i++;
    }
}
``` 
**运行以上代码，存在数结果不是99的情况,当某个两个线程同时读取到i的一样时候，就会出现结果错误。即使加上了volatile修饰i,也阻止不了加操作的同时进行，这个一个原子性问题。**
**使用原子变量即可解决以上问题。如 AtomicInteger**
**https://blog.csdn.net/qq_31580305/article/details/79736226**


