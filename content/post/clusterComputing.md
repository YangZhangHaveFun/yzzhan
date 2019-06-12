---
title : "Notes for Cluster and Cloud Computing(COMP90024)"
date: 2019-04-17T01:37:56+08:00
lastmod: 2019-04-17T01:37:56+08:00
draft: false
tags: ["cluster", "auto-deploying","distributed"]
categories: ["cluster computing related"]
author: "Yang Zhang"
---
## Information Session

**Cloud Characteristics:**

- **On-demand self-service**: A consumer can provision computing capabilities as needed without requiring human interaction with each service provider.
- **Networked access**: Capabilities are available over the network and accessed through standard mechanisms that promote use by hetergeneous client platforms.
- **Resource pooling**: The provider's computing resources are pooled to serve multiple consumers using a multi-tenant model potentially with different physical and virtual resources that can be dynamically assigned and reassigned according to consumer demand.
- **Rapid elasticity**: Capabilities can be elastically provisioned and released, in some cases automatically, to scale rapidly upon demand.
- **Measured service**: Cloud systems automatically control and optimize resource use by leveraging a metering capability at some level of abstraction appropriate to the type of service. 

**Cloud Computing Flavours:**

- Compute Clouds:
  - Amazon Elastic Compute Cloud
  - Azure
- Data Clouds:
  - Amazon Simple Storage Service
  - Google docs
  - iCloud
  - Dropbox
- Application Clouds:
  - App Store
  - Virtual Image Factories
  - Community-specific
- Public/Private/Hybrid/Mobile/Health Clouds

### History

#### Distributed System
- The first focus is on **Transparency** and **Heterogeneity** of computer-computer interactions. (Finding -> Binding -> checking -> invoking) in heterogeneous environment.

#### Grid Computing
- Grid Computing transfer computer-computer focus to organization-organization focus. 
- Grid computing is distinguished from conventional high-performance computing systems such as cluster computing in that grid computers have each node set to perform a different task/application. Grid computers also tend to be more heterogeneous and geographically dispersed (thus not physically coupled) than cluster computers.
- Although a single grid can be dedicated to a particular application, commonly a grid is used for a variety of purposes. Grids are often constructed with general-purpose grid middleware software libraries. Grid sizes can be quite large.

#### Hard of compute Grid
- Information System: What resources are available
- Monitoring and Discovery Systems: What status of those resources
- Job scheduling/resource brokering: Please run these {jobs}
- Virtual organization support
- Security

## Domain Driver

Challenges happen in multiple perspectives in research domains.
- Big data
- Big compute
- Big distribution
- Big collaboration
- Big security

The solution to these challenges focus on `Computing Scaling`, `Network Scaling`.
### Computing Scaling
#### General Analysis
Computing Scaling could be divided into:
- Vertical Computational Scaling:
  - Have faster processors
  - Limits of fundamental physics/matter(nanoMOS)
- Horizontal Computational Scaling:
  - Have more processors: Easy to add more processors but hard to design, develop, test, debug and delpoy.
  - HTC is far more important than HPC

Multiple meaning of "add more":
- Single machine multiple cores
  - Typical laptop/PC/server these days
- Loosely coupled cluster of machines
  - Pooling/Sharing of resources:
    - Dedicated vs available only when not in use by others
    - Web services
- Tightly coupled cluster of machines
  - Typical HPC/HTC set-up (SPARTAN)
    - Many servers in same rack/server room(often with fast message passing interconnects)
- Widely distributed clusters of machines
- Hybrid combinations of the above
  - Leads to many challenges with distributed systems
    - Shared state
    - Message Passing Paradigms(DANGER of delays/lost messages)
#### Mathematical Analysis
Question: If n processors(cores) are thrown at a problem how much faster will it go?
##### Amdahl's Law

Terminologes:

- Basic terms
  - $\sigma$: The time that costs by unparallelable part.
  - $\pi$: The time that costs by parallelable part.
  - N: The number of processors
- Derived terms
  - $T(1)=\sigma+\pi$: Time for serial computation without enhanced by parallelism.
  - $T(N)=\sigma+\frac{\pi}{N}$: Time for N parallel computations with max enhanced by parallelism.
  - S(N)=T(1)/T(N): Time proportion between no parallelism applied and max parallelism applied.
  - $\pi/\sigma=\frac{1-\alpha}{\alpha}$: Time proportion between cost on parallelable part and unparallelable part.
  - $\alpha$: The proportion of unparallelable part.
  - $S=T(1)/T(N)=\frac{1}{\alpha+(1-\alpha)/N}\approx\frac{1}{\alpha}$: Speed up proportion.

Equation:
$S = 1/\alpha$. For example, 95% of paralleled program with infinite numbers of cores will lead to 20 times faster than unparalleled program  
##### Gustafson-Barsis's Law

Different Terminologies:

- Basic terms:
  - $\pi$: The time that costs by parallelable part for a single process.
- Derived terms
  - $T(1)=\sigma+N\pi$: Time for serial computation without enhanced by parallelism.
  - $T(N)=\sigma+\pi$: Time for N parallel computations with max enhanced by parallelism.
  - $\alpha$: Fraction of running time sequential program spends on parallel parts. 

Equation:
$S(N)=\alpha+N(1-\alpha)$
Speed up S using N processes is given as a linear formula dependent on the number of processes and the fraction of time to run sequential parts.

##### Comparison
**Amdahl's Law** suggests that with limited task, speed up could not be too fast. 
**Gustafson-Barsis's Law** suggests that with enough processors and remaining tasks, speed up will always meet the requirement.

> Correctness in parallelisation requires synchronisation. Synchronization and atomic operations causes loss of performance and communication latency.

### Further parallel improvement
#### Basic Architecture in a simplest level
A computer comprises:
- CPU for executing programs
- Memory that stores/executing programs and related data
- I/O systems
- Permanent Storage for read/writing data into out of memory

Important issue is to balance the components especially for HPC systems.

To deal with data, here are four different kinds of combination.

|               | Single Instruction           | Multi Instruction  |
| ------------- |:-------------:| -----:|
| Single Data   | SISD | MISD |
| Multiple Data | SIMD      |   MIMD |

- Single Instruction, Single Data Stream (SISD): 
  - Single control unit (CU/CPU) fetches single Instruction Stream from memory.
  - Sequential computer which exploits no parallelism in either the instruction or data streams
- Multiple Instruction, Single Data stream (MISD):
  - **Parallel** computing architecture where many functional units (PU/CPU) perform different operations on the same data 
  - Example: fault tolerant computer architectures, multiple error checking on the same date source
-  Single Instruction, Multiple Data stream (SIMD):
   -  focus is on data level parallelism, only a single process (instruction) at a given moment(**Concurrency**)
   -  multiple processing elements that perform the same operation on multiple data points simultaneously
   -  Example: to improve performance of multimedia use such as for image processing
-  Multiple Instruction, Multiple Data stream (MIMD)
   -  Number of processors that function **asynchronously** and independently.
   -  Machines can be shared memory or distributed memory categories.
   -  Example: HPC.

#### Approaches for parallelism in different level
- Explicit vs Implicit parallelism
  - Explicit Parallelism: programmer is responsible for parallelism effort. 
  - Implicit Parallelism: Compiler is responsible for identifying parallelism and scheduling of calculations and the placement of data.  
- Hardware: Multiple cores that can process data and perform computational tasks in parallel. To address the issue that CPU not doing anything whilst waiting for caching. Many chips have mixture cache L1 for single core, L2 for pair cores and L3 shared with all cores. 
- Operating System:
  - Compute parallelism
    - Processes
    - Threads
  - Data parallelism
    - Caching
- Software/Application: Programming language supports a range of parallelisation/concurrency features.

#### Data Parallelism Approaches
- Distributed Data/Distributed File System -> CAP(Consistency, Availability, Partition Tolerance)
- To guarantee the data/transaction is reliable, ACID(Atomicity, Consistency, Integrity, Durability) need to be satisfied.

#### Erroneous Assumptions of Distributed Systems

- The network is reliable
  - **Reliable** means that there is guarantee that data can send and arrive successfully over the network.
  - **Reliable** means that lower layers in the networking stack protect me from these issues
- Latency is zero
  - **Latency** means that the time cost by transmission. Latency becomes more and more significant if distance become further.
- Bandwidth is infinite
  - **Bandwidth** means that amount of data could be sent in a unit of time.
- The network is secure
  - SQL injections
  - Man in the middle attacks
  - Masquerade
  - Trojans/Viruses
- Topology doesn't change
  - **Topology** means that the route taken from source to destination. Routine selected will decide:
    - Latency
    - IP
    - Service 
  - Routine is not fixed unless specific routing protocols selected.
- There is  one administrator
  - **Administrator**: There should be an adiminstrator for every module.
- Transport cost is zero
- The network is homogeneous
  - There are multiple approaches to design parallel or distributed systems
    - No single algorithm
    - No single technical solution
    - Eco-system of approaches explored over time and many open research questions/challenges
- Time is ubiquitous
  - Protocols: NTP synchronizes participating computers to within a few milliseconds of Coordinated Universal Time(UTC).

#### Design of parallel Program
The flow of parallel program design could be divided into four key points.

- Partitioning
  - Primary idea: decomposition of computational activities and data into smaller tasks
  - Strategies: 
    - Master-worker/Slave Model: Master decomposes the problem into samll tasks, distributes to workers and gathers partial results to produce the final result.
    - SPMD: common exploited model(Mapreduce). The main idea is that each process executes the same piece of code, but on different parts of the data.
    - Pipeline: Suitable for applications involving multiple stages of execution, typically operate on large number of datasets.
    - Divide and conquer: A problem is divided into two or more sub problems, and each of these sub problems are solved independently, and their results are combined.(eg: merge sort)
    - Speculation: Used when it is quite difficult to achieve parallelism through the previous paradigms.
- Communication
  - informaton stream and coordination among tasks that are created in the partitioning stage.
- Agglomeration
  - Tasks and communication created in above stages are evaluated for performance and implementation cost
- Mapping/Scheduling
  - Assign tasks to processors such that job completion time is minimized and resource utilization is maximized.


## Parallel System, Distributed Computing and HPC/HTC
### HTC (high throughput computing)
HTC jobs generally involve running multiple independent instances of the software on multiple processors, at the same time. Serial systems are suitable for these requirements.

HTC is worthwhile when one needs to:

- run many jobs that are typically similar but not highly parallel;
- run the same program with varying inputs;
- run jobs that do not communicate with each other;
- execute on physically distributed resources using grid-enabled technologies;
- make use of many computing resources over long periods of time to accomplish a computational task.

### HPC (high performance computing)
HPC jobs generally involve running a single instance of parallel software over many processors. Results at various instances throughout the computation are communicated among the processors, requiring a parallel environment.

HPC: use when one needs to:

- run jobs where rapid communication of intermediate results is required to perform the computations;
- make intense use of large amounts of computing resources in relatively short time periods.
## Cloud Computing
### Concepts & Development
Before cloud computing, the main issue is that capacity and utilization could not tightly meet. 
- Even if peak load can be correctly anticipated, without **elasticity** we waste resources during nopeak times.
- Underprovisioning 1: potential revenue from users not serverd
- Underprovisioning 2: some users desert the site permanently after experiencing poor service; this attrition and possible negative press result in a permanent loss of a portion of the revenue stream.

Then cloud computing starts busting developing.
> Cloud computing is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources(networks, servers, storage, applications, services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

Common Cloud Model could be divided into three aspects:

- Deployment Models:
  - Private
    - Pros: 1. Control 2. Consolidation of resources 3. Easier to secure 4. More trust
    - Cons: 1. Not relevance to core business 2. Staff/Management overheads 3. Hardware obsolescence 4. Over/under utilisation challenges.
  - Community
  - Public
    - Pros: 1. Utility computing 2. Can focus on core business 3. Cost-effective 4. Right-sizing 5.Democratisation of computing
    - Cons: 1.Security 2. Loss of control 3. Possible lock-in 4. Dependency of Cloud provider continued existence
  - Hybrid
    - Pros: 1.Cloud-bursting
    - Cons: Difficult to 1. move data.resources when needed 2. decide what data can go to public cloud 3. public cloud compliant with PCI-DSS
- Delivery Models: Separation of Responsibilities
  ![Delivery Model](/media/posts/delivery_models.png)

x-as-a-Service:
- Testing-as-a-Service
- Management/Governance-as-a-Service
- **Software-as-a-Service**
  
  - Examples: 
    - Gmail
    - On-live
    - Microsoft Office 365
- Process-as-a-Service
- Information-as-a-Service
- Database-as-a-Service
- Storage-as-a-Service
- **Infrastructure-as-a-Service**
  
  - Examples:
    - Amazon Web Services
    - Oracle Public Cloud
    - Rackspace Cloud
    - NeCTAR
- Integration-as-a-Service
- Security-as-a-Service
- **Platform-as-a-Service**
  
  - Examples:
    - Google App Engine
    - Microsoft Azure
    - Amazon Elastic MapReduce

### Auto-Deployment ---- Ansible
Reasons for auto-deployment:
- Esay to forget what software you installed, and what steps you took to configure the system
- Manual process is error-prone, can be non-repeatable
- Snapshots are monolithic - provide no record of what has changed.

Automation provides:
- A record of what you did
- Codifes knowledge about the system
- Makes process repeatable
- Makes it programmable -- "Infrastructure as Code"

**Configuration management(CM) tools**
> Configuration management refers to the process of systematically hadnling changes to a system in a way that it maintains integrity over time.

Automation is the mechanism used to make servers reach a desirable state.
#### Ansible
> Ansible is an automation tool for configuring and managing computers.

Pros: 
- Easy to learn
- Minimal requirements
- Idempotent(repeatable)
- Extensible
- Supports push or pull
- Rolling updates
- Inventory management
- Ansible Vault for encrypted data

## ReST and Docker(Containalization)
### Web Service 

#### Basic Idea -- Service-oriented Architecture
When an architecture is completely contained within the same machine. components can communicate directly. However, when components are distributed such a direct approach typically can not be used.

**Service**(combination and commonality) are often used to form a **Service-oriented Architecture(SoA)**.

**Service-oriented Architecture(SoA) Core Ideas**:

- A set of externally facing services that a business wants to provide to their customers or partners/collaborators.
- An architectural pattern based on service providers, one or more brokers, and service requestors based on agreed service descriptions.
- A set of architectural principles, patterns and criteria that support modularity, encapsulation, loose coupling, separation of concerns, reuse and composability.
- A programming model complete with standards, tools and technologies that supports development and support of services(could be many flavours of service)
- A middleware solution optimized for service assembly, orchestration, monitoring, and management. Can include tools and approaches that combine services together.

**Service-oriented Architecture(SoA) Design Principle**:

- Standardized service contract: Services adhere to a communications agreement as defined collectively by one or more service-description documents.
- Service loose coupling: Services maintain a relationship that minimizes dependencies and only requires that they maintain an awareness of each other.
- Service abstraction: Beyond descriptions in the service contract, services hide logic from the outside world.
- Service autonomy: Services have control over the logic they encapsulate.
- Service statelessness: Services minimize resource consumption by deferring the management of state information when necessary.
- Service discoverability: Services are supplemented with communicative meta data by which they can be effectively discovered and interpreted.
- Service composability: Services are effective composition participants, regardless of the size and complexityh of the composition.
- Service Granularity: A design consideration to provide optimal scope at the right granular level of the business functionality in a service operation.
- Service normalization
- Service optimization
- Service relevance
- Service encapsulation
- Service location transparency

#### SoA for Web
Web services implement SoA with two main flavours

- SOAP-based Web Services
- ReST-based Web Services
- (Both flavours to call services over HTTP)

and many other flavous
- Geospatial Services(WFS, WMS, WPS)
- Health services(HL7)
- SDMX(Statistical Data Markup exchange)

##### SOAP/WS
**SOAP(Simple Object Access Protocol)** is built upon Remote Procedure Call paradigm, a language independent function call that spans another system.

SOAP/WS is a stack of protocols that covers every aspect of using a remote service, from service discovery, to service description, to the actual request/response.

- Can use HTTP and other protocols
- Build up on remote procedure call (a language independent function call that spans another system)
- Covers all aspects of a remote services including discovery, description, request/response.
- WS-* (security) goes into SOAP headers for additional functionality. 
##### RESTful/WS and ROA Architecture
> **ReST(Representation State Transfer)** is intended to evoke an image of how a well-designed Web app behaves. A network for web pages, where the user progresses through an application by selecting links and resulting in next page then being transferred to the user and rendered for their use.
- REST leverages less bandwidth, making it more suitable for internet usage.

Steps For $Client \rightleftharpoons Resource$, Client sends "http://amazon.com/product/123" and Resource return Product.html:

1. Client requests Resource through Identifier(URL)
2. Server/proxy sends representation of Resource
3. This puts the client in a certain state.
4. Representation contains URLS allowing navigation.
5. Client follows URL to fetch another resource.
6. This transitions client into yet another state.
7. Representational State Transfer

> **A ROA(Resource-Oriented Architecture) is a way of turning a problem into a RESTful web service: an arrangement of URIs, HTTP, and XML that works like the rest of the Web.**

A resource is anything that's important enough to be referenced as a thing in itself.

- **PUT** should be used when target resource URL is known by the client.
- **POST** should be used when target resource URL is server generated

**ReST** - Uniform Interface

- **Identification of Resources**: All important resources are identified by one(uniform) resource identifier mechanism(HTTP URL)
- **Manipulation of Resources through representations**: Each resource can have one or more representations. Such as application/xml, application/json, text/html. Clients and Servers negotiate to select representation.
- **Self-descriptive messages**: Requests and resources contain not only data but additional headers describing how the content should be handled. Such as if it should be cached, authentication requirements, etc. Access methods mean the same for all resources
- Keywords: **HTTP**, **GET**, **HEAD**, **OPTIONS**, **PUT**, **POST**, **DELETE**, **CONNECTION**, **TRACE**, **PATCH**.
  
> HTTP Methods can be 
> - **safe**: Do not change, repeating a call is equivalent to not making a call at all $GET, OPTION, HEAD$
> - **Idempotent**: Effect of repeating a call is equivalent to making a single call. $PUT, DELETE$
> - **Neither**: $POST$ 


##### ReST 2.0
Motivation: Everything as a service(EaaS) paradigm, Vast number of entities and services, Link services together to create workflows and mashups.

Extend the API(Application Programming Interface) notation to facilitate the sharing and usage of services among developers, testers, and in some cases end users.

##### Comparison between SOA and ROA
- An application built with a Service Oriented Architecture is more a 'Facade', e.g. it combines or composes its outgoing functionality based on functionality that is in the services it uses 'behind the screens' (possibly over the network). E.g. its core processing consists of calling external services, supplying them with parameters, and combining the results with possibly some extra processing or algorithms for the user.
- An application built with a Resource Oriented Architecture does more of its processing internally (e.g. as opposed to calling external components) but uses external resources as input. E.g. its core processing consists of retrieving static resources and then doing more calculating internally.
## Big Data and Related Technologies

### DBMS design in Big Data Environment
Four "Vs" relates to Big data:

- **Volume**: volume is one of most important criteria.
- **Velocity**: the frequency at which new data is being brought into system and analytics performed.
- **Variety**: the variability and complexity of data schema. The more complex the data schemas you have, the higher the probability of them changing along the way, adding more complexity.
- **Veracity**: the level of trust in the data accuracy(provenance); the more diverse sources you have, the more unstructured they are, the less veracity you have.

Why Big Data need special database rather than Relational DBMSs
- RDBMS finds it challenging to handle such huge data volumes. To address this, RDBMS added more central processing units (or CPUs) or more memory to the database management system to scale up vertically.
- the majority of the data comes in a semi-structured or unstructured format from social media, audio, video, texts, and emails. However, the second problem related to unstructured data is outside the purview of RDBMS because relational databases just can’t categorize unstructured data. They’re designed and structured to accommodate structured data such as weblog sensor and financial data.
- “big data” is generated at a very high velocity. RDBMS lacks in high velocity because it’s designed for steady data retention rather than rapid growth.
  
DBMSs for Distributed Environment
- A key-value store is a DBMS that allows the retrieval of a chunk of data given a key: fast but crude. (Redis, PostgreSQL,Hstore)
- A BigTable DBMS stores data in columns grouped into column families, with rows potentially containing different columns of the same family.(Apache Cassandra, Apache Accumulo)
- A Document-oriented DBMS stores data as structured documents, usually expressed as XML or JSON. (Apache CouchDB, MongoDB) 

### Clustered NoSQL DBMS Comparison
Clusters needs to 
- Distribute the computing load over multiple computers. (eg. for availability)
- Store multiple copies of data. (eg. to achieve redundancy)
#### Sharding
> Sharding is the partitioning of a database "horizontally"(ie. the database rows or documents) are partitioned into subsets that are stored on different servers. Every subset of rows is called a shard.

- Number of shards:
  - larger than the numebr of replica
  - the max number of nodes is equal to the number of shards
- Advanage of shards: 
  - Improve the performance through the distribution of computing load across nodes.
  - Make it easier to move data files around(eg. when adding new nodes to cluster)
- Strategy of shards:
  - Hash sharding: distribute evenly across the cluster.
  - Range sharding: similar rows are stored on the same node. 

#### Replication
> Replication is the action of storing the same row(or document) on different nodes to make the database fault-tolerant.

Combined relication and sharding aim to maximize the availability while maintaining a minimum level of data safety. 

#### CouchDB Cluster Architecture
![CouchDb Cluster](/media/posts/couchdb.png)
CouchDB sharding process:
- Sharding is done on every node(One node contains multiple shards)
- All nodes answer requests(read or write) at the same time
- Nodes can be added/removed easily, and their shards are re-balanced automatically upon addition/deletion of nodes.
- If node request don't have respective node, node will request another node for document and return it to client.
#### MongoDB Cluster Architecture
![MongoDb Cluster](/media/posts/mongodb.png)
A mongoDB sharded cluster consists of three components:
- shard: Each shard contains a subset of the sharded data. Each shard can be deployed as a replica set.
- mongos: The mongos acts as a query router, providing an interface between client application and the sharded cluster.
- config server: Config servers store metadata and configuration settings for the cluster.

Sharding process:
- Sharding is done at the replica set level(One node contains one shard).
- Write requests is based on primary node in a replica set, Read requests depends on configuration, be answered by every node.
- Updates flow only from primary to secondary.
- Read request **by default** bases on primary node, it can be changed by modifying the configuration.

#### MongoDB vs CouchDB Clusters
- MongoDB cluster is considerably more complex.
- MongoDB cluster is less available(only primary nodes can talk to client)
- MongoDB software routers(Mongos) must be embedded in application servers, while any HTTP client can connect to CouchDB.
- Losing two nodes out of three 
  - in CouchDB, losing access to one quarter of data.
  - in MongoDB, losing access to half the data.(Actually there are 10 nodes.)

#### CAP Theorem
> Consisteny: every client receiving an answer receivers the same answer from all nodes in the cluster.

> Aailability: every client receives an answer from any node in the cluster

> Partition-tolerance: the cluster keeps on operating when one or more nodes cannot communicate with rest of the cluster.

![CAP](/media/posts/cap.png)
##### Consistency and Availability: Two phase commit
> A two-phase commit is a standardized protocol that ensures that a database commit is implementing in the situation where a commit operation must be broken into two separate parts.

 The two-phase commit is implemented as follows:
 - Phase 1: Each server that needs to commit data writes its data records to the log. If a server is unsuccessful, it responds with a failure message. If successful, the server replies with an OK message.
 - Phase 2:  This phase begins after all participants respond OK. Then, the coordinator sends a signal to each server with commit instructions. After committing, each writes the commit as part of its log record for reference and sends the coordinator a message that its commit has been successfully implemented. If a server fails, the coordinator sends instructions to all servers to roll back the transaction. After the servers roll back, each sends feedback that this has been completed.

Two-Phase Commit **enforce consistency** and **reduce availability**. It is a good solution when the cluster is co-located, less good when it is distributed.
##### Consistency and Partition-Tolerance: Paxos
In Paxos, every node is either a proposer or an accepter:
- a proposer proposes a value (with a timestamp)
- an accepter can accept or refuse it

Paxos clusters can recover from partitions and maintain consistency, but the smaller part of a partition will not send responses, hence the availability is compromised.
##### Availability and Partition-Tolerance: MVCC
In MVCC, concurrent updates are possible without distributed locks, since the updates will have different revision number; the transaction that completes last will get a higher revision number, hence will be considered as the current value.

CouchDB uses MVCC and MongoDB uses a mix of two-phase commit(for replicating data from primary to secondary nodes) and Paxos-like(to elect a primary node in a replica-set)
### MapReduce Algorithm
- MAP: distributes data across machines
- REDUCE:  hierarchically summarizes them until the result is obtained.
## Big Data Analytics
### Introduction
Challenges of Big Data Analytics
- Reading and writing distributed dataset
- Preserving data in the presence of failing data nodes 
- Supporting the execution of MapReduce tasks
- Being fault-tolerant
- Coordinating the execution of tasks across a cluster.
### Apache Hadoop
Hadoop
: an open source software platform for distributed storage and distributed processing of very large data sets on computer clusters built fro commodity hardware.

The core of Hadoop is a fault tolerant file system that has been explicitly designed to span many nodes.
#### Hadoop Distributed File System(HDFS)
HDFS break large file into smaller blocks. However, HDFS blocks are much larger than blocks used by an ordinary file system. 
Reansons:

- Reduced need for memory to store information about where the blocks are(some kind of metadata)
- More efficient use of the network(with a large block, a reduced number network connections needs to be kept open)
- Reduced need for seek operations on big files
- Efficient when most data of a block have to be processed

A HDFS file is a collection of blocks stored in **datanodes**, the corresponding metadata is stored in **namenodes**.
#### Hadoop Resource Manager(YARN) --- MapReduce Task Manager
YARN deals with executing MapReduce jobs on a cluster.
- Central Resource Manager (on the master)
- Node Manager (on slave machines)

When a MapReduce job is scheduled for execution on a Hadoop cluster, YARN starts Application Master that negotiates resources with the Resource Manager and starts Containers on the slave nodes.

### Apache Spark
Spark
: A fast and general engine for large-scale data processing.

Hadoop MadpReduce works well towards performing relatively simple jobs on large datasets. Complex jobs requires strong incentive for caching data in memory and in having finer-grained control on the execuion of jobs. 

Spark can operate within the Hadoop architecture, using YARN and Zookeeper to manage computing resources, and storing data on HDFS.

One of the strong points of Spark is the tightly-coupled nature of its main components:

![Biased Locking](/media/posts/spark_ar.png)

#### Spark Runtime Architecture
##### Included Components:

- Job: the data processing that has to be performed on a dataset.
- Task: a single operation on a dataset
- Executors: the processes in which tasks are executed
- Cluster Manager: the process assigning tasks to executors
- Driver program: the main logic of the application
- Spark application: Driver program + Executors
- Spark Context: the general configuration of the job

##### Local Mode
In local mode, every Spark component runs within the same JVM. However, the Spark application can still run in parallel, as there may be more than one executor active.
![Biased Locking](/media/posts/spark_lm.png)
##### Cluster Mode
In cluster mode, every component is executed on the cluster. Job can run autonomously. This is the common way of running non-interactive Spark jobs.
![Biased Locking](/media/posts/spark_cluster.PNG)
##### Client Mode
In client mode, the driver program talks directly to the executors on the worker nodes. Therefore, the machine hosting the driver program has to be connected to the cluster until job completion. Client mode must be used when the applications are interactive.
![Biased Locking](/media/posts/spark_client.png)
#### Resilient Distributed Dataset(RDD)
- Resilient: data are stored redundantly
- Distributed: data are split into chunks, and these chunks are sent to different nodes
- Dataset: a dataset is just a collection of objects, hence very generic

Properties of RDD:
- immutable
- transient
- lazily-evaluated: Declared transformation will no be executed immediately. Until the action found and executed, the tranformation will start executing.  

**Transformation is executed only after action is executed.**
```Java
/* 
 *  Example of Transformations of RDDs
*/

//selects elements from an RDD
rdd.filter(lambda); 

//returns an RDD without duplicated elements
rdd.distinct();

//merges two RDDs
rdd.union(otherRdd);

//returns elements common to both 
rdd.intersection(otherRdd)

//removes elements of otherRdd
rdd.substract(otherRdd)

//returns the Cartesian product of both RDDs
rdd.cartesian(otherRdd)

rdd.map(lambda)
rdd.flatMap(lambda)
rdd.reduceByKey(lambda)
rdd.join(otherRdd)
```

---

```Java
/* 
 *  Example of Action of RDDs
*/

//returns all elements in an RDD
rdd.collect();

//returns the number of elements in an RDD
rdd.count();

//applies the function to all elements repeatedly, resulting in one result(say, the sum of all elements)
rdd.reduce(lambda)

//applies lambda to all elements of an RDD
rdd.foreach(lambda)

//save files
sf.saveAsTextFile("./counts");
```

## Cloud Underpinning and Other Things
### Virtualization
Virtual Machine Monitor/Hypervisor
: The virtualisation layer between the underlying hardware the virtual machines and guest operating systems it supports.

Virtual Machine
: A representation of a real machine using hardware/software that can host a guest operating system

Guest Operating System
: An operating system that runs in a virtual machine environment that would otherwise run directly on a separate physical system.

$User\ space\rightleftharpoons Kernel\ Space$ by context switch.

Inside the virtual machine, there are Virtual Network Device, VHD(Virtual Hard disk), VMDK(Virtual Machinie Disk), qcow2(QEMU Copy on Write)

#### Motivation
- Server Consolidation: to improve utilization and reduce energy consumption
- Personal virtual machines can be created on demand
- Security/Isolation
- Hardware independency
#### Classification of Instructions
- Privileged Instructions: Instructions that trap if the processor is in user mode and do not trap in kernel mode
- Sensitive Instructions: Instructions whose behaviour depends on the mode or configuration of the hardware.(different behaviour in different mode)
- Innocuous Instructions: instructions that are neither Privileged nor Sensitive

> For any conventional third generation computer, a virtual machine monitor may be constructed if the set of **sensitive instructions** for that computer is a subset of the set of of **privileged instructions**.

#### Aspect of VMMs
##### Full virtualisation
Intercept every system-level calls and try to process and deal with it.

- User Apps directly executes code with host hardware
- Privileged instruction by Guest OS is translated to hardware specific instructions.

. Guest OS uses extra rings, VMM traps privileged instructions and translates to hardware specific instructions.

Pros:
- Guest is unware it is executing within a VM
- Guest OS need not be modified
- No hardware or OS assistance required
- Can run legacy OS

Cons:
- can be less efficient

![Biased Locking](/media/posts/fv.png)
##### Para-virtualisation
provide some available interface for sensitive calls.
Pros:
- Lower virtualisation overheads
- Performance

Cons:
- Need to modify guest OS
- Less portable
- Less compatibility
##### Hardware-assisted virtualisation
Hardware provides architectural support for running a Hypervisor
- New processors typically have this
- Requires that all sensitive instructions trappable

Pros:
- Good performance
- Easier to implement
- Advanced implementation supports hardware assisted DMA, memory virtualization

Cons:
- Needs hardware support

##### Binary Translation
Trap and execute occurs by scanning guest instruction stream and replacing sensitive instruction with emulated code
Pros:
- Guest OS need not be modified
- No hardware or OS assistance required
- Can run legacy OS

Cons:
- Overhead
- Complicated
- Need to replace instructions "on-the-fly"
##### Bare Metal Hyperviosr
VMM runs directly on actual hardware
- Boots up and runs on actual physical machine
- VMM has to support device driver
##### Hosted Virtualization
VMM runs on top of another operating system
##### Operating System Level Virtualisation
It's a lightweight VMs. Instead of whole-system virtualisation, the OS creates mini-containers
- A subset of the OS is often good enough for many use cases
- Similar to an advanced version of "chroot"

Pros:
- Lightweight
- Many more VMs on same hardware
- Can be used to package applications and all OS dependencies into container.
- Uses same resources as other containers

Cons:
- Can only run apps designed for the same OS
- Cannot host a different guest OS
- Can only use native file systems
- Uses same resources as other containers

#### Memory Virtualisation
- Conventional case, page tables store the logical page number and physical page number mappings
- In VMM case, VMM maintains shadow page tables in lock-step with the page tables. Additional management overhead is added.

### OpenStack Components
Services:
- Compute Service(code-named Nova)
- Image Service(code-named Glance)
- Block Service(code-named Cinder)
- Object Service(code-named Swift)
- Security Service(code-named Keystone)
- Orchestration Service(code-named Heat)
- Network Service(code-named Neutron)
- Container Service(code-named Zun)
- Database Service(code-named Trove)
- Dashboard Service(code-named Horizon)
- Search Service(code-named Searchlight)

Nova
- Manages the lifecycle of compute instances in an OpenStack environment
- Responsibilities include spawning, scheduling and decommissioning of virtual machines on demand 
- Virtualisation agnostic

### Serverless(Function as a Service---FaaS)
FaaS is also know as Serverless computing. The idea behind Serverless/Faas is to develop software applications without bothering with the infrastructure.

It's Server-unseen rather than Server-less. A FaaS service allows functions to be added, removed, updated, executed, and auto-scaled. FaaS is an extreme form of microservice architecture.

Pros: 
- Simpler deployment
- Reduced computing costs
- Reduced application complexity due to loosely-coupled architecture

A function that does not modify the state of the system is said to be side-effect free. A function that changes the system somehow is not side-effect free(eg. a function that writes to the file system the thumbnail of an image)

Side-effect free functions can be run in parallel. However, side-effect function is inevitable in complex system.

#### Stateful/Stateless Function 
Stateful Function
: is one whose output changes in relation to internally stored information(its input cannot entirely predict its output).

Stateless Function
: is one that does not store information internally.

#### Synchronous/Asynchronous Function
By default functons in FaaS are synchronous, hence they return their result immediately. 

Asynchronous functions return a code that informs the client that the execution has started, and then trigger an event when the execution completes.
## Security and Clouds
### Authentication
Authentication
: is the establishment and propagation of a user's identity in the system.

- Does not check what user is allowed to do, only that we know who they are.
- Masquerading danger
- Public Key Infrastructures(PKI)

Certification Authority -- Central component of PKI
CA's responsibilities:
- policy and procedures
  - Processes that should be followed by users, organisations, service providers.
- Issuing certificates
  - Often need to delegate to local Registration Authority(Prove who you are)
- Revoking certificates
  - Certificate Revocation List(CRL) for expired/compromised certificates
- Storing, archiving
  - Keeping  track of existing certificates, various other information

- PKI is an arrangement that binds public key with respective identities of entities(like people and organization). 
- The binding is established through a process of registration and issurance of certificates at and by a certificate authority
- The PKI role that assures valid and correct registration is called a registration authority(RA). RA is responsible for accepting requests for digital certificates and authenticating the entity making the request.

### Authorisation
Authorisation
: is concerned with controlling access to resources based on policy.

There are many approaches for authorisation
- Group Based Access Control
- Role Based Access Control
- Identity Based Access Control
- Attribute Based Access Control

Authorization defines what they can do and define and enforce rules. It is realized through **Virtual Organization(VO)**.

- Provide conceptual framework for rules and regulations for resources to be offered/shared between VO members.
- Different domains place greater/lesser emphasis on expression and enforcement of rules and regulations

RBAC
- Basic idea is to define:
  - Roles: roles are often hierachical
  - Actions: allowed/not allowed for VO members
  - Resources: comprising VO infrastructure(computer, data)
- Policy consists sets of these rules: $Role\times Action \times Target$ 
  - eg. Can user with VO role X invoke service Y on resource Z
- Policy engines consume this information to make access decisions

### Other Cloud Security Challenges
- Single sign-on
  - The grid model needed but not solved for Cloud-based IaaS currently.
- Auditing: logging, instrusion detection, auditing of security in external computer facilities.
  - well established in theory and practice and for local systems.
- Deletion and Encryption
  - Data deletion with no direct hard disk
- Liability
- Licensing
  - Challenges with the Cloud delivery model
- Workflows
- The Ever Changing Techinal/Legal Landscape

---

### Auto-deploying by Ansible
### Docker
#### Background Introduction
##### Virtualization vs. Containerization
The advantages of virtualization, such as application containment and horizontal scalability come at a cost: resources.

The containerization allows virtual instances to share a single host OS to reduce wasted resources since each container only holds the application and related binaries. The rest are shared among the containers.
![Biased Locking](/media/posts/vmcon.png)
##### Container
Container is similar to concept of resouce isolation and allocation as a virtual machine.

Without bunding the entire hardware environment and full OS.

##### Container Orchestration Tools
Container orchestration technologies provides a framework for integrating and managing container at scale.
- Features:
  - Networking
  - Scaling
  - Service discovery and load balancing
  - Health check and self-healing
  - Security  
  - Rolling updates
- Goals:
  - Simplify container management processes
  - Help to manage availability and scaling of containers
#### Introduction to Docker
Docker is by far the most successful containerization technology. It uses resource isolation features of the Linux kernel to allow independent "containers" to run within a single Linux instance.
##### Concepts
- **Container**:a process that behaves like an independent machine, it is a runtime instance of a docker image.
- **Image**: a blueprint for a container.
- **Dockerfile**: the recipe to create an  image
- **Registry**: a hosted service containing repositories of images. (Docker Hub)[https://hub.docker.com]
- **Tag**: a label applied to a Docker image in a repository.
- **Docker Compose**: Compose is a tool for defining and running multi-containers Docker applications.
- **Docker SWARM**: a standalone native clutering / orchestration tool for Docker.
##### Manage Data in Docker
By default, data inside a Docker container won't be persisted when a container is no longer exist.
Docker provides two options for containers to store files on the host machine so that the files are persisted even after the container stops.
- Docker volumes(Managed by Docker, /var/lib/docker/volume/)
- Bind mounts(Managed by user, anywhere on the file system)
![Docker data management](/media/posts/dockermount.png)

##### Networking
- Network mode "host": every container uses the host network stack(share the same IP address). Ports cannot be shared across container(Linux only)
- Network mode "bridge": container can re-use the same port, as they have different IP addresses, and expose a port of thier own that belongs to the hosts, allowing the container to be somewhat visible from the outside.

##### Extra
Normal Web Service requires multiple dependencies such as PHP, MySQL, Redis, Node, etc. Ususally we will install the dependencies manually, which however introduces several problems.

- Different services require the same software with different versions.
- Different services in a single environment will modify the file at the same time, such as config of system, nginx
- The same service needs to be deploy manually, which greatly waste the resources.

The following solution is to pack the whole system, which introduces the virtualization technology. It saves the resources but there are lots of other problems.
- Packed virtual machine file including image is pretty large.
- Packed virtual machine file 
- The process of pack could not be automatic.


### CouchDB