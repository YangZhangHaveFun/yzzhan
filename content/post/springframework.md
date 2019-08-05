---
title : "Spring Analysis"
date: 2019-07-29
draft: false
tags: ["Java", "Spring", "Interview"]
categories: ["Spring Framework"]
author: "Yang Zhang"
---
#### JDBC的简单操作
- JdbcTemplate
  - query
  - queryForObject
  - queryForList
  - update
  - execute

####事务抽象的核心接口
- PlatformTransactionManager
  - DataSourceTransactionManager
  - HibernateTransactionManager
  - JtaTransactionManger
- TransactionDefition
  - Propagation
  - Isolation
  - Timeout
  - Read-only status
