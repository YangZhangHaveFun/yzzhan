---
title : "对Elastic Search的实现和理解"
date: 2019-03-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Python", "Search Engine"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---

# Elastic Search Engine

## Specification
1. Coding Language : python
2. FrameWork : django + scrapy
3. Scrapied website: www.zhihu.com www.jobbole.com www.lagou.com 

## Knowledge Preparation

### 浏览器 driver
由于知乎等网站需要登录后才能访问所需要的内容，于是我们需要安装chrome driver 和 selenium去模拟登录

**下载链接** http://chromedriver.chromium.org/downloads 
但是，直接使用还是凉凉， 因为服务器会从Chrome driver的某些js文件中检测出这是个driver不是正常浏览器，然后返回403.我们的解决方法是开启chrome remote debug 模式
**配置**
关闭所有chrome正在运行的实例
在Cmd里找到Chrome.exe所在的目录，输入chrome.exe --remote-debugging-port=9222
然后访问127.0.0.1:9222/json

### ElasticSearch 概念
1. 集群： 一个或多个节点组织在一起
2. 节点： 一个节点是集群中的一个服务器，由一个名字来标识
3. 分片： 将索引划分为多份的能力，允许水平分割和扩展容量，多个分片响应请求，提高性能和吞吐量
4. 副本： 创建分片的一份或多份的能力，在一个节点失败是其余节点可以顶上


| ElasticSearch | Mysql |
|---------------|-------|
|index(索引)     |数据库 |
|type(类型)      | 表    |
|documents(文档) | 行    |
|Fields         |  列   |