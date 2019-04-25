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

We evaluate experimentally two sharing method- ologies, based on their original prototype systems, that exploit work sharing opportunities among concurrent queries at run-time:
**Si- multaneous Pipelining (SP)**
**Global Query Plans (GQP)**
These two methods build and evaluate a single query plan with shared operators. We show that SP and GQP are orthogonal techniques.

并发控制(Concurrency Control)就是保证并发执行的事务在某一隔离级别正确执行的机制. 并发控制由数据库的调度器负责,事务本身并不感知.

在这个过程中,对可能破坏数据正确性的冲突事务,调度器可以选择两种处理模式:
- Delay: 延迟某个事务的执行知道合法的时刻
- Abort: 直接放弃事务的提交,并回滚该事务可能造成的影响.

锁用来在并发下控制多个操作的顺序执行,以此来保证数据安全的变动.乐观锁(Optimistic Concurrency Control)和悲观锁(Pessimistic Concurrency Control)是基于数据库层面的.

#### 悲观锁(Pessimistic Concurrency Control)
悲观锁的基本理论是被悲观锁保护的数据是极其不安全的,任何不顺序执行的操作都会使其数据发生错误.
> 我们经常使用的数据库是Mysql, mysql最常用的引擎是Innodb,Innodb默认使用的是行锁.而行锁是基于索引的,因此要想加上行锁,在加锁时必须命中索引,否则将使用表锁.



#### 乐观锁(Optimistic Concurrency Control)
乐观锁的基本理论是受乐观锁保护的数据变动不会太频繁,因此可以允许多个事务同时对数据进行变动.但是乐观不代表不负责,当多个事务执行发生冲突时,只会有一个事务执行成功,剩下的进行补充操作,例如回滚(rollback).

##### 多版本(Multiversion MVCC)
多版本的优势在于,可以让读写事务与只读事务互不干扰,因而获得更好的并行度,为了实现多版本的并发控制,需要给每个事务在开始时分配一个唯一标识TID,并对数据库对象增加以下信息:
- txd-id, 创建该版本的事务TID
- begin-ts及end-ts分别记录该版本创建和过期时的事务TID
- poiter 指向该对象的其他版本的链表.

基本思路是,每次对数据库对象的写操作都生成一个新的版本,用自己的TID标记新版本begin-ts以及上个版本的end-ts,并将自己加入链表.读操作对比自己的TID与数据版本的begin-ts,end-ts,找到其可见的最新的版本进行访问.
- Two-phase Locking(MV2PL)
- Timestamp Ordering(MVTO)
- Optimistic Concurrency Control(MVOCC)

无论何种并发控制的机制,归根结底都是在确定的隔离级别上尽可能的提高系统吞吐,可以说**隔离级别**选择决定了吞吐上限,而并发控制实现决定吞吐下限.


However, a transaction is serializable if we can guarantee that it would see exactly the same data if all its reads were repeated at the end of the transaction.

To ensure that a transaction T is serializable we must guarantee that the following two properties hold:
- **Read stability**:  If T reads some version V1 of a record during its processing, we must guarantee that V1 is still the version visible to T as of the end of the transaction, that is, V1 has not been replaced by another committed version V2. This can be implemented either by read locking V1 to prevent updates or by validating that V1 has not been updated before commit. This ensures that nothing has disappeared from the view.
- **Phantom avoidance**: We must also guarantee that the transaction’s scans would not return additional new versions. This can be implemented in two ways: by locking the scanned part of an index/table or by rescanning to check for new versions before commit. This ensures that nothing has been added to the view.

#### To meet the different level of isolation by multi-version.
- read uncommitted
- read committed 
- repeatable read 
- serializable

A transaction can be in one of four states: Active, Preparing, Committed, or Aborted.
![Four States of transaction](/media/posts/trastate.png)

write-write conflict. We follow the first-writer-wins rule and force transaction T to abort.

A serializable optimistic transaction keeps track of its reads, scans and writes. To this end, a transaction object contains three sets:
- ReadSet: contains pointers to every version read;
- ScanSet: stores information needed to repeat every scan;
- WriteSet: contains pointers to versions updated (old and new), versions deleted (old) and versions inserted (new).


Processing a read: a transaction reads only a committed version (created by a committed transaction). If the transaction which is processing a version is active, then the reader transaction waits until the creator transaction has committed or aborted. If the creator is aborted then the read request is scheduled for another version. Thus a read operation may be delayed but is never rejected and can be described as follows:
- find the version with largest write TS < the requesting transaction’s TS;
```
if transaction which created this version has committed then
  begin
    grant read; create new read TS for the version
  end
else block the read until the creator transaction of this version has committed or aborted;
```

 Internally, data consistency is maintained by using a multiversion model (Multiversion Concurrency Control, MVCC). This means that each SQL statement sees a snapshot of data (a database version) as it was some time ago, regardless of the current state of the underlying data.

 This prevents statements from viewing inconsistent data produced by concurrent transactions performing updates on the same data rows, providing transaction isolation for each database session. 

 MVCC, by eschewing the locking methodologies of traditional database systems, minimizes lock contention in order to allow for reasonable performance in multiuser environments.

 The main advantage of using the MVCC model of concurrency control rather than locking is that in MVCC locks acquired for querying (reading) data do not conflict with locks acquired for writing data, and so reading never blocks writing and writing never blocks reading. PostgreSQL maintains this guarantee even when providing the strictest level of transaction isolation through the use of an innovative Serializable Snapshot Isolation (SSI) level.

 Serializable level isolation indicates that any concurrent execution of a set of Serializable transactions is guaranteed to produce the same effect as running them one at a time in some order.

**dirty read**: A transaction reads data written by a concurrent uncommitted transaction.

**nonrepeatable read**: A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read).

**phantom read**: A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.

**serialization anomaly**: 
The result of successfully committing a group of transactions is inconsistent with all possible orderings of running those transactions one at a time.

| Isolation level | Dirty Read | Nonrepeatable Read | Phantom Read | Serialization Anomaly|
| ----------- | ----------- | ----------- | ----------- | ----------- |
| Read uncommitted | Allowed, but not in PG | Possible | Possible | Possible |
| Read committed | Not possible | Possible | Possible | Possible |
| Repeatable read | Not possible | Not possible | Allowed, but not in PG | Possible |
| Serializable | Not possible | Not possible | Not possible | Not possible |

#### Read Committed
Read Committed is the default isolation level in PostgreSQL. When a transaction uses this isolation level, a SELECT query (without a FOR UPDATE/SHARE clause) sees only data committed **before the query began**. It never sees 
- uncommitted data
- changes committed during query execution by concurrent transactions

The effect is that a `SELECT` query sees a snapshot of the database as of the instant the query begins to run. 

However, `SELECT` does see the effects of previous updates executed within its own transaction, even though they are not yet committed. Also note that two successive `SELECT` commands can see different data, even though they are within a single transaction, if other transactions commit changes after the first `SELECT` starts and before the second `SELECT` starts.

`UPDATE`, `DELETE`, `SELECT FOR UPDATE`, and `SELECT FOR SHARE` commands behave the same as `SELECT` in terms of searching for target rows: they will only find target rows that were committed as of the command start time. 

However, such a target row might have already been updated (or deleted or locked) by another concurrent transaction by the time it is found. In this case, the would-be updater will wait for the first updating transaction to commit or roll back (if it is still in progress). If the first updater rolls back, then its effects are negated and the second updater can proceed with updating the originally found row. 

If the first updater commits, the second updater will ignore the row if the first updater deleted it, otherwise it will attempt to apply its operation to the updated version of the row. The search condition of the command (the `WHERE` clause) is re-evaluated to see if the updated version of the row still matches the search condition. If so, the second updater proceeds with its operation using the updated version of the row. 


`INSERT with an ON CONFLICT DO UPDATE` clause behaves similarly.In Read Committed mode, each row proposed for insertion will either insert or update. Unless there are unrelated errors, one of those two outcomes is guaranteed. If a conflict originates in another transaction whose effects are not yet visible to the `INSERT` , the `UPDATE` clause will affect that row, even though possibly no version of that row is conventionally visible to the command.

`INSERT with an ON CONFLICT DO NOTHING` clause may have insertion not proceed for a row due to the outcome of another transaction whose effects are not visible to the INSERT snapshot. Again, this is only the case in Read Committed mode.

Because of the above rules, it is possible for an updating command to see an inconsistent snapshot: it can see the effects of concurrent updating commands on the same rows it is trying to update, but it does not see effects of those commands on other rows in the database. This behavior makes Read Committed mode unsuitable for commands that involve complex search conditions.

The partial transaction isolation provided by Read Committed mode is adequate for many applications, and this mode is fast and simple to use; however, it is not sufficient for all cases. Applications that do complex queries and updates might require a more rigorously consistent view of the database than Read Committed mode provides.


#### Repeatable Read Isolation Level

The Repeatable Read isolation level only sees data committed **before the transaction began**; it never sees either uncommitted data or changes committed during transaction execution by concurrent transactions. (However, the query does see the effects of previous updates executed within its own transaction, even though they are not yet committed.)

This is a stronger guarantee than is required by the SQL standard for this isolation level, and prevents all of the phenomena described in Table 13.1 except for serialization anomalies.

This level is different from Read Committed in that a query in a repeatable read transaction sees a snapshot as of the start of **the first non-transaction-control statement** in the transaction, not as of the start of the current statement within the transaction. Thus, successive SELECT commands within a single transaction see the same data, i.e., they do not see changes made by other transactions that committed after their own transaction started.

Applications using this level must be prepared to retry transactions due to serialization failures.

`UPDATE`, `DELETE`, `SELECT FOR UPDATE`, and `SELECT FOR SHARE` commands behave the same as `SELECT` in terms of searching for target rows: they will only find target rows that were committed as of the transaction start time. However, such a target row might have already been updated (or deleted or locked) by another concurrent transaction by the time it is found. In this case, the repeatable read transaction will wait for the first updating transaction to commit or roll back (if it is still in progress). If the first updater rolls back, then its effects are negated and the repeatable read transaction can proceed with updating the originally found row. But if the first updater commits (and actually updated or deleted the row, not just locked it) then the repeatable read transaction will be rolled back with the message.

#### Serializable Isolation Level
The Serializable isolation level provides the strictest transaction isolation. This level emulates serial transaction execution for all committed transactions; as if transactions had been executed one after another, serially, rather than concurrently. However, like the Repeatable Read level, applications using this level must be prepared to retry transactions due to serialization failures. 

In fact, this isolation level works exactly the same as Repeatable Read except that it monitors for conditions which could make execution of a concurrent set of serializable transactions behave in a manner inconsistent with all possible serial (one at a time) executions of those transactions.

When relying on Serializable transactions to prevent anomalies, it is important that any data read from a permanent user table not be considered valid until the transaction which read it has successfully committed. This is true even for read-only transactions.

To guarantee true serializability PostgreSQL uses predicate locking, which means that it keeps locks which allow it to determine when a write would have had an impact on the result of a previous read from a concurrent transaction, had it run first.

### Explicit Locking
#### Table-level Locks
#### Row-level Locks
#### Page-level Locks

### Concurrency Control Protocol
#### Two-phase Lock
**Two-phase lock** is a standard solution and broad mathematical theory has been developed to analyse the problem.
>Two-phase locking(2PL) is a concurrency control protocol that determines whethere a txn is allowed to access an object in the database on the fly.
>The protocol does not need to know all of the queries that a txn will execute ahead of time. 
##### Process
- **Phase 1: Growing**
  - Each txn requests the locks that it needs from the DBMS's lock manager.
  - The lock manager grants/denies lock requests.
- **Phase 2: Shrinking**
  - The txn is allowed to only release the locks that it previously acquried. It can not acquire new locks.

The 2PL guarantees the conflict serialibility. However, there are two problems.
- There are potential schedules that are serializable but would not be allowed by 2PL. => Locking limits concurrency.
- May still have "dirty read"       *solutions* => Strict 2PL
- May lead to deadlocks         *solution* => Detection or Prevention

> A schedule is **strict** if a value written by a txn is not read or overwritten by other txns until it finishes. 
> The advantanges:
> - Does not incur cascading aborts
> - Aborted txns can be undone by just restoring original values of modified tuples

**Deadlock Detection** 
The DBMS creates a waits-for graph to keep track of what locks each txn is waiting to acquire:
- Nodes are transactions
- Edge from $T_i$ to $T_j$ if $T_i$ is waiting for $T_j$ to release a lock. 
The system will periodically check for cycles in waits-for graph and then make a decision on how to break it.

**Deadlock Handling**
When the DBMS detecks a deadlock, it will select a "victim" txn to rollback to break the cycle. The victim txn will either restart or abort(common way) depending on how it was invoked.

There is a trade-off between the frequency of checking for deadlocks and how long txns have to wait before deadlocks are broken.

**Deadlock Handling--Victim Selection**
Selecting the proper victim depends on a lot of different variables
- By age(lowest timestamp)
- By progress(least/most queries executed)
- By the # of items already locked
- By the # of txns that we have to rollback with it

**Deadlock Handling--Rollback Length**
After selecting a victim txn to abort, the DBMS can also decide on how far to rollback the txn's changes
- Completely
- Minimally 

**Deadlock Prevention**
When a txn tries to acquire a lock that is held by another txn, the DBMS kills one of them to prevent a deadlock. This approach does not require a wait-for graph or detection algorithm. 

Assign priorities based on timestamps: Older Timestamp = Higher Priority($T_1$ > $T_2$)

- Wait-Die("Old Waits for Young"): If requesting txn has higher priority than holding txn, then *requesting txn* wait else abort
- Wound-Wait("Young Waits for Old"): If requesting txn has higher priority than holding txn, then *holding txn* aborts and release lock else *requesting txn* wait.

##### Lock Granularity
**Intention Locks**: allows a higher level node to be locked in shared or exclusive mode without having to check all descendent nodes. If a node is in an intention mode, then explicit locking is being done at a lower level in the tree.

- Intension-Shared(IS): Indicates explicit locking at a lower level with shared locks.
- Intension-Exclusive(IX): Indicates locking at a lower level with exclusive or shared locks.
- Shared+Intention-Exclusive(SIX): The subtree rooted by that node is locked explicitly in shared mode and explicit locking is being done at a lower level with exclusive-mode locks.

To get S or IS lock on a node, the txn must hold at least IS on parent node.
To get X, IX or SIX on a node, must hold at least IX on parent node.

Intention locks help improve concurrency:
- Intention-Shared(IS):Intent to get S lock(s) at finer granularity.
- Intention-Exclusive(IX):Intent to get X lock(s) at finer granularity.
- Shared+Intention-Exclusive(SIX): Like S and IX at the same time.


Two-phase locking (2PL) synchronizes reads and writes by explicitly detecting and preventing conflicts between concurrent operations.

Before reading data item x, a transaction must "own" a readlock on x. Before writing into x, it must "own" a writelock on x. The ownership of locks is governed by two rules:
- different transactions cannot simultaneously own conflicting locks; 
- once a transaction surrenders ownership of a lock, it may never obtain additional locks. 

The definition of conflicting lock depends on the type of synchronization being performed:
- for rw synchronization two locks conflict:
  - both are locks on the same data item
  - one is a readlock and the other is a writelock
- for ww synchronization two locks conflict:
  -  both are locks on the same data item
  -  both are writelocks.

The second lock ownership rule causes every transaction to obtain locks in a two-phase manner. During the growing phase the transaction obtains locks without releasing any locks. By releasing a lock the transaction enters the **shrinking phase**.  

During this phase the transaction releases locks, and, by rule 2, is prohibited from obtaining additional locks. When the transaction terminates (or aborts), all remaining locks are automatically released. 

Two-phase locking is a correct synchronization technique, meaning that 2PL attains an acyclic `rwr` or `ww` relation when used for rw (ww) synchronization.

The serialization order attained by 2PL is determined by the order in which transactions obtain locks. 

The point at the end of the growing phase, when a transaction owns all the locks it ever will own, is called the **locked point** of the transaction.

##### Basic 2PL Implementation

A distributed database management system (DDBMS) is a collection of sites interconnected by a network. Each site is a computer running one or both of the following software modules: a transaction manager (TM) or a data manager (DM). TMs supervise interactions between users and the DDBMS while DMs manage the actual database.