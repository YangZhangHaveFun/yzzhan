---
title: "COMP-90054 AI Planning"
tags: ["algorithm", "AI", "Planning"]
date: 2019-9-27
draft: false
---

### Markov Decision Processes

> Classical Planning tools can produce solutions quickly in large search space; but assume:
>
> - Deterministic events
> - Environments change only as result of an action
> - Perfect knowledge (omniscience)
> - Single actor (omniscience)


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