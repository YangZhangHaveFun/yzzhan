---
title: "HashMap 源码解析"
tags: ["java", "HashMap", "source code analysis"]
date: 2019-6-18
draft: false
---

#### Construction
An instance of has two parameters that affect its performance: initial capacity and load factor.

- **capacity** is the number of buckets in the hash table, and the initial capacity is simply the capacity at the time the hash table is created. 
- **load factor** is a measure of how full the hash table is allowed to get before its capacity is automatically increased.

When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is <i>rehashed</i> (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.

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

#### hash
```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
异或运算 与 与运算 相比, 可以增加hash后的散列度


#### PUT
```Java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

> 为什么初始容量设置为2的倍数
> - 减少内存碎片的产生, 因为操作系统分出的内存为2的倍数, 例如分页一页为4KB.
> - 提高性能, 计算机更擅长移位计算. 2的次方所需要操作的数量最少.
> - 提高hash运算的散列度. 因为2的次方-1 2进制下剩余位数都为1, 与操作不会降低hash散列度.