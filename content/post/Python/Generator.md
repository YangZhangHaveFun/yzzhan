---
title : "对生成器和协程的理解"
date: 2019-01-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Python", "generator"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---

# 生成器

生成器不仅可以输出值还可以输入值

## 生成器的四个方法

1. next()
2. send(*args)
3. close()
4. throw()

## Advanced function --- yield from