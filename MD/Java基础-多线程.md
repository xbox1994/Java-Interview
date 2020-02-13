## 多线程
### 线程的生命周期
新建 -- 就绪 -- 运行 -- 阻塞 -- 就绪 -- 运行 -- 死亡

![](https://github.com/xbox1994/Java-Interview/raw/master/images/j13.png)

### 问：你怎么理解多线程的

1. 定义：多线程是指从软件或者硬件上实现多个线程并发执行的技术。具有多线程能力的计算机因有硬件支持而能够在同一时间执行多于一个线程，进而提升整体处理性能。
2. 存在的原因：因为单线程处理能力低。打个比方，一个人去搬砖与几个人去搬砖，一个人只能同时搬一车，但是几个人可以同时一起搬多个车。
3. 实现：在Java里如何实现线程，Thread、Runnable、Callable。
4. 问题：线程可以获得更大的吞吐量，但是开销很大，线程栈空间的大小、切换线程需要的时间，所以用到线程池进行重复利用，当线程使用完毕之后就放回线程池，避免创建与销毁的开销。

### 线程间通信的方式
1. 等待通知机制 wait()、notify()、join()、interrupted()
2. 并发工具synchronized、lock、CountDownLatch、CyclicBarrier、Semaphore

### 锁
#### 锁是什么
锁是在不同线程竞争资源的情况下来分配不同线程执行方式的同步控制工具，只有线程获取到锁之后才能访问同步代码，否则等待其他线程使用结束后释放锁

#### [synchronized](https://mp.weixin.qq.com/s/0qyNS6wQUShhJHoMuyziMg)
通常和wait，notify，notifyAll一块使用。    
wait：释放占有的对象锁，释放CPU，进入等待队列只能通过notify/all继续该线程。  
sleep：则是释放CPU，但是不释放占有的对象锁，可以在sleep结束后自动继续该线程。  
notify：唤醒等待队列中的一个线程，使其获得锁进行访问。  
notifyAll：唤醒等待队列中等待该对象锁的全部线程，让其竞争去获得锁。

#### lock
拥有synchronize相同的语义，但是添加一些其他特性，如中断锁等候和定时锁等候，所以可以使用lock代替synchronize，但必须手动加锁释放锁

#### 两者的区别
* 性能：资源竞争激烈的情况下，lock性能会比synchronized好；如果竞争资源不激烈，两者的性能是差不多的
* 用法：synchronized可以用在代码块上，方法上。lock通过代码实现，有更精确的线程语义，但需要手动释放，还提供了多样化的同步，比如公平锁、有时间限制的同步、可以被中断的同步
* 原理：synchronized在JVM级别实现，会在生成的字节码中加上monitorenter和monitorexit，任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。monitor是JVM的一个同步工具，synchronized还通过内存指令屏障来保证共享变量的可见性。lock使用AQS在代码级别实现，通过Unsafe.park调用操作系统内核进行阻塞
* 功能：比如ReentrantLock功能更强大
  1. ReentrantLock可以指定是公平锁还是非公平锁，而synchronized只能是非公平锁，所谓的公平锁就是先等待的线程先获得锁
  2. ReentrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程
  3. ReentrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制

**我们写同步的时候，优先考虑synchronized，如果有特殊需要，再进一步优化。ReentrantLock和Atomic如果用的不好，不仅不能提高性能，还可能带来灾难。**

#### 锁的种类
* 公平锁/非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁  
ReentrantLock通过构造函数指定该锁是否是公平锁，默认是非公平锁。Synchronized是一种非公平锁  

* 可重入锁

指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁  
ReentrantLock, 他的名字就可以看出是一个可重入锁，其名字是Re entrant Lock重新进入锁  
Synchronized,也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁  

* 独享锁/共享锁

独享锁是指该锁一次只能被一个线程所持有。共享锁是指该锁可被多个线程所持有  
ReentrantLock是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁  
读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的  
Synchronized是独享锁  

* 乐观锁/悲观锁

悲观锁在Java中的使用，就是各种锁  
乐观锁在Java中的使用，是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新  

* 偏向锁/轻量级锁/重量级锁

针对Synchronized的锁状态：  
偏向锁是为了减少无竞争且只有一个线程使用锁的情况下，使用轻量级锁产生的性能消耗。指一段同步代码一直被一个线程所访问，在无竞争情况下把整个同步都消除掉  
轻量级锁是为了减少无实际竞争情况下，使用重量级锁产生的性能消耗。指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过CAS自旋的形式尝试获取锁，不会阻塞  
重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁  

* 自旋锁/自适应自旋锁

指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU，默认自旋次数为10  
自适应自旋锁的自旋次数不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态决定，是虚拟机对锁状况的一个预测  

### volatile
功能：

1. 主内存和工作内存，直接与主内存产生交互，进行读写操作，保证可见性；
2. 禁止 JVM 进行的指令重排序。

### ThreadLocal
使用`ThreadLocal<UserInfo> userInfo = new ThreadLocal<UserInfo>()`的方式，让每个线程内部都会维护一个ThreadLocalMap，里边包含若干了 Entry（K-V 键值对），每次存取都会先获取到当前线程ID，然后得到该线程对象中的Map，然后与Map交互。

### 线程池
#### 起源
new Thread弊端：

* 每次启动线程都需要new Thread新建对象与线程，性能差。线程池能重用存在的线程，减少对象创建、回收的开销。
* 线程缺乏统一管理，可以无限制的新建线程，导致OOM。线程池可以控制可以创建、执行的最大并发线程数。
* 缺少工程实践的一些高级的功能如定期执行、线程中断。线程池提供定期执行、并发数控制功能

#### 线程池时核心参数

* corePoolSize：核心线程数量，线程池中应该常驻的线程数量
* maximumPoolSize：线程池允许的最大线程数，非核心线程在超时之后会被清除
* workQueue：阻塞队列，存储等待执行的任务
* keepAliveTime：线程没有任务执行时可以保持的时间
* unit：时间单位
* threadFactory：线程工厂，来创建线程
* rejectHandler：当拒绝任务提交时的策略（抛异常、用调用者所在的线程执行任务、丢弃队列中第一个任务执行当前任务、直接丢弃任务）

#### 创建线程的逻辑
以下任务提交逻辑来自ThreadPoolExecutor.execute方法：  

1. 如果运行的线程数 < corePoolSize，直接创建新线程，即使有其他线程是空闲的
2. 如果运行的线程数 >= corePoolSize  
    2.1 如果插入队列成功，则完成本次任务提交，但不创建新线程  
    2.2 如果插入队列失败，说明队列满了  
        2.2.1 如果当前线程数 < maximumPoolSize，创建新的线程放到线程池中  
        2.2.2 如果当前线程数 >= maximumPoolSize，会执行指定的拒绝策略

#### [阻塞队列的策略](https://blog.csdn.net/hayre/article/details/53291712)
* 直接提交。SynchronousQueue是一个没有数据缓冲的BlockingQueue，生产者线程对其的插入操作put必须等待消费者的移除操作take。将任务直接提交给线程而不保持它们。
* 无界队列。当使用无限的 maximumPoolSizes 时，将导致在所有corePoolSize线程都忙时新任务在队列中等待。
* 有界队列。当使用有限的 maximumPoolSizes 时，有界队列（如ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。

### 并发包工具类
#### CountDownLatch
计数器闭锁是一个能阻塞主线程，让其他线程满足特定条件下主线程再继续执行的线程同步工具。

![](https://github.com/xbox1994/Java-Interview/raw/master/images/countdownlatch.png)
图中，A为主线程，A首先设置计数器的数到AQS的state中，当调用await方法之后，A线程阻塞，随后每次其他线程调用countDown的时候，将state减1，直到计数器为0的时候，A线程继续执行。

使用场景:  
并行计算：把任务分配给不同线程之后需要等待所有线程计算完成之后主线程才能汇总得到最终结果  
模拟并发：可以作为并发次数的统计变量，当任意多个线程执行完成并发任务之后统计一次即可  

#### Semaphore
信号量是一个能阻塞线程且能控制统一时间请求的并发量的工具。比如能保证同时执行的线程最多200个，模拟出稳定的并发量。

```java
public class CountDownLatchTest {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Semaphore semaphore = new Semaphore(3); //配置只能发布3个运行许可证
        for (int i = 0; i < 100; i++) {
            int finalI = i;
            executorService.execute(() -> {
                try {
                    semaphore.acquire(3); //获取3个运行许可，如果获取不到会一直等待，使用tryAcquire则不会等待
                    Thread.sleep(1000);
                    System.out.println(finalI);
                    semaphore.release(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        executorService.shutdown();
    }
}
```

由于同时获取3个许可，所以即使开启了100个线程，但是每秒只能执行一个任务

使用场景:  
数据库连接并发数，如果超过并发数，等待（acqiure）或者抛出异常（tryAcquire）  

#### CyclicBarrier
可以让一组线程相互等待，当每个线程都准备好之后，所有线程才继续执行的工具类

![](https://github.com/xbox1994/Java-Interview/raw/master/images/cyclicbarrier.png)

与CountDownLatch类似，都是通过计数器实现的，当某个线程调用await之后，计数器减1，当计数器大于0时将等待的线程包装成AQS的Node放入等待队列中，当计数器为0时将等待队列中的Node拿出来执行。

与CountDownLatch的区别：  
1. CountDownLatch是一个线程等其他线程，CyclicBarrier是多个线程相互等待
2. CyclicBarrier的计数器能重复使用，调用多次

使用场景：
有四个游戏玩家玩游戏，游戏有三个关卡，每个关卡必须要所有玩家都到达后才能允许通过。其实这个场景里的玩家中如果有玩家A先到了关卡1，他必须等到其他所有玩家都到达关卡1时才能通过，也就是说线程之间需要相互等待。

### 编程题
交替打印奇偶数

```java
public class PrintOddAndEvenShu {
    private int value = 0;

    private synchronized void printOdd() {
        while (value <= 100) {
            if (value % 2 == 1) {
                System.out.println(Thread.currentThread() + ": -" + value++);
                this.notify();
            } else {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }

    }

    private synchronized void printEven() {
        while (value <= 100) {
            if (value % 2 == 0) {
                System.out.println(Thread.currentThread() + ": --" + value++);
                this.notify();
            } else {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        PrintOddAndEvenShu print = new PrintOddAndEvenShu();
        Thread t1 = new Thread(print::printOdd);
        Thread t2 = new Thread(print::printEven);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }
}
```

欢迎光临[我的博客](http://www.wangtianyi.top/?utm_source=github&utm_medium=github)，发现更多技术资源~
