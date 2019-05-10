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
volatile 关键字最原始的意义是禁用CPU缓存.

极客时间版权所有: https://time.geekbang.org/column/article/84017