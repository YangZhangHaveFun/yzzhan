---
title: "COMP-90054 AI Planning"
tags: ["algorithm", "AI", "Planning"]
date: 2019-9-27
draft: false
---

#### Agent

Agent has two functionalities:

- Perceive the environment through sensors (->percepts).
- Act upon the environment through actuators (->actions).

#### AI Solver

The basic models and tasks include:

- Constraint Satisfaction/SAT: find state that satisfies constraints
- Planning Problems: find action sequence that produces desired state
- Planning with Feedback: find strategy for producing desired state


### Classical Planning Model

- Planning is the model-based approach to autonomous behavior
- A system can be in one of many states
- States assign values to a set of variables
- Actions change the values of certain variables
- Basic task: find action sequence to drive initial state into goal state

Search algorithms for planning exploit the correspondence between (classical) **states model S(P)** and **directed graphs**:

- The nodes of the graph represent the states s in the model
- The edges (s, s') capture corresponding transition in the model with same cost

### Search Algorithms

##### Blind search vs. heuristic (or informed) search:

- Blind search algorithms: Only use the basic ingredients for general search algorithms.
  - DFS, BFS, Uniform Cost(Dijkstra), Iterative Deepening (ID)
- Heuristic search algorithms: Additionally use heuristic functions which estimate the distance (or remaining cost) to the goal.
  - A*, IDA*, Hill Climbing, Best First, WA*, DFS B&B, LRTA*, . . .

- For satisficing planning, heuristic search vastly outperforms blind algorithms pretty much everywhwere.
- For optimal planning, heuristic search also is better (but the difference is less pronounced).

##### Systematic search vs. local search:

- Systematic search algorithms: Consider a large number of search nodes simultaneously.
- Local search algorithms: Work with one (or a few) candidate solutions (search nodes) at a time.

This is not a black-and-white distinction; there are crossbreeds (e.g., enforced hill-climbing).

- For satisficing planning, there are successful instances of each.
- For optimal planning, systematic algorithms are required.

##### Search Terminology

- Search node n: Contains a state reached by the search, plus information about how it was reached.
  - Search states s: States (vertices) of the search space.
  - Search nodes $\sigma$: Search states, plus information on where/when/how they are encountered during search.
  - typical information includes:
    - state($\sigma$): Associated search state.
    - parent($\sigma$): Pointer to search node from which $\sigma$ is reached.
    - action($\sigma$): An action leading from state(parent($\sigma$)) to state($\sigma$).
    - g($\sigma$): Cost of $\sigma$ (cost of path from the root node to $\sigma$).
- Path cost g(n): The cost of the path reaching n.
- Optimal cost g*: The cost of an optimal solution path. For a state s, g*(s) is the cost of a cheapest path reaching s.
- Node expansion: Generating all successors of a node, by applying all actions applicable to the node’s state s. Afterwards, the state s itself is also said to be expanded.
- Search strategy: Method for deciding which node is expanded next.
- Open list: Set of all nodes that currently are candidates for expansion. Also called frontier.
- Closed list: Set of all states that were already expanded. Used only in graph search, not in tree search (up next). Also called explored set.

###### Analysis
- Guarantees:
  - Completeness: Is the strategy guaranteed to find a solution when there is one?
  - Optimality: Are the returned solutions guaranteed to be optimal?
- Typical state space features governing complexity:
  - Branching factor b: How many successors does each state have?
  - Goal depth d: The number of actions required to reach the shallowest goal state.

#### Blind Systematic Search Algorithms

##### Breadth-First Search

- Completeness? Yes.
- Optimality? Yes, for uniform action costs. Breadth-first search always finds a shallowest goal state. If costs are not uniform, this is not necessarily optimal.
- the time complexity is O($b^d$). And if we were to apply the goal test at node-expansion time, rather than node-generation time? O($b^d$+1) because then we’d generate the first d + 1 layers in the worst case.
- Space Complexity: the same as time complexity

##### Depth-First Search

- Optimality? No. After all, the algorithm just “chooses some direction and hopes for the best”.
- Completeness? No, because search branches may be infinitely long: No check for cycles along a branch!
  - Depth-first search is complete in case the state space is acyclic, e.g., Constraint Satisfaction Problems. If we do add a cycle check, it becomes complete for finite state spaces.


##### Iterative Deepening Search

“Iterative Deepening Search=Keep doing the same work over again until you find a solution.”

Optimality? Yes! Completeness? Yes! Space complexity? O(b d).

IDS combines the advantages of breadth-first and depth-first search. It is the preferred blind search method in large state spaces with unknown solution depth.

#### Informed Systematic Search Algorithms 

Heuristic function h estimates the cost of an optimal path to the goal; search gives a preference to explore states with small h.

Heuristic searches require a heuristic function to estimate remaining cost:

Heuristic Function
: Let $\prod$ be a **planning task** with state space $\theta_{\prod}$. A heuristic function, short heuristic, for $\prod$ is a function $h: S \rightarrow \real_{0}^{+} \cup\{\infty\}$. Its value h(s) for a state s is referred to as the state’s heuristic value, or h-value.

Remaining Cost($h^*$)
: Let $\prod$ be a **planning task** with state space $\theta_{\prod}$. For a state s $\in$ S, the state’s **remaining cost** is the cost of an optimal plan for s, or $\infty$ if there exists no plan for s. The **perfect heuristic for $\prod$**, written $h^*$, assigns every s$\in$S its remaining cost as the heuristic value.

>"Search performance depends crucially on the informedness of h and on the computational overhead of computing h!!"

Extreme cases:

- h = h*: Perfectly informed; computing it = solving the planning task in the first place.
- h = 0: No information at all; can be “computed” in constant time.

Successful heuristic search requires a good trade-off between h’s informedness and the computational overhead of computing it.

##### Properties of Heuristic Functions

Safe/Goal-Aware/Admissible/Consistent
: Let $\prod$ be a planning task with state space $\theta_{\prod}$ = (S, L, c, T, I, $S^G$), and let h be a heuristic for $\prod$. The heuristic is called:

- **safe** if h*(s) = $\infty$ for all s $\in$ S with h(s) = $\infty$;
  - safe是预测State s不可达，实际State s也确实不可达
  - a heuristic function is said to be admissible if it never overestimates the cost of reaching the goal
- **goal-aware** if h(s) = 0 for all goal states s $\in$ $S^G$;
- **admissible** if h(s) $\leq$ h*(s) for all s $\in$ s;
  - admissible是预测的cost永远小于等于实际的cost
- **consistent** if h(s) $\leq$ h(s') + c(a) for all transitions s $\rightarrow^{a}$ s'.

##### Relationships

- If h is consistent and goal-aware, then h is admissible.
- If h is admissible, then h is goal-aware.
- If h is admissible, then h is safe.
##### A*
f(n) 是从初始状态经由状态n到目标状态的代价估计，

g(n) 是在状态空间中从初始状态到状态n的实际代价，( the cost from the start node to the current node)

h(n) 是从状态n到目标状态的最佳路径的估计代价。(estimated cost from current node to goal.)
##### Weighted A*

##### Greedy best-first search.

##### Others

IDA*, depth-first branch-and-bound search, breadth-first heuristic search, . . .

#### Heuristic Search Algorithms: Local

##### Hill-climbing

##### Enforced hill-climbing

##### Others

Beam search, tabu search, genetic algorithms, simulated annealing, ...
### Autonomous Behavior in AI
The key problem is to select the action to do next. This is the so-called control problem. Three approaches to this problem:

- Programming-based: Specify control by hand
  - Advantage: domain-knowledge easy to express
  - Disadvantage: cannot deal with situations not anticipated by programmer
- Learning-based: Learn control from experience
  - Advantage: does not require much knowledge in principle
  - Disadvantage: in practice, hard to know which features to learn, and is slow
- Model-based: Specify problem by hand, derive control automatically
  - Advantage:
    - Powerful: In some applications generality is absolutely necessary
    - Quick: Rapid prototyping. 10s lines of problem description vs. 1000s lines of C++ code.
    - Flexible & Clear: Adapt/maintain the description.

### Markov Decision Processes

> Classical Planning tools can produce solutions quickly in large search space; but assume:
>
> - Deterministic events
> - Environments change only as result of an action
> - Perfect knowledge (omniscience)
> - Single actor (omniscience)

#### Model Properties

- a state space S
- initial state $s_0$ $\in$ S
- a set G $\subseteq$ S of goal states
- actions A(s) $\subseteq$ A applicable in each state s $\in$ S
- transition probabilities $P_a(s'|s)$ for s $\in$ S and a $\in$ A(s)
- action costs c(a, s) > 0

Solutions are functions (policies) mapping states into actions. Optimal solutions minimize expected cost to goal.

- If actions deterministic and initial location known, planning problem is classical
- If actions stochastic and location observable, problem is an MDP
- If actions stochastic and location partially observable, problem is a POMDP

#### STRIP & PDDL

A planner is a solver over a class of models; it takes a model description, and computes the corresponding controller

##### STRIP problem

A problem in STRIPS is a tuple P = \<F,O,I,G\>:

- F stands for set of all **atoms** (boolean vars)
- O stands for set of all **operators** (actions)
  - Operators o $\in$ O represented by:
    - the Add list Add(o) $\subseteq$ F
    - the Delete list Del(o) $\subseteq$ F
    - the Precondition list Pre(o) $\subseteq$ F
- I $\subseteq$ F stands for initial situation
- G $\subseteq$ F stands for goal situation

##### STRIP model

A STRIPS problem P = \<F,O,I,G\> determines state model S(P) where

- the states s $\in$ S are collections of atoms from F. S = $2^F$
- the initial state $s_0$ is I
- the goal states s are such that G $\subseteq$ s
- the actions a in A(s) are ops in O s.t. Prec(a) $\subseteq$ s
- the next state is s' = s - Del(a) + Add(a)
- action costs c(a, s) are all 1

#### Desc
Markov Decision Processes(MDPs) remove the assumption of deterministic events and instead assume that each action could have multiple outcomes, with each outcome associated with a probability. 

MDPs have been successfully applied to planning in many domains: robot navigation, planning which areas of a mine to dig for minerals, treatment for patients, maintainance scheduling on vehicles, and many others.
#### Markov Property

Markov Property
: "The future is independent of the past given the present" 

Relative Equation:  $P(S_{t+1}|S_t) = P(S_{t+1}|S_1,....,S_t)$

- $S_t$状态能够捕获历史状态的相关信息
- 当当前状态$S_t$已知，历史可以被忽视

#### Markov Process

> A Markov process is a memoryless random process, i.e. a sequence of random states $S_1$, $S_2$,... with the Markov property.
>
> - S is a (finite) set of states
> - P is a state transition probability matrix, $P_{ss'} = P[S_{t+1}=s'|S_t=s]$

Markov Process是一个无记忆的随机过程, 是一些具有Markov Property的**随机状态队列**构成, 可以用一个元组$<S,P>$表示, 其中S是有限数量的状态集, P是状态转移概率矩阵.

#### Markov Reward Process

在Markov Process的基础上增加了奖励R和衰减系数$\gamma$. 此时的元组为: $<S,P,R,\gamma>$

> R是一个奖励函数, S状态下的奖励是某一时刻(t)处在状态s下在下一时刻(t+1)能获得的奖励期望.

A Markov reward process is Markov chain with values.
Markov Reward Process
: contains a tuple $<S,P,R,\gamma>$ 

- S is a finite set of states
- P is a state transition probability matrix $P_{ss'} = P[S_{t+1}=s'|S_t=s]$
- R is a reward function, $R_s = E[R_{t+1|S_t=s}]$
- $\gamma$ is a discount factor, $\gamma \in [0,1]$

##### Return

收获$G_t$为在一个Markov Reward Chain上从t时刻开始往后所有的奖励的有衰减的收益总和.

$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2R_{t+3} +...=\sum^{\infty}_{k=0} \gamma^k R_{t+k+1}$

$\gamma$接近0表示趋向于"近视"性评估, $\gamma$接近1表明偏重考虑远期的利益.

#### Bellman Equations for MDPs

> Bellman Equations decribe the condition that must hold for a policy to be optimal.

For discounted-reward MDPs, the Bellman equation is: 
$V(s)=max_{a\in A(s)}\sum_{s'\in S}P_a(s'|s)[r(s,a,s')+\gamma V(s')]$

the reward of an action is : sum of immediate reward for all state possibly + discounted future reward of those states

Another representation of Bellman equation for discounted-reward MDPs. If V(s) is the expected value of being in state s and acting optimally according to our policy, then we can also describe the Q-value of being in a state s, choosing action a and then acting optimally according to our policy as:

- Q(s,a) = $\sum_{s'\in S}P_a(s'|s)[r(s,a,s')+\gamma V(s')]$
- V(s) = $max_{a\in A(s)} Q(s,a)$

#### Value Iteration

> Value Iteration find the optimal function V* solving the Bellman equations iteratively, using the following algorithm:
> - Set $V_0$ to arbitrary value function; eg. $V_0$ = 0 for all s.
> - Set $V_{i+1}$ to result of Bellman's right hand side using $V_i$ in place of V:
> $V_{i+1}(s)=max_{a\in A(s)}\sum_{s'\in S}P_a(s'|s)[r(s,a,s')+ \gamma V_i(s')]$

When $V_i$ converges to $V_{\infty}$. That is, given an infinite amount of iterations, it will be optimal.

### Q-Learning

- offline-policy learning(Q-learning): estimates the policy($Q(s',a')$ for the best estimated future state) independent of the current behaviour.
- on-policy learning(SARSA): estimate $Q^\pi(s,a)$ for the current behaviour policy $\pi$. Use the action chosen by the policy for the update. 

#### Q-learning(off-policy learning)

Process:

1. selects an action a;
2. takes that actions and observes the reward & next state s'
3. updates optimistically by assuming the future reward is $max_{a'}Q(s', a')$--that is, it assumes that future behaviour will be optimal



#### SARSA(on-policy learning)

Process:

1. selects action a' for the next loop iteration
2. in the next iteration, takes that action and observes the reward & next state s'
3. only then chooses a' for the next action chosen
4. updates using the estimate for the actual next action chosen - which may not be the greediest one

On-policy learning is more appropriate when we want to optimise the behaviour of an agent who learns while operating in its environment.


- If we know the MDP:
  - Offline: Value Iteration, Policy Iteration
  - Online: Monte Carlo Search Tree and related
- If we do not know MDP:
  - Offline: Reinforcement learning
  - Online: Monte Carlo Tree Search/ SARSA