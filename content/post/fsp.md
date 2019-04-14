---
title : "FSP and LTSA"
date: 2019-04-02T01:37:56+08:00
lastmod: 2019-04-08T01:37:56+08:00
draft: false
tags: ["SWEN90004", "Learning"]
categories: ["Subjects reviews in Uni Melb"]
author: "Yang Zhang"
---
## Introduction to FSP
When dealing with large code bases, identifying, locating, and removing concurrency problems can be a nightmare. The size of the system and the number of possible interleavings and synchronisations become so large that it is difficult to even understand a single problem, isolate it, figure out how to fix it, or prevent it in the first place.

s. In particular, we will look at a language called **Finite State Processes (FSP)**, based on the well-known **Communicating Sequential Processes (CSP)** and **Calculus of Communicating Systems (CCS)**.

We shall use finite state machines as models of programs. Our particular type of state machine is the **labelled transition system (LTS)**

The LTS only specifies the sequence of legal actions in system.

Consequently, process models are described in an algebraic language called **finite state processes (FSP)**.

Non-termination is common in concurrent, real-time systems.

## FSP syntax
### the action prefix
**The action prefix operator, “->”, is fundamental**:
> “If x is an action and P a process then the action prefix x -> P describes a process that initially engages in action x and then behaves exactly as described by P.”

The “->” operator always has an atomic action as the left operand, and a process as the right operand.

eg:
```Java
TRAFFIC_LIGHT = ( green - > yellow - >red - > TRAFFIC_LIGHT ).
```

**Atomic actions use lower case; process names use upper case.**

The traffic light behaviour could be modelled using more than one
process names.
```Java
TRAFFIC_LIGHT = GREEN ,
GREEN = ( green -> YELLOW ) ,
YELLOW = ( yellow -> RED ) ,
RED = ( red -> GREEN ).
```

**Note the use of “,” and “.” at the end of lines. The “,” indicates that the processes named GREEN, YELLOW, and RED are part of and local to the definition of the process named TRAFFIC LIGHT.**

### Choice Option
**The choice operation, “|” is used to describe a process that can execute more than one possible sequence of actions.**
> “If x and y are actions, then (x -> P | y -> Q) describes a process which initially engages in either of the actions x or y. After the first action has occurred, the subsequent behaviour is described by P if the first action was x and Q if the first action was y.”

example: This describes a traffic light with a button for a pedestrian, which turns the light red. If the button is not pushed, the light remains green:
```Java
TRAFFIC_LIGHT = ( button -> YELLOW | none -> GREEN ) ,
GREEN = ( green -> TRAFFIC_LIGHT) ,
YELLOW = ( yellow -> RED ) ,
RED = ( red -> TRAFFIC_LIGHT ).
```

This example may also raise a question: What is the input to the process, and what is the output? 
FSP makes no distinction make between input and output actions. button is an input, while green is an output, but FSP does not distinguish these semantically—they are both just actions.


### Non-deterministic Choice
To complicate matters regarding input, output, and choice, FSP allows non-deterministic choice.

> “Process (x -> P | x -> Q) describes a process that engages in x and then behaves as P or Q.”

**Note that x is the prefix in both options of the choice.** In this instance, the choice is made by the process, not the environment. Therefore, **x could be an input from the environment, but the choice of P or Q is not controlled by the environment.**

```Java
TRAFFIC_LIGHT = ( button -> YELLOW
                | button -> green -> YELLOW
                | none -> GREEN
) ,
GREEN = ( green -> TRAFFIC_LIGHT) ,
YELLOW = ( yellow -> RED ) ,
RED = ( red -> TRAFFIC_LIGHT ).
```

### Indexed processes
Consider a buffer that can contain a single value. The value is input into the buffer, and can be then output. Values range from 0 to 3:

```Java
BUFF = ( in [i :0..3] -> out[ i] -> BUFF ).
```
This is equivalent to:
```Java
BUFF = ( in [0] -> out [0] -> BUFF
        | in [1] -> out [1] -> BUFF
        | in [2] -> out [2] -> BUFF
        | in [3] -> out [3] -> BUFF
).
```

### Constants and Ranges
```Java
const N = 3
range T = 0.. N
BUFF = ( in [i: T] -> STORE [i ]) ,
STORE [i :T] = ( out [i] -> BUFF ).
```

### Guarded actions
**A guarded action allows a context condition to be added to options in a choice.**

> “The choice (when B x -> P | y -> Q) means that when the guard B is true, then the actions x and y are both eligible to be chosen, otherwise if B is false, then action x cannot be chosen.”

```Java
COUNT ( N =3) = COUNT [0] ,
COUNT [i :0.. N] = ( when (i <N) inc - > COUNT [i +1]
                    | when (i >0) dec - > COUNT [i -1]
).

```

### The STOP Process
The STOP process is a special, pre-defined process that engages in no further actions. It is used for defining processes that terminate.

```Java
ONESHOT = ( once -> STOP ).
```

### Parallel Composition
The parallel composition operator allows us to describe the concurrent execution of two processes.

> “If P and Q are processes, then (P || Q) represents the concurrent execution of P and Q.”

example:
```Java
ITCH = ( scratch - > STOP ).
CONVERSE = ( think - > talk - > STOP ).
|| CONVERSE_ITCH = ( ITCH || CONVERSE ).
```
**FSP insists that when a process P is defined by parallel composition, the name of the composite process is prefixed with vertical bars: ||P.**

Currently, the combined processes have not interacted with each other yet. How can the pedestrian interact with our traffic light?
```Java
TRAFFIC_LIGHT = ( button -> YELLOW | idle -> GREEN ) ,
GREEN = ( green -> TRAFFIC_LIGHT ) ,
YELLOW = ( yellow -> RED ) ,
RED = ( red -> TRAFFIC_LIGHT ).
PEDESTRIAN = ( wander -> PEDESTRIAN ).
|| LIGHT_PEDESTRIAN = ( TRAFFIC_LIGHT || PEDESTRIAN ).
```
Interaction happens through shared actions:
> “If the processes in a composition have actions in common, these actions are said to be shared. This is how process interaction is modelled. While unshared actions may be arbitrarily interleaved, a shared action must be executed at the same time by all processes that participate in that shared action.”


```Java
TRAFFIC_LIGHT = ( button -> YELLOW | idle -> GREEN ) ,
GREEN = ( green -> TRAFFIC_LIGHT ) ,
YELLOW = ( yellow -> RED ) ,
RED = ( red -> TRAFFIC_LIGHT ).
PEDESTRIAN = ( button -> PEDESTRIAN
            | wander -> PEDESTRIAN
            ).
|| LIGHT_PEDESTRIAN = ( TRAFFIC_LIGHT || PEDESTRIAN ).
```

### Action relabelling

> “Given a process P, the process P/{new1/old1, ..., newN/oldN} is the same as P but with action old1 renamed to new1, etc.”

We can re-label wander in PEDESTRIAN to idle:
```Java
TRAFFIC_LIGHT = ( button -> YELLOW | idle -> GREEN ) ,
GREEN = ( green -> TRAFFIC_LIGHT ) ,
YELLOW = ( yellow -> RED ) ,
RED = ( red -> TRAFFIC_LIGHT ).
PEDESTRIAN = ( button -> PEDESTRIAN
| wander -> PEDESTRIAN
).
|| LIGHT_PEDESTRIAN =
( TRAFFIC_LIGHT || PEDESTRIAN /{ idle / wander }).
```

another example---client and server
```Java
CLIENT = ( call -> wait -> continue -> CLIENT ).
SERVER = ( request -> service -> reply -> SERVER ).
|| CLIENT_SERVER = ( CLIENT || SERVER ) /{ call / request , reply / wait }.
```
### Process Labelling
We must instead have different action labels in both clients. Writing a number of different definitions for multiple similar processes would be cumbersome and error prone, so instead, we use process labels.

> “a:P prefixes each action label in P with a”

```Java
|| TWO_CLIENTS = ( a : CLIENT || b : CLIENT ).
```

### Parameterised Processes and Labelling
An array of prefix labelled processes can be described using parameterised processes:

```Java
|| N_CLIENTS ( N =3) = ( c [ i :1.. N ]: CLIENT ).
```

```Java
|| N_CLIENTS ( M =3) = ( forall [ i :1.. M ] c [ i ]: CLIENT ).
```

we want to compose our server with multiple clients. However, with the action label in the clients being prefixed, they are no longer the same as the server labels, so will not be shared. To get around this, we can use a set of prefix labels:

> “{a1,..,ax}:: P replaces every action label n in the
alphabet of P with the labels a1.n,..,ax.n.
Further, every transition n->X in the definition of P is
replaced with the transitions ({a1.n,..,ax.n} -> X)”

({a1,..,ax} -> X) is just shorthand for the set of transitions (a1 -> X), ..., (ax -> X).

```Java
|| TWO_CLIENT_SERVER
  = ( a : CLIENT || b : CLIENT ||
    {a , b }::( SERVER /{ call / request , wait / reply })).
```
more generally,
```Java
|| N_CLIENT_SERVER ( N =2) =
    (( c [ i :1.. N ]: CLIENT )
    ||
    { c [ i :1.. N ]}::( SERVER /{ call / request , wait / reply })
    ).
```
Sometimes, we don't really need specify the index of process while it could only be declared once in a composed process. Therefore, a more efficient and general way to represent the whole group of processes is by range directly. Here I shows part of my assignment answer as example:
```
||SPACESYS = ((p[PilotRange]:PILOT)||
	{p[PilotRange]}::tugs:TUGS(NumOfTugs)||
	{p[PilotRange]}::arrZone:WAITZONE/{shipArriveArrivalZone/p[PilotRange].arrZone.shipArrive}||
	{p[PilotRange]}::depZone:WAITZONE/{shipDepartDepartureZone/p[PilotRange].depZone.shipDepart}||
	{p[PilotRange]}::berthsys:BERTHSYS/{shieldActivate/p[PilotRange].berthsys.shieldActivate, 
										shieldDeactivate/p[PilotRange].berthsys.shieldDeactivate}
).
```

The syntax process1::process2 represents the mapping process in my understanding, which means that all the actions for process2 will be shared with each member of process1. Different from syntax process1||process2 to execute process1 and process2 concurrently. In :: syntax, all the actions in process2 will be executed with a forehead process1. Action Relabeling could be used to remove this forehead.


### Variable Hiding
To reduce complexity, it is often useful to hide variables:
> ‘Given a process P, the process P\\{a1,..,aN} is the same as P, but with actions names a1,..,aN removed from P, making these silent. Silent actions are named tau and are never shared”.

An alternative is to list the variables that are not to be hidden:
> ‘Given a process P, the process P@{a1,..,aN} is the same as P, but with actions names other than a1,..,aN removed from P”.

The following two codes result in the same situation.
```java
SERVER_1 = ( request -> service -> reply -> SERVER_1 )
@ { request , reply }.

SERVER_2 = ( request -> service -> reply -> SERVER_2 )
\{ service }.
```
## Synchronization, Safety & Liveness in FSP
### Synchronization
#### Definition for Deadlock and Livelock
A process is in a **deadlock** if it is blocked waiting for a condition that will never become true.

A process is in a **livelock** (a busy wait deadlock) if it is spinning while waiting for a condition that will never become true. Either can happen if concurrent processes or threads are mutually waiting for each other.

#### Coffman Conditions
Coffman, Elphick, and Shoshani identify four necessary and sufficient conditions (the Coffman conditions) that all must occur for deadlock to happen:

- **Serially Reusable Resources**: the processes involved must share some reusable resources between themselves under mutual exclusion.
- **Incremental acquisition**: processes hold on to resources that have been allocated to them while waiting for additional resources.
- **No preemption**: once a process has acquired a resource, it can only release it voluntarily—it cannot be forced to release it.
- **Wait-for cycle**: a cycle exists in which each process holds a resource which its successor in the cycle is waiting for.

#### FSP vs Java
FSP monitors map well to Java monitors. In particular, the design
template for waiting in Java monitors can be mapped directly from
FSP guarded processes, such as below.

```Java
when cond act -> NEWSTAT
```
becomes
```Java
public synchronized void act () throws InterruptedException {
    while (! cond ) wait ();
        // modify monitor data
    notifyAll ();
}

```
### Alphabet Extension
alphabet_extension ::= "+" action_label_set
> Recall: The alphabet of a process s the set of actions in which it engages. 

alphabet extension focuses on extending the actions of process.
### Safety
#### Desirable properties of a Mutex solution
> **Mutual exclusion:** only one process may be active in its critical section at a time.
> **No deadlock:** if one or more processes are trying to enter their critical section, one must eventually succeed.
> **No starvation:** if a process is trying to enter its critical section, it must eventually succeed.
> **Handles lack of contention:** if only one process is trying to enter its critical section, it must succeed with minimal overhead.


To prevent the interference, we need a solution similar to the solutions seen in earlier lectures for mutual exclusion. One advantage of working with FSP instead of Java is that we can disregard all irrelevant details.

```Java
LOCK = ( acquire -> release -> LOCK ).
```

#### Checking safety and liveness properties
In concurrent systems, there are two categories of property that are of interest:
- Safety properties: A safety property asserts that nothing “bad” happens during execution. A deadlock is an example of this.
- Liveness properties: A liveness property asserts that some “good” eventually happens. For example, that all processes trying to access a critical section eventually do get access.

**As for safety**,
In sequential systems, the most common safety property is that the system satisfies some assertion each time a given program point is reached.

In concurrent systems, we have seen that additional important safety properties are absence of deadlock and interference.

**As for safety**,
The most common liveness property for a sequential system is that the system terminates.

Concurrent systems are often designed to be non-terminating, and liveness properties are most commonly related to resource access.

#### Safety Property in FSP
```Java
AN_ERROR = ( start -> do_something -> ERROR ).
```
`Note:`The state labelled -1 is a special state that indicates the ERROR process. It has no outgoing transitions.

Safety properties
When modelling complex systems, it is better practice to consider only the desired system behaviour, rather than also try to enumerate all possible undesirable behaviours. So, given a model, specify some desirable properties and check that the model maintains them.

In many formalisms, safety properties are used for this purpose. In FSP, a safety property is just a process, but to identify it clearly as a safety property, FSP uses the property keyword:

```Java
property SAFE_ACTUATOR
= ( command -> respond -> SAFE_ACTUATOR ).
```
**This property says that, whenever a command action is observed, a respond action should occur before another command action occurs.**

The LTSA compiler generates the LTS as if SAFE ACTUATOR is a normal FSP process, and then, for every state in the LTS, it adds an outgoing action for all actions in the process’s alphabet that are not already outgoing actions. These new outgoing actions go to the error state. As a result, the LTS is complete: all actions can occur from all states. Invalid actions go to the error state. Therefore, every possible combination of actions are permitted.

**To maintain this transparency, safety properties must be deterministic processes. That is, they must not contain non-deterministic choices.**

An LTS generated from a property process allows every possible combination of actions in a process’s alphabet. As a result, composing a property process with a normal process, as we have done with ACTUATOR and SAFE ACTUATOR, means that we do not affect the **normal** behaviour of the original process: all previous transitions remain because all shared actions can be synchronised. If behaviour that violates the safety property occurs, the result in the composite process will be the error state.

##### Safe Property for Interference
##### Safe Property for Mutual Exclusion

### Liveness
```JAVASCRIPT
range M = 0..4

SEND(E=3) = (chan[3] -> STOP).

RECEIVE = (chan[v:M]->received[v]->RECEIVE).

||SYS = (SEND || RECEIVE).
```
