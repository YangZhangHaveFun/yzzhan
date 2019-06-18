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