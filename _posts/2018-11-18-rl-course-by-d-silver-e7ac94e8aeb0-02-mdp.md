---
ID: 91
post_title: RL Course by D.Silver 笔记 02 – MDP
author: chin340823
post_excerpt: ""
layout: post
permalink: https://blog.tvect.cn/?p=91
published: true
post_date: 2018-11-18 22:23:22
---
[toc]

Markov decision processes formally describe an environment for reinforcement learning，where the environment is fully observable, i.e. The current state completely characterises the process.

Almost all RL problems can be formalised as MDPs

<!--more-->

<h1>Markov Processes</h1>

A <strong>Markov process</strong> is a memoryless random process, i.e. a sequence of random states $S_1, S_2, ...$ with the Markov property.

<blockquote>
  Definition
  A <strong>Markov Process</strong> (or <strong>Markov Chain</strong>) is a tuple $left langle S, P right rangle$
  1. $S$ is a (finite) set of states
  2. $P$ is a state transition probability matrix: $P_{ss'} = P [S_{t+1} = s' | S_t = s]$
</blockquote>

<h1>Markov Reward Processes</h1>

<h2><strong>Definition of MRP</strong></h2>

A <strong>Markov reward process</strong> is a Markov chain with values.

<blockquote>
  Definition
  A Markov Reward Process is a tuple $left langle S, P, {color{Red}R, gamma} right rangle$
  1. $S$ is a (finite) set of states
  2. $P$ is a state transition probability matrix: $P_{ss'} = P [S_{t+1} = s' | S_t = s]$
  3. <font color="red"> $R$ is a reward function, $R_s = E [R_{t+1} | S_t = s]$ </font>
  4. <font color="red"> $gamma$ is a discount factor, $gamma in [0, 1]$ </font>
</blockquote>

<h2><strong>Return</strong></h2>

<blockquote>
  Definition
  The <strong>return</strong> $G_t$ is the total discounted reward from time-step t.
  $$G_t = R_{t+1} + gamma R_{t+2} + ... =  sum limits_{k=0}^{} gamma^{k} R_{t+k+1}$$
</blockquote>

<h2><strong>Value Function</strong></h2>

The value function $v(s)$ gives the long-term value of state $s$.

<blockquote>
  Definition
  The <strong>state value function</strong> $v(s)$ of an MRP is the expected return starting from state $s$
  $$v(s) = E [G_t | S_t = s]$$
</blockquote>

<h2><strong>Bellman Equation</strong></h2>

The value function can be decomposed into two parts:

<ul>
<li>immediate reward $R_{t+1}$</p></li>
<li><p>discounted value of successor state $gamma v(S_{t+1})$</p></li>
</ul>

<p>$$
begin{aligned}
v(s) &amp;= E [G_t | S_t = s] &#92;
&amp;= E[R_{t+1} + gamma R_{t+2} + gamma^2 R_{t+3} + ... | S_t = s] &#92;
&amp;= E[R_{t+1} + gamma (R_{t+2} + gamma R_{t+3} + ...) | S_t = s] &#92;
&amp;= E[R_{t+1} + gamma G_{t+1} | S_t = s] &#92;
&amp;= E[R_{t+1} + gamma v(S_{t+1}) | S_t = s]
end{aligned}
$$

进一步有: $v(s) = R_s + gamma sum limits_{s' in S} P_{ss'} v(s')$

<ul>
<li><strong>Bellman Equation in Matrix Form</strong></li>
</ul>

The Bellman equation can be expressed concisely using matrices,

$$v = R + gamma P v$$

where $v$ is a column vector with one entry per state

$$
begin{bmatrix}
v(1)&#92; 
...&#92; 
v(n)
end{bmatrix} = begin{bmatrix}
R_1&#92; 
...&#92; 
R_n
end{bmatrix} + gamma begin{bmatrix}
P_{11} &amp; ... &amp; P_{1n}&#92; 
... &amp; ... &amp; ...&#92; 
P_{n1} &amp; ... &amp; P_{nn}
end{bmatrix} begin{bmatrix}
v(1)&#92; 
...&#92; 
v(n)
end{bmatrix}
$$

<ul>
<li><strong>Solving the Bellman Equation</strong></li>
</ul>

上述的 Bellman Equation 可以当做线性方程组来求解.

$$
begin{aligned}
v &amp;= R + gamma Pv &#92;
(I − gamma P) v &amp;= R &#92;
v &amp;= (I − gamma P)^{-1} R
end{aligned}
$$

其中涉及到矩阵逆的计算, 考虑到计算量, 可能只适合于状态空间比较小的 MRPs.

另外也可以用迭代的方式求解, 包括后面会介绍的 DP, MC based methods, TD based methods.

<h1>Markov Decision Processes</h1>

A <strong>Markov decision process (MDP)</strong> is a Markov reward process with decisions. It is an environment in which all states are Markov.

<h2>Definition of MDP</h2>

<blockquote>
  Definition
  A <strong>Markov Decision Process</strong> is a tuple $left langle S, {color{Red} A}, P, R, gamma right rangle$
  1. $S$ is a (finite) set of states
  2. <font color="red"> $A$ is a (finite) set of actions </font>
  3. $P$ is a state transition probability matrix: $P_{ss'}^{a} = P [S_{t+1} = s' | S_t = s, A_t=a]$
  4.  $R$ is a reward function, $R_s^a = E [R_{t+1} | S_t = s, A_t=a]$
  5. $gamma$ is a discount factor, $gamma in [0, 1]$
</blockquote>

<h2>Policy, Value Function &amp; Bellman Expectation Equation</h2>

<ul>
<li><strong>Policy</strong></li>
</ul>

<blockquote>
  Definition
  A <strong>policy</strong> $pi$ is a distribution over actions given states,
  
  $$ pi(a|s) = P [A_t = a | S_t = s]$$
</blockquote>

<strong>MDP 与 MP, MRP 的关系</strong>:

Given an MDP $M = left langle S, A, P, R, gamma right rangle$ and a policy $pi$.
The state sequence $S1, S2, ...$ is a Markov process $M = left langle S, P^pi right rangle$
The state and reward sequence $S1, R2, S2, ...$ is a Markov reward process $left langle S, P^pi, R^pi, gamma right rangle$
where

$$ P_{s,s'}^pi = sum_{a in A} pi(a|s)P_{ss'}^a$$

$$ R_{s}^pi = sum_{a in A} pi(a|s)R_{s}^a$$

<ul>
<li><strong>Value Function</strong></li>
</ul>

<blockquote>
  Definition
  The <strong>state-value function</strong> $v_pi(s)$ of an MDP is the expected return starting from state s, and then following policy $pi$
  
  $$v_pi(s) = E_{pi} [G_t | S_t = s]$$
  
  The <strong>action-value function</strong> $q_pi(s, a)$ is the expected return starting from state s, taking action a, and then following policy $pi$
  
  $$q_{pi}(s, a) = E_{pi} [G_t | S_t = s, A_t = a]$$
</blockquote>

<ul>
<li><strong>Bellman Expectation Equation</strong></li>
</ul>

The state-value function can again be decomposed into immediate reward plus discounted value of successor state,

$$v_{pi}(s) = E_{pi} [R_{t+1} + gamma v_{pi}(S_{t+1}) | S_t = s]$$

The action-value function can similarly be decomposed,

$$q_{pi}(s, a) = E_{pi} [R_{t+1} + gamma q_{pi}(S_{t+1}, A_{t+1}) | S_t = s, A_t = a]$$

进一步有:

$$
begin{aligned}
v_{pi}(s) &amp;= sum_{a in A} pi(a|s)q_{pi}(s,a) &#92;
q_{pi}(s, a) &amp;= R_s^a + gamma sum_{s' in S} P_{ss'}^a v_{pi}(s') &#92;
v_{pi}(s) &amp;= sum_{a in A} pi(a|s) (R_s^a + sum_{s' in S} P_{ss'}^a v_{pi}(s')) &#92;
q_{pi}(s, a) &amp;= R_s^a + gamma sum_{s' in S} P_{ss'}^a sum_{a' in A} pi(a'|s')q_{pi}(s',a')
end{aligned}
$$

<ul>
<li><strong>Bellman Expectation Equation (Matrix Form)</strong></li>
</ul>

The Bellman expectation equation can be expressed concisely using the induced MRP,

$$v_{pi} = R_{pi} + gamma P^{pi}v_{pi}$$

with direct solution

$$v_{pi} = (I − gamma P^{pi})^{−1} R^{pi}$$

<h2>Optimal Policy, Optimal Value Function &amp; Bellman Optimality Equation</h2>

<ul>
<li><strong>Optimal Value Function</strong></li>
</ul>

<blockquote>
  Definition
  The <strong>optimal state-value function</strong> $v_∗(s)$ is the maximum value function over all policies: $v_∗(s) = max_{pi} v_{pi}(s) $.
  The <strong>optimal action-value function</strong> $q_∗(s, a)$ is the maximum action-value function over all policies: $q_∗(s, a) = max_pi q_{pi}(s, a)$
</blockquote>

<ul>
<li><strong>Optimal Policy</strong></li>
</ul>

Define a partial ordering over policies:

$$ pi ≥ pi_0 quad if  quad v_{pi}(s) ≥ v_{pi 0}(s) quad  forall s$$

<strong>Optimal Policy, Optimal Value Function 存在性定理</strong>:

<blockquote>
  Theorem
  For any Markov Decision Process
  1. There exists an optimal policy π∗ that is better than or equal to all other policies, $pi_∗ ≥ pi   forall pi $
  2. All optimal policies achieve the optimal value function, $v_{pi^∗}(s) = v_∗(s)$
  3. All optimal policies achieve the optimal action-value function, $q_{pi^∗}(s, a) = q_∗(s, a)$
</blockquote>

Moreover, there is always a deterministic optimal policy for any MDP. If we know $q_∗(s, a)$, we immediately have the deterministic optimal policy by maximising over $q_∗(s, a)$.

<ul>
<li><strong>Bellman Optimality Equation</strong></li>
</ul>

The optimal value functions are recursively related by the <strong>Bellman optimality equations</strong>:

$$
begin{aligned}
v_∗(s) &amp;= max_a q_∗(s, a) &#92;
q_∗(s, a) &amp;= R_s^a + gamma sum_{s' in S} P_{ss'}^a v_∗(s') &#92;
v_∗(s) &amp;= max_a R_s^a + gamma sum_{s' in S} P_{ss'}^a v_∗(s') &#92;
q_∗(s, a) &amp;= R_s^a + gamma sum_{s' in S} P_{ss'}^a max_{a'}q_∗(s', a')
end{aligned}
$$

<ul>
<li><strong>Solving the Bellman Optimality Equation</strong></li>
</ul>

Bellman Optimality Equation is non-linear, there is no closed form solution (in general).

Many iterative solution methods can be used to solve the Bellman Optimality Equation: Value Iteration, Policy Iteration, Q-learning, Sarsa ...

<h1>Extensions to MDPs</h1>

<strong>参考资料</strong>：Reinforcement Learning Course by David Silver