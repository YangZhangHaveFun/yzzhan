---
title : "深入理解java并发知识的归纳"
date: 2019-06-02T01:37:56+08:00
lastmod: 2019-04-14T01:37:56+08:00
draft: false
tags: ["Java", "Concurrency"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---

<!-- TOC -->

- [1. 多线程基础](#1-多线程基础)
    - [1.1. 多线程的三个问题](#11-多线程的三个问题)
    - [1.2. Java内存模型](#12-java内存模型)
        - [1.2.1. Volatile](#121-volatile)
        - [1.2.2. Happen-Before规则](#122-happen-before规则)
    - [1.3. 如果解决并发原子性问题---互斥锁](#13-如果解决并发原子性问题---互斥锁)
        - [1.3.1. 死锁](#131-死锁)
    - [1.4. "等待-通知"机制](#14-等待-通知机制)
    - [1.5. 编程中需要注意的三类问题: 安全性问题, 活跃性问题和性能问题.](#15-编程中需要注意的三类问题-安全性问题-活跃性问题和性能问题)
        - [1.5.1. 安全性问题](#151-安全性问题)
        - [1.5.2. 活跃性问题](#152-活跃性问题)
        - [1.5.3. 性能问题](#153-性能问题)
    - [1.6. 管程](#16-管程)
        - [1.6.1. 管程的发展-Hasen模型, Hoare模型, MESA模型](#161-管程的发展-hasen模型-hoare模型-mesa模型)
    - [1.7. 线程](#17-线程)
        - [1.7.1. 线程的生命周期](#171-线程的生命周期)
        - [1.7.2. Java中线程的生命周期](#172-java中线程的生命周期)
    - [1.8. 并发量](#18-并发量)
    - [1.9. 局部变量 -- 线程封闭](#19-局部变量----线程封闭)
- [2. Java.util.Concurrent JUC并发包详解](#2-javautilconcurrent-juc并发包详解)
    - [2.1. Lock接口](#21-lock接口)
        - [2.1.1. 重造管程而不使用自带的synchronized的理由](#211-重造管程而不使用自带的synchronized的理由)
        - [2.1.2. 如何保证可见性](#212-如何保证可见性)
        - [2.1.3. 可重入锁](#213-可重入锁)
        - [2.1.4. 公平与非公平锁](#214-公平与非公平锁)
    - [2.2. Condition接口](#22-condition接口)
        - [2.2.1. Dubbo源码分析](#221-dubbo源码分析)
    - [2.3. 信号量模型](#23-信号量模型)
    - [2.4. ReadWriteLock 实现](#24-readwritelock-实现)
        - [2.4.1. 读写锁的升级与降级](#241-读写锁的升级与降级)
    - [2.5. StampedLock](#25-stampedlock)
        - [2.5.1. StampedLock使用注意事项](#251-stampedlock使用注意事项)
    - [2.6. CountDownLatch和CyclicBarrier](#26-countdownlatch和cyclicbarrier)
    - [2.7. 并发容器](#27-并发容器)
    - [2.8. 原子类](#28-原子类)
        - [2.8.1. 原理](#281-原理)
        - [2.8.2. ABA问题](#282-aba问题)
        - [2.8.3. Java实现原子化的count += 1](#283-java实现原子化的count--1)
        - [2.8.4. 原子类概览](#284-原子类概览)
    - [2.9. 线程池Executor](#29-线程池executor)
        - [2.9.1. 使用线程池的注意事项](#291-使用线程池的注意事项)
    - [2.10. Future](#210-future)
        - [2.10.1. FutureTask工具类](#2101-futuretask工具类)
    - [2.11. CompletableFuture](#211-completablefuture)
        - [2.11.1. 创建CompletableFuture对象](#2111-创建completablefuture对象)
        - [2.11.2. 如何理解CompletionStage接口](#2112-如何理解completionstage接口)

<!-- /TOC -->

## 1. 多线程基础

### 1.1. 多线程的三个问题
- 多核和缓存导致的可见性问题: 一个线程对共享变量的修改，另外一个线程能够立刻看到，我们称为**可见性**.
- 线程切换带来的原子性问题: 我们把一个或者多个操作在 CPU 执行的过程中不被中断的特性**原子性**.
- 编译优化带来的有序性问题: 代码按照预期的顺序执行,称为**有序性**

### 1.2. Java内存模型
Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括 **volatile**, **synchronized**和**final**关键字, 以及六项**Happen-Before**规则.

#### 1.2.1. Volatile
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

#### 1.2.2. Happen-Before规则
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

### 1.3. 如果解决并发原子性问题---互斥锁
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
#### 1.3.1. 死锁
死锁指一组互相竞争资源的线程因互相等待,导致"永久"阻塞的现象. 触发死锁必须满足四个条件
- 互斥, 共享资源X和Y只能被一个线程占用.
- 占有且等待, 线程T已经取得共享资源X,在等待共享资源Y的时候,不释放共享资源X.
- 不可抢占,其他线程不能强行抢占线程T1占用的资源.
- 循环等待,线程T1等待线程T2占用的资源,线程T2等待线程T1占有的资源,就是循环等待.

破坏其中一条就可以避免死锁的发生. 一般较为实用的方法是破坏第二和第四条,第二条可以通过设置timeout或者循环等待所有条件就绪再占有来预防,第四条可以通过序列化共享资源的占用来避免.

### 1.4. "等待-通知"机制
在并发冲突较小时,循环等待所有条件尚可行.但当冲突量增大时,这种自旋的方式会白白浪费CPU. 另外一种可行的方式当线程要求不满足时,线程阻塞自己进入阻塞状态. 当线程线程的要求被满足时,别的线程可以通知线程继续执行.

在Java语言里, "等待-通知"机制可以由Java语言内置的synchronized配合wait(), notify(), notifyAll()来实现.

每个互斥锁都有自己的**等待队列**,这个等待和互斥锁是一对一的关系. 在并发程序中,当一个线程进入临界区后,由于某个条件不满足,需要进入等待状态, Java对象的wait()方法就需要被调用. 当调用wait()方法后, 当前线程就会被阻塞并进入到**等待队列**.这个等待队列也是互斥锁的等待队列. 在线程进入**等待队列**的同时,会释放持有的互斥锁. 随后其他线程就有机会获得锁并进入临界区.

当条件满足时,可以通过notifyAll()方法, 这个方法会通知等待队列(**对应互斥锁的等待队列**)中的线程,告诉它**条件曾经满足过**. 但被通知的线程想要重新被执行,仍然需要获取到互斥锁.
> 需要注意的是,wait(), notify(), notifyAll()这三个方法能够被调用的前提是已经获取了相应的互斥锁. 通常来说, 使用notifyAll(),并在wait()处设置重复判断是绝大多数的选择.

### 1.5. 编程中需要注意的三类问题: 安全性问题, 活跃性问题和性能问题.

#### 1.5.1. 安全性问题
安全性问题主要讨论的方向是方法或类是否线程安全. 所谓线程安全就是指程序按照预期的执行. 发生不安全的源头则是前面介绍过的可见性, 原子性和有序性问题.

但是只有一种情况需要具体分析这三个问题. 这种情况是**存在共享数据并且该数据会发生变化,通俗的说就是多个线程会同时读写同一数据.** 如果能够做到不共享数据或者数据状态不发生变化,就能够保证线程的安全性. 有许多方案是针对这个理论, 例如**线程本地存储(Thread Local Storage)**, 不变模式等等.

当**必须共享数据**时,多个线程同时访问同一数据且又一线程要写数据就会引发**数据竞争(Data race)**. 程序的执行结果依赖线程执行的顺序,就是所谓的**竞态条件**. 

竞态条件（Race Condition）：计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件。常见的竞态条件有:
- 先检测后执行
- 两个线程同时修改统一数据

#### 1.5.2. 活跃性问题
活跃性问题指的是某个操作无法执行下去. 常见的典型活跃性问题有三类:
- 死锁: 线程互相等待, 表现为线程永久阻塞
- 活锁: 线程并未发生阻塞, 但仍然会存在执行不下去的情况. 通常设置随机等待时间可以解决此类问题
- 饥饿: 线程因无法访问所需资源而无法执行下去的情况. 通常是因为CPU繁忙且线程优先级的差异导致的. 使用公平锁可以极大程度上解决此类问题.

#### 1.5.3. 性能问题
我们应尽量使程序并行而提高性能. 有一个阿姆达尔(Amdahl)定律代表处理器并行计算之后效率提升的能力,它可以解决此类问题. 具体公式是:$S=1/(1-p)+\frac{p}{n})$. 在这个公式里, n为CPU的核数, p为并行百分比. 当n无穷大, 并行百分比为95%时, 最高也只能提高20倍的性能.

在方案设计层面,有两个方向可以用来提高性能.
- 第一, 最好的方案是避免用锁, 尽量多使用无锁的算法和数据结构. 这方面的相关技术有线程本地存储**(Thread Local Storage TLS)**, 写入复制(Copy-On-Write), 乐观锁等; Java并发包里的原子类也是无锁的数据结构, Disruptor则是无锁的内存队列.
- 第二, 可以尽量多的减少锁的持有时间, 互斥锁本质上就是将并行的程序串行化.实现这个方案的方法也很多,例如细粒度锁. ConcurrentHashMap它使用了所谓分段锁的技术;还有读写锁等.

在性能方面的度量指标有三个:
- 吞吐量: 指的是在单位时间内能处理请求的数量.
- 延迟: 指的是发出请求到收到响应这个过程的时间.
- 并发量.

### 1.6. 管程
管程(Monitor), 在Java中经常称之为监视器,在操作系统中被常称为管程.
> 管程,指管理共享变量以及对共享变量的操作过程,使其支持并发.

#### 1.6.1. 管程的发展-Hasen模型, Hoare模型, MESA模型
并发编程领域有两大问题,两大核心问题:**互斥**和**同步**.
- 互斥: 同一时刻只允许一个线程访问共享资源.
- 同步: 线程之间如果通信与协作.

首先关注管程如何解决互斥问题. 其思路是将共享变量及其对共享变量的操作统一封装起来.


其次管程解决同步的方法需引入条件变量,每个条件变量都有一个对应的等待队列. 同时,只有一个线程允许进入管程.

![monitor illustration](/media/posts/monitor.png)

### 1.7. 线程
#### 1.7.1. 线程的生命周期
通用的线程模型包括五种状态,通常以"五态模型"来描述. 这五态分别是:
- 初始状态: 线程已经被创建,但还不允许分配CPU执行.这个状态只存在于编程语言中, 换言之,这里的线程只是在编程语言层面被创建, 而在操作系统层面,真正的系统还没有创建.
- 可运行状态: 指线程可以分配CPU执行.
- 运行状态: 当空闲CPU分配给一个处于可运行状态的线程,被分配到CPU的线程的状态就转换成了可运行状态.
- 休眠状态: 运行的线程如果调用一个阻塞的API(阻塞的读取文件)或者等待某个事件(条件变量),那么线程状态就会转换到休眠状态,同时释放CPU的使用权. 当等待的时间出现并通知此线程,线程就会从休眠状态转换到可运行状态.
- 终止状态: 线程执行完或出现异常就会进入终止状态,终止状态的线程不会切换到其他任何状态. 这个状态意味着线程的生命周期结束了.

![monitor illustration](/media/posts/threadstate.png)

#### 1.7.2. Java中线程的生命周期
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

### 1.8. 并发量
在并发编程领域,提升性能本质上是提升硬件的利用率, 提升I/O的利用率和CPU的利用率. 最佳的线程数量的决定可以分为CPU密集型计算场景和I/O密集型计算场景.
- CPU密集型计算场景: 理论上"线程的数量=CPU核数"是最适合的,但在工程上,一般会设置为#CPU+1,因为偶尔当内存失效或其他原因导致阻塞的时候,额外的线程可以顶上来保证CPU的利用率.
- I/O密集型计算场景: 最佳的线程数量与程序中CPU计算和I/O操作的耗时比相关的.
 > 最佳线程数 = 1+ (I/O耗时/CPU耗时)

### 1.9. 局部变量 -- 线程封闭
方法里的局部变量, 因为不会和其他线程共享, 所以没有并发问题. 这是一个解决并发问题的重要思路和技术,称之为**线程封闭**.
> 线程封闭: 仅在单线程内访问数据.

采用线程封闭的案例非常多,例如在数据库连接池里获取的连接Connection, 在JDBC里没有要求这个Connection必须是线程安全的. 数据库连接池通过线程封闭技术,保证一个Connection一旦被一个线程获取之后,在这个线程关闭Connection之前的这段时间里, 不会再分配给其他线程,从而保证了Connection不会有并发问题.

## 2. Java.util.Concurrent JUC并发包详解
### 2.1. Lock接口
JUC通过lock和condition这两个接口来重新实现管程, 其中Lock用于解决互斥问题,Condition用于解决同步问题. 
#### 2.1.1. 重造管程而不使用自带的synchronized的理由
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
#### 2.1.2. 如何保证可见性
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
#### 2.1.3. 可重入锁
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
#### 2.1.4. 公平与非公平锁
ReentrantLock的构造器支持传递一个fair参数,fair代表锁的公平策略. 默认fair = false.

- 当fair == true, 然后有线程释放锁的时候,从等待队列中唤醒一个等待线程时,谁等待的时间长就唤醒谁.
- 当fair == false, 等待的时间并不决定唤醒线程的次序.

**锁的实现建议**
锁的最佳实践可以分为三个方面

- 永远只在更新对象的成员变量时加锁
- 永远只在访问可变的成员变量时加锁
- 永远不在调用其他对象的方法时加锁

### 2.2. Condition接口
JUC的Condition实现了管程模型里面的条件变量.Java内置的管程里只有一个条件变量, 而Lock&Condition实现的管程是支持多个条件变量.

在很多并发场景下, 支持多个条件变量能够让我们的并发程序的可读性更好, 实现也更加容易. 例如,实现一个阻塞队列就需要两个条件变量, 队列不空和队列不满.
```Java
public class BlockedQueue<T>{
  final Lock lock =
    new ReentrantLock();
  // 条件变量：队列不满  
  final Condition notFull =
    lock.newCondition();
  // 条件变量：队列不空  
  final Condition notEmpty =
    lock.newCondition();

  // 入队
  void enq(T x) {
    lock.lock();
    try {
      while (队列已满){
        // 等待队列不满
        notFull.await();
      }  
      // 省略入队操作...
      // 入队后, 通知可出队
      notEmpty.signal();
    }finally {
      lock.unlock();
    }
  }
  // 出队
  void deq(){
    lock.lock();
    try {
      while (队列已空){
        // 等待队列不空
        notEmpty.await();
      }  
      // 省略出队操作...
      // 出队后，通知可入队
      notFull.signal();
    }finally {
      lock.unlock();
    }  
  }
}
```
#### 2.2.1. Dubbo源码分析
**同步与异步**:当被调用方执行后返回结果,则为同步.当被调用方立即返回结果时则为异步.
![monitor illustration](/media/posts/async.png)


在Java中,默认的代码方式就是同步,如果要异步可以用两种方式来实现:
1. 调用方创建子线程, 在子线程中执行方法调用,这种调用称之为异步调用.
2. 当方法实现的时候,创建一个新的线程执行主要的逻辑,主线程直接return,这种方法我们一般称之为异步方法.

TCP协议本身就是非阻塞的. 在TCP协议层面, 发送完RPC请求之后,线程不会等待RPC的响应结果. 但日常使用时RPC调用大多数都是同步的, 这里以Dubbo为例, 展示如何从异步转化为同步.

DefaultFuture类实现了当RPC返回结果之前,阻塞调用线程,让调用线程等待;当RPC返回结果后,唤醒线程,让调用线程重新执行.
```Java
// 创建锁与条件变量
private final Lock lock 
    = new ReentrantLock();
private final Condition done 
    = lock.newCondition();

// 调用方通过该方法等待结果
Object get(int timeout){
  long start = System.nanoTime();
  lock.lock();
  try {
	while (!isDone()) {
	  done.await(timeout);
      long cur=System.nanoTime();
	  if (isDone() || 
          cur-start > timeout){
	    break;
	  }
	}
  } finally {
	lock.unlock();
  }
  if (!isDone()) {
	throw new TimeoutException();
  }
  return returnFromResponse();
}
// RPC 结果是否已经返回
boolean isDone() {
  return response != null;
}
// RPC 结果返回时调用该方法   
private void doReceived(Response res) {
  lock.lock();
  try {
    response = res;
    if (done != null) {
      done.signal();
    }
  } finally {
    lock.unlock();
  }
}
```

### 2.3. 信号量模型

信号量的模型有一个计数器,一个等待队列和三个方法. 信号量模型中, 计数器和等待队列是透明的, 所以只能通过信号量模型提供的三个方法来访问它们.
- init(): 设置计数器的初始值
- down(): 计数器的值减1; 如果此时计数器的值小于0, 则当前线程将被阻塞, 否则当前线程可以继续执行.
- up(): 计数器的值加1; 如果此时计数器的值小于或者等于0, 则唤醒等待队列的一个线程,并将其从等待队列中移除.

这三个方法都具有原子性. 信号量模型是由J.U.C.Semaphore实现的.
```Java
class Semaphore{
  // 计数器
  int count;
  // 等待队列
  Queue queue;
  // 初始化操作
  Semaphore(int c){
    this.count=c;
  }
  // 
  void down(){
    this.count--;
    if(this.count<0){
      // 将当前线程插入等待队列
      // 阻塞当前线程
    }
  }
  void up(){
    this.count++;
    if(this.count<=0) {
      // 移除等待队列中的某个线程 T
      // 唤醒线程 T
    }
  }
}
```
在JUC中, up()和down()方法对应的是acquire()和release()方法.

信号量相较于锁和条件变量, 最突出的优点是可以控制多个线程进入同一个临界区. 实际的例子就是限流器, 例如连接池,对象池,线程池. 这里的例子是对象池.
```Java
class ObjPool<T, R> {
  final List<T> pool;
  // 用信号量实现限流器
  final Semaphore sem;
  // 构造函数
  ObjPool(int size, T t){
    pool = new Vector<T>(){};
    for(int i=0; i<size; i++){
      pool.add(t);
    }
    sem = new Semaphore(size);
  }
  // 利用对象池的对象，调用 func
  R exec(Function<T,R> func) {
    T t = null;
    sem.acquire();
    try {
      t = pool.remove(0);
      return func.apply(t);
    } finally {
      pool.add(t);
      sem.release();
    }
  }
}
// 创建对象池
ObjPool<Long, String> pool = 
  new ObjPool<Long, String>(10, 2);
// 通过对象池获取 t，之后执行  
pool.exec(t -> {
    System.out.println(t);
    return t.toString();
});
```
Java并发编程重点在管程模型, 解决了信号量模型的易用性和工程化方面. 

### 2.4. ReadWriteLock 实现

JUC提供很多工具类, 目的就是分场景优化性能, 提供易用性.

一种非常普遍的并发场景, 读多写少的场景. 我们经常使用缓存,例如缓存元数据,缓存基础数据等, 这是一种典型的读多写少的应用场景. 缓存之所以能提升性能, 一个重要的条件就是缓存的数据一定是读多写少的, 例如元数据和基础数据基本不会发生变化,但使用的地方却很多. 针对这种并发场景, JUC包中的ReadWriteLock用来解决此类型的并发问题并极大提高性能.

读写锁的三个基本原则
- 允许多个线程同时读共享变量
- 只允许一个线程写共享变量
- 如果一个线程正在执行写操作, 此时禁止读线程读共享变量``
```Java
class Cache<K,V> {
  final Map<K, V> m =
    new HashMap<>();
  final ReadWriteLock rwl = 
    new ReentrantReadWriteLock();
  final Lock r = rwl.readLock();
  final Lock w = rwl.writeLock();
 
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         ①
    try {
      v = m.get(key); ②
    } finally{
      r.unlock();     ③
    }
    // 缓存中存在，返回
    if(v != null) {   ④
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         ⑤
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); ⑥
      if(v == null){  ⑦
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }
}
```
#### 2.4.1. 读写锁的升级与降级
在ReadWriteLock中,不支持先获得读锁再升级成写锁的功能. 虽然锁的升级是不允许的, 锁的降级却是允许的.同时,只有写锁支持条件变量,读锁是不支持条件变量的.
```Java
class CachedData {
  Object data;
  volatile boolean cacheValid;
  final ReadWriteLock rwl =
    new ReentrantReadWriteLock();
  // 读锁  
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  
  void processCachedData() {
    // 获取读锁
    r.lock();
    if (!cacheValid) {
      // 释放读锁，因为不允许读锁的升级
      r.unlock();
      // 获取写锁
      w.lock();
      try {
        // 再次检查状态  
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // 释放写锁前，降级为读锁
        // 降级是可以的
        r.lock(); ①
      } finally {
        // 释放写锁
        w.unlock(); 
      }
    }
    // 此处仍然持有读锁
    try {use(data);} 
    finally {r.unlock();}
  }
}
```

### 2.5. StampedLock
StampedLock类似于数据库并发模型中的Timestamp模型. StampedLock有三种模式, 写锁, 悲观读锁和乐观读. 其中写锁和悲观读锁与ReadWriteLock相似. 允许多个线程读操作和一个线程写操作, 不同的是StampedLock里的悲观读和写锁加锁成功后,会返回一个stamp. 在解锁的时候,需要传进这个stamp.
```Java
final StampedLock sl = 
  new StampedLock();
  
// 获取 / 释放悲观读锁示意代码
long stamp = sl.readLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockRead(stamp);
}

// 获取 / 释放写锁示意代码
long stamp = sl.writeLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockWrite(stamp);
}
```
乐观读是StampedLock比ReadWriteLock性能高的关键所在. StampedLock支持的乐观读允许在多个线程读数据时一个线程写数据.同时,这个乐观读操作是无锁的.
```Java
class Point {
  private int x, y;
  final StampedLock sl = 
    new StampedLock();
  // 计算到原点的距离  
  int distanceFromOrigin() {
    // 乐观读
    long stamp = 
      sl.tryOptimisticRead();
    // 读入局部变量，
    // 读的过程数据可能被修改
    int curX = x, curY = y;
    // 判断执行读操作期间，
    // 是否存在写操作，如果存在，
    // 则 sl.validate 返回 false
    if (!sl.validate(stamp)){
      // 升级为悲观读锁
      stamp = sl.readLock();
      try {
        curX = x;
        curY = y;
      } finally {
        // 释放悲观读锁
        sl.unlockRead(stamp);
      }
    }
    return Math.sqrt(
      curX * curX + curY * curY);
  }
}
```
#### 2.5.1. StampedLock使用注意事项

- 对于读多写少的场景, StampedLock的性能优于ReadWriteLock
- StampedLock只是ReadWriteLock的子集.
- StampedLock不支持重入
- StampedLock 的悲观读锁、写锁都不支持条件变量
- 如果线程阻塞在 StampedLock 的 readLock()或者WriteLock()上会导致CPU飙升

### 2.6. CountDownLatch和CyclicBarrier
CountDownLatch和CyclicBarrier是Java并发包提供的两个非常易用的线程同步工具类. 当每次循环都生成新线程时,join()可以被用来做为线程同步的方式. 当我们用线程池不新建线程时, CountDownLatch和CyclicBarrier来使线程同步.
```Java
// 创建 2 个线程的线程池
Executor executor = 
  Executors.newFixedThreadPool(2);
while(存在未对账订单){
  // 计数器初始化为 2
  CountDownLatch latch = 
    new CountDownLatch(2);
  // 查询未对账订单
  executor.execute(()-> {
    pos = getPOrders();
    latch.countDown();
  });
  // 查询派送单
  executor.execute(()-> {
    dos = getDOrders();
    latch.countDown();
  });
  
  // 等待两个查询操作结束
  latch.await();
  
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}

``` 
```Java
// 订单队列
Vector<P> pos;
// 派送单队列
Vector<D> dos;
// 执行回调的线程池 
Executor executor = 
  Executors.newFixedThreadPool(1);
final CyclicBarrier barrier =
  new CyclicBarrier(2, ()->{
    executor.execute(()->check());
  });
  
void check(){
  P p = pos.remove(0);
  D d = dos.remove(0);
  // 执行对账操作
  diff = check(p, d);
  // 差异写入差异库
  save(diff);
}
  
void checkAll(){
  // 循环查询订单库
  Thread T1 = new Thread(()->{
    while(存在未对账订单){
      // 查询订单库
      pos.add(getPOrders());
      // 等待
      barrier.await();
    }
  });
  T1.start();  
  // 循环查询运单库
  Thread T2 = new Thread(()->{
    while(存在未对账订单){
      // 查询运单库
      dos.add(getDOrders());
      // 等待
      barrier.await();
    }
  });
  T2.start();
}
```

- CountDownLatch主要用来解决一个线程等待多个线程的场景.
- CyclicBarrier是一组线程之间互相等待.

CountDownLatch不能循环利用,一旦计数器减到0, 后面的线程调用await()会直接通过. 但CyclicBarrier的计数器是可以循环使用的, 而且具备自动重置的功能.


### 2.7. 并发容器
Java的容器可以分为四大类: List, Map, Set和Queue. 

- 把线程不安全的容器封装在安全的容器对象内部, 并控制好访问路径是最简单的构造线程安全容器的方法. 但潜在的组合操作导致竞态条件问题并没有解决. 例如SafeArrayList方法.
```Java
SafeArrayList<T>{
  // 封装 ArrayList
  List<T> c = new ArrayList<>();
  // 控制访问路径
  synchronized
  T get(int idx){
    return c.get(idx);
  }

  synchronized
  void add(int idx, T t) {
    c.add(idx, t);
  }

  synchronized
  boolean addIfNotExist(T t){
    if(!c.contains(t)) {
      c.add(t);
      return true;
    }
    return false;
  }
}
```
对于addIfNotExist()方法, 存在组合操作的竞态条件问题. 一个容易被忽略的问题是
迭代器遍历容器导致并发问题.
```Java
List list = Collections.
  synchronizedList(new ArrayList());
synchronized (list) {  
  Iterator i = list.iterator(); 
  while (i.hasNext())
    foo(i.next());
}    
```
Java1.5版本之前所谓的线程安全的容器, 主要指的就是**同步容器**, 但是同步容器的问题就是性能差, 所有的方法都用synchronized来保证互斥,这种串行度太高.

Java1.5版本之后,提供了性能更高的容器,一般为**并发容器**.
![Concurrent Container](/media/posts/conc.png)

- List: List实现类只有CopyOnWriteList. CopyOnWriteList内部维护了一个数组, 成员变量array就指向这个内部数组,所有的读操作都是基于array进行的. 迭代器Iterator遍历的就是array数组. 迭代器Iterator遍历的就是array数组. 当在遍历array的同时,还有一个写操作,例如增加元素, CopyOnWriteList会将array复制一份, 然后在新复制处理的数据上执行增加元素的操作,执行完之后再将array指向这个新的数组. 因此读写操作是可以并行的, 遍历操作一直都是基于原array执行的, 而写操作则是基于新array执行的.
  - 在应用场景中,CopyOnWriteList仅适用于写操作非常少的场景, 而且能够容忍读写的短暂不一致.
  - CopyOnWriteList迭代器是只读的, 不支持增删改.
- Map: Map接口的两个实现是ConcurrentHashMap和ConcurrentSkipMap, 它们从应用的角度看,主要的区别是ConcurrentHashMap的key是无序的,而ConcurrentSkipMap的key是有序的.
![Maps Comparison Table](/media/posts/cmap.PNG)
- Set: Set的两个接口是CopyOnWriteArraySet和ConcurrentListSet, 它们使用的场景类似于CopyOnwriteArrayList和ConcurrentSkipMap.
- Queue: Queue的并发容器相对较为复杂. 从两个维度来看
  - 阻塞与非阻塞: 阻塞指当队列已满时,入队操作阻塞; 队列空时,出队操作阻塞.
  - 单端与双端: 单端只能队尾入队,队首出队; 双端指队首队尾皆可入队出队. JUC里, 阻塞队列用Blocking标识, 单双队列分别用Queue和Deque标识.
    - 单端阻塞队列: ArrayBlockingQueue, LinkedBlockingQueue, SynchronousQueue, LinkedTransferQueue, PriorityBlockingQueue和DelayQueue. 一般来说, 单端阻塞队列中会持有一个队列,可以是数组(ArrayBlockingQueue), 可以是链表(LinkedBlockingQueue), 甚至可以不持有队列(SynchronousQueue), 此时生产者线程的入队操作必须等待消费者线程的出队操作. 而LinkedBlockingQueue融合了LinkedBlockingQueue和 SynchronousQueue的功能,性能更好.PriorityBlockingQueue和DelayQueue支持按照优先级出队, DelayQueue支持延迟出队.
    ![Blocking queue](/media/posts/bqueue.png)
    - 双端阻塞队列: LinkedBlockingDeque
    ![Blocking deque](/media/posts/bdeque.png)
    - 单端非阻塞队列: ConcurrentLinkedQueue
    - 双端非阻塞队列: ConcurrentLinkedDeque
  - 在使用时,需格外注意是否支持有界(容量限制), 数据量大了的时候会引发OOM问题. 只有ArrayBlockingQueue, LinkedBlockingQueue支持有界. 在使用其他无界队列时,一定要充分考虑是否存在导致OOM的隐患.
  - 选对所需要的容器是最关键的
  - Fail-Fast机制

### 2.8. 原子类
之前提到可见性问题一般用volatile解决,原子性问题一般用互斥锁的方式解决. 但对于简单的原子性问题.还有一种方法就是**无锁方案**. JUC提供一系列的原子类, 下面提供一个例子.
```Java
public class Test {
  AtomicLong count = 
    new AtomicLong(0);
  void add10K() {
    int idx = 0;
    while(idx++ < 10000) {
      count.getAndIncrement();
    }
  }
}
```
#### 2.8.1. 原理
原子类通过硬件支持的CAS(Compare and Swap)指令来解决并发问题.CAS指令包含三个参数:共享变量的内存地址A, 用于比较的值B和共享变量的新值C. 只有当内存中地址A处的值等于B时, 才能将内存中地址A处的值更新为新值C. CAS是一条CPU指令,指令本身可以保证其原子性. 这里代码模拟工作原理.
```Java
class SimulatedCAS{
  int count；
  synchronized int cas(int expect, int newValue){
    // 读目前 count 的值
    int curValue = count;
    // 比较目前 count 值是否 == 期望值
    if(curValue == expect){
      // 如果是，则更新 count 的值
      count = newValue;
    }
    // 返回写入前的值
    return curValue;
  }
}
```
使用CAS来解决并发问题, 一般都会伴随着自旋, 下面为演示代码

```Java
class SimulatedCAS{
  volatile int count;
  // 实现 count+=1
  addOne(){
    do {
      newValue = count+1; //①
    }while(count !=
      cas(count,newValue) //②
  }
  // 模拟实现 CAS，仅用来帮助理解
  synchronized int cas(
    int expect, int newValue){
    // 读目前 count 的值
    int curValue = count;
    // 比较目前 count 值是否 == 期望值
    if(curValue == expect){
      // 如果是，则更新 count 的值
      count= newValue;
    }
    // 返回写入前的值
    return curValue;
  }
}
```

#### 2.8.2. ABA问题

简单来说,ABA问题就是一个共享变量A被线程2修改成了B,又被线程3修改成了A, 这样线程1看到的共享变量的值就一直是A. 但实际上,共享变量已经被修改过了. 大多数情况下我们无需关心ABA问题. 但当原子化的更新对象可能就需要关心ABA问题,因为两个A虽然相等, 但是第二个A的属性可能已经发生变化了.

#### 2.8.3. Java实现原子化的count += 1

在Java1.8中,getAndIncrement()方法会转调unsafe.getAndAddLong()方法. this和valueOffset两个参数可以唯一确定共享变量的内存地址.
```Java
final long getAndIncrement() {
  return unsafe.getAndAddLong(
    this, valueOffset, 1L);
}
```
之后是unsafe.getAndAddLong()的源码, 该方法首先会在内存中读取共享变量的值, 之后循环调用compareAndSwapLong()方法尝试设置共享变量的值, 直到成功为止. compareAndSwapLong()是native的方法. 共享变量的值成功更新返回true否则返回false.
```Java
public final long getAndAddLong(
  Object o, long offset, long delta){
  long v;
  do {
    // 读取内存中的值
    v = getLongVolatile(o, offset);
  } while (!compareAndSwapLong(
      o, offset, v, v + delta));
  return v;
}
// 原子性地将变量更新为 x
// 条件是内存中的值等于 expected
// 更新成功则返回 true
native boolean compareAndSwapLong(
  Object o, long offset, 
  long expected,
  long x);
```
compareAndSwapLong的语义和CAS指令的语义差别仅仅是返回值不同而已. 另外getAndAddLong方法的实现基本是CAS使用的典范.
```Java
do {
  // 获取当前值
  oldV = xxxx；
  // 根据当前值计算新值
  newV = ...oldV...
}while(!compareAndSet(oldV,newV);

```
#### 2.8.4. 原子类概览
![Atomic Class](/media/posts/atomic.png)

##### 原子化的基本数据类型

AtomicBoolean、AtomicInteger 和 AtomicLong

```Java
getAndIncrement() // 原子化 i++
getAndDecrement() // 原子化的 i--
incrementAndGet() // 原子化的 ++i
decrementAndGet() // 原子化的 --i
// 当前值 +=delta，返回 += 前的值
getAndAdd(delta) 
// 当前值 +=delta，返回 += 后的值
addAndGet(delta)
//CAS 操作，返回是否成功
compareAndSet(expect, update)
// 以下四个方法
// 新值可以通过传入 func 函数来计算
getAndUpdate(func)
updateAndGet(func)
getAndAccumulate(x,func)
accumulateAndGet(x,func)
```

##### 原子化的对象引用类型
相关实现有 AtomicReference, AtomicStampedReference和AtomicMarkableReference, 利用它们实现对象引用的原子化更新. 后两者是用来解决ABA问题.

##### 原子化数组
相关实现有 AtomicIntegerArray, AtomicLongArray和AtomicReferenceArray, 利用这些原子类, 我们可以原子化地更新数组里面的每一个元素. 这些类提供的方法和原子化地基本数据类型的区别是: **每个方法多了一个数组的索引参数**.

##### 原子化对象属性更新器
相关实现有AtomicIntegerFieldUpdater, AtomicLongFieldUpdater和AtomicReferenceFieldUpdater, 利用它们可以原子化地更新对象的属性, 这三个方法都是利用反射机制实现的, 创建更新器的方法如下
```Java
public static <U>
AtomicXXXFieldUpdater<U> 
newUpdater(Class<U> tclass, 
  String fieldName)
```

##### 原子化的累加器
DoubleAccumulator, DoubleAdder, LongAccumulator和LongAdder, 这四个类仅仅支持执行累加的操作, 相比于原子化的基本数据类型, 速度更快,但不支持compareAndSet()方法. 如果仅仅需要累加操作, 使用原子化的累加器性能更佳.

原子类的方法都是针对一个共享变量的, 如果需要解决多个变量的原子性问题, 建议还是使用互斥锁的方案. 原子类虽然好,但使用需要慎之又慎.

### 2.9. 线程池Executor
- 创建对象, 仅需要在JVM的堆上分配一块内存
- 创建一个线程,却需要调用操作系统内核的API, 然后操作系统要为线程分配一系列的资源.

作为一个重量级的对象,线程应避免频繁创建和销毁. 因此线程池孕育而生. 不同于其他池化资源通过acquire()方法申请资源通过release()方法释放资源. 线程池采用生产-消费者模式. 我们可以根据这个模式组建一个简单的线程池.

```Java
// 简化的线程池，仅用来说明工作原理
class MyThreadPool{
  // 利用阻塞队列实现生产者 - 消费者模式
  BlockingQueue<Runnable> workQueue;
  // 保存内部工作线程
  List<WorkerThread> threads 
    = new ArrayList<>();
  // 构造方法
  MyThreadPool(int poolSize, 
    BlockingQueue<Runnable> workQueue){
    this.workQueue = workQueue;
    // 创建工作线程
    for(int idx=0; idx<poolSize; idx++){
      WorkerThread work = new WorkerThread();
      work.start();
      threads.add(work);
    }
  }
  // 提交任务
  void execute(Runnable command){
    workQueue.put(command);
  }
  // 工作线程负责消费任务，并执行任务
  class WorkerThread extends Thread{
    public void run() {
      // 循环取任务并执行
      while(true){ ①
        Runnable task = workQueue.take();
        task.run();
      } 
    }
  }  
}

/** 下面是使用示例 **/
// 创建有界阻塞队列
BlockingQueue<Runnable> workQueue = 
  new LinkedBlockingQueue<>(2);
// 创建线程池  
MyThreadPool pool = new MyThreadPool(
  10, workQueue);
// 提交任务  
pool.execute(()->{
    System.out.println("hello");
});
```
JUC里的线程池复杂和强大的多, 最核心部分是ThreadPoolExecutor. 最完备的ThreadPoolExecutor构造函数有7个参数
```Java
ThreadPoolExecutor(
  int corePoolSize,
  int maximumPoolSize,
  long keepAliveTime,
  TimeUnit unit,
  BlockingQueue<Runnable> workQueue,
  ThreadFactory threadFactory,
  RejectedExecutionHandler handler) 
```
- corePoolSize: 表示线程池保有的最小线程数.
- maximumPoolSize: 表示线程池创建的最大线程数.
- keepAliveTime & unit: 用来定义线程是否繁忙, 如果一个线程在unit下的keepAliveTime这么长的时间内都处于空闲,并且线程池的数量大于corePoolSize,那么这个线程将被回收.
- workQueue: 工作队列
- threadFactory: 通过这个参数可以自定义如何创建线程,例如给线程命名.
- handler: 自定义任务的拒绝策略,当线程池所有线程都在忙碌且工作队列已满时, 此时提交任务, 线程池会拒绝接收.
  - CallerRunsPolicy: 提交任务的线程自己去执行任务
  - AbortPolicy: 默认拒绝策略, 会throws RejectedExecutionException
  - DiscardPolicy: 直接丢弃任务,没有异常抛出
  - DiscardOldestPolicy: 丢弃最老的任务(最早进入工作队列的任务)

#### 2.9.1. 使用线程池的注意事项
- 不建议使用Executors, 原因是Executors提供的很多方法默认使用都是无界的LinkedBlockingQueue,在高负载情境下, 容易导致OOM, OOM会导致所有请求都无法处理.
- 谨慎使用默认拒绝策略. RejectedExecutionException是运行时异常, 编译器并不强制要求catch, 因此容易被忽略.
### 2.10. Future
ThreadPoolExecutor的void execute(Runnable command) 方法, 利用这个方法虽然可以提交任务, 但是却没有办法获取任务的执行结果.

Java通过ThreadPoolExecutor提供的3个submit()方法和1个FutureTask工具类来支持获得任务执行结果的需求
```Java
// 提交 Runnable 任务
Future<?> submit(Runnable task);
// 提交 Callable 任务
<T> Future<T> submit(Callable<T> task);
// 提交 Runnable 任务及结果引用
<T> Future<T> submit(Runnable task, T result);
```

Future接口有五个方法.
- cancle(): 取消任务
- isCancelled(): 判断任务是否已经取消
- isDone(): 判断任务是否已经结束
- get(): 获取任务执行结果(阻塞)
- get(timeout, unit): 获取任务执行结果,并支持超时机制.

submit()方法之间的区别
- 提交Runnable任务submit(Runnable task): 这个方法的参数是一个RUNNABLE接口, Runnable接口的run()方法是没有返回值的,所以submit(Runnable task)这个方法返回的Future仅可以用来断言任务已经结束了,类似于Thread.join().
- 提交Callable任务submit(Callable<T> task): 这个方法的参数是一个Callale接口, 它只有一个call()方法, 并且这个方法是有返回值的,所以这个方法返回的Future对象可以通过调用其get()方法来获取任务的执行结果.
- 提交Runnable任务及结果引用submit(Runnable task, T result): Runnable接口的实现类Task声明了一个有参构造函数Task(Result r), 创建Task对象的时候传入了result对象, 这样就能在类Task的run()方法中对result进行各种操作. result相当于主线程和子线程之间的桥梁, 通过它主子线程可以共享数据.

```Java 
ExecutorService executor = Executor.newFixedThreadPool(1);
//创建Result对象
Result r = new Result();
r.setAAA(a);
//提交任务
Future<Result> future = executor.submit(new Task(r),r);
Result fr = future.get()

class Task implements Runnable{
    Result r;
    //通过构造函数传入result
    Task(Result r){
        this.r = r;
    }
    void run() {
        //可以操作 result
        a = r.getAAA();
        r.setXXX(x);
    }
}
```

#### 2.10.1. FutureTask工具类

Future前面提到是个接口,而FutureTask是一个工具类, 这个工具类有两个构造函数.
```Java
FutureTask(Callable<V> callable);
FutureTask(Runnable runnable, V result);
```

FutureTask实现了Runnable和Future接口, 由于其实现了Runnable接口, 所以可以将FutureTask对象作为任务提交给ThreadPoolExecutor执行, 也可以直接被Thread执行; 又因为实现了Future接口, 所以也可以用来获得任务的执行结果.

```Java
// 创建 FutureTask
FutureTask<Integer> futureTask
  = new FutureTask<>(()-> 1+2);
// 创建线程池
ExecutorService es = 
  Executors.newCachedThreadPool();
// 提交 FutureTask 
es.submit(futureTask);
// 获取计算结果
Integer result = futureTask.get();
```

**实现"烧水泡茶"程序**

![烧水泡茶](/media/posts/boil.png)

一种方式可以分为两个FutureTask----ft1和ft2, 
- ft1完成洗水壶, 烧开水, 泡茶的任务. 
- ft2完成洗茶壶, 洗茶杯, 拿茶叶的任务. 

需要注意的是ft1在执行泡茶任务前,需要等待ft2完成所有任务, 换言之ft1需要在内部引用ft2,在执行泡茶之前, 调用ft2的get()方法实现等待.
```Java
// 创建任务 T2 的 FutureTask
FutureTask<String> ft2
  = new FutureTask<>(new T2Task());
// 创建任务 T1 的 FutureTask
FutureTask<String> ft1
  = new FutureTask<>(new T1Task(ft2));
// 线程 T1 执行任务 ft1
Thread T1 = new Thread(ft1);
T1.start();
// 线程 T2 执行任务 ft2
Thread T2 = new Thread(ft2);
T2.start();
// 等待线程 T1 执行结果
System.out.println(ft1.get());

// T1Task 需要执行的任务：
// 洗水壶、烧开水、泡茶
class T1Task implements Callable<String>{
  FutureTask<String> ft2;
  // T1 任务需要 T2 任务的 FutureTask
  T1Task(FutureTask<String> ft2){
    this.ft2 = ft2;
  }
  @Override
  String call() throws Exception {
    System.out.println("T1: 洗水壶...");
    TimeUnit.SECONDS.sleep(1);
    
    System.out.println("T1: 烧开水...");
    TimeUnit.SECONDS.sleep(15);
    // 获取 T2 线程的茶叶  
    String tf = ft2.get();
    System.out.println("T1: 拿到茶叶:"+tf);

    System.out.println("T1: 泡茶...");
    return " 上茶:" + tf;
  }
}
// T2Task 需要执行的任务:
// 洗茶壶、洗茶杯、拿茶叶
class T2Task implements Callable<String> {
  @Override
  String call() throws Exception {
    System.out.println("T2: 洗茶壶...");
    TimeUnit.SECONDS.sleep(1);

    System.out.println("T2: 洗茶杯...");
    TimeUnit.SECONDS.sleep(2);

    System.out.println("T2: 拿茶叶...");
    TimeUnit.SECONDS.sleep(1);
    return " 龙井 ";
  }
}
// 一次执行结果：
T1: 洗水壶...
T2: 洗茶壶...
T1: 烧开水...
T2: 洗茶杯...
T2: 拿茶叶...
T1: 拿到茶叶: 龙井
T1: 泡茶...
上茶: 龙井
```

### 2.11. CompletableFuture

CompletableFuture是一个更为复杂的异步化工具类, 同时提供更加直接的功能.

- 无需手动维护线程, 没有繁琐的手动维护线程的工作
- 语义更清晰
- 代码更加简练并且专注于业务逻辑

```Java
// 任务 1：洗水壶 -> 烧开水
CompletableFuture<Void> f1 = 
  CompletableFuture.runAsync(()->{
  System.out.println("T1: 洗水壶...");
  sleep(1, TimeUnit.SECONDS);

  System.out.println("T1: 烧开水...");
  sleep(15, TimeUnit.SECONDS);
});
// 任务 2：洗茶壶 -> 洗茶杯 -> 拿茶叶
CompletableFuture<String> f2 = 
  CompletableFuture.supplyAsync(()->{
  System.out.println("T2: 洗茶壶...");
  sleep(1, TimeUnit.SECONDS);

  System.out.println("T2: 洗茶杯...");
  sleep(2, TimeUnit.SECONDS);

  System.out.println("T2: 拿茶叶...");
  sleep(1, TimeUnit.SECONDS);
  return " 龙井 ";
});
// 任务 3：任务 1 和任务 2 完成后执行：泡茶
CompletableFuture<String> f3 = 
  f1.thenCombine(f2, (__, tf)->{
    System.out.println("T1: 拿到茶叶:" + tf);
    System.out.println("T1: 泡茶...");
    return " 上茶:" + tf;
  });
// 等待任务 3 执行结果
System.out.println(f3.join());

void sleep(int t, TimeUnit u) {
  try {
    u.sleep(t);
  }catch(InterruptedException e){}
}
// 一次执行结果：
T1: 洗水壶...
T2: 洗茶壶...
T1: 烧开水...
T2: 洗茶杯...
T2: 拿茶叶...
T1: 拿到茶叶: 龙井
T1: 泡茶...
上茶: 龙井
```
#### 2.11.1. 创建CompletableFuture对象

```Java
//没有返回值, 默认ForkJoinPool线程池
static CompletableFuture<Void> runAsync(Runnable runnable);
//有返回值, 默认ForkJoinPool线程池
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
//没有返回值, 可以指定线程池.
static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
//有返回值, 可以指定线程池.
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
```
JUC默认使用ForkJoinPool线程池, 这个线程池默认创建的线程数是CPU的核数.

创建完成后, 会自动运行runnable.run()或者supplier.get()方法. 因为CompletableFuture实现了Future接口, 所以关于异步操作什么时候结束和如果获取异步操作的执行结果的问题都可以通过Future接口来解决.

#### 2.11.2. 如何理解CompletionStage接口

首先需要理解的是任务的时序关系.

- 串行关系
- 并行关系
- 汇聚关系

CompletionStage接口可以清晰地描述任务之间的这种时序关系.

- **描述串行关系**: 
有关的方法有thenApply, thenAccept, thenRun和thenCompose这四个系列的接口.
  - then
  



