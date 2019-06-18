---
title : "hadoop的归纳"
date: 2019-05-03T01:37:56+08:00
lastmod: 2019-05-03T01:37:56+08:00
draft: false
tags: ["Hadoop"]
categories: ["Hadoop"]
author: "Yang Zhang"
---

### Concepts of Hadoop
"an open source softeware platform for distributed storage and distributed processing of very large data sets on computer clusters built from commodity hardware" - Hortonworks

### 构造器
初始化HashMap时,可以选择提供initialCapacity和loadFactor. 若不提供initialCapacity=64, loadFactor=0.75.
```Java
public HashMap(int initialCapacity, float loadFactor);

public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);

public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

HashMap内部包括的数据结构有 **数组**, **单向链表**, **红黑树**.
#### 单向链表

### General methods

#### PUT
```Java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
