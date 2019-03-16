---
title : "Java面试的知识储备"
date: 2019-03-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Java", "Interview"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---
#Content
1. Difference between Hashtable and HashMap




---------------------------------------------------
## 1.Difference between Hashtable and HashMap
1. 作者不一样。。。
Hashtable阵容
```Java
 * @author Arthur van Hoff
 * @author Josh Bloch
 * @author Neal Gafter
```
HashMap阵容
```Java
 * @author Arthur van Hoff
 * @author Josh Bloch
 * @author Neal Gafter
 * @author Doug Lea
```
Doug Lea大神写了util.concurrent包。 并著有并发编程圣经Concurrent Programming in Java: Design Principles and Patterns 他的个人主页 http://g.oswego.edu/
Josh Bloch 著有Effective Java, Arthur vanHoff最早任职于硅谷的SunMicrosystems公司，从事Java程序语言的早期开发工作
Neal Gafter是Java SE 4和5语言增强的主要设计者和实现者

2. 产生时间
HashMap 晚于 Hashtable, Hashtable一开始发布就提供的键值映射的数据结构，而HashMap产生于JDK1.2。

3. 继承的父类不同
HashMap是继承自AbstractMap类，而HashTable是继承自Dictionary类。不过它们都实现了同时实现了map、Cloneable（可复制）、Serializable（可序列化）这三个接口。
![HashMap](hashmap.png)
![Hashtable](hashtable.png)

4. 对外提供的接口不同
Hashtable比HashMap多提供了elments() 和contains() 两个方法。

5. 对Null key 和Null value的支持不同
Hashtable既不支持Null key也不支持Null value。Hashtable的put()方法的注释中有说明。 当key为Null时，调用put() 方法，运行到下面这一步就会抛出空指针异常。因为拿一个Null值去调用方法了。

HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

6. 