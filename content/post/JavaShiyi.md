---
title : "Java拾遗"
date: 2019-04-02T01:37:56+08:00
lastmod: 2019-04-08T01:37:56+08:00
draft: false
tags: ["Java", "Interview"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"


------------------------------------

### 面向对象和面向过程的区别与面向对象的三大基本特征和五大基本原则

**面向过程**指把task分解成一个一个的步骤, 每个步骤用函数来实现,依次调用即可. 

进一步解释:在软件开发初期,程序编写都是以算法为核心,程序员会把数据和过程分别作为独立的部分来考虑, 数据代表空间中的客体,程序代码则用于处理这些数据,这种直接站在计算机的角度去抽象问题和解决问题的思想称为面向过程的编程思想.

**面向对象**指把task分解成一个一个的步骤之后, 并对各个步骤进行相应的抽象, 形成对象, 同时建立对象之间的联系. 随后组合解决问题.
在面向对象编程的过程中,要把步骤进行相应的抽象形成对象, 对象封装了相应的属性和方法. 最后基于对象和对象的能力来完成业务逻辑的实现.

进一步解释:面向对象的编程思想是站在现实角度去抽象和解决问题,它把数据和行为都看做是对象的一部分, 这样可以让程序员能以符合现实世界的思维方式来编写和组织程序.

**面向对象的三个基本特征**
- 封装
- 继承
- 多态
>_封装(Encapsulation)_:是指利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体，数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。系统的其他对象只能通过包裹在数据外面的已经授权的操作来与这个封装的对象进行交流和交互。也就是说用户是无需知道对象内部的细节，但可以通过该对象对外的提供的接口来访问该对象。

>_继承(Inheritance)_: 继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承我们能够非常方便地复用以前的代码，能够大大的提高开发的效率。
 继承所描述的是“is-a”的关系，如果有两个对象A和B，若可以描述为“A是B”，则可以表示A继承B，其中B是被继承者称之为父类或者超类，A是继承者称之为子类或者派生类。

 《Think in java》：问一问自己是否需要从子类向父类进行向上转型。如果必须向上转型，则继承是必要的，但是如果不需要，则应当好好考虑自己是否需要继承。

>_多态(Polymorphism)_: 所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。因为在程序运行时才确定具体的类，这样，不用修改源程序代码，就可以让引用变量绑定到各种不同的类实现上，从而导致该引用调用的具体方法随之改变，即不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。

对于面向对象而言，多态分为编译时多态和运行时多态。其中编辑时多态是静态的，主要是指方法的重载，它是根据参数列表的不同来区分不同的函数，通过编辑之后会变成两个不同的函数，在运行时谈不上多态。而运行时多态是动态的，它是通过动态绑定来实现的，也就是我们所说的多态性。

总结:OOP是一种编程思想,一种程序范式,提倡用类来实现对象,用继承来实现关系,用封装来实现类所提供的功能,用多态来实现角色的动态转换

原文：https://blog.csdn.net/jianyuerensheng/article/details/51602015 


**面向对象的五个基本原则**
>_单一职责原则(Single-Resposibility Principle)_: 其核心思想为：一个类，最好只做一件事，只有一个引起它的变化。单一职责原则可以看做是低耦合、高内聚在面向对象原则上的引申，将职责定义为引起变化的原因，以提高内聚性来减少引起变化的原因。职责过多，可能引起它变化的原因就越多，这将导致职责依赖，相互之间就产生影响，从而大大损伤其内聚性和耦合度。通常意义下的单一职责，就是指只有一种单一功能，不要为类实现过多的功能点，以保证实体只有一个引起它变化的原因。
专注，是一个人优良的品质；同样的，单一也是一个类的优良设计。交杂不清的职责将使得代码看起来特别别扭牵一发而动全身，有失美感和必然导致丑陋的系统错误风险。

>_开放封闭原则(Open-Closed principle)_: 其核心思想是：软件实体应该是可扩展的，而不可修改的。也就是，对扩展开放，对修改封闭的。开放封闭原则主要体现在两个方面1、对扩展开放，意味着有新的需求或变化时，可以对现有代码进行扩展，以适应新的情况。2、对修改封闭，意味着类一旦设计完成，就可以独立完成其工作，而不要对其进行任何尝试的修改。
实现开开放封闭原则的核心思想就是对抽象编程，而不对具体编程，因为抽象相对稳定。让类依赖于固定的抽象，所以修改就是封闭的；而通过面向对象的继承和多态机制，又可以实现对抽象类的继承，通过覆写其方法来改变固有行为，实现新的拓展方法，所以就是开放的。
“需求总是变化”没有不变的软件，所以就需要用封闭开放原则来封闭变化满足需求，同时还能保持软件内部的封装体系稳定，不被需求的变化影响。

>_Liskov替换原则(Liskov-Substituion Principle)_: 其核心思想是：子类必须能够替换其基类。这一思想体现为对继承机制的约束规范，只有子类能够替换基类时，才能保证系统在运行期内识别子类，这是保证继承复用的基础。在父类和子类的具体行为中，必须严格把握继承层次中的关系和特征，将基类替换为子类，程序的行为不会发生任何变化。同时，这一约束反过来则是不成立的，子类可以替换基类，但是基类不一定能替换子类。
Liskov替换原则，主要着眼于对抽象和多态建立在继承的基础上，因此只有遵循了Liskov替换原则，才能保证继承复用是可靠地。实现的方法是面向接口编程：将公共部分抽象为基类接口或抽象类，通过Extract Abstract Class，在子类中通过覆写父类的方法实现新的方式支持同样的职责。
Liskov替换原则是关于继承机制的设计原则，违反了Liskov替换原则就必然导致违反开放封闭原则。
Liskov替换原则能够保证系统具有良好的拓展性，同时实现基于多态的抽象机制，能够减少代码冗余，避免运行期的类型判别。

>_依赖倒置原则(Dependecy-Inversion Principle)_: 其核心思想是：依赖于抽象。具体而言就是高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象。
我们知道，依赖一定会存在于类与类、模块与模块之间。当两个模块之间存在紧密的耦合关系时，最好的方法就是分离接口和实现：在依赖之间定义一个抽象的接口使得高层模块调用接口，而底层模块实现接口的定义，以此来有效控制耦合关系，达到依赖于抽象的设计目标。
抽象的稳定性决定了系统的稳定性，因为抽象是不变的，依赖于抽象是面向对象设计的精髓，也是依赖倒置原则的核心。
依赖于抽象是一个通用的原则，而某些时候依赖于细节则是在所难免的，必须权衡在抽象和具体之间的取舍，方法不是一层不变的。依赖于抽象，就是对接口编程，不要对实现编程。

>_接口隔离原则(Interface-Segregation Principle)_: 其核心思想是：使用多个小的专门的接口，而不要使用一个大的总接口。
具体而言，接口隔离原则体现在：接口应该是内聚的，应该避免“胖”接口。一个类对另外一个类的依赖应该建立在最小的接口上，不要强迫依赖不用的方法，这是一种接口污染。
接口有效地将细节和抽象隔离，体现了对抽象编程的一切好处，接口隔离强调接口的单一性。而胖接口存在明显的弊端，会导致实现的类型必须完全实现接口的所有方法、属性等；而某些时候，实现类型并非需要所有的接口定义，在设计上这是“浪费”，而且在实施上这会带来潜在的问题，对胖接口的修改将导致一连串的客户端程序需要修改，有时候这是一种灾难。在这种情况下，将胖接口分解为多个特点的定制化方法，使得客户端仅仅依赖于它们的实际调用的方法，从而解除了客户端不会依赖于它们不用的方法。
分离的手段主要有以下两种：1、委托分离，通过增加一个新的类型来委托客户的请求，隔离客户和接口的直接依赖，但是会增加系统的开销。2、多重继承分离，通过接口多继承来实现客户的需求，这种方式是较好的。

以上就是5个基本的面向对象设计原则，它们就像面向对象程序设计中的金科玉律，遵守它们可以使我们的代码更加鲜活，易于复用，易于拓展，灵活优雅。不同的设计模式对应不同的需求，而设计原则则代表永恒的灵魂，需要在实践中时时刻刻地遵守。就如ARTHUR J.RIEL在那边《OOD启示录》中所说的：“你并不必严格遵守这些原则，违背它们也不会被处以宗教刑罚。但你应当把这些原则看做警铃，若违背了其中的一条，那么警铃就会响起。”

------------------------------------
### 面向对象和面向过程的区别与面向对象的三大基本特征和五大基本原则 <a id="#1"></a>
平台无关性就是一种语言在计算机上的运行不受平台的约束，一次编译，到处执行（Write Once ,Run Anywhere).

对于Java的平台无关性的支持，就像对安全性和网络移动性的支持一样，是分布在整个Java体系结构中的。其中扮演者重要的角色的有Java语言规范、Class文件、Java虚拟机（JVM）等。
- **Java虚拟机**: 虽然Java语言是平台无关的，但是JVM确实平台有关的，不同的操作系统上面要安装对应的JVM。所以，Java之所以可以做到跨平台，是因为Java虚拟机充当了桥梁。他扮演了运行时Java程序与其下的硬件和操作系统之间的缓冲角色。
- **字节码**: 各种不同的平台的虚拟机都使用统一的程序存储格式——字节码（ByteCode）是构成平台无关性的另一个基石。Java虚拟机只与由自己码组成的Class文件进行交互。
- **Java语言规范**: Java中基本数据类型的值域和行为都是由其自己定义的。而C/C++中，基本数据类型是由它的占位宽度决定的，占位宽度则是由所在平台决定的。所以，在不同的平台中，对于同一个C++程序的编译结果会出现不同的行为。举一个简单的例子，对于int类型，在Java中，int占4个字节，这是固定的。但是在C++中却不是固定的了。在16位计算机上，int类型的长度可能为两字节；在32位计算机上，可能为4字节；当64位计算机流行起来后，int类型的长度可能会达到8字节。
-----------------------------

-----------------------------
#### 快速失败(fail-fast)
在用一个迭代器遍历一个集合对象时, 如果遍历过程中对集合对象的内容进行了修改(增删改), 则会抛出ConcurrentModificationException.

原理是: 迭代器在遍历时直接访问集合中的内容,并且在遍历过程中使用一个modCount变量.集合在被遍历期间如果内容产生变化, 就会改变modCount的值. 每当迭代器使用hasNext()/next()遍历下一个元素之前, 都会检测modCount变量是否为expectedmodCount值,是的话就返回遍历, 否则抛出异常,终止遍历.

当集合发生变化时修改modCount值刚好又设置为了expectedmodCount值,则异常不会抛出.因此,不能依赖于这个异常是否抛出而进行并发操作的编程, 这个异常建议仅用来检测修改的bug.

#### 安全失败(fail--safe)
采用安全失败机制的集合容器不会在遍历时直接访问集合内容,而是先复制原有集合内容,在拷贝的集合上进行遍历.

这种方法避免了ConcurrentModificationException 但缺点也很明显, 迭代器不能访问到修改后的内容,即在迭代器遍历期间,原集合发生的修改迭代器是不知道的.

java.util.concurrent包下的容器都是安全失败, 可以在多线程下并发使用, 并发修改.


### Java反射机制与类加载机制
#### 反射
基础类
```Java
package edu.unimelb.learningspringboots.javabasic;

public class Robot {
    private String name;

    public void PublicSayHello(String welcomeSentence){
        System.out.println(welcomeSentence + this.name);
    }

    private String throwHello(String name){
        return "hello" + name;
    }

    private static String staticSayHello() {
        return "Hello Static";
    }
}
```
调用目标类的方法首先要加载一个目标类对象通过Class.forName()
```java
Class robotClass = Class.forName("edu.unimelb.learningspringboots.javabasic.Robot");
```
若不是静态方法, 则还需通过这个类新建一个实例
```java
Robot robot = (Robot) robotClass.getDeclaredConstructor().newInstance();
```
当调用类里的方法时, 若是静态方法, 则getDeclaredMethod的第一个参数传null, 若是实例方法, 第一个参数传对应的实例
```java
    Method prSayHello = robotClass.getDeclaredMethod("throwHello", String.class);
    prSayHello.setAccessible(true);
    String str = (String) prSayHello.invoke(robot, "NoN");
    Field field = robotClass.getDeclaredField("name");
    field.setAccessible(true);
    field.set(robot, "Alice");
    log.info("Current Reflected Class is {}, invoked method is {}, result is {}, related field is {}",
            robotClass.getName(), prSayHello.getName(), str, field.getName());

    Method staticMethod = robotClass.getDeclaredMethod("staticSayHello");
    staticMethod.setAccessible(true);
    String res2 = (String) staticMethod.invoke(null);
    log.info(res2);
```

#### ClassLoader
ClassLoader在Java中有着非常重要的作用, 它主要工作在Class装载的加载阶段, 其主要作用是从系统外部获得Class二进制数据流. 它是Java的核心组件, 所有的Class都是由ClassLoader进行加载的, ClassLoader负责通过将Class文件里的二进制数据流装载进系统, 然后交给Java虚拟机进行连接,初始化等操作.

##### ClassLoader的种类

- BootStrapClassLoader: C++编写, 加载核心java.*
- ExtClassLoader: Java编写, 加载扩展库javax.*
- AppClassLoader: Java编写, 加载程序所在目录
- 自定义ClassLoader: Java编写, 定制化加载

##### 双亲加载机制

#### 类的加载方式

- 隐式加载: 
- 显式加载: 
  - LoadClass: 得到的class是还没有链接的
  - forName: 得到的class是已经初始化完成的


### JVM

#### 调制参数
- -Xss: 规定了每个线程虚拟机栈的大小
- -Xms: 堆的初始值
- -Xmx: 堆能达到的最大值

#### 堆栈的区别
- 静态存储: 编译时确定每个数据目标在运行时的存储空间需求
- 栈式存储: 数据区需求在编译时未知,运行时模块入口前确定
- 堆式存储: 编译时或运行时模块入口都无法确定,动态分配

##### 栈
- 管理方式: 栈自动释放,堆需要GC
- 空间大小: 栈比堆小
- 碎片相关: 栈产生的碎片远小于堆
- 分配方式: 栈支持静态和动态分配
- 效率: 栈的效率比堆高

### HashMap源码分析
![](/mdeia.posts/hashmapillu.png)

#### 基本结构
HashMap继承AbstractMap, 实现Map, Cloneable, Serializable. HashMap有 

#### 基本属性
##### 静态属性
```java
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    static final int MAXIMUM_CAPACITY = 1 << 30;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
```
##### 实例属性
```java
    transient Node<K,V>[] table;
    transient Set<Map.Entry<K,V>> entrySet;
    transient int size;
    transient int modCount;
    int threshold;
    final float loadFactor;
```

#### 构造器方法
```java
public HashMap(int initialCapacity, float loadFactor)

public HashMap(int initialCapacity)

public HashMap()

public HashMap(Map<? extends K, ? extends V> m)
```
#### 内部类
##### 静态内部类
```java
static class Node<K,V> implements Map.Entry<K,V>{}
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V>{}
```
##### 实例内部类
```java
//
final class EntrySet extends AbstractSet<Map.Entry<K,V>>{}
//Returns a set view of the keys contained in this map.
final class KeySet extends AbstractSet<K>{}
final class Values extends AbstractCollection<V>{}
```

#### 基本方法
##### 静态方法
```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    static final int tableSizeFor(int cap) {
        int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
hash方法通过一次异或运算使数据更加均匀分布. tableSizeFor方法用来获取size为大于cap的第一个2的n次方的值.
##### 实例操作方法
###### Public Actions
**GET方法**

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```
get方法通过输入的key和其hashcode来取得对应的node.
```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
在具体的实现逻辑中, 通过tab.length-1和hashcode的与计算来找到数组的索引, 由于初始化HashMap时, 通过tableForSize方法确使HashMap的初始长度都设置成了2的n次方,所以tab.length-1则后面(n-1)位数都是1. 因此key的hash值的n-1位确定了这组key,value对在table数组里的位置. 

但只通过这个hash值找到的是entrySet, 然后通过key值本身找到对应的Node. 这时可能有三种情况, 一种情况第一个就是要找的Node, 第二种情况是后继节点为链表结构,通过next找到对应的值, 第三种后继节点为红黑树结构, 通过对应的搜索方法找到对应的Node.

**PUT方法**
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

首先介绍后两个boolean值, 这两个值默认都是false.

- @param onlyIfAbsent if true, don't change existing value
- @param evict if false, the table is in creation mode.

接着是putValue的逻辑.

1. 首先判断table的长度, 如果长度是0或者table不存在执行resize()操作生成新的table.
2. 然后通过hash & table.length-1来计算出索引值
   1. 如果当前索引没有node, 就在此索引处创建一个node.
   2. 如果当前索引有node
      1. 判断当前第一个node的key值和要添加的node的key值相同,并且onlyIfAbsent的值是false,则更新当前Node的val. 添加过程结束
      2. 判断当前索引的数据结构是不是红黑树结构, 如果是, 通过树结构的添加操作填加val.
      3. 否则则证明当前索引的数据结构是链表结构.在链表的尾部添加node,并判断是否到达需要树化的阈值来决定是否把链表结构转化为树形结构.
3. 最后判断table的结构是否够用和