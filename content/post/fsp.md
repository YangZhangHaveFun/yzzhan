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


