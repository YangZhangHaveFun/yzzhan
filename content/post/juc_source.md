---
title : "解析J.U.C源码"
date: 2019-06-02T01:37:56+08:00
lastmod: 2019-04-14T01:37:56+08:00
draft: false
tags: ["Java", "Concurrency"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---

### Atomic Classes
Atomic包的实现主要依赖硬件原子指令CAS的支持.例如在getAndIncrement()方法中调用了unsafe.getAndAddInt()方法. 其中调用了native的compareAndSwapInt方法.
```Java
    //例如要在并发环境中对count这个对象执行加一的操作,count原有指为5.
    public final int getAndAddInt(Object var1, long var2, int var4) {
        //var1 指被操作的对象, var2指被操作对象的偏移量, var4值要加的值
        //对应例子var1=count, var2=5, var4=1
        int var5;
        do {
            //通过对象和offset得到内存中这个对象真正的值.
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        当内存中的值等于对应的偏移量(输入的值)时,执行值的交换操作.  
        return var5;
    }
```

### 队列同步器AQS(AbstractQueuedSynchronizer)
AQS是用来构建锁或其他同步组件的基本框架, 其中成员变量有两个

- int型表示同步状态: 果我们使用锁同步共享变量的时候，我们首先应该要知道这个共享变量的状态（是否已经被其他线程锁住等），这也是这个int成员变量的作用；
- 内置的FIFO队列: 第二个就是既然是同步访问共享资源，肯定会有一些线程无法获取到共享资源等待获取锁而进入一个容器中进行保存而这容器就是这个内置的FIFO队列。


子类推荐被定义为自定义同步组件的静态内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用，同步器既可以支持独占式地获取同步状态，也可以支持共享式地获取同步状态，这样就可以方便实现不同类型的同步组（ ReentrantLock、ReentrantReadWriteLock、CountDownLatch 等）。
同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者之间的关系：

- **锁是面向使用者的**，它定义了使用者与锁交互的接口（比如可以允许两个线程并行访问），隐藏了实现细节；
- **同步器面向的是锁的实现者**，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。或者可以把AQS认为是锁的实现者的一个父类。

#### AQS的主要方法
按照可见性和继承性可分成四类:

- public final
- protected final 
- private 
- protected

##### protected
```java
    /**
     * Attempts to acquire in exclusive mode. This method should query
     * if the state of the object permits it to be acquired in the
     * exclusive mode, and if so to acquire it.
     */
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * Attempts to set the state to reflect a release in exclusive
     * mode.
     *
     * <p>This method is always invoked by the thread performing release.
     */
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * Attempts to acquire in shared mode. This method should query if
     * the state of the object permits it to be acquired in the shared
     * mode, and if so to acquire it.
     */
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * Attempts to set the state to reflect a release in shared mode.
     *
     * <p>This method is always invoked by the thread performing release.
     */
    protected boolean tryReleaseShared(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * Returns {@code true} if synchronization is held exclusively with
     * respect to the current (calling) thread.  This method is invoked
     * upon each call to a {@link ConditionObject} method.
     */
    protected boolean isHeldExclusively() {
        throw new UnsupportedOperationException();
    }
```
- protected boolean tryAcquire(int arg): 独占式获取同步状态, 实现该方法需要查询当前状态并判断同步状态是否符合预期, 然后再进行CAS设置同步状态.
- protected boolean tryRelease(int arg): 独占式释放同步状态, 等待获取同步状态的线程将有机会获取同步状态.
- protected int tryAcquireShared(int arg): 共享式获取同步状态, 返回大于等于0的值, 表示获取成功, 反之, 获取失败.
- protected boolean tryReleaseShared(int arg): 共享式释放同步状态
- protected boolean isHeldExclusively(): 当前同步器是否在独占模式下被线程占用, 一般该方法表示是否被当前线程独占.



AbstractQueuedSynchronizer虽然没有抽象方法，但是提供了五个方法可以让我们在子类中重载，并且这五个方法都是空实现直接抛出异常，也就是说我们要使用这五个方法提供的功能，我们必须要自己在子类中进行实现，这也是“模板方法模式”的一种体现和使用。
##### public final
![](/media/posts/publicfinal.png)

除了上述 protected 类别的方法，还有一个关键的类别就是 public final 类别，这是因为，这是我们可以直接使用的方法，称之为“模板方法”，当我们实现自定义的同步组件的时候，我们可以调用这些模板方法获取我们需要的东西。主要有如下方法：

- void acquire(int arg): 独占式获取同步状态, 如果当前线程获取同步状态成功, 则由该方法返回, 否则, 将会进入同步队列等待, 该方法将会调用重写的tryAcquire(int arg)方法
- void acquireInterruptibly(int arg): 与acquire(int arg)相同, 但是该方法响应中断, 当前线程未获取到同步状态而进入同步队列中, 如果当前线程被中断, 则该方法会抛出InterruptedException并返回
- boolean tryAcquireNanos(int arg, long nanos): 在acquireInterruptibly(int arg)基础上增加了超时限制, 如果当前线程未获取到同步状态而进入同步队列中, 如果当前线程被中断, 则该方法会抛出InterruptedException.
- void acquireShared(int arg): 共享式获取同步状态, 如果当前线程未获取到同步状态, 将会进入同步队列等待, 与独占获取的主要区别是在同一时刻可以有多个线程获取同步状态.
- boolean acquireSharedInterruotibly(int arg): 与acquireShared(int arg)相同, 该方法响应中断
- boolean tryAcquireSharedNanos(int arg, long nanos): 在acquireSharedInterruotibly(int arg)基础上增加了超时限制
- boolean release(int arg): 独占式的释放同步状态, 该方法会在释放同步状态之后, 将同步队列中第一个节点包含的线程唤醒
- boolean releaseShared(int arg): 共享式的释放同步状态
- Collection<Thread> getQueuedThreads(): 获取等待在同步队列上的线程集合

同步器提供的上述模板方法基本上分为3类：**独占式获取与释放同步状态**、**共享式获取与释放同步状态**、**查询同步队列中的等待线程情况**。
##### protected final
- getState(): int
- setState(): void
- compareAndSetState(int,int): boolean
- hasWaiters(): boolean
- getWaitQueueLength(): int
- getWaitingThreads(): Collection<Thread>

#### AQS的内部类
- ConditionObject: 这个我们知道在使用synchronized的时候是使用wait和notify进行线程间通信，使用ReentrantLock的时候是使用Condition实现的线程间通信，而这正是 AbstractQueuedSynchronizer帮我们进一步封装的Condition接口.
- Node: 用来实现FIFO队列节点

##### ConditionObject
```java
public class ConditionObject implements Condition, java.io.Serializable {
        private static final long serialVersionUID = 1173984872572414699L;
        /** First node of condition queue. */
        private transient Node firstWaiter;
        /** Last node of condition queue. */
        private transient Node lastWaiter;
```
- Condition接口:
  - await(): void
  - awaitUninterruptibly(): void
  - awaitNanos(long): long
  - awaitUntil(Date): boolean
  - signal(): void
  - signalAll(): void

同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成为一个节点（Node）并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。
###### 同步队列的基本结构
同步队列中的节点（Node）用来保存获取同步状态失败的线程引用、等待状态以及前驱和后继节点，节点的属性类型与名称以及描述如下：

- int waitStatus: 等待状态, 包含如下状态
  - CANCELLED: 值为1, 由于同步队列中等待的线程等待超时或者被中断, 需要从同步队列中取消等待, 姐弟啊年进入该状态将不会变化.
  - SIGNAL: 值为-1, 后续节点的线程处于等待状态, 而当前节点的线程如果释放了同步状态或者被取消, 将会通知后继节点, 使后继节点的线程得以运行.
  - CONDITION: 值为-2, 节点在等待队列中,节点线程等待在Condition上, 当其他线程对Condition调用了signal()方法后, 该节点会从等待队列中转移到同步队列中,加入到对同步状态的获取中
  - PROPAGATE: 值为-3, 表示下一次共享式同步状态获取将会无条件地被传播下去
  - INITIAL: 值为0, 初始状态
- Node prev: 前驱节点, 当节点加入同步队列时被设置(尾部添加)
- Node next: 后继节点
- Node nextWaiter: 等待队列中的后继节点. 如果当前节点是共享的, 那么这个字段僵尸一个SHARED常量, 也就是说节点类型(独占和共享)和等待队列中的后继节点共用一个字段
- Thread thread: 获取同步状态的线程

节点是构成同步队列（等待队列）的基础，同步器拥有首节点（head）和尾节点（tail），没有成功获取同步状态的线程将会成为节点加入该队列的尾部，同步队列的基本结构如下图：
![](/media/posts/aqsstruct.png)

关于Node节点的细节还有很多，最重要的是我们理解他就是实现的是队列同步的存储功能就行，这个存储功能在尾部存放的是需要排队等待的线程，在头部获取的是获取到锁的线程信息

#### 同步状态
```Java
/**
* The synchronization state.
*/
private volatile int state;

/**
    * Returns the current value of synchronization state.
    * This operation has memory semantics of a {@code volatile} read.
    * @return current state value
    */
protected final int getState() {
    return state;
}

/**
    * Sets the value of synchronization state.
    * This operation has memory semantics of a {@code volatile} write.
    * @param newState the new state value
    */
protected final void setState(int newState) {
    state = newState;
}
```
同步状态被设计为是AQS中的一个整形变量，用于表示当前共享资源的锁被线程获取的次数，并且是多线程可见的。

- 如果是独占式的话state的值0表示该共享资源没有被其他线程所锁住可以被使用，其他值表示该锁被当前线程重入的次数；例如下文中的重入锁 ReentrantLock 。
- 如果是共享式，该 state值被分为高16位和低16位，高16位表示读状态，低16位表示写状态，用一个整形维护多种状态。例如： ReentrantReadWriteLock 实现读写锁，用整数state表示读写锁状态

#### 总结
##### aquire的步骤：
1）tryAcquire()尝试获取资源。
2）如果获取失败，则通过addWaiter(Node.EXCLUSIVE), arg)方法把当前线程添加到等待队列队尾，并标记为独占模式。
3）插入等待队列后，并没有放弃获取资源，acquireQueued()自旋尝试获取资源。根据前置节点状态状态判断是否应该继续获取资源。如果前驱是头结点，继续尝试获取资源；
4）在每一次自旋获取资源过程中，失败后调用shouldParkAfterFailedAcquire(Node, Node)检测当前节点是否应该park()。若返回true，则调用parkAndCheckInterrupt()中断当前节点中的线程。若返回false，则接着自旋获取资源。当acquireQueued(Node,int)返回true，则将当前线程中断；false则说明拿到资源了。
5）在进行是否需要挂起的判断中，如果前置节点是SIGNAL状态，就挂起，返回true。如果前置节点状态为CANCELLED，就一直往前找，直到找到最近的一个处于正常等待状态的节点，并排在它后面，返回false，acquireQueed()接着自旋尝试，回到3）。
6）前置节点处于其他状态，利用CAS将前置节点状态置为SIGNAL。当前置节点刚释放资源，状态就不是SIGNAL了，导致失败，返回false。但凡返回false，就导致acquireQueed()接着自旋尝试。
7）最终当tryAcquire(int)返回false，acquireQueued(Node,int)返回true，调用selfInterrupt()，中断当前线程。
##### 内部类ConditionObject
###### await流程
await()：当前线程处于阻塞状态，直到调用signal()或中断才能被唤醒。

1）将当前线程封装成node且等待状态为CONDITION。
2）释放当前线程持有的所有资源，让下一个线程能获取资源。
3）加入到条件队列后，则阻塞当前线程，等待被唤醒。
4）如果是因signal被唤醒，则节点会从条件队列转移到等待队列；如果是因中断被唤醒，则记录中断状态。两种情况都会跳出循环。
5）若是因signal被唤醒，就自旋获取资源；否则处理中断异常。
###### signal流程
1) signal()：唤醒一个被阻塞的线程。
2) doSignal()：将条件队列的头节点从条件队列转移到等待队列，并且，将该节点从条件队列删除。
3) transferForSignal()：将节点放入等待队列并唤醒。并不需要在条件队列中移除，因为条件队列每次插入时都会把状态不为CONDITION的节点清理出去。
### Lock接口
在《第五节：使用Lock对象实现同步以及线程间通信》介绍了如何使用Lock实现和synchronized关键字类似的同步功能，只是Lock在使用时需要显式地获取和释放锁，synchronized实现的隐式的获取所和释放锁。

虽然Lock它缺少了（通过synchronized块或者方法所提供的）隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性. 特性有:

- 尝试非阻塞地获取锁: 当前线程尝试获取锁, 如果这一时刻锁没有被其他线程获取到, 则成功获取并持有锁.
- 能被中断地获取锁: 与synchronized不同, 获取到锁的线程能够响应中断, 当获取到锁的线程被中断时, 中断异常将会被抛出, 同时锁会被释放
- 超时获取锁: 在指定的截止时间之前获取锁, 如果截止时间到了仍旧无法获取锁,则返回

![](/media/posts/lockstruct.png)

#### Lock各接口的含义
- void lock(): 获取锁, 调用该方法当前线程将会获取锁, 当锁获得后, 从该方法返回
- void lockInterruptibly() throws InterruptedException: 可中断地获取锁, 和lock()方法的不同之处在于该方法会响应中断, 即在锁的获取中可以中断当前位置.
- boolean tryLock(): 尝试非阻塞的获取锁, 调用该方法后立即返回, 如果能获取则返回true, 否则false
- boolean tryLock(long time, TimeUnit unit) throws InterruptedException: 超时的获取锁, 当前线程在以下三个情况下会返回:
  - 当前线程在超时时间内获得了锁
  - 当前线程在超时时间内被中断
  - 超时时间结束, 返回false
- void unlock(): 释放锁
- Condition newCondition(): 获取等待通知组件, 该组件和当前的锁绑定, 当前线程只有获取了锁, 才能调用该组件的wait()方法, 而调用后, 当前线程将释放锁.

### ReentrantLock设计与实现
![](/media/posts/reenstruct.png)
可以看出 ReentrantLock 的内部类包含： **Sync**、**NonfairSync（非公平锁）**、**FairSync（公平锁）** 。而**Sync**正是继承**AbstractQueuedSynchronizer** 这个抽象类，而 **NonfairSync** 和 **FairSync** 又是继承了 **Sync** 的两个静态内部类。

因为我们在上述的学习中已经知道了 AbstractQueuedSynchronizer 同步器面向的是锁的实现者，即其内部已经封装了一些关于锁的操作。这也是上文中提到的两句话：

- 同步器的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态；
- 子类推荐被定义为自定义同步组件的静态内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用。

#### Sync内部类
可以看出对于我们上述说的那5个方法，Sync只重写了一个： tryRelease（） ，那么其他的几个方法那？

这里需要注意的是：Sync也是一个abstract类，并且这5个方法并不是一定要在子类中进行重写的， ReentrantLoc k的几个内部类只重写了 tryRelease 和 tryAcquire方法，其他的使用是在 ReentrantReadWriteLock 中用到的，这也是根据具体的ReentrantLock 的实现的实际需求，而其他的方法具体（其实在 ReentrantLock就是指 tryAcquire ）的重写这就需要： NonfairSync 和 FairSync 上场了！

##### FairSync


### Condition接口

#### Condition框架
##### 当ReentrantLock只有一个Condition时
一个Condition对象包含一个等待队列，Condition拥有首节点和尾节点。当前线程调用Condition.await() 方法，将会以当前线程构造节点，并将该节点从尾部加入到等待队列，等待队列的基本结构如下图：
![](/media/posts/onecondition.png)

Condition拥有首尾节点的引用，而新增节点只需要将原有的尾节点nextWaiter指向它，并且更新尾节点即可。上述节点引用更新的过程并没有使用到CAS保证，这是因为当前线程调用await（） 方法的时候必定是获取了锁的线程，也就是说该过程是由锁来保证线程安全的。
##### 当ReentrantLock有多个Condition时
我们知道在使用synchronized的时候，是使用的对象监视器模型的，即在Object的监视器模型上，一个对象拥有一个同步队列和等待队列，而Lock可以拥有一个同步队列和多个等待队列，这是因为通过lock.newCondition() 可以创建多个Condition条件，而这多个Condition对象都是在同一个锁的基础上创建的，在同一时刻也只能由一个线程获取到该锁。
![](/media/posts/syncwait.png)

#### Condition等待实现
当前线程调用`Condition.await()` 方法的时候，相当于将当前线程从同步队列的首节点移动到Condition的等待队列中，并释放锁，同时线程变为等待状态。当前线程加入到等待队列的过程如下：
![](/media/posts/syncwaiter.png)
可以看出同步队列的首节点并不是直接加入到等待队列的尾节点，而是封装成等待队列的节点才插入到等待队列的尾部的。

#### Condition通知实现
调用当前线程的`Condition.signal()` 方法，将会唤醒在等待队列中等待时间最长的节点也就是首节点，在唤醒节点之前，会将该节点移到同步队列中。
![](/media/posts/syncnotify.png)

通过调用同步器的方法将等待队列中的头结点线程安全的移到同步队列的尾节点，当前线程在使用LockSupport唤醒该节点的线程。

被唤醒后的线程，将会从`await()` 方法中的while循环中退出，进而调用同步器的方法加入到获取同步状态的竞争中。

成功获取同步状态之后，被唤醒的线程从先前调用的await饭发个返回，此时该线程已经成功的获取了锁。

Condition的signalAll() 方法，相当于对等待队列中的每一个节点均执行一次signal（） 方法，效果就是将等待队列中的所有节点全部移到同步队列中，并唤醒每个节点的线程。
