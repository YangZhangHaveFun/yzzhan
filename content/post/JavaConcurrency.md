---
title : "深入理解java并发知识的归纳"
date: 2019-05-10T01:37:56+08:00
lastmod: 2019-04-14T01:37:56+08:00
draft: false
tags: ["Java", "Concurrency"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---
### 多线程
在Java中创建线程的方式有且仅有一种就是Thread myThread = new Thread();

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

每个互斥锁都有自己的**等待队列**,这个等待和互斥锁是一对一的关系. 在









