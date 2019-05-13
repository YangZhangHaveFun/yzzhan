---
title : "深入理解java并发知识的归纳"
date: 2019-05-13T01:37:56+08:00
lastmod: 2019-04-14T01:37:56+08:00
draft: false
tags: ["Java", "Concurrency"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---
### 多线程基础

#### 多线程的三个问题
- 多核和缓存导致的可见性问题: 一个线程对共享变量的修改，另外一个线程能够立刻看到，我们称为**可见性**.
- 线程切换带来的原子性问题: 我们把一个或者多个操作在 CPU 执行的过程中不被中断的特性**原子性**.
- 编译优化带来的有序性问题: 代码按照预期的顺序执行,称为**有序性**

#### Java内存模型
Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括 **volatile**, **synchronized**和**final**关键字, 以及六项**Happen-Before**规则.

##### Volatile
volatile 关键字最原始的意义是禁用CPU缓存.变量的读写必须从内存中读取或者写入.
```java
class VolatileExample {
    int x = 0;
    volatile boolean v = false;
    public void writer() {
        x = 42;
        v = true;
    }
    public void reader() {
        if (v == true) {
            // 若有两个线程,一个线程进行writer()一个线程进行reader()则
            // 这里 X 在1.5之前 是 0 或者42, X在1.5之后一定是42.
        }
    }
} 
```

##### Happen-Before规则
Java1.5 用**Happen-Before**规则来解决这个问题. 下面介绍六项**Happen-Before**的规则. 首先**Happen-Before**表示前面一个操作的结果对后续操作是可见的.
- **程序的顺序性规则**: 这条规则是指在一个线程中, 按照程序顺序, 前面的操作Happens-Before于后续的任意操作.
- **Volatile规则**: 这条规则是指对一个volatile的写操作, Happens-Before于后续对这个volatile变量的读操作.
- **传递性规则**: 这条规则指如果A Happens-Before B, 且 B Happens-Before C, 那么A Happens-Before C. 
- **管程中的锁规则**: 这条规则指一个对锁的解锁 Happens-Before 于后续对这个锁的枷锁.
- **线程的start的规则**: 这条规则指主线程启动子线程B后,子线程能够看到主线程在启动子线程B前的操作.
    ```java
    Thread sub = new Thread(()->{
        // 主线程调用B.start() 之前
        //所有对共享变量的修改,此处皆可见
        //这里var == 77 ---> return true
    });
    int var = 77;
    sub.start()
    ```
- **线程的join的规则**: 这条规则指主线程A等待子线程B完成(主线程A通过调用子线程B的join()方法实现), 当子线程B完成后(主线程A中join()方法返回), 主线程能够"看到"子线程的操作. 能够看到的部分是指对**共享变量**的修改.
  ```java
    Thread sub = new Thread(()->{
        var = 66
    });
    int var = 77;
    sub.start()
    sub.join()
    //此时var的数值变为了66
    ```

Java内存模型关于JVM的部分见深入理解JVM的读书总结.

#### 如果解决并发原子性问题---互斥锁
我们称 **同一时刻只有一个线程执行**这个条件为**互斥(mutex)**. 一段需要互斥执行的代码称为**临界区(critical area)**, 需要注意的是我们需要标注出来临界区内受保护的资源. Java中互斥锁的关键词为synchronized. 加锁的本质就是在锁对象的对象头中添加当前线程的ID.
```java
class X {
    //lock non-static method
    synchronized void foo() {
        //critical area
    }

    //lock static method
    synchronized static void bar() {
        //critical area
    }
    Object obj = new Object();
    void baz() {
        synchronized(obj) {
            //critical area
        }
    }
}
```
当synchronized修饰非静态方法时锁定当前实例对象this, 当synchronized修饰静态方法时锁定当前类的Class对象, 上面的例子可以等同为
```java
class X {
    //lock non-static method
    synchronized(this) void foo() {
        //critical area
    }

    //lock static method
    synchronized(X.class) static void bar() {
        //critical area
    }
}
```
受保护资源与锁的关联关系应该是N:1的关系, 也就是说一把锁可以保护多个资源,而多把锁保护同一资源并没有可见性的保证.
```java
class SafeCalc {
    static long value = 0L;
    synchronized long get() {
        return value;
    }

    synchronized static void addOne(){
        value += 1
    }
}
```
简单的来说,我们可以用一把锁锁住所有需要的数据,但是这样会导致性能问题.因为一把锁会使所有的方法串行但很多方法是可以并行运行的. 用不同的锁对保护资源进行精细化管理,能够很大程度上提高性能.这种锁叫做**细粒度锁**. 使用**细粒度锁**可以提高并行度,但设计不良会导致**死锁**.
##### 死锁
死锁指一组互相竞争资源的线程因互相等待,导致"永久"阻塞的现象. 触发死锁必须满足四个条件
- 互斥, 共享资源X和Y只能被一个线程占用.
- 占有且等待, 线程T已经取得共享资源X,在等待共享资源Y的时候,不释放共享资源X.
- 不可抢占,其他线程不能强行抢占线程T1占用的资源.
- 循环等待,线程T1等待线程T2占用的资源,线程T2等待线程T1占有的资源,就是循环等待.

破坏其中一条就可以避免死锁的发生. 一般较为实用的方法是破坏第二和第四条,第二条可以通过设置timeout或者循环等待所有条件就绪再占有来预防,第四条可以通过序列化共享资源的占用来避免.

#### "等待-通知"机制
在并发冲突较小时,循环等待所有条件尚可行.但当冲突量增大时,这种自旋的方式会白白浪费CPU. 另外一种可行的方式当线程要求不满足时,线程阻塞自己进入阻塞状态. 当线程线程的要求被满足时,别的线程可以通知线程继续执行.

在Java语言里, "等待-通知"机制可以由Java语言内置的synchronized配合wait(), notify(), notifyAll()来实现.

每个互斥锁都有自己的**等待队列**,这个等待和互斥锁是一对一的关系. 在并发程序中,当一个线程进入临界区后,由于某个条件不满足,需要进入等待状态, Java对象的wait()方法就需要被调用. 当调用wait()方法后, 当前线程就会被阻塞并进入到**等待队列**.这个等待队列也是互斥锁的等待队列. 在线程进入**等待队列**的同时,会释放持有的互斥锁. 随后其他线程就有机会获得锁并进入临界区.

当条件满足时,可以通过notifyAll()方法, 这个方法会通知等待队列(**对应互斥锁的等待队列**)中的线程,告诉它**条件曾经满足过**. 但被通知的线程想要重新被执行,仍然需要获取到互斥锁.
> 需要注意的是,wait(), notify(), notifyAll()这三个方法能够被调用的前提是已经获取了相应的互斥锁. 通常来说, 使用notifyAll(),并在wait()处设置重复判断是绝大多数的选择.

#### 编程中需要注意的三类问题: 安全性问题, 活跃性问题和性能问题.

##### 安全性问题
安全性问题主要讨论的方向是方法或类是否线程安全. 所谓线程安全就是指程序按照预期的执行. 发生不安全的源头则是前面介绍过的可见性, 原子性和有序性问题.

但是只有一种情况需要具体分析这三个问题. 这种情况是**存在共享数据并且该数据会发生变化,通俗的说就是多个线程会同时读写同一数据.** 如果能够做到不共享数据或者数据状态不发生变化,就能够保证线程的安全性. 有许多方案是针对这个理论, 例如**线程本地存储(Thread Local Storage)**, 不变模式等等.

当**必须共享数据**时,多个线程同时访问同一数据且又一线程要写数据就会引发**数据竞争(Data race)**. 程序的执行结果依赖线程执行的顺序,就是所谓的**竞态条件**. 

竞态条件（Race Condition）：计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件。常见的竞态条件有:
- 先检测后执行
- 两个线程同时修改统一数据

##### 活跃性问题
活跃性问题指的是某个操作无法执行下去. 常见的典型活跃性问题有三类:
- 死锁: 线程互相等待, 表现为线程永久阻塞
- 活锁: 线程并未发生阻塞, 但仍然会存在执行不下去的情况. 通常设置随机等待时间可以解决此类问题
- 饥饿: 线程因无法访问所需资源而无法执行下去的情况. 通常是因为CPU繁忙且线程优先级的差异导致的. 使用公平锁可以极大程度上解决此类问题.

##### 性能问题
我们应尽量使程序并行而提高性能. 有一个阿姆达尔(Amdahl)定律代表处理器并行计算之后效率提升的能力,它可以解决此类问题. 具体公式是:$S=1/(1-p)+\frac{p}{n})$. 在这个公式里, n为CPU的核数, p为并行百分比. 当n无穷大, 并行百分比为95%时, 最高也只能提高20倍的性能.

在方案设计层面,有两个方向可以用来提高性能.
- 第一, 最好的方案是避免用锁, 尽量多使用无锁的算法和数据结构. 这方面的相关技术有线程本地存储**(Thread Local Storage TLS)**, 写入复制(Copy-On-Write), 乐观锁等; Java并发包里的原子类也是无锁的数据结构, Disruptor则是无锁的内存队列.
- 第二, 可以尽量多的减少锁的持有时间, 互斥锁本质上就是将并行的程序串行化.实现这个方案的方法也很多,例如细粒度锁. ConcurrentHashMap它使用了所谓分段锁的技术;还有读写锁等.

在性能方面的度量指标有三个:
- 吞吐量: 指的是在单位时间内能处理请求的数量.
- 延迟: 指的是发出请求到收到响应这个过程的时间.
- 并发量.

#### 管程
管程(Monitor), 在Java中经常称之为监视器,在操作系统中被常称为管程.
> 管程,指管理共享变量以及对共享变量的操作过程,使其支持并发.

##### 管程的发展-Hasen模型, Hoare模型, MESA模型
并发编程领域有两大问题,两大核心问题:**互斥**和**同步**.
- 互斥: 同一时刻只允许一个线程访问共享资源.
- 同步: 线程之间如果通信与协作.

首先关注管程如何解决互斥问题. 其思路是将共享变量及其对共享变量的操作统一封装起来.


其次管程解决同步的方法需引入条件变量,每个条件变量都有一个对应的等待队列. 同时,只有一个线程允许进入管程.

![monitor illustration](/media/posts/monitor.png)

#### 线程
##### 线程的生命周期
通用的线程模型包括五种状态,通常以"五态模型"来描述. 这五态分别是:
- 初始状态: 线程已经被创建,但还不允许分配CPU执行.这个状态只存在于编程语言中, 换言之,这里的线程只是在编程语言层面被创建, 而在操作系统层面,真正的系统还没有创建.
- 可运行状态: 指线程可以分配CPU执行.
- 运行状态: 当空闲CPU分配给一个处于可运行状态的线程,被分配到CPU的线程的状态就转换成了可运行状态.
- 休眠状态: 运行的线程如果调用一个阻塞的API(阻塞的读取文件)或者等待某个事件(条件变量),那么线程状态就会转换到休眠状态,同时释放CPU的使用权. 当等待的时间出现并通知此线程,线程就会从休眠状态转换到可运行状态.
- 终止状态: 线程执行完或出现异常就会进入终止状态,终止状态的线程不会切换到其他任何状态. 这个状态意味着线程的生命周期结束了.

![monitor illustration](/media/posts/threadstate.png)

##### Java中线程的生命周期
Java语言中线程共有六种状态, 分别是
1. NEW (初始化状态)
2. RUNNABLE (可运行/运行状态)
3. BLOCKED (阻塞状态)
4. WAITING (无时限等待)
5. TIMED_WAITING (有时限等待)
6. TERMINATED (终止等待)

BLOCKED, WAITING, TIMED_WAITING均属于线程的休眠状态, 换言之, 只要Java线程处于这三种状态之一, 那么这个线程就永远没有CPU的使用权. 这三种状态的划分是源于导致线程休眠的三个原因
- RUNNABLE -> BLOCKED: 只有一种场景会触发这种转换, 就是线程等待synchronized的隐式锁. synchronized修饰的方法,代码块同一时刻只允许一个线程执行,其他线程只能等待,这种情况下,等待的线程就会从RUNNABLE转换到BLOCK状态.
需要注意的是, 当线程调用阻塞式API时,操作系统层面,线程会转换到休眠状态,但是在JVM层面, Java线程的状态不会发生变化.也就是说Java线程的状态依然会保持RUNNABLE状态. JVM层面不关心操作系统的调度状态因为在JVM看来,等待CPU使用权和等待I/O没有区别, 都是在等待某个系统资源,于是都属于RUNNABLE状态.
- RUNNABLE -> WAITING: 有三种情况会发生这种状态的转化.
  - 第一种是在获得synchronized隐式锁后,调用无参数的wait()方法. 
  - 第二种是调用无参数的Thread.join()方法.其中的join()是一种线程同步方法.例如一个线程对象thread A,当调用A.join().执行这条语句的线程会等待thread A执行完, 而等待中的这个线程,其状态会从RUNNABLE转换到WAITING. 当线程thread A执行完, 原来等待它的线程又会从WAITING变回RUNNABLE. 
  - 第三种情况是调用LockSupport.park()方法.Java并发包中的锁都是基于其实现的.调用LockSupport.park()方法,当前线程会阻塞,线程的状态会从RUNNABLE转换到WAITING. 调用LockSupport.unpack(Thread thread)可以唤醒目标线程, 目标线程的状态会从WAITING状态转换到RUNNABLE.
- RUNNABLE -> TIMED_WAITING: 
  - 带超时参数的Thread.sleep(long millis)
  - 获得synchronized隐式锁的线程, 调用wait(long timeout)
  - 调用带超时参数的Thread.join()
  - 调用带超时参数的LockSupport.parkNanos(Object blocker, long deadline)方法
  - 调用带超时参数的LockSupport.parkUntil(long deadline)
- NEW -> RUNNABLE: 可以分为NEW线程的部分和NEW->RUNNABLE部分.
  - NEW线程部分可以通过继承Thread对象或者实现Runnable接口.
  - NEW->RUNNABLE只能通过myThread.start()方法.
- RUNNABLE -> TERMINATED: 当线程执行完run()方法之后,线程的状态转化为TERMINATED,或者当执行run()方法时有异常抛出. 当我们想主动结束线程时,建议使用interrupt()方法. stop()已经被标记为了@Deprecated.

stop()方法和interrupt()方法的区别是stop()方法会直接杀死线程,被杀死的线程不会自动释放ReentrantLock锁,致使其他线程也无法获取到相应的锁.这样的方法因为太危险而被不建议使用,类似的还有suspend()和resume()方法.

interrupt()方法只是通知线程,线程可以选择忽略或者继续执行一些收尾操作. 线程由两种方法接收到通知.一种是异常,一种是主动监测
- 异常: 
  - 当线程处于WAITING, TIMED_WAITING状态时,如果其他线程调用线程A的interrupt()方法,会使线程A返回到RUNNABLE的状态, 同时线程A的代码会触发InterruptedException异常. 上面提到过的wait(), join(), sleep()等方法的签名都有throws InterruptedException这个异常. 这个异常的触发条件是: 其他线程调用了该线程的interrupt()方法.
  - 当线程处于RUNNABLE状态时,并且阻塞在java.nio.channels.InterruptibleChannel上时, 如果其他线程调用线程A的interrupt()方法, 线程A会触发java.nio.channels.ClosedByInterruptException这个异常.
  - 当线程处于RUNNABLE状态时,并且阻塞在java.nio.channels.Selector上时,如果其他线程调用线程A的interrupt()方法,线程A的java.nio.channels.Selector会立即返回.
- 主动监测: 如果线程处于RUNNABLE状态,并且没有阻塞在某个I/O操作上,这时依赖线程A主动监测中断状态.如果其他线程调用线程的interrupt()方法,那么线程可以通过isInterrupted()方法,监测是不是自己被中断了.

#### 并发量
在并发编程领域,提升性能本质上是提升硬件的利用率, 提升I/O的利用率和CPU的利用率. 最佳的线程数量的决定可以分为CPU密集型计算场景和I/O密集型计算场景.
- CPU密集型计算场景: 理论上"线程的数量=CPU核数"是最适合的,但在工程上,一般会设置为#CPU+1,因为偶尔当内存失效或其他原因导致阻塞的时候,额外的线程可以顶上来保证CPU的利用率.
- I/O密集型计算场景: 最佳的线程数量与程序中CPU计算和I/O操作的耗时比相关的.
 > 最佳线程数 = 1+ (I/O耗时/CPU耗时)

#### 局部变量 -- 线程封闭
方法里的局部变量, 因为不会和其他线程共享, 所以没有并发问题. 这是一个解决并发问题的重要思路和技术,称之为**线程封闭**.
> 线程封闭: 仅在单线程内访问数据.

采用线程封闭的案例非常多,例如在数据库连接池里获取的连接Connection, 在JDBC里没有要求这个Connection必须是线程安全的. 数据库连接池通过线程封闭技术,保证一个Connection一旦被一个线程获取之后,在这个线程关闭Connection之前的这段时间里, 不会再分配给其他线程,从而保证了Connection不会有并发问题.

### Java.util.Concurrent JUC并发包详解
#### Lock和Condition
JUC通过lock和condition这两个接口来重新实现管程, 其中Lock用于解决互斥问题,Condition用于解决同步问题. 
##### 重造管程而不使用自带的synchronized的理由
- 能够相应中断. synchronized的问题是, 持有锁A后,如果尝试获取锁B失败, 那么线程就进入阻塞状态.一旦发生死锁, 就没有任何机会来唤醒阻塞的线程. 我们的期望是如果处于阻塞状态的线程能够相应中断信号, 换言之当我们给阻塞的线程发送中断信号时,线程能够被唤醒,那它就有机会释放锁A,也就破坏了不可抢占的条件.
- 支持超时. 如果线程在一段时间内都没有获取到锁, 不是进入阻塞状态而是返回一个错误,那这个线程也有机会释放曾经的锁.
- 非租塞的获取锁. 如果尝试获取锁失败可以立即返回.

这三个理由构成了API上就是Lock接口的三个方法.
```Java
// 支持中断的API
void lockInterruptibly() throws InterruptedException;
// 支持超时的API
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
// 支持非阻塞获取锁的API
boolean tryLock();
```
JUC里用锁的经典范例就是try{}finally{}
##### 如何保证可见性
```Java
class X {
    private final Lock rtl = new ReentrantLock();
    int value;
    public void addOne() {
        //获取锁
        rtl.lock();
        try {
            value += 1;
        } finally {
            //保证锁能够释放
            rtl.unlock();
        }
    }
}
```
之前介绍的sychronized之所以能够保证可见性是因为有一条Happens-before规则,即synchronized的解锁Happens-before于后续对这个锁的加锁. 而在JUC中,实现可见性利用了volatile相关的Happens-before规则. 例如ReentrantLock, 内部持有一个volatile成员变量state,获取锁的时候,会读写state的值,释放锁的时候也会读写state的值.也就是说,在执行value+=1之前,程序需要先读写一次Volatile变量state, 在执行value+=1之后,又读写了一次volatile变量state.根据相关的Happens-before规则:
1. 顺序规则: 对于线程T1, value+=1 Happens-before 释放锁的操作unlock();
2. volatile变量规则: 由于state=1 会先读取state, 所以线程T1的unlock()操作 Happens-Before 线程T2的lock()操作;
3. 传递性规则: 线程T1的value+=1 Happens-before 线程T2的lock()操作

```Java
class SamleLock {
    volatile int state;
    //加锁
    lock(){
        //省略代码...
        state = 1;
    }
    //解锁
    unlock(){
        //省略代码...
        state = 0;
    }
}
``` 
##### 可重入锁
所谓可重入锁,指的是线程可以重复获取同一把锁
```Java
class X {
  private final Lock rtl =
  new ReentrantLock();
  int value;
  public int get() {
    // 获取锁
    rtl.lock();         ②
    try {
      return value;
    } finally {
      // 保证锁能释放
      rtl.unlock();
    }
  }
  public void addOne() {
    // 获取锁
    rtl.lock();  
    try {
      value = 1 + get(); ①
    } finally {
      // 保证锁能释放
      rtl.unlock();
    }
  }
}
```
##### 公平与非公平锁
ReentrantLock的构造器支持传递一个fair参数,fair代表锁的公平策略. 默认fair = false.

- 当fair == true, 然后有线程释放锁的时候,从等待队列中唤醒一个等待线程时,谁等待的时间长就唤醒谁.
- 当fair == false, 等待的时间并不决定唤醒线程的次序.

**锁的实现建议**
锁的最佳实践可以分为三个方面

- 永远只在更新对象的成员变量时加锁
- 永远只在访问可变的成员变量时加锁
- 永远不在调用其他对象的方法时加锁