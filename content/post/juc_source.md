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