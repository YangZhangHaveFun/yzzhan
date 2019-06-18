---
title : "Notes for Advanced Database System(COMP90050)"
date: 2019-04-17T01:37:56+08:00
lastmod: 2019-04-17T01:37:56+08:00
draft: false
tags: ["Concurrency", "Database"]
categories: ["Databased related"]
author: "Yang Zhang"
---
## Database Architecture

Database technologies:

- Simple file systems linke Unix file system
- Relational database systems
- Object Oriented Systems
- Deductive Database System
- Key-Value pair based Database System
- NoSQL

Database architectures:

- Centralised Database systems
- Client-Server
- Distributed Database systems
- World Wide Web
- Grid Computing/Database & Cloud Computing/Data Centres

## Hardware Structure

### Memory Hierarchy

The design and chip and cache is relatively similar. Cache design aims to catch several stage of 

![On chip for single chip](/media/posts/single-core.png)

![On chip for two chip](/media/posts/two-core.png)

> In computing, a cache is a hardware or software component that stores data so future requests for that data can be served faster; the data stored in a cache might be the result of an earlier computation, or the duplicate of data stored elsewhere.

> A **cache hit** occurs when the requested data can be found in a cache, while a **cache miss** occurs when it cannot. 
> Cache hits are served by reading data from the cache, which is faster than recomputing a result or reading from a slower data store; thus, the more requests can be served from the cache, the faster the system performs.

> The percentage of accesses that result in cache hits is known as the hit rate or hit ratio of the cache. 

$Hit\ ratio=\frac{references\ satisfied\ by\ cache}{total\ references}$

Effective memory access time, EA=H*C + (1-H)*S where H=hit ratio, C=cache access time, S = memory access time.

Disk access time = seek time + rotational time + transferlength/bandwidth


### Storage System

> Storage Systems can determine the performance and reliability of Data base system.

> RAID (redundant array of independent disks; originally redundant array of inexpensive disks) is a way of storing the same data in different places on multiple hard disks to protect data in the case of a drive failure. However, not all RAID levels provide redundancy.

> RAID (Redundant Array of Inexpensive Disks or Drives, or Redundant Array of Independent Disks) is a data storage virtualization technology that combines multiple physical disk drive components into one or more logical units for the purposes of data redundancy, performance improvement, or both. This was in contrast to the previous concept of highly reliable mainframe disk drives referred to as "single large expensive disk" (SLED).

#### How RAID works

RAID works by placing data on multiple disks and allowing input/output (I/O) operations to overlap in a balanced way, improving performance. Because the use of multiple disks increases the mean time between failures (MTBF), storing data redundantly also increases fault tolerance.

RAID employs the techniques of disk mirroring or disk striping. Mirroring copies identical data onto more than one drive. Striping partitions each drive's storage space into units ranging from a sector (512 bytes) up to several megabytes. The stripes of all the disks are interleaved and addressed in order.

A RAID controller can be used as a level of abstraction between the OS and the physical disks, presenting groups of disks as logical units. Using a RAID controller can improve performance and help protect data in case of a crash.

#### RAID levels

- **RAID 0**: This configuration has striping, but no redundancy of data. It offers the best performance, but no fault tolerance.
  ![raid-0](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_00.png)
- **RAID 1**: Also known as disk mirroring, this configuration consists of at least two drives that duplicate the storage of data. There is no striping. Read performance is improved since either disk can be read at the same time. Write performance is the same as for single disk storage.
  ![raid-1](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_01.png)
- **RAID 2**: This configuration uses striping across disks, with some disks storing error checking and correcting (ECC) information. It has no advantage over RAID 3 and is no longer used. 
  ![raid-2](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_02.png)
- **RAID 3**: This technique uses striping and dedicates one drive to storing parity information. The embedded ECC information is used to detect errors. Data recovery is accomplished by calculating the exclusive OR (XOR) of the information recorded on the other drives. Since an I/O operation addresses all the drives at the same time, RAID 3 cannot overlap I/O. For this reason, RAID 3 is best for single-user systems with long record applications.
  ![raid-2](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_03.png)
- **RAID 4**: This level uses large stripes, which means you can read records from any single drive. This allows you to use overlapped I/O for read operations. Since all write operations have to update the parity drive, no I/O overlapping is possible. RAID 4 offers no advantage over RAID 5.
  ![](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_04.png)
- **RAID 5**: This level is based on block-level striping with parity. The parity information is striped across each drive, allowing the array to function even if one drive were to fail. The array's architecture allows read and write operations to span multiple drives. This results in performance that is usually better than that of a single drive, but not as high as that of a RAID 0 array. RAID 5 requires at least three disks, but it is often recommended to use at least five disks for performance reasons.
RAID 5 arrays are generally considered to be a poor choice for use on write-intensive systems because of the performance impact associated with writing parity information. When a disk does fail, it can take a long time to rebuild a RAID 5 array. Performance is usually degraded during the rebuild time, and the array is vulnerable to an additional disk failure until the rebuild is complete.
  ![](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_05.png)
- **RAID 6**: This technique is similar to RAID 5, but includes a second parity scheme that is distributed across the drives in the array. The use of additional parity allows the array to continue to function even if two disks fail simultaneously. However, this extra protection comes at a cost. RAID 6 arrays have a higher cost per gigabyte (GB) and often have slower write performance than RAID 5 arrays.
  ![](https://cdn.ttgtmedia.com/rms/onlineImages/storage_raid_06.png)
- **RAID 10**: Combining RAID 1 and RAID 0, this level is often referred to as RAID 10, which offers higher performance than RAID 1, but at a much higher cost. In RAID 1+0, the data is mirrored and the mirrors are striped.
  ![raid-2](https://cdn.ttgtmedia.com/rms/editorial/storage_raid_10.png)

Storage can be organized as RAID systems. Storage is partitioned and allocated to each system and can also be shared.


## Fault Tolerance \& Database Reliability

> **Meantime to failure(MTTF)** means the time I need to wait to see the failure(which means running period).

> **Meantime to repair(MTTR)** means the time I need to wait to see the repair finished.



### Fault Tolerance 
#### Basic concepts

- P(A) is the probability of an event A is happening in **certain period**
- Assuming event A and B is independent, 
  - P(A and B) = P(A) * P(B)
  - P(A or B) = P(A) + P(B) - P(A) * P(B) = P(A) + P(B)
- **Mean time** to the event A is 1/P(A). MT(A) = 1/P(A)

#### Fault Tolerance Analysis
> **Module Availability** measures the ratio of service accomplishment to elapsed time.
MA = MTTF/(MTTF + MTTR)

If all n different events have same mean time m the Mean time to the first one of the events = m/n

#### Fault Tolerance Solutions

- Failvote - use two are more modules and compare their outputs. Stops if there are no majority outputs agreeing. It fails twice as often with duplication but gives clean failure semantics
- Failfast - this scheme is similar to fail voting except the system senses which modules are available and then uses the majority of the available modules. (which means we only concern of majority among the working ones.)
  > In systems design, a fail-fast system is one which immediately reports at its interface any condition that is likely to indicate a failure. Fail-fast systems are usually designed to stop normal operation rather than attempt to continue a possibly flawed process. Such designs often check the system's state at several points in an operation, so any failures can be detected early. The responsibility of a fail-fast module is detecting errors, then letting the next-highest level of the system handle them.

A 10-plex module system continues to operate until the failure of 9 modules in Failfast.
A 10-plex module system continues to operate until the failure of 5 modules in Failvote.

#### Repair Process

N-plex repair: in this configuration the faulty equipment is repaired with an average time of MTTR(mean time to repair) as soon as a fault is detected.

- Probability of a particular module is not available: $MTTR/(MTTR+MTTF) \cong MTTR/MTTF$ if MTTF >> MTTR
- Probability of (n-1) modules unavailable = $P_{n-1}=(\frac{MTTR}{MTTF})^{n-1}$
- Probability of a module N fails $P_{f}$ = 1/MTTF
- Probability that the system fails with the particular $i^{th}$ module failing last = $P_{f}*P_{n-1}$ = $(\frac{1}{MTTF})(\frac{MTTR}{MTTF})^{n-1}$
- Probability that the N-plex system fails  = $(\frac{n}{MTTF})(\frac{MTTR}{MTTF})^{n-1}$
- MTTF of N-plex system = $(\frac{MTTF}{n})(\frac{MTTF}{MTTR})^{n-1}$

### Software Reliability

- hardware reliability: requires tolerating component failures
- software reliability: requires tolerating design and coding faults

- Dense faults: these are the faults that a system can tolerate up to N faults within a period. If there are more than N faults the system may be interrupted.
- Byzantine faults: these are the faults which are not part of the model and the design is not catered to tolerate such faults such as earthquake.

### Solution to improving software reliability

#### Processing pair

- Periodic Transfer of data: One process called Primary does all the work until it fails. Second process called Backup takes over the Primary and continues the computation. In order to do this, Primary need to tell on a regular basis that it is alive and also transmit its state to the Backup.
- Checkpoint-restart: Primary records its state on a duplexed storage module. When taking over, Backup starts reading the state of the primary from the duplexed storage and resumes the application.
- Checkpoint-messages: Primary sends its state changes as messages to the Backup. When taking over, the Backup gets its  current state from the most recent checkpoint message.
- Persistent: Backup restarts in the null state and let Transaction Mechanism to clean up all uncommitted transactions. This is the approach taken by the most Database System.

#### Availability related

- Highly available storage: write to several storage modules and have some kind of checksum to make sure that the data read is correct with a very high probability. eg. Disk mirroring/Shadowing.
- Highly available Processes

Flat transaction
: In a flat transaction, each transaction is decoupled from and independent of other transactions in the system. Another transaction cannot start in the same thread until the current transaction ends.

Disadvantage: Flat transactions do not model many real applications, which means UNDO operation will roll back lots of unnecessary computations.

Nested transaction
: A nested transaction is a transaction that is created inside another transaction.

- Commit Rules
  - A subtransaction can either commit or abort, however, commit cannot take place unless the parent itself commits.
  - Subtransactions have A, C, and I properties but not have D property unless all its ancestors commit.
  - Commit of a sub transaction makes its results available only to its parents.
- Rollback Rules
  - If a subtransaction rolls back all its children are forced to roll back.
- Visibility Rules
  - Changes made by a sub transaction are visible to the parent only when the sub transaction commits. Whereas all objects of parent are visible to its children. Implication of this is that the parent should not modify objects while children are accessing them. This is not a problem as parent is not run in parallel with its children.

#### Transaction Processing Monitor(TP monitor) 
TP Monitor
:  is a control program that monitors the transfer of data between multiple local and remote terminals to ensure that the transaction processes completely or, if an error occurs, to take appropriate actions.

- Batch Processing
- Time-sharing
- Real-Time processing

TP service in distributed system need to deal with Heterogeneity and Control communication.
- Local TP monitors to interact with other TP monitors to ensure ACID properties.
- the local TP monitor should maintain the communicaiton status among the processes for it be able to recover from a crash.

TP service should provide:

- Terminal management
- Presentation service
- Context management

Components of a TP monitor

- Presentation Service: defines interface between the application and the devices it has to interact to.
- Queue management: performs the queuing of transactions.

> Transaction processing provides the following:
> - Resource mangers
> - Durable state
> - Transaction programming language
>
> There are two types of message passing:
> - session based - usually efficient and reliable for an interaction that requires several message transfers. Package comes sequence.
> - datagrams - efficient for single message transfers. Only one package, just send it and do not care the receiver receive it or not.
## ACID Properties

Transaction Processing
: A transaction is collection of operations that need to be performed on the physical and abstract application state.

### ACID

Atomicity
: A transaction’s changes to the state (Database) are atomic implying either all actions happen or none happen.

Consistency
: Transaction are a correct transformation of the state. Actions taken as a whole by a transaction do not violate the integrity of the application state. That is we are assuming transactions are correct programs. Concurrent execution of transactions does not violate consistency of data.

Isolation
: Even when several transactions are executed simultaneously, it appears to each transaction T that others executed either happen before T or after T but not at the same time.

Durability
: State changes committed by a transaction survive failures.

### Application Development

Resource mangers
: Responsible for providing ACID operations on objects they implement. E.g. Database systems, persistent programming languages, spoolers, reliable communication managers, etc.

Durable state
: The application designer represents the application state as durable data stored by the resource manager.

Transaction programming language
: Allows expression of group of actions as an atomic operation. E.g. SQL.

![](/media/posts/tra-s.png)
## Type of transaction model

### ACID properties

Atomicity
: all or none. This means the application would not be able to find reasons for failure directly. One may be able to find reasons of failures through other means - e.g., by looking at the system logs, or making a fake success.

Consistency
: only restricted type of consistency can be ganranteed.

Isolation
: transaction are executed as if it was only the one on the system.

Durability
: the system should tolerate system failures and any committed updates should not be lost.

### Write Types
Atomic Disk Writes
: either entire block is written correctly on disk or the contents of the block is unchanged. To achieve atomic disk writes we require **duplex write**.

> **Duplex write** means a block of data is written two disk blocks sequentially. We can determine whether the contents of a disk block has an error or not by means of its CRC. Each block is associated with a version number. The block which contains the latest version number is the one which contains most recent data. If one of the writes fail system can issue another write to the disk block that failed. It always guarantees at least one block has consistent data. 

Logged writes
: it is similar to the duplex write, except one of the writes goes to a log. This method is very efficient if the changes to a block are small. 



### Action Types

- Unprotected actions: no ACID property.
- Protected actions: these action are **not externalised** before they are completed done. These actions are controlled and can be rolled back if required. These have ACID property.
- Real actions: these are real physical actions once performed cannot be undone. In many situations, atomicity is not possible with real action

### Data Types

### Host Types

### Transaction Types

### Termial Management

## Isolation concepts, degree of isolation and their usefulness

### Dekker's algorithm

> Dekker's algorithm is the first known correct solution to the mutual exclusion problem in concurrent programming. 
>
> If two processes attempt to enter a critical section at the same time, the algorithm will allow only one process in, based on whose turn it is. If one process is already in the critical section, the other process will busy wait for the first process to exit.

- needs almost no hardware support although it needs atomic reads and writes to main memory, which is exclusive access of one time cycle of memory access time.
- code will be complicated and hard to understand when more than two processes exists.
- inefficient if the lock contention is low.

### OS supported primitives
Lock and Unlock: through an interrupt call, the request is passed to the operating system to implement desired lock request.

- no need special hardware
- very expensive in general due to saving of context of the requesting process.
- solution is independent of number of processes.
- need to use spin locks in the Kernel

### Spin Locks
Atomic machine instructions such as test and set or swap instructions.

- need hardware support 
- are efficient if the lock contentions are low
- are necessary in symmetric multiprocessor machines.

### Lock Manager
```c
boolean cs(int *cell, int *old, int *new){
  /* the following is executed atomically*/
  if (*cell == *old){ 
    *cell = *new; return TRUE;}
  else { 
    *old = *cell; 
    return FALSE;
    }
}
```

### Deadlocks
> In concurrent computing, a deadlock is a state in which each member of a group is waiting for another member, including itself, to take action, such as sending a message or more commonly releasing a lock.

Reason:

- **Mutual exclusion**: At least one resource must be held in a non-shareable mode. Otherwise, the processes would not be prevented from using the resource when necessary. Only one process can use the resource at any given instant of time.
- **Hold and wait or resource holding**: a process is currently holding at least one resource and requesting additional resources which are being held by other processes.
- **No preemption**: a resource can be released only voluntarily by the process holding it.
- **Circular wait**: each process must be waiting for a resource which is being held by another process, which in turn is waiting for the first process to release the resource. In general, there is a set of waiting processes, P = \{$P_{1}$, $P_{2}$, …, $p_{N}$\}, such that $P_{1}$ is waiting for a resource held by $P_{2}$, $P_{2}$ is waiting for a resource held by P3 and so on until $p_{N}$ is waiting for a resource held by $P_{1}$.

solutions:
- enough resources provided
- do not wait, roll back diirectly
- Linearly order the resources and request of resources should follow this order

#### DeadLock avoidance and mitigation

- pre-declare all the resources needed and allocate reources in a single request.
- allow a transaction to wait for certain max time on a lock and force it.
- periodically check the resource dependency graph for cycles.
- distributed database systems maintain only local dependency graphs and use time outs for global deadlocks.
- Phantom Deadlocks: Phantom dead locks can happen if we cannot build consistent dependency graphs across many independent database systems.

#### Theoretical analysis
![](media/posts/env.png)

![](media/posts/proc.png)

![](media/posts/res.png)

## Isolation

isolation guarantees consistency provided each transaction itself is consistent. In order to achieve isolation in concurrent environment, we need the concept of locking and/or restricting type of concurrent operations on an object.

Isolation properties can be studied by means of dependency graphs. There are three types of dependencies:
- Write --> Read
- Read --> Write
- Write --> Write

When dependency graph has cycles then there is a violation of isolation and a possibility of inconsistency.

- A transaction is a sequence of READ, WRITE, SLOCK, XLOCK actions on objects ending with COMMIT or ROLLBACK.
- A transaction is **well formed** if each READ, WRITE and UNLOCK operation is covered earlier by a corresponding lock operation.
- A history is **legal** if does not grant conflicting grants.
- A transaction is **two phase** if its all lock operations precede its unlock operations.

### Concurrency Issues

**dirty read**: A transaction reads data written by a concurrent uncommitted transaction.

**nonrepeatable read**: A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read).

**phantom read**: A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.

**serialization anomaly**: 
The result of successfully committing a group of transactions is inconsistent with all possible orderings of running those transactions one at a time.

### Isolation level

- **Read Uncommitted**: This is the lowest isolation level. In this level, dirty reads are allowed, so one transaction may see not-yet-committed changes made by other transactions.
- **Read Committed**:  a lock-based concurrency control DBMS implementation keeps write locks (acquired on selected data) until the end of the transaction, but read locks are released as soon as the SELECT operation is performed (so the non-repeatable reads phenomenon can occur in this isolation level). As in the previous level, range-locks are not managed.
- **Repeatable Read**:  a lock-based concurrency control DBMS implementation keeps read and write locks (acquired on selected data) until the end of the transaction. However, range-locks are not managed, so phantom reads can occur.
- **Serializable**: With a lock-based concurrency control DBMS implementation, serializability requires read and write locks (acquired on selected data) to be released at the end of the transaction. Also range-locks must be acquired when a SELECT query uses a ranged WHERE clause, especially to avoid the phantom reads phenomenon.

![](media/posts/il.png)

### Concurrency Control Strategy
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
> 
> - Does not incur cascading aborts
> - Aborted txns can be undone by just restoring original values of modified tuples


#### Optimistic Lock
The DBMS creates a private workspace for each txn.
- Any object read is copied into workspace
- Modifications are applied to workspace.

When a txn commits, the DBMS compares workspace write set to see whether it conflicts with other txns.

If there are no conflicts, the write set is installed into the "global" database.
**OCC Phases**

**Read Phase**:Track the read/write sets of txns and store their writes in a private workspace.

**Validation Phase**:When a txn commits, check whether it conflicts with other txns.

**Write Phase**:If validation succeeds, apply private changes to database. Otherwise abort and restart the txn.

**Validation Phase**:The DBMS needs to guarantee only serializable schedules are permitted.

$T_i$ checks other txns for RW and WW conflicts and makes sure that all conflicts go one way(from older txns to young txns). Record read set and write set while txns are running and write into private workspace.

Execute Validation and Write phase inside a protected critical section.

Each txn's timestamp is assigned at the beginning of the validation phase. Check the timestamp ordering of the committing txn with all other running txns.

If TS($T_i$) < TS($T_j$), then one of the following three conditions must hold.
- $T_i$ completes all three phases before $T_j$ begins.
- $T_i$ completes before $T_j$ starts its Write phase, and $T_i$ does not write to any object read by $T_j$  $\implies$  WriteSet($T_i$) $\cap$ WriteSet($T_j$) = $\emptyset$
- $T_i$ completes its Read phase before $T_j$ completes its Read phase. And $T_i$ does not write to any object that is either read or written by $T_j$:
  - WriteSet($T_i$) $\cap$ ReadSet($T_j$) = $\emptyset$
  - WriteSet($T_i$) $\cap$ WriteSet($T_j$) = $\emptyset$

OCC works well when the # of conflicts is low:
- All txns are read-only(ideal)
- Txns access disjoint  subsets of data.

If the database is large and the workload is not skewed, then there is a low probability of conflict, so again locking is wasteful. Therefore, the performance issues could be concluded as :
- High overhead for copying data locally.
- Validation/Write phase bottlenecks.
- Aborts are more wasteful than in 2PL because they only occur after a txn has already executed.

#### Timestamp Ordering 
Using timestamps to determine the serializability order of txns

if TS($T_i$)<TS($T_j$),then the DBMS must ensure that the execution schedule is equivalent to a serial schedule where $T_i$ appears before $T_j$. 

Each txn $T_i$ is assigned a unique fixed timestamp that is monotonically increasing.
- Let TS($T_i$) be the timestamp allocated to txn $T_i$.
- Different schemes assign timestamps at different times during the txn.

Implementation strategies could be System Clock, Logical Counter, Hybrid.

##### Basic Timestamp Ordering Protocol
Txns read and write objects without locks.

Every object X is tagged with timestamp of the last txn that successfully did read/write:
- W-TS(X): Write timestamp on X
- R-TS(X): Read timestamp on X

**READ PROCESS**
`If TS($T_i$)<W-TS(X)`, this violates timestamp order of $T_i$ with regard to the writer of X. 
- Abort $T_i$ and restart it with same TS

`Else`:
- Allow $T_i$ to read X.
- Update R-TS(X) to max(R-TS(X), TS($T_i$))
- Have to make a local copy of X to ensure repeatable reads for $T_i$

**WRITE PROCESS**
`If TS($T_i$)<W-TS(X) OR TS($T_i$)<W-TS(X)`
- Abort and restart $T_i$

`Else`:
- Allow $T_i$ to write X and update W-TS(X)
- Also have to make a local copy of X to ensure repeatable reads for $T_i$.

**THOMAS WRITE RULE**
`If TS($T_i$)<R-TS(X)`:
- Abort and restart $T_i$

`If TS($T_i$)<W-TS(X)`:
- Thomas Write Rule: Ignore the write and allow the txn to continue.
- This violates timestamp order of $T_i$

`Else`:
- Allow $T_i$ to write X and update W-TS(X)

**Basic T/O-Performance Issue**:
- High overhead from copying data to txn's workspace and from updating timestamps
- Long running txns can get starved.
#### MVCC
The DBMS maintains multiple physical versions of a single logical object in the database:
- When a txn writes to an object, the DBMS creates a new version of that object.
- When a txn reads an objects, it reads the newest version that existed when the txn started.

The principles of MVCC are:
- Writers don't block readers.
- Readers don't block writers.

Read-only txns can read a **consistent snapshot** without acquiring locks and timestamps is used to determine the visibility. Meanwhile, MVCC easily support **time-travel** queries

MVCC is more than just a concurrency control protocol. It completely affects how the DBMS manages transactions and the database.

##### MVCC DESIGN DECISIONS
- Concurrency Control Protocol
- Version Storage
- Garbage Collection
- Index Management
## Granular Locks
**Intention Locks**: allows a higher level node to be locked in shared or exclusive mode without having to check all descendent nodes. If a node is in an intention mode, then explicit locking is being done at a lower level in the tree.

- Intension-Shared(IS): Indicates explicit locking at a lower level with shared locks.
- Intension-Exclusive(IX): Indicates locking at a lower level with exclusive or shared locks.
- Shared+Intention-Exclusive(SIX): The subtree rooted by that node is locked explicitly in shared mode and explicit locking is being done at a lower level with exclusive-mode locks.
- U (Update lock) allows read to the node and its descendants and prevents others holding X, U, SIX, IX and IS locks on this node or its descendants.
- X (exclusive lock) allows writes to the node and prevents others holding X, U, S, SIX, IX locks on this node and all its descendants.

To get S or IS lock on a node, the txn must hold at least IS on parent node.
To get X, IX or SIX on a node, must hold at least IX on parent node.

Intention locks help improve concurrency:
- Intention-Shared(IS):Intent to get S lock(s) at finer granularity.
- Intention-Exclusive(IX):Intent to get X lock(s) at finer granularity.
- Shared+Intention-Exclusive(SIX): Like S and IX at the same time.

Crabbing: if a data structure is collection of nodes linked via pointers, we can guarantee atomicity in the following way:
- take an Xlock for entire list, do appropriate operation and release the lock.
- take an Xlock at the head and test for need to go further down; if required take Xlock at the next level and release the previous lock. The process is repeated until the required node is found. Perform the actions needed and release the lock. This gives more concurrency at the expense more locks.

## Database Recovery
### Handling Buffer Pool
The Recovery Manager guarantees **Atomicity** & **Durability**. 

![](media/posts/sys.png)

- **Force**: Force write to disk at commit
- **Steal**: Steal buffer-pool frames from uncommitted Xacts.

No Force && Steal is most desired and efficient way. 

- STEAL:
  - will cause atomicity issue
  - must remember the old value of P at the steal time to support UNDO
- No Force:
  - will cause durability issue
  - write as little as possible at commit time in a convenient place to support REDO.

Using Logging to record relative information to REDO and UNDO

### Logging -- Write Ahead Logging(WAL)
The Write-Ahead Logging Protocol:
1. Must force the log record which has both old and new values for an update before the corresponding data page gets to disk(stolen)
2. Must write all log records to disk(force) for a Xact before commit.

### ARIES algorithm
- In the RAM, there is transaction table and dirty page table.
- In stable Storage, there is Logging table
- In disk, there is DB for committed page.

![](media/posts/ar.png)

#### Records
##### Log Records
- LogRecord: prevLSN, XID, type, pageID, length, offset, before-image, after-image
- Data pages: each with a pageLSN
- RAM: 
  - Xact Table: lastLSN, status
  - Dirty Page Table: recLSN
  - flushedLSN

LSNs identify log records, linked into backwards chains per transaction(via prevLSN).

pageLSN allows comparison of data page and log records.
##### DB in disk

#### Checkpointing
Periodically, the DBMS creates a checkpoint, in order to minimize the time taken to recover in the event of a system crash (a quick way to limit the amount of log to scan on recovery).
Write to log:

- Begin checkpoint record: Indicates when chkpt began.
- End checkpoint record: Contains current Xact table and dirty page table.
  - Other Xacts continue to run. So these tables accurate only as of the time of the begin checkpoint record.
  - No attempt to force dirty pages to disk. Effectiveness of checkpoint is limited by the oldest unwritten change to a dirty page.
  - it's good idea to periodically flush dirty pages to disk.

Start from a checkpoint(found via master record)
Three Phases. Need to:
- **Analysis**: Figure out which Xacts committed since checkpoint, which failed.
- **REDO**: redo all actions(repeat history)
- **UNDO**: undo effects of failed Xacts.

## Distributed Recovery -- Two Phase Commit
### two-phase process
#### Phase 1

- The coordinator sends a prepare message to each suboridnate
- On receiving a prepare message a subordinate decides to either commit or abort its sub-transaction. It force-writes an abort or prepare log record and sends yes or no message to the coordinate.

#### Phase 2

- If the coordinator receives all “yes” messages from all subordinates it force-writes a commit log record and then sends commit messages to all subordinates. If it receives even one no message or does not receive a message within some specified time it force-writes an abort log and sends abort message to all subordinates.
- When a subordinate receives an abort message it forcewrites an abort log record and sends an acknowledgement to the coordinator. It aborts the subtransaction.

### Restart after a Failure
On recovery, the recovery node does the following:

- If we have a commit or abort log of T, the status is clear. We redo or undo T as in centralized database systems. The coordinator needs sending messages to its subordinates abort or commit messages until it receives acknowledgements from them.
- If we have only prepare log for T but no commit or abort log record and the site is a subordinate we must repeatedly contact the the coordinator to respond for commit or abort instruction. After receiving a message the rest of actions are 2PC.
- If there are no prepare, commit or abort log records the subordinate aborts the transaction unilaterally. If the site is the coordinator it then has to inform all subordinates to abort their subtransactions.

## Database Architecture

**Parallelization**
Here are some guidelines to effectively benefit from parallelization:

1. Maximize the fraction of your program that can
be parallelized
2. Balance the workload of parallel processes
3. Minimize the time spent for communication

**Replicating Data**

Replicating data across servers helps in:

- Avoiding performance bottlenecks
- Avoiding single point of failures
- And, hence, enhancing scalability and availability

### CAP Theorem
The limitations of distributed databases can be described
in the so called the CAP theorem

- Consistency: every node always sees the same data at any given instance (i.e., strict consistency)
- Availability: the system continues to operate, even if nodes in a cluster crash, or some hardware or software parts are down due to upgrades
- Partition Tolerance: the system continues to operate in the presence of network partitions CAP theorem: any distributed database with shared

CAP theorem: any distributed database with shared data, can have at most two of the three desirable properties, C, A or P

- AP: Best Effort Consistency
  - Example:
    - Web Caching
    - DNS
  - Trait:
    - Optimistic
    - Expiration/Time-to-live
    - Conflict resolution

- CP: Best Effort Availability
  - Example:
    - Majority protocols
    - Distributed Locking (Google Chubby Lock service)
  - Trait:
    - Pessimistic locking
    - Make minority partition unavailable

#### Type of consistency

- Strong Consistency: After the update completes, any subsequent access
will return the same updated value.
- Weak Consistency: It is not guaranteed that subsequent accesses will return the updated value.
- Eventual Consistency
  - Specific form of weak consistency
  - It is guaranteed that if no new updates are made to object, eventually all accesses will return the last updated value (e.g., propagate updates to replicas in a lazy fashion)
  - Variations:
    - Causal consistency: Processes that have causal relationship will see consistent data
    - Read-your-write consistency: A process always accesses the data item after it’s update operation and never sees an older value
    - Session consistency: As long as session exists, system guarantees read-your-write consistency
    - Monotonic read consistency: If a process has seen a particular value of data item, any subsequent processes will never return any previous values
    - Monotonic write consistency: The system guarantees to serialize the writes by the same process
    - Monotonic reads and read-your-writes are most desirable
  - Pros:
    - reduce the load and improve availability

#### Partitioning

- **Data Partitionin**g: Different data may require different consistency and availability
- **Operational Partitioning**: Each operation may require different balance between consistency and availability
- **Functional Partitioning**: System consists of sub-services, Different sub-services provide different balances
- **User Partitioning**: Try to keep related data close together to assure better performance
- **Hierarchical Partitioning**: Different location in hierarchy may use different consistency

#### The BASE Properties
The CAP theorem proves that it is impossible to guarantee strict Consistency and Availability while being able to tolerate network partitions. In particular, such databases apply the BASE properties:

- Basically Available: the system guarantees Availability
- Soft-State: the state of the system may change over time
- Eventual Consistency: the system will eventually become consistent

#### CAP -> PACELC
A more complete description of the space of potential tradeoffs for distributed system:

> If there is a **partition (P)**, how does the system trade off **availability and consistency (A and C)**; **else (E)**, when the system is running normally in the absence of partitions, how does the system trade off **latency (L) and consistency (C)**?

- PA/EL Systems: Give up both Cs for availability and lower latency
- PC/EC Systems: Refuse to give up consistency and pay the cost of availability and latency
- PA/EC Systems: Give up consistency when a partition happens and keep consistency in normal operations
- Keep consistency if a partition occurs but gives up consistency for latency in normal operations

### NoSQL Database

Main characteristics of NoSQL databases include:

- No strict schema requirements
- No strict adherence to ACID properties
- Consistency is traded in favor of Availability

#### Types of NoSQL Database

- Document Stores: Documents are stored in some standard format or encoding
  - Documents can be indexed
  - Eg: MongoDB and CouchDB
- Graph Databases: Data are represented as vertices and edges
  - Graph databases are powerful for graph-like queries
  - Eg:Neo4j and VertexDB
- Key-Value Stores: Keys are mapped to (possibly) more complex value (e.g., lists)
  - Such stores typically support regular CRUD operations
  - Eg: Amazon DynamoDB and Apache Cassandra
- Columnar Databases: Columnar databases are a hybrid of RDBMSs and Key- Value stores
  - Values are queried by matching keys
  - E.g., HBase and Vertica

## Indexing structures

## Relational operators

## Transaction processing

## Performace benchmarks: TPC

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
- $P_{2}$P Databases

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
### Concurrency Control Approach
- **Two Phase Locking(2PL)**: Determine serializability order of conflicting operations at runtime while txns executes
- **Timestamp Ordering(T/O)**: Determine serializability order of txns before they execute. 
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


#### Timestamp Ordering(T/O) 

##### Optimistic Concurrency Control

##### Partition-based Timestamp Ordering
Txns are assigned timestamps based on when they arrive at the DBMS.

Partitions are protected by a single lock:
- Each txn is queued at the partitions it needs.
- The txn acquires a partition's lock if it has the lowest timestamp in that partition's queue.
- The txn starts when it has all of the locks for all the partitions that is will read/write.

**READ PROCESS**
Txns can read anything that they want at the partitions that they have locked. If a txn tries to access a partition that it does not have the lock, it is aborted+restarted.
**WRITE PROCESS**
All updates occur in place.
- Maintain a separate in-memory buffer to undo changes if the txn aborts.

If a txn tries to write a partition that it does not have the lock, it is aborted+restarted.

The Partition-based T/O protocol is fast if:
- The DBMS knows waht partitions the txn needs before it starts.
- Most(if not all)txns only need to access a single partition.

However, Multi-partition txns causes partition to be idle while txn executes.


## Multi-version Concurrency Control

## MVCC Extra

#### Concurrency Control Protocol
- Timestamp Ordering $\implies$ Assign txns timestamps that determine serial order.
- Optimistic Cocurrency Control $\implies$ Three-phase protocol and private workspace for new version
- Two-Phase Locking $\implies$ Txns acquire appropriate lock on physical they can read/write a logical tuple.

#### Version Storage
The DBMS uses the tuples' pointer filed to create a **version chain** per logical tuple.
- this allows the DBMS to find the version that is visible to a particular txn at runtime.
- Indexes always point to the "head" of the chain.

Different storage schemes determine where/what to store for each version. Here shows the approaches.

- Append-Only Storage: New versions are appended to the same table space.
- Time-Travel Storage: Old versions are copied to separate table space.
- Delta Storage: The original values of the modified attributes are copied into a separate delta record space.

#### Garbage Collection
The DBMS needs to remove reclaimable physical versions from the database over time.

- No active txn in the DBMS can "see" that version(SI)
- The version was created by an aborted txn.

Two additional design decision:

- How to look for expired versions
- How to decide when it is safe to reclaim the memory

These are two collection level.

- Tuple level
- Transaction level

**Tuple Level**: find old versions by examining tuples directly. Background Vacuuming vs Cooperative Cleaning.

> **Background Vacuuming**: Separate threads periodically scan the table and look for reclaimable versions. Works with any storage.

> **Cooperative Cleaning**: Worker threads identify reclaimable versions as they traverse version chain. Only works with O2N.


**Transaction Level**: Txns keep trak of their old versions so the DBMS does not have to scan tuples to determine visibility.

In transaction level GC, each txn keeps track of its read/write set. The DBMS determines when all versions created by a finished txn are no longer visible.

#### Index Management
Primary key indexes point to version chain head.
- How often the DBMS has to update the pkey index depends on whether the system creates new versions when a tuple is updated.
-  If a txn updates a tuple’s pkey attribute(s), then this is treated as an `DELETE` followed by an `INSERT`. Secondary indexes are more complicated...

For secondary Index, there are two approaches.
- Logical Pointers: Use a fixed identifier per tuple that does not change. 
  - Requires an extra indirection layer.
  - Primary Key(MySQL) vs. Tuple Id
- Physical Pointers: Use the physical address to the version chain head.
![MVCC Implimentations](/media/posts/adbgc.png)

MVCC is the widely used scheme in DBMSs. Even systems that do not support multi-statement txns(NoSQL) to use it.


#### Concepts & History
At first, we apply 2PL to ensure the consistency of concurrency. However, lock operation will extremely limit the performance and scalability in concurrent database or distributed databse. Therefore, the researchers camp up with the idea of version to get rid of most lock operations. The final aim of multi-version concurrency control is that readers don't block writers and writers don't block readers. 

Protocol was first proposed in 1978 MIT PhD 

First implementations was Rdb/VMS and InterBase at DEC in early 1980s. 

The core solution to avoid lock is snapshot. Based on different 



sed ri ’s/session required required pam_loginuid.so/g ’ / etc/pam .d/sshd pam_loginuid .so/#session



## Another EXPLAINANTION
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