---
title : "对深入理解java虚拟机的读书总结"
date: 2019-03-24T01:37:56+08:00
lastmod: 2019-03-25T01:37:56+08:00
draft: false
tags: ["Java", "JVM"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---

# Part 1(Java 概括)
**Java技术体系的组成部分**
- Java 程序设计语言
- 各种硬件平台上的Java虚拟机
- Class文件格式
- Java API 类库
- 来自商业机构和开源社区的第三方Java类库

**JDK(Java Development Kit)**
包括Java 程序设计语言, Java虚拟机, JavaAPI类库这三部分。是用于支持Java程序开发的最小环境

**JRE(Java Runtime Environment)**
包括Java SE API子集和Java虚拟机这两个部分。是支持Java程序运行的标准环境。

**Java的四个平台**
- Java Card: 支持一些Java小程序(Applets) 运行在小内存设备（如智能卡）上的平台
- Java ME(Micro Edition): 支持Java程序运行在移动终端上的平台，是对Java API有所精简并加入了针对移动终端支持，以前称为J2ME。
- Java SE（Standard Edition）: 支持面向桌面级应用。
- Java EE (Enterprise Edition) : 支持使用多层架构的企业应用的Java平台，除了提供Java SE API外，还对其做了大量的扩充和部署支持。

# Part 2（自动内存管理机制）
## Java内存区域与内存溢出异常
### 运行时的数据区域
Java虚拟机在执行Java程序的过程中会把所管理的内存划分为若干个不同的数据区域.
![数据区](/media/posts/dataArea.png)
1. **程序计数器(Program Counter Register)**
    * 当前线程所执行的字节码的行号指示器
    * 字节码解释器工作时通过**程序计数器**来选取下一条需要执行的字节码指令.
    * 每个线程都需要一个独立的程序计数器,各条线程之间的计数器互不影响,独立存储.称这类内存区域为"线程私有"的内存
    * 若线程正在执行java方法,则**程序计数器**记录正在执行的虚拟机字节码指令的地址.若执行的是Native方法,则这个计数器值为空(Undefined).
    * 此内存是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域.
        > Native方法
        > "A native method is a Java method whose implementation is provided by non-java code."
        > 使用Native Method的原因 1.需要与java环境外交互2.需要与操作系统交互

2. **Java虚拟机栈(Java Virtual Machine Stacks)**
    * **Java虚拟机栈**也是线程私有的.
    * **Java虚拟机栈**描述的是Java方法执行的内存模型: 每个方法在执行的同时会创建一个栈帧(Stack Frame)用于存储局部变量表,操作数栈,动态链接,方法出口等信息.每一个方法从调用直到执行完成的过程就对应着一个栈帧在虚拟机栈中入栈到出栈的过程.
    * 局部变量表所需的内存空间在编译时完成分配,当进入一个方法时,这个方法需要在帧中分配多大的局部变量空间是完全确定的
    * 如果线程请求的栈深度大于虚拟机所允许的深度,将抛出StackOverFlowError异常.
    * 虚拟机栈动态扩展的过程中,如果扩展时无法申请到足够的内存,就会抛出OutOfMemoryError异常.

3. **本地方法栈(Native Method Stack)**
    * **本地方法栈**类似于**Java虚拟机栈**. 前者为Native Method服务,后者为Java方法(字节码)服务.

4. **Java堆(Java Heap)**
    * 对大多数应用来说, Java Heap是Java虚拟机所管理的内存中最大的一块, 并被所有线程共有. 在虚拟机启动时就已创建.
    * 此内存区域的唯一目的就是存放对象实例,几乎所有的对象实例都在这里分配内存(Java虚拟机规范: 所有的对象实例以及数组都要在堆上分配.)
    * JIT编译器的发展与逃逸分析技术逐渐成熟, 栈上分配,标量替换优化技术会导致一些微妙的变化.上述第二点描述变得并不绝对.
        > Java程序最初是仅仅通过解释器解释执行的，即对字节码逐条解释执行，这种方式的执行速度相对会比较慢，尤其当某个方法或代码块运行的特别频繁时，这种方式的执行效率就显得很低。于是后来在虚拟机中引入了JIT编译器（即时编译器），当虚拟机发现某个方法或代码块运行特别频繁时，就会把这些代码认定为“Hot Spot Code”（热点代码），为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各层次的优化 (摘自https://blog.csdn.net/ns_code/article/details/18009455)

        > **逃逸分析(Escape Analysis)**的基本行为就是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为方法逃逸。如果能证明一个对象不会逃逸到方法或线程外，则可能为这个变量进行一些高效的优化。如果能够通过逃逸分析确定某些对象不会逃出方法之外，那就可以让这个对象在栈上分配内存，这样该对象所占用的内存空间就可以随栈帧出栈而销毁，就减轻了垃圾回收的压力。在一般应用中，如果不会逃逸的局部对象所占的比例很大，如果能使用栈上分配，那大量的对象就会随着方法的结束而自动销毁了。
    * Java堆是垃圾收集器管理的主要区域. 因此很多时候也被称为GC堆.
    * Java堆可以处于物理上不连续的内存空间中,只要逻辑上连续的即可(//TODO: 是否与虚存相关?)
    * 如果在堆中没有内存完成实例分配, 并且堆也无法再扩展时, 将会抛出OutOfMemoryError.
    
5. **方法区(Method Area)**
   * **方法区**与Java堆一样都是所有线程共享的内存区域.它用于存储已被虚拟机加载的类信息,常量,静态变量,即时编译器编译后的代码等数据.别名Non-Heap.
   * 这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载.
6. **运行时常量池(Runtime Constant Pool)**
   * **运行时常量池**是方法区中的一部分. Class文件中出了有类的版本,字段,方法,借口等描述信息外,还有一项信息是常量池.
7. **直接内存(Direct Memory)**
    * **直接内存**并不是虚拟机运行时数据区的一部分,也不是Java虚拟机规范中定义的内存区域.但是这部分内存也被频繁的使用.而且也可能导致OutOfMemoryError异常出现.
    * JDK1.4中新加入NIO(New Input/Output)类, 引入基于Channel与Buffer的I/O方式.这个类就用到**直接内存**避免数据在Java堆和Native堆中来回复制数据,让其在一些场景中显著提高性能.

### 对象的创建流程(new)
当虚拟机遇到一条new指令时,
1. 检查这个指令的参数是否能在常量池中定位到一个类的符号引用.并检查这个符号引用代表的类是否已经被加载,解析和初始化过.如果没有,那必须先执行相应的类加载过程.

2. 在类加载检查通过后,接下来虚拟机将为新生对象分配内存. 对象所需内存的大小在类加载完成后便可完全确定,为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来.
    * **指针碰撞(Bump the Pointer)**把分界点的指针向空闲空间那边挪动一段与对象大小相等的距离        ---Serial, ParNew
    * **空闲列表(Free List)** 更新记录可用内存块的列表  ---CMS

3. 内存分配完成后,虚拟机需要将分配到的内存空间都初始化为零值(不包括对象头).这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用,程序能访问到这些字段的数据类型所对应的零值.
4. 虚拟机对对象进行一些必要的设置
   * 这个对象是哪个类的实例
   * 如何才能找到类的元数据信息
   * 对象的hashcode
   * 对象的GC分代年龄等信息

5. 结束后以上工作后,从虚拟机的视角看,一个新的对象已经产生了,但从Java程序的视角看,对象的创建才刚刚开始--init方法还没有执行,所有字段依旧为零值
6. 接着执行init方法, 把对象按照程序员的意愿进行初始化,这样一个真正可用的对象才算完全产生出来.

### 对象的内存布局
对象在内存中存储的布局可以分为3块区域: 对象头(Header), 实例数据(Instance Data), 和对齐填充(Padding)
1. **对象头(Header)**包括两部分
   * 自身的运行时数据 -hashcode, GC分代年龄, 锁状态标志, 线程持有的锁, 偏向线程ID, 偏向时间戳
   * 类型指针, 即对象指向它的类元数据的指针,虚拟机同过其来确定这个对象是哪个类的实例.
2. **实例数据**是对象真正存储的有效信息,也是在程序代码中所定义的各种类型的字段内容
3. **对齐填充**因对象的大小必须是8字节的整数倍,没对齐是用其来填充.

### 对象的访问定位
1. **句柄访问**在对象移动时只需要改变句柄中的实例数据指针,reference本身无需修改
![state access](/media/posts/access_ref.png)
2. **直接指针访问**速度更快,它节省了一次指针定位的开销
![direct pointer access](/media/posts/access_direct.png)

## 垃圾收集器(GC)与内存分配策略
### 判断对象是否可以被回收的算法
1. **引用计数算法**
> 给对象中添加一个引用计数器，每当有一个地方引用它时，计数器+1；当引用失效时，计数器就减1；任意时刻计数器为0的对象就是不可能再被使用的。
大部分情况下这是一个不错的算法，但是最大的问题是它很难解决对象之间的互相循环引用的问题。
```Java
public class ReferenceCountingGC{
    public Object instance = null;
    private static final int _1MB = 1024*1024
    private byte[] bigsize = new byte[2* _1MB]

    public static void testGC() {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();
        objA = null;
        objB = null;
        //即使两个对象相互引用,Java的GC也把两个对象回收了
        System.gc();
    }
}
```
2. **可达性分析算法**
> 这个算法的基本思想就是通过一系列的称为"GC Roots"的对象作为起始点,从这些节点开始向下搜索,搜索所走过的路径称为引用链(Reference Chain),当一个对象到GC Roots没有任何引用链相连时,则证明此对象是不可用的.
![Reachable analysis](/media/posts/reachable.PNG)

3. **对引用更多的定义**
> 初步的定义: 如果reference类型的数据中存储的数值代表的是另外一块内存的起始地址,就称这块内存代表着一个引用.
但是我们需要描述一些"当内存空间还够时则保留在内存中,若进行了垃圾收集后还是非常紧张,则可以抛弃这些对象."的引用.于是引用的概念被扩充至四个, 强引用,软引用,弱引用和虚引用.
   - **强引用**: 只要强引用还在,垃圾收集器就永远不会收掉被引用的对象.例如Object obj = new Object()这类的引用.
   - **软引用**: 还有用但非必须的对象. 对于软引用关联着的对象, 在系统将要发生内存溢出异常之前,将会把这些对象列进回收范围之中,进行第二次回收. 若这次回收后还没有足够的内存则抛出内存溢出的异常. SoftReference类来实现.
   - **弱引用**: 弱引用的强度比软引用更弱, 被弱引用关联的对象只能生存到下一次垃圾收集发生之前.当垃圾收集器工作时, 无论当前内存是否足够, 都会回收掉只被弱引用关联的对象. WeakReference类来实现
   - **虚引用**: 最弱的引用关系, 为一个对象设置虚引用唯一的目的就是能在这个对象被收集器回收时收到一个系统通知. PhantomReference类来实现.

4. **对象回收前的工作**
要真正宣告一个对象死亡, 至少需要经历两次标记过程: 如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链, 那它将会被第一次标记并且进行一次筛选, 筛选的条件是此对象是否有必要执行finalize()方法, 当对象没有覆盖finalize()方法, 或者finalize()方法已经被虚拟机调用过,虚拟机将这两种情况都视为"没有必要执行".

```Java
public class FinalizeEscapeGC {
    public static FinalizeEscapeGC SAVE_HOOK = null;

    public void isAlive() {
        System.out.println("yes, i'm still alive:)");
    }

    @Override
    protected void finalize() throws Throwable{
        super.finalize();
        System.out.println("finalize method executed!");
        FinalizeEscapeGC.SAVE_HOOK = this;
    }

    public static void main(String[] args) throws Throwable {
        SAVE_HOOK = new FinalizerEscapeGC();
        //第一次自救
        SAVE_HOOK = null;
        System.gc();
        //finalize方法优先级很低,所以暂停0.5秒
        Thread.sleep(500);
        if (SAVE_HOOK != null){
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("no, i'm dead :(");
        }

        //下面与上面完全相同,但是自救失败
        SAVE_HOOK = null;
        System.gc();
        //finalize方法优先级很低,所以暂停0.5秒
        Thread.sleep(500);
        if (SAVE_HOOK != null){
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("no, i'm dead :(");
        }
    }
}
```
5. **回收方法区**
    - **回收废弃常量**: 与回收Java堆中的对象非常类似. 
    - **回收无用的类**: 判断"无用类"的三个条件
        * 该类所有的实例都已经被回收, 也就是Java堆中不存在该类的任何实例
        * 加载该类的ClassLoader已经被回收
        * 该类对应的java.lang.Class对象没有在任何地方被引用, 无法在任何地方通过反射访问该类的方法.

### 垃圾回收算法
介绍垃圾回收算法之前需要先明白Java堆的新生代和老年代
> 在 Java 中，堆被划分成两个不同的区域：年轻代(Young)、老年代 (Tenured).
> 
> 年轻代 ( Young ) 又被划分为三个区域：Eden、From Survivor、To Survivor。 这样划分的目的是为了使 JVM 能够更好的管理堆内存中的对象，包括内存的分配以及回收。
> 
> 堆大小 = 年轻代 + 老年代
> 
> 年轻代 = eden space (新生代) + from survivor + to survivor

![java-heap](/media/posts/javaheap.JPG)

年轻代的特点是产生大量的死亡对象,并且要是产生连续可用的空间, 所以使用复制清除算法和并行收集器进行垃圾回收.对年轻代的垃圾回收称作初级回收 (minor gc)

新生代几乎是所有 Java 对象出生的地方，即 Java 对象申请的内存以及存放都是在这个地方。Java 中的大部分对象通常不需长久存活，具有朝生夕灭的性质。 

当一个对象被判定为 "死亡" 的时候，GC 就有责任来回收掉这部分对象的内存空间。新生代是 GC 收集垃圾的频繁区域。 当对象在 Eden 出生后，在经过一次 Minor GC 后，如果对象还存活，并且能够被另外一块 Survivor 区域所容纳，则使用复制算法将这些仍然还存活的对象复制到另外一块 Survivor 区域中，然后清理所使用过的 Eden 以及 Survivor 区域，并且将这些对象的年龄设置为1，以后对象在 Survivor 区每熬过一次 Minor GC，就将对象的年龄 + 1，当对象的年龄达到某个值时 ( 默认是 15 岁，可以通过参数 -XX:MaxTenuringThreshold 来设定 )，这些对象就会成为老年代。 
但这也不是一定的，对于一些较大的对象 ( 即需要分配一块较大的连续内存空间 ) 则是直接进入到老年代。
(摘自https://gblog.sherlocky.com/java-xin-sheng-dai-lao-nian-dai/)
1. **标记-清除算法(Mark-Sweep)**
   算法分为"标记"和"清除"两个阶段:首先标记出所有需要回收的对象, 在标记完成后统一回收所有被标记的对象. 它时最基础的收集算法.不足有二
   * 效率问题: 标记和清除两个过程效率都不高
   * 空间问题: 标记清除之后会产生大量的不连续的内存碎片,空间碎片太多可能会导致以后再程勋运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾回收动作.
   ![mark-sweep](/media/posts/marksweep.JPG)
2. **复制算法(Copying)**
   此算法为了解决效率问题而产生. 它将可用内存按照容量划分成大小相等的两块, 每次只使用其中的一块.当一块用完时,就将还存活的对象都复制到另一块,然后把已使用过得内存空间一次全部清理掉. 这种算法不用考虑内存碎片等复杂情况但代价是将内存缩小为了原来的一半. 这种算法多用于新生代中的对象的回收.
   ![copying](/media/posts/copying.JPG)
3. **标记-整理算法(Mark-Compact)**
   复制算法并不适用于老年代的对象. 因此出现了**标记-整理算法**, 标记过程与标记-清除算法一样, 但后续会让所有存活的对象都向一端移动, 然后直接清理掉端界面以外的内存.
   ![mark-compact](/media/posts/markcompact.JPG)
4. **分代收集算法(Generational Collection)**
   当前商业虚拟机的垃圾收集都采用**分代收集算法**, 它根据对象存活周期的不同将内存划分为几块, 一般是把Java堆分成新生代和老年代.在新生代中,每次垃圾收集时都发现有大批的对象死去, 只有少量存活,就选用复制算法.而老年代中因为对象存活率高,没有额外空间对他进行分配担保,就要使用**标记-清理**或者**标记-整理**算法.

### 垃圾收集器
HotSpot虚拟机的垃圾收集器如图所示
![GC_HotSpot](/media/posts/GCmachine.JPG)
图中展示了七种不同分代的收集器, 如果两个收集器之间存在连线,说明他们可以搭配使用. 虚拟机所处的区域表示它属于新生代还是老年代的收集器.
1. **Serial收集器**
    ![Serial](/media/posts/serial.JPG)
    * 这是最基本的收集器, 单线程. 并且"单线程"的意义不仅仅说明他只会使用一个CPU或一条收集线程去完成垃圾收集工作,更重要的是它在进行垃圾收集时,必须暂停其他所有的工作线程,直到它收集结束. ---"Stop The World"
    * 这项工作是自动发起和完成的,在用户不可见的情况下把正常工作的线程全部停掉.
    * 简单而高效,对于运行在Client模式下的虚拟机来说是一个很好的选择.
2. **ParNew收集器**
    ![ParNew](/media/posts/parnew.JPG)
    * 其实只是Serial收集器的多线程版本.
    * 是运行在Server模式下的虚拟机中首选的新生代收集器,因为这是目前唯一能和CMS收集器配合工作的新生代收集器.
    * ParNew收集器在单CPU环境中绝不会有比Serial收集器更好的效果

3. **Parallel Scavenge收集器**
    * 与ParNew一样,他也是新生代收集器,使用复制算法,也是并行的多线程
    * Parallel Scavenge收集器的目的是达到一个可控制的吞吐量(Throughput)=>CPU用于运行用户代码的时间与CPU总消耗时间的比值=>吞吐量=运行用户代码的时间/(运行用户代码的时间 + 垃圾回收收集时间)
    * 也被称为"吞吐量优先"收集器
    * 参数: MaxGCPauseMillis控制最大垃圾收集停顿时间, GCTimeRatio设置吞吐量大小, UseAdaptiveSizePolicy开启GC自适应调节策略(GC Ergonomics)
    
4. **Serial Old 收集器**
    * Serial收集器的老年代版本
    * 用来与Parallel Scavenge收集器搭配使用
    * 用来作为CMS收集器的后备预案

5. **Parallel Old 收集器**
    ![Parallel Old](/media/posts/parallelold.JPG)
    * **Parallel Old 收集器**是Parallel Scavenge收集器的老年代版本
    * 在**Parallel Old 收集器**出现之前, 新生代的Parallel Scavenge收集器一直处在比较尴尬的位置由于其无法与CMS收集器配合工作
    * "吞吐量优先"收集器终于有了比较给力的应用组合,在注重吞吐量以及CPU资源敏感的场合,都可以优先考虑Parallel Scavenge加Parallel Old收集器.

6. **CMS 收集器**
    > CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器. 从名字上看CMS收集器是基于"标记-清除"算法实现的.

    它的运作过程相对于前面几种收集器来说更复杂一些,整个过程分为4个步骤.
    + 初始标记(CMS initial mark)
    + 并发标记(CMS concurrent mark)
    + 重新标记(CMS remark)
    + 并发清除(CMS concurrent sweep)

    其中,初始标记和重新标记这两个步骤仍然需要"Stop The World". 
    * 初始标记仅仅只是标记一下GC Roots能关联到的对象,速度很快
    * 并发标记阶段就是进行GC Roots Tracing的过程
    * 重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录, 这个阶段的停顿时间一般会比初始标记阶段稍长一些, 但远比并发标记的时间段.
    
    ![cms](/media/posts/cms.JPG)
    虽然CMS有并发收集和低停顿的优点,但也具有三个明显的缺点. (//TODO: 详细阐述三个缺点)
    - CMS收集器对CPU资源非常敏感
    - CMS收集器无法处理浮动垃圾(Floating Garbage)
    - CMS基于"标记-清除"算法,意味着它收集结束时会有大量的空间碎片产生.
7. **G1收集器(Garbage-First)**
   * 是一款面向服务端应用的垃圾收集器.
   * 特点是并发与并行,分代收集,空间整合,可预测的停顿
   * 步骤类似CMS,分别为初始标记,并发标记,最终标记,筛选回收
## 虚拟机性能监控与故障处理工具
## 调优案例分析
### 书中实战: 优化Elipse启动时间

# 虚拟机执行子系统
## 类文件结构
### 平台无关性
对于Java来说,平台无关性的基石就是**字节码(Byte Code)**.实现语言无关性的基础是**虚拟机**和**字节码存储格式**.**Java虚拟机**不与任何语言绑定, 它只与"Class文件"这种特定的二进制文件格式所关联.
>Class文件中包含了Java虚拟机指令集和符号表以及若干其他辅助信息. 基于安全方面的考虑, Java虚拟机规范要求在Class文件中使用许多强制性的语法和结构化约束.虚拟机并不关心Class的来源是何种语言

## 虚拟机类加载机制
## 虚拟机字节码执行引擎
## 类加载及执行子系统的案例与实战 

# 程序编译与代码优化
## 早期(编译期)优化
## 晚期(运行期)优化

# 高效并发
## Java内存模型与线程
## 线程安全与锁优化