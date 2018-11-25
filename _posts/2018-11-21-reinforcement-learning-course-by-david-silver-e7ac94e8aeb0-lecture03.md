---
ID: 910
post_title: 'RL Course by D.Silver 笔记 03 &#8211; Dynamic Programming'
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/910
published: true
post_date: 2018-11-21 23:29:27
---
[toc]

<!--more-->

<h1>Introduction</h1>

Dynamic Programming is a very general solution method for problems which have two properties:
1. Optimal substructure
2. Overlapping subproblems

Markov decision processes satisfy both properties：
1. Bellman equation gives recursive decomposition
2. Value function stores and reuses solutions

<strong>Dynamic Programming 在 MDP 中应用</strong>

Dynamic programming assumes full knowledge of the MDP. It is used for planning in an MDP.

<ul>
<li>For prediction:
Input: MDP $\left \langle S, A, P, R, \gamma \right \rangle$ and policy $\pi$, or MRP $\left \langle S, P^{\pi}, R^{\pi}, \gamma \right \rangle$
Output: value function $v_{\pi}$</p></li>
<li><p>For control:
Input: MDP $\left \langle S, A, P, R, \gamma \right \rangle$
Output: optimal value function $v_* $ and optimal policy $\pi_*$</p></li>
</ul>

<h1>Policy Evaluation</h1>

<p><strong>Problem</strong>: evaluate a given policy $\pi$

<strong>Solution</strong>: iterative application of Bellman expectation backup

<ul>
<li><strong>synchronous backups</strong>
At each iteration k + 1
For all states $s \in S$
Update $v_{k+1}(s)$ from $v_k(s')$, where $s'$ is a successor state of $s$.</li>
</ul>

<h1>Policy Iteration</h1>

<strong>基本流程</strong>

<blockquote>
  Evaluate the policy $\pi$,
  
  $$v_\pi(s) = E[R_{t+1} + \gamma R_{t+2} + ... | S_t=s]$$
  
  Improve the policy by acting greedily with respect to $v_\pi$
  
  $$ {\pi}^{\prime} = greedy(v_{\pi})$$
</blockquote>

this process of policy iteration always converges to $\pi^{*}$.

<strong>Policy Improvement 有效性</strong>

<blockquote>
  Consider a deterministic policy, $a = \pi(s)$, We can improve the policy by acting greedily: ${\pi}^{\prime}(s) = \mathop{\arg \max}<em>{a \in A} q</em>{\pi}(s, a)$
</blockquote>

<ul>
<li>证明</li>
</ul>

$$
q_{\pi}(s, {\pi}^{\prime}(s)) = \mathop{\max}<em>{a \in A} q</em>{\pi}(s,a) \geq q_{\pi}(s, \pi(s)) = v_{\pi}(s)
$$

接下来, 重复上面的不等式, 证明 $v_{{\pi}^{\prime}}(s) \geq v_\pi(s)$.

$$
\begin{aligned}
v_{\pi}(s) &amp; \leq q_{\pi}(s, {\pi}^{\prime}(s)) = E_{{\pi}^{\prime}}[R_{t+1} + \gamma v_{\pi}(S_{t+1}) | S_t=s] &#92;
&amp; \leq E_{{\pi}^{\prime}}[R_{t+1} + \gamma q_{\pi}(S_{t+1}, {\pi}^{\prime}(S_{t+1})) | S_t=s] = E_{{\pi}^{\prime}}[R_{t+1} + \gamma R_{t+2} + \gamma^2 v_{\pi}(S_{t+2})| S_t=s] &#92;
&amp; \leq E_{{\pi}^{\prime}}[R_{t+1} + \gamma R_{t+2} + \gamma^{2} q_{\pi}(S_{t+2}, {\pi}^{\prime}(S_{t+2})) | S_t=s] &#92;
&amp; \leq ... &#92;
&amp; \leq E_{{\pi}^{\prime}} [R_{t+1} + \gamma R_{t+2} + \gamma^{2} R_{t+3} + ... | S_t=s] = v_{\pi}(s)
\end{aligned}
$$

在 policy improvement 终止的时候有:

$$
q_{\pi}(s, {\pi}^{\prime}(s)) = \mathop{\max}<em>{a \in A} q</em>{\pi}(s,a) = q_{\pi}(s, \pi(s)) = v_{\pi}(s)
$$

此时, Bellman optimality equation $v_{\pi}(s)=\mathop{\max}<em>{a \in A} q</em>{\pi}(s,a)$ 得到满足.
因此, $v_{\pi}(s) = v_*(s) \quad \forall s \in S$, 即有 $\pi$ 是 an optimal policy.

<strong>Generalised Policy Iteration</strong>

<img src="https://i.loli.net/2018/11/25/5bfa58e2e1b45.png" width="600" align=center />

<h1>Value Iteration</h1>

<h1>Extensions to Dynamic Programming</h1>

<h1>Contraction Mapping</h1>