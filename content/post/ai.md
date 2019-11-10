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
- **consistent** if h(s) $\leq$ h(s') + c(a) for all transitions s $\underrightarrow{a}$ s'.

##### Relationships

- If h is consistent and goal-aware, then h is admissible.
- If h is admissible, then h is goal-aware.
- If h is admissible, then h is safe.
##### A*
f(n) 是从初始状态经由状态n到目标状态的代价估计，

g(n) 是在状态空间中从初始状态到状态n的实际代价，( the cost from the start node to the current node)

h(n) 是从状态n到目标状态的最佳路径的估计代价。(estimated cost from current node to goal.)

![reactor model](/media/posts/a_star.png)
##### Weighted A*

##### Greedy best-first search.

![reactor model](/media/posts/greedy_best_first.png)

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

#### Planning

- Satisficing Planning:
  - Input: A planning task P.
  - Output: A plan for P, or ‘unsolvable’ if no plan for P exists.
  - Satisficing planning is much more effective in practice
- Optimal Planning:
  - Input: A planning task P.
  - Output: An optimal plan for P, or ‘unsolvable’ if no plan for P exists.

Programs solving these problems are called (optimal) planners, planning systems, or planning tools.

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


### Relaxtions & Heuristic Functions

The **delete relaxation** simplifies STRIPS by removing all delete effects of the actions. The cost of **optimal relaxed plans** yields the heuristic function $h^+$, which is admissible but hard to compute.

We can approximate $h^+$ optimistically by $h^{max}$, and pessimistically by $h^{add}$. $h^{max}$ is admissible, hadd is not. $h^{add}$ is typically much more informative, but can suffer from over-counting.

Either of $h^{max}$ or $h^{add}$ can be used to generate a **closed well-founded best-supporter function**, from which we can extract a **relaxed plan**. The resulting relaxed plan heuristic hFF does not do over-counting, but otherwise has the same theoretical properties as hadd; it typically does not over-estimate h.

#### Delete relaxation

> Delete relaxation is a method to relax planning tasks, and thus automatically compute heuristic functions h.

Every h yields good performance only in some domains! (Search reduction vs. computational overhead) 

We cover the 4 different methods currently known:

- Critical path heuristics:
- Delete relaxation.
- Abstractions.
- Landmarks.

Delete Relaxation
: For a STRIPS action a, by $a^+$ we denote the corresponding delete relaxed action, or short relaxed action, defined by $pre_{a+}$ := $pre_a$, $add_{a+}$ := $add_a$, and $del_{a+}$ := $\emptyset$.

For a set A of STRIPS actions, by $A^+$ we denote the corresponding set of relaxed actions, $A^+$ := {$a^+|a\in A$}; similarly, for a sequence $\overrightarrow{a}$ = <$a_1,...,a_n$> of STRIPS actions, by $\overrightarrow{a}^+$ we denote the corresponding sequence of relaxed actions, $\overrightarrow{a}^+$ = <$a_1^+,...,a_n^+$>.

For a STRIPS planning task $\prod$ = (F, A, c, I, G), by $\prod ^+$ := (F, $A^+$, c, I,G) we denote the corresponding (delete) relaxed planning task.

Relaxed Plan
: Let $\prod$ = (F, A, c, I,G) be a STRIPS planning task, and let s be a state. An (optimal) relaxed plan for s is an (optimal) plan for $\prod^+_s$. A relaxed plan for I is also called a relaxed plan for $\prod$.

**It is always better to have more facts true.**

deletes do not matter because “shortest paths never walk back”, which means $h^+$ is equal to h* unless the shortest path contains walking back steps.

#### The Additive and Max Heuristics

![](/media/posts/hadd.PNG)

**Definitions:**

- $h^{add}(s,G)$: cost from current state to goal state
- G stands for goal state, g stands for any fact currently available. 
- c(a): cost of action a
- $h^{add}(s,pre_a)$: cost of achieving the precondition of current state.

**Propositions:**

- $h^{max}$ is Optimistic, admissible
  - $h^{max} \leq h^+$, and thus $h^{max} \leq h* $.
- $h^{add}$ is Pessimistic, not admissible
  - For all STRIPS planning tasks $\prod$, $h^{add} \geq h^+$. There exist $\prod$ and s so that $h^{add}(s) > h*(s)$.
  - typically a lot more informed than $h^{max}$
  - is sometimes better informed than $h^+$, rather than accounting for deletes, it overcounts by ignoring positive interactions
- $h^{max}$ and $h^{add}$ Agree with $h^+$ on $\infty$.
  - For all STRIPS planning tasks $\prod$ and states s in $\prod$, $h^+$(s) = $\infty$ if and only if $h^{max}$(s) = $\infty$ if and only if $h^{add}$(s) = $\infty$.

Both $h^{max}$ and $h^{add}$ approximate $h^+$ by assuming that singleton sub-goal facts are achieved independently. $h^{max}$ estimates optimistically by the most costly singleton sub-goal, $h^{add}$ estimates pessimistically by summing over all singleton sub-goals.

#### Best-Supporter Functions

> First compute a best-supporter function bs, which for every fact p $\in$ F returns an action that is deemed to be the cheapest achiever of p (within the relaxation). Then extract a relaxed plan from that function, by applying it to singleton sub-goals and collecting all the actions.

the best-supporter function only choose the cheaper action to add a certain fact. The best-supporter function can be based directly on $h^{max}$ or $h^{add}$, simply selecting an action a achieving p that minimizes the sum of c(a) and the cost estimate for $pre_a$

**Starting with the top-level goals, iteratively close open singleton sub-goals by selecting the best supporter.**

Best-Supporter Function
: Let $\prod$ = (F, A, c, I, G) be a STRIPS planning task, and let s be a state. A best-supporter function for s is a partial function bs : (F \ s) $\rightarrow$ A such that p $\in add_a$ whenever a = bs(p).

- We say that bs is **closed** if bs(p) is defined for every p $\in$ (F\s) that has a path to a goal g $\in$ G in the support graph.
- We say that bs is well-founded if the support graph is acyclic.

> Relaxed plan extraction starts at the goals, and chains backwards in the support graph. If there are cycles, then this backchaining may not reach the currently true state s, and thus not yield a relaxed plan.

##### Proportions

Let $\prod$ = (F, A, c, I, G) be a STRIPS planning task such that, for all a $\in$ A, c(a) > 0. Let s be a state where $h^+$(s) < $\infty$. Then both $bs^{max}_s$ and $bs^{add}_s$ are closed well-founded supporter functions for s.

Proposition. Let $\prod$ = (F, A, c, I, G) be a STRIPS planning task, let s be a state, and let bs be a closed well-founded best-supporter function for s. Then the action set RPlan returned by relaxed plan extraction can be sequenced into a relaxed plan $\overrightarrow{a}^+$ for s.

Relaxed Plan Heuristic
: A heuristic function is called a relaxed plan heuristic, denoted $h^{FF}$, if, given P a state s, it returns $\infty$ if no relaxed plan exists, and otherwise returns $\sum_{a\in RPlan}c(a)$ where RPlan is the action set returned by relaxed plan extraction on a closed well-founded best-supporter function for s.

- $h^{FF}$ is Pessimistic and Agrees with h* on $\infty$
- $h^{FF}$ may be inadmissible, just like hadd, but for more subtle reasons
- In practice, $h^{FF}$ typically does not over-estimate h* (or not by a large amount, anyway);

#### Helpful Actions

Helpful Actions
: Let $h^{FF}$ be a relaxed plan heuristic, let s be a state, and let RPlan be the action set returned by relaxed plan extraction on the closed well-founded best-supporter function for s which underlies hFF. Then an action a **applicable** to s is called helpful if it is contained in RPlan.

### Width Based Planning

- Benchmark domains have small width when goals restricted to single atoms
- Joint goals easy to serialize into a sequence of single goals

#### Width-Based Search

the novelty w(s) of a state s
: is the size of the smallest subset of atoms in s that is true for the first time in the search.

e.g. w(s) = 1 if there is one atom p 2 s such that s is the first state that makes p true.

##### Iterated Width (IW)

IW(k) = breadth-first search that prunes newly generated states whose novelty(s) > k. IW is a sequence of calls IW(k) for i = 0, 1, 2, ... over problem P until problem solved or i exceeds number of variables in problem.

For problems $\prod \in$ P, where width($\prod$) = k:

- IW(k) solves $\prod$ in time O($n^k$);
- IW(k) solves $\prod$ optimally for problems with uniform cost functions
- IW(k) is complete for $\prod$

##### Serialized Iterated Width (SIW)

Simple way to use IW for solving real benchmarks P with joint goals is by simple form of “hill climbing” over goal set G with |G| = n, achieving atomic goals one at a time.

- SIW uses IW for both decomposing a problem into subproblems and for solving subproblems
- It’s a **blind search procedure**, no heuristic of any sort, IW does not even know next goal $G_i$ “to achieve”.

#### Balancing Exploration and Exploitation

##### Best-First Width Search (BFWS)

Best-First Width Search
: BFWS(f) for f = <w, f1,..fn> where w is a novelty-measure, is a plain best-first search where nodes are ordered in terms of novelty function w, with ties broken by functions $f_i$ in that order.

### Markov Decision Processes

> Classical Planning tools can produce solutions quickly in large search space; but assume:
>
> - Deterministic events
> - Environments change only as result of an action
> - Perfect knowledge (omniscience)
> - Single actor (omniscience)

Markov Decision Processes (MDPs) remove the assumption of deterministic events and instead assume that each action could have multiple outcomes, with each outcome associated with a probability.

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

The planning problem for discounted-reward MDPs is different to that of classical planning because the actions are non-deterministic. Instead of a sequence of actions, an MDP produces a policy.

A policy, $\pi$ is a function that tells an agent which is the best action to choose in each state. A policy can be deterministic or stochastic.

A deterministic policy $\pi$ : S $\rightarrow$ A is a mapping from states to actions. It specifies which action to choose in every possible state. Thus, if we are in state s, our agent should choose the action defined by $\pi$(s).

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
$V(s)=max_{a\in A(s)}\sum_{s'\in S}P_a (s'|s)[r(s,a,s')+\gamma V(s')]$

the reward of an action is : sum of immediate reward for all state possibly + discounted future reward of those states

Another representation of Bellman equation for discounted-reward MDPs. If V(s) is the expected value of being in state s and acting optimally according to our policy, then we can also describe the Q-value of being in a state s, choosing action a and then acting optimally according to our policy as:

- Q(s,a) = $\sum_{s'\in S}P_a(s'|s)[r(s,a,s')+\gamma V(s')]$
- V(s) = $max_{a\in A(s)} Q(s,a)$


A policy can now be easily defined: in a state s, given V, choose the action with the highest expected cost (or reward).
#### Value Iteration

> Value Iteration find the optimal function V* solving the Bellman equations iteratively, using the following algorithm:
> - Set $V_0$ to arbitrary value function; eg. $V_0$ = 0 for all s.
> - Set $V_{i+1}$ to result of Bellman's right hand side using $V_i$ in place of V:
> $V_{i+1}(s)=max_{a\in A(s)}\sum_{s'\in S}P_a(s'|s)[r(s,a,s')+ \gamma V_i(s')]$

When $V_i$ converges to $V_{\infty}$. That is, given an infinite amount of iterations, it will be optimal.

#### Policy Iteration

policy iterationis an approach that is similar to value iteration. While value iteration iterates over value functions, policy iteration iterates over policies themselves, creating a strictly improved policy in each iteration.

However, the expected costs V(s) can also be characterised as a solution to the equation

### Monte Carlo Tree

Build up an MDP tree using simulation. The evaluated states are stored in a search tree. The set of evaluated states is incrementally built be iterating over the following four steps:

- **Select**: Given a tree policy, select a single leaf node in the tree to assess.
- **Expand**: Expand this node by applying one available action (as dened by the MDP) from the node.
- **Simulation**: From one of the new nodes, perform a complete random simulation of the MDP to a terminating state. This therefore assumes that the search tree is nite, but versions for innitely large trees exist in which we just execute for some time and then estimate the outcome.
- **Backpropagate**: Finally, the value of the node is backpropagated to the root node, updating the value of each ancestor node on the way using expected value.

### Q-Learning

#### Reinforcement Learning Intro

- We execute many different episodes of the problem we want to solve, and from that we learnt a policy.
- During learning, we try to learn the value of applying particular actions in particular states.
- During each episode, we need to execute some actions. After each action, we get a reward (which may be 0) and we can see the new state.
- From this, we reinforce our estimates of applying the previous action in the previous state.
- We terminate when: (1) we run out of training time; (2) we think our policy has converged to the optimal policy (for each new episode we see no improvement); or (3) our policy is ‘good enough’ (for each new episode we see minimal improvement).

---

There are many different models of reinforcement learning, all with the same basis:

- offline-policy learning(Q-learning): estimates the policy($Q(s',a')$ for the best estimated future state) independent of the current behaviour.
- on-policy learning(SARSA): estimate $Q^\pi(s,a)$ for the current behaviour policy $\pi$. Use the action chosen by the policy for the update. 

#### Q-learning(off-policy learning)

Process:

1. selects an action a;
2. takes that actions and observes the reward & next state s'
3. updates optimistically by assuming the future reward is $max_{a'}Q(s', a')$--that is, it assumes that future behaviour will be optimal

##### Equation:

$Q(s,a) \leftarrow Q(s,a) + \alpha [r+\gamma max_{a'}Q(s',a')-Q(s,a)]$

Params:

- Q(s,a): old value
- $\alpha$: learning rate
- $\gamma$: discount factor
- r: reward, r(s,a,s')
- $max_{a'}Q(s',a')$: estimate of optimal future value


#### SARSA(on-policy learning)

Process:

1. selects action a' for the next loop iteration
2. in the next iteration, takes that action and observes the reward & next state s'
3. only then chooses a' for the next action chosen
4. updates using the estimate for the actual next action chosen - which may not be the greediest one

##### Equation:

$Q(s,a) \leftarrow Q(s,a) + \alpha [r+\gamma Q(s',a')-Q(s,a)]$

##### Difference between Q-learning

On-policy learning is more appropriate when we want to optimise the behaviour of an agent who learns while operating in its environment.

On-Policy SARSA learns action values relative to the policy it follows, while Off-Policy Q-Learning does it relative to the greedy policy.

The difference is all in how the update happens in the loop body.

Q-learning: (1) selects an action a; (2) takes that actions and observes the reward & next state s'; and (3) updates optimistically by assuming the future reward is $max_{a'}Q(s',a')$ – that is, it assumes that future behaviour will be optimal (according to its policy).

SARSA: (1) selects action a' for the next loop iteration; (2) in the next iteration, takes that action and observes the reward & next state s'; (3) only then chooses a' for the next iteration; and (4) updates using the estimate for the actual next action chosen – which may not be the greediest one.

So what difference does this really make? There are two main differences:

- Q-learning will converge to the optimal policy irrelevant of the policy followed, because it is off-policy: it uses the greedy reward estimate in its update rather than following the policy such as $\epsilon$-greedy). Using a random policy, Q-learning will still converge to the optimal policy, but SARSA will not (necessarily).
- Q-learning learns an optimal policy, but this can be ’unsafe’ or risky during training.

### Reinforcement Learning – Some Improvements

Reinforcement Learning – Some Weaknesses:

- Unlike Monte-Carlo methods, which reach a reward and then backpropagate this reward, TD methods use bootstrapping (they estimate the future discounted reward using Q(s, a)), which means that for problems with sparse rewards, it can take a long time to for rewards to propagate throughout a Q-function.
- Both methods estimate a Q-function Q(s, a), and the simplest way to model this is via a Q-table. However, this requires us to maintain a table of size |A| * |S|, which is prohibitively large for any non-trivial problem.
- Using a Q-table requires that we visit every reachable state many times and apply every action many times to get a good estimate of Q(s, a). Thus, if we never visit a state s, we have no estimate of Q(s, a), even if we have visited states that are very similar to s.
- Rewards can be sparse, meaning that there are few state/actions that lead to non-zero rewards. This is problematic because initially, reinforcement learning algorithms behave entirely randomly and will struggle to find good rewards.

#### n-step temporal difference learning
Updating the Q-function:

- First, we need to calculate the truncated reward for n steps, in which $\tau$ is the time step that we are updating for:
  - $G\leftarrow \sum_{i=\tau+1}^{min(\tau+n,T)}\gamma^{i-\tau-1}r_i$
  - This just sums the rewards from time step $\tau$ + 1 until either n steps ($\tau$ + n) or termination of the episode (T), whichever comes first.
- For n-step SARSA, we have:
  - If $\tau+n<T$ then $G\leftarrow G+\gamma^nQ(S_{\tau+n},A_{\tau+n})$
    $Q(S_\tau,A_\tau)\leftarrow Q(S_\tau,A_\tau)+\alpha[G-Q(S_\tau,A_\tau)]$
  - The first line just adds the future expect reward if we are not at the end of the episode (if $\tau$ + n < T).
  - For the first n 􀀀 1 steps of the any episode, we do not update Q at all.
  - we have to continue updating n 􀀀 1 steps after the end of the episode.
#### Approximate methods:

problem is that the size of the state space increases exponentially with every variable added. This causes two main issues:

- Storing a value of Q(s, a) for every s and a, such as in Q-tables, is infeasible.
- Propagation of Q-values takes a long time.

The key idea is to approximate the Q-function using a linear combination of features and their weights.

The overall process is:

- For the states, consider what are the features that determine its representation.
- During learning, perform updates based on the weights of features instead of states.
- Estimate Q(s, a) by summing the features and their weights.

a Q-function is represented using the features and their weights, instead of a Q-table. To represent this, we have two vectors:

- A feature vector, f (s), which is a vector of n * |A| different functions, where n is the number of state features and |A| the number of actions. Each function extracts the value of a feature for state-action pair (s, a). We say $f_i(s, a)$ extracts the ith feature from the state-action pair (s, a):
  ![reactor model](/media/posts/aqfunctions.png)
- A weight vector w of size n * |A|: one weight for each feature-action pair. $w^a_i$ defines the weight of a feature i for action a.

Give a feature vector f and a weight vector w, the Q-value of a state is a simple linear combination of features and weights:

$Q(s, a) = f_1(s, a)*w_1^a+f_2(s, a)*w_2^a+...+f_n(s, a)*w_n^a$
$Q(s, a) = \sum^n_{i=0}f_i(s,a)w^a_i$

To use approximate Q-functions in reinforcement learning, there are two steps we need to change from the standard algorithsm: (1) initialisation; and (2) update.

For initialisation, initialise all weights to 0. Alternatively, you can try Q-function initialisation and assign weights that you think will be ‘good’ weights.

For update, we now need to update the weights instead of the actions. For Q-learning, the update rule is now:

![reactor model](/media/posts/qandsarsa.png)

#### Reward shaping and Value-Function Initialisation:

### Game Theory
#### Games in Normal Form
a normal form game, or a game in strategic form:

- The set of players is N = {1,...,n}.
- Player i has a set of actions, $a_i$, available. These are generally referred to as pure strategies. This set might be nite or innite.
- Let a = $a_1 \times$ . . . $\times a_n$ be the set of all proles of pure strategies or actions, with a generic element denoted by a = ($a_1$,...,$a_n$).
- Player i's payoff as a function of the vector of actions taken is described by a function $u_i$: A $\rightarrow$ IR, where $u_i$(a) is i's payo if the a is the prole of actions chosen in the society.

Given a game in normal form, we then can make predictions about which actions will be chosen.
#### Dominant Strategies

A dominant strategy for a player is one that produces the highest payoff any strategy available for every possible action by the other players.

Strict Dominance: Strategy x dominates strategy y for a player if x cost a less payoff than y regardless of what the other player do.

Iterated Elimination of Strictly Dominated Strategies(IEDS): 

- If you ever see a strictly dominated strategy, eliminate it immediately.
  - Order does not matter
  - If IEDS leads to a single outcome, you will arrive at that outcome whether you eliminate strategy #1 or strategy #2 first

#### Pure Nash Equilibrium
A Nash Equilibrium is a set of strategies, one for each player, such that no player has incentive to change his or her strategy.

- We only care about individual deviations, not group deviations.
- Nash equilibrium are inherently stable.
  - What you are doing is optimal given what I am doing and vice versa.
  - No regrets.

A pure strategy Nash equilibrium is when players do not randomize between two or more strategies.

#### Best Responses

Given what all other players are doing, a strategy is best response if and only if a player cannot gain more utility from switching to a different strategy.

A game is in a Nash Equilibrium if and only if all players are playing best responses to what the other players are doing.

A pure strategy Nash equilibrium is a prole of strategies such that each player's strategy is a best response (results in the highest available payo) against the equilibrium strategies of the other players.

#### Mixed Nash Equilibrium

Nash Theorem: There must be at least one Nash Equilibrium for all finite games.

Mixed Nash Equilibrium: 

- If no equilibrium exists in pure strategies, one must exist in mixed strategies.
- A mixed strategy is a probability distribution over two or more pure strategies.
  - That is, the players choose randomly among their options in equilibrium.
  - If mixtures are mutual best responses, the set of strategies is a mixed strategy Nash Equilibrium.

There could be mixed Nash Equilibrium while the pure Nash Equilibrium exists.

##### Strict Dominance and Mixed Strategy

If you find a mixture between two strategies(or more) strictly dominates another strategy, eliminate that last strategy immediately.

Strictly dominance strategies are irrational, whether pure strategies or mixed strategies dominates them.

##### Weak Dominance

Left weakly dominates right for play two, that is, left as at least as good as right and sometimes better.
#### Randomization and Mixed Strategies

#### Sequentiality, Extensive Form Games, and Backward Induction


##### Extra

- If we know the MDP:
  - Offline: Value Iteration, Policy Iteration
  - Online: Monte Carlo Search Tree and related
- If we do not know MDP:
  - Offline: Reinforcement learning
  - Online: Monte Carlo Tree Search/ SARSA