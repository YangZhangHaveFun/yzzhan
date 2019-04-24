---
title : "Notes for Advanced Database System(COMP90050)"
date: 2019-04-17T01:37:56+08:00
lastmod: 2019-04-17T01:37:56+08:00
draft: false
tags: ["Concurrency", "Database"]
categories: ["Databased related"]
author: "Yang Zhang"
---

#### Different types of Database System
There are multiple database technologies related to database structure.
- Simple file systems like Unix File System
- Relational Database System(RDBS)
- Object Oriented(oo) Database System
- Deductive Database System(DDBS)
- Key-Value pair based Database System
- NoSQL

#### Different types of Database Architectures
- Centralised Database System
- Client-Server
- Distributed Database System
- World Wide Web
- Grid Computing/Databases that lead to Cloud Computing/Data Centres
- P2P Databases

### Basic Memory Knowledge
#### Memory Hierarchy
In single core circumstance, the CPU communication case is :
![Sinle core](/media/posts/cache.png)
In muliple cores circumstance, the CPU communication case is :
![Multi cores](/media/posts/multiCores.png)
In Inter i7 Block Diagram, the CPU interaction is:
![I7 cores](/media/posts/i7Cores.png)

#### Transaction Processing
> A transaction is collection of operations that need to be performed on the physical and abstract application state.

Structure of a transaction should be:
```sql
begin_Transaction()
        <Sequence of operations to be performed>
    if (successful)
        commit_Trans() % Any changes made durable.
    else rollback() %Any changes made are all undone
end_Transaction()
```

**Transaction processing** contributed many concepts in distributed computing and fault tolerant computing.

##### ACID(atomicity, consistency, isolation, durability)
**Atomicity**
:  A transaction’s changes to the state (Database) are atomic implying either all actions happen or none happen.

**Consistency**
: A the transaction is a correct transformation of the state. Actions taken as a whole do not violate the integrity of the application state assuming transactions are correct programs.

**Isolation**
:  Even when several transactions are executed simultaneously, it appears to each transaction T that others executed either happen before T or after T but not at the same time.

**Durability**
: State changes committed by a transaction survive failures.

##### distributed transaction processing system
As for the distributed transaction processing system, it should look like this.
![distributed transaction processing system](/media/posts/dtbs.png)

The features of distributed transaction processing system.
- Application development features
- Repository features
- Database features

#### Probability and Fault Tolerance
##### Concepts
- P(A) is the probability of an event A is happening in **certain period**
- Assuming event A and B is independent, 
  - P(A and B) = P(A) * P(B)
  - P(A or B) = P(A) + P(B) - P(A) * P(B) = P(A) + P(B)
- **Mean time** to the event A is 1/P(A). MT(A) = 1/P(A)
- **Meantime to failure(MTTF)** means the time I need to wait to see the failure(which means running period).
- **Meantime to repair(MTTR)** means the time I need to wait to see the repair finished.
- **Module Availability** measures the ratio of service accomplishment to elapsed time.
  MA = MTTF/(MTTF + MTTR)
 
##### Fault tolerance in multiple events case
If all **N** different events have same mean time **M**. The mean time to the first one of the evenets = M/N. It means that the more devices we use, the shorter time the system meets the failure.

##### Fault types
##### Fault Tolerance
- Failvote - use two are more modules and compare their outputs. Stops if there are no majority outputs agreeing. It fails twice as often with duplication but gives clean failure semantics
- Failfast - this scheme is similar to fail voting except the system senses which modules are available and then uses the majority of the available modules. (which means we only concern of majority among the working ones.)
  > In systems design, a fail-fast system is one which immediately reports at its interface any condition that is likely to indicate a failure. Fail-fast systems are usually designed to stop normal operation rather than attempt to continue a possibly flawed process. Such designs often check the system's state at several points in an operation, so any failures can be detected early. The responsibility of a fail-fast module is detecting errors, then letting the next-highest level of the system handle them.

A 10-plex module system continues to operate until the failure of 9 modules in Failfast.
A 10-plex module system continues to operate until the failure of 5 modules in Failvote.
#### How to improve hardware reliability
#### How to improve software reliability
##### Process-pairing
##### Transaction based recovery
#### How to improve communication system reliability

### Lock Mechanisms
#### Pessimistic Concurrency Control
#### Optimistic Concurrency Control
#### Two Phase Locking(2PL)
#### Priority Ceiling Protocol
####  Read-Write Priority Ceiling 