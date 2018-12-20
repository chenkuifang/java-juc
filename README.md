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

**java.util.concurrent.atomic包提供了一些常用的原子变量类**
- AtomicInteger,AtomicBoolean,AtomicLong,AtomicReference
- AtomicIntegerArray,AtomicLongArray
- AtomicMarkableReference 
- AtomicReferenceArray
- AtomicStampedReference

## 4.CAS算法  

- CAS(Compare-And-Swap) 算法是硬件对于并发的支持,针对多处理器操作而设计的处理器中的一种特殊指令,用于管理对共享数据的并发访问;
- CAS 是一种无锁的非阻塞算法的实现;
- CAS 包含了三个操作数:
    1. 需要读写的内存值: V
    2. 进行比较的预估值: A
    3. 拟写入的更新值: B
    4. 当且仅当 V == A 时, V = B, 否则,将不做任何操作;
**以下是模拟CAS算法代码** 
 
```
public class TestCompareAndSwap {

    @Test
    public void test1() {
        final CompareAndSwap cas = new CompareAndSwap();

        for (int i = 0; i < 10; i++) {
            // 创建10个线程,模拟多线程环境
            new Thread(() -> {
                Object expectedValue = cas.getValue();
                boolean b = cas.compareAndSet(expectedValue, (int) (Math.random() * 100));
                System.out.println(b);
            }).start();
        }
    }

}

class CompareAndSwap {
    private volatile Object value;

    // 读取内存中的值
    public synchronized Object getValue() {
        return value;
    }

    // 比较
    public synchronized Object compareAndSwap(Object expectedValue, Object updateValue) {
        Object oldValue = value;

        if (oldValue == expectedValue) {
            this.value = updateValue;
        }

        // 最后返回旧值
        return oldValue;
    }

    // 设置值
    public synchronized boolean compareAndSet(Object expectedValue, Object updateValue) {
        return expectedValue == compareAndSwap(expectedValue, updateValue);
    }
}
```   
## 5.ConcurrentHashMap 锁分段机制  
**ConcurrentHashMap同步容器类是java5增加的一种线程安全的哈希表。对于多线程操作，介于HashMap和HashTable之间。内部采用了“锁分段”机制，替换了HashTable的独占锁，进而提高了性能。**  
**我们知道HashMap与HashTable的区别在线程安全与线程不安全，HashTable采用了独占锁的方式，当多个线程同时访问HashTable的时候，最多只能有一个线程访问(并行转串行),所以效率不高。并且会存在符合操作不安全的情况**
**ConcurrentHashMap的“锁分段”机制是把每个HashMap默认分为16个并发级别(ConcurrentLevel),每个级别分为一个段(segment),每个段中存放16个Entry,这样每个段维护一个独立的锁，当多个线程同时访问HashMap的时候，可以并行执行。**
**注意：ConcurrentHashMap在jdk1.8后“锁分段”机制被CAS替换。使用更轻量的安全机制。**
**此包还提供了用于设计多线程上下文中的Collection实现：**  
**ConcurrentHashMap,ConcurrentSkipListMap,ConcurrentSkipListSet,CopyOnWriteArrayList,CopyOnWriteArraySet.**  
**当许多线程访问同一个Collection时,通常ConcurrentHashMap要比同步的HashMap效率高，ConcurrentSkipListMap通常优于同步的TreeMap.**  
**当读数和遍历数远远大于更新的时候，CopyOnWriteArrayList通常优于同步的ArrayList.**  

## 6.CountDownLatch 闭锁  
- CountDownLatch 是juc包提供的一个同步辅助类。
- CountDownLatch 它可以让一组或一个线程在执行完成某个操作前，出于等待状态。如同所有运动员在没有做好预备的动作前，先做好预备的运动员必须等待其他没有做好预备的运动员。
  直到所有运动员都做好了预备，才开始跑步。
- 使用场景
  1. 确保某个计算在其需要的所有资源都执行完毕才继续执行（如启动游戏时，先加载完毕再执行其他）。
  2. 确保某个服务在其需要的其他所有服务都启动之后再启动。
  3. 等待直到某个操作的所有参与者都准备完成了才继续执行（运动员例子）。 
   
``` 
public class CountDownLatchTest {

    private static volatile CountDownLatch countDownLatch = new CountDownLatch(5);

    /**
     * Boss线程，等待员工到达开会
     */
    static class BossThread extends Thread {
        @Override
        public void run() {
            System.out.println("Boss在会议室等待，总共有" + countDownLatch.getCount() + "个人开会...");
            try {

                //Boss等待
                countDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("所有人都已经到齐了，开会吧...");
        }
    }

    // 员工到达会议室线程
    static class EmployeeThread extends Thread {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "，到达会议室....");
            //员工到达会议室 count - 1
            countDownLatch.countDown();
        }
    }

    @Test
    public void test() {
        //Boss线程启动
        new BossThread().start();

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
        }

        for (int i = 0; i < countDownLatch.getCount(); i++) {
            new EmployeeThread().start();
        }
    }

}
```  
# 7.Callable接口实现有返回的线程  
**复习：创建线程方式有4种**
- 继承Thread类，-->重新run() --> 启动：new MyThread().start()
- 实现Runnable接口,--->重写run()--->启动：new Thread(new MyRunnable()).start()
- 实现Callable接口(配合ExecutorService使用)，--->重写call()--->创建线程池：ExecutorService executorService = Executors.newFixedThreadPool(5);
  Future result = executorService.submit(new MyCallable()); -->result.get() 获取返回值
- 当然还可以使用JDK提供的线程池创建方式，一般使用Executors工具类。  
**首先说一下什么任务：实现Callable接口或Runnable接口的类，其实例就可以成为一个任务提交给ExecutorService去执行,这样的好处是任务和执行解耦。
其中Callable任务可以返回执行结果，Runnable任务无返回结果；**  

**通过Executors工具类可以创建各种类型的线程池，如下为常见的四种：**
- newCachedThreadPool ：大小不受限，当线程释放时，可重用该线程；
- newFixedThreadPool ：大小固定，无可用线程时，任务需等待，直到有可用线程；
- newSingleThreadExecutor ：创建一个单线程，任务会按顺序依次执行；
- newScheduledThreadPool：创建一个定长线程池，支持定时及周期性任务执行。  
``` 
    /**
     * 创建一个单线程执行器
     */
    @Test
    public void test1() {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        executorService.submit(new Task());
        executorService.shutdown();
    }

    class Task implements Runnable {
        @Override
        public void run() {
            System.err.println("running...");
        }
    }
``` 
``` 
    /**
     * 创建一个指定大小的线程池，并提交一个有返回的任务
     * <p>
     * 使用场景：
     * FutureTask可用于异步获取执行结果或取消执行任务的场景。通过传入Runnable或者Callable的任务给FutureTask
     * ，直接调用其run方法或者放入线程池执行，之后可以在外部通过FutureTask的get方法异步获取执行结果，
     * 因此，FutureTask非常适合用于耗时的计算，主线程可以在完成自己的任务后，
     * 再去获取结果。另外，FutureTask还可以确保即使调用了多次run方法，它都只会执行一次Runnable或者Callable任务
     * ，或者通过cancel取消FutureTask的执行等。
     */
    @Test
    public void test2() throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        Future result = executorService.submit(new CallBackTask());

        // 模拟主线程执行时间6S
        System.err.println(Thread.currentThread().getName());
        Thread.sleep(6000);

        // 主线程可以在完成自己的任务后，再去获取结果
        System.err.println(result.get()); //  get()方法会使所在线程阻塞，直到返回结果。
        executorService.shutdown();
    }

    /**
     * 注意：get()方法会使所在线程阻塞，直到返回结果。
     */
    class CallBackTask implements Callable<String> {
        @Override
        public String call() {
            try {
                System.err.println(Thread.currentThread().getName());
                Thread.sleep(5000);
                System.err.println(Thread.currentThread().getName() + "执行完成");
            } catch (InterruptedException e) {

            }
            return "I'm callback info!!";
        }
    }
``` 
# 8.同步锁 Lock
**JDK1.5之前同步使用Synchronized和volatile，JDK1.5后增加了显示锁Lock。**
- Synchronized属于内置的一种隐式锁。同步方式有两种
  1. 同步块
  2. 同步方法
- Lock 是JDK1.5后现在的显示锁，使用lock()获取锁，然后使用unlock()方法释放锁。
**Lock接口主要有两个实现类1.ReentrantLock(公平/非公平锁) 2.ReentrantReadWriteLock(读写锁)**  
**Lock相对于synchronized更灵活，粒度更细，需要手动获取锁和释放锁。**

