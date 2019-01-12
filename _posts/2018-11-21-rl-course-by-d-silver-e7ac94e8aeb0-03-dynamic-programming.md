---
ID: 93
post_title: 'RL Course by D.Silver 笔记 03 &#8211; Dynamic Programming'
author: chin340823
post_excerpt: ""
layout: post
permalink: https://blog.tvect.cn/?p=93
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
Input: MDP $left langle S, A, P, R, gamma right rangle$ and policy $pi$, or MRP $left langle S, P^{pi}, R^{pi}, gamma right rangle$
Output: value function $v_{pi}$</p></li>
<li><p>For control:
Input: MDP $left langle S, A, P, R, gamma right rangle$
Output: optimal value function $v_* $ and optimal policy $pi_*$</p></li>
</ul>

<h1>Policy Evaluation</h1>

<p><strong>Problem</strong>: evaluate a given policy $pi$

<strong>Solution</strong>: iterative application of Bellman expectation backup

<ul>
<li><strong>synchronous backups</strong>
At each iteration k + 1
For all states $s in S$
Update $v_{k+1}(s)$ from $v_k(s')$, where $s'$ is a successor state of $s$.</li>
</ul>

<h1>Policy Iteration</h1>

<strong>基本流程</strong>

<blockquote>
  Evaluate the policy $pi$,
  
  $$v_pi(s) = E[R_{t+1} + gamma R_{t+2} + ... | S_t=s]$$
  
  Improve the policy by acting greedily with respect to $v_pi$
  
  $$ {pi}^{prime} = greedy(v_{pi})$$
</blockquote>

this process of policy iteration always converges to $pi^{*}$.

<strong>Policy Improvement 有效性</strong>

<blockquote>
  Consider a deterministic policy, $a = pi(s)$, We can improve the policy by acting greedily: ${pi}^{prime}(s) = mathop{arg max}<em>{a in A} q</em>{pi}(s, a)$
</blockquote>

<ul>
<li>证明</li>
</ul>

$$
q_{pi}(s, {pi}^{prime}(s)) = mathop{max}<em>{a in A} q</em>{pi}(s,a) geq q_{pi}(s, pi(s)) = v_{pi}(s)
$$

接下来, 重复上面的不等式, 证明 $v_{{pi}^{prime}}(s) geq v_pi(s)$.

$$
begin{aligned}
v_{pi}(s) &amp; leq q_{pi}(s, {pi}^{prime}(s)) = E_{{pi}^{prime}}[R_{t+1} + gamma v_{pi}(S_{t+1}) | S_t=s] &#92;
&amp; leq E_{{pi}^{prime}}[R_{t+1} + gamma q_{pi}(S_{t+1}, {pi}^{prime}(S_{t+1})) | S_t=s] = E_{{pi}^{prime}}[R_{t+1} + gamma R_{t+2} + gamma^2 v_{pi}(S_{t+2})| S_t=s] &#92;
&amp; leq E_{{pi}^{prime}}[R_{t+1} + gamma R_{t+2} + gamma^{2} q_{pi}(S_{t+2}, {pi}^{prime}(S_{t+2})) | S_t=s] &#92;
&amp; leq ... &#92;
&amp; leq E_{{pi}^{prime}} [R_{t+1} + gamma R_{t+2} + gamma^{2} R_{t+3} + ... | S_t=s] = v_{pi}(s)
end{aligned}
$$

在 policy improvement 终止的时候有:

$$
q_{pi}(s, {pi}^{prime}(s)) = mathop{max}<em>{a in A} q</em>{pi}(s,a) = q_{pi}(s, pi(s)) = v_{pi}(s)
$$

此时, Bellman optimality equation $v_{pi}(s)=mathop{max}<em>{a in A} q</em>{pi}(s,a)$ 得到满足.
因此, $v_{pi}(s) = v_*(s) quad forall s in S$, 即有 $pi$ 是 an optimal policy.

<strong>Generalised Policy Iteration</strong>

<img src="https://i.loli.net/2018/11/25/5bfa58e2e1b45.png" width="600" align=center />

<h1>Value Iteration</h1>

<h2>Value Iteration in MDPs</h2>

<strong>Principle of Optimality</strong>

Any optimal policy can be subdivided into two components:

<ul>
<li>An optimal first action $A_*$</li>
<li>Followed by an optimal policy from successor state $S^{prime}$</li>
</ul>

<blockquote>
  Theorem (<strong>Principle of Optimality</strong>)
  A policy $pi(a|s)$ achieves the optimal value from state $s$, $v_pi(s) = v_ast(s)$ ,
  if and only if, for any state $s^{prime}$ reachable from $s$, $pi$ achieves the optimal value from state $s^{prime}$, $v_pi(s^{prime}) = v_ast(s^{prime})$
</blockquote>

<strong>Deterministic Value Iteration</strong>

If we know the solution to subproblems $v_ast(s^{prime})$, then solution $v_ast(s)$ can be found by one-step lookahead

$$v_ast(s) leftarrow max_{a in A} R_s^a + gamma sum_{s^{prime} in S} P_{ss^{prime}}^a v_ast(s^{prime}) $$

<strong>Value Iteration</strong>

<strong>Problem</strong>: find optimal policy $pi$
<strong>Solution</strong>: iterative application of Bellman optimality backup

<ul>
<li><strong>synchronous backups</strong>
At each iteration k + 1
For all states $ s in S $
Update $v_{k+1}(s)$ from $v_k(s^{prime})$</li>
</ul>

Unlike policy iteration, there is no explicit policy. And intermediate value functions may not correspond to any policy.

<h2>Summary of DP Algorithms</h2>

<strong>Synchronous Dynamic Programming Algorithms</strong>
<table>
<thead>
<tr>
  <th>Problem</th>
  <th>Bellman Equation</th>
  <th>Algorithm</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Prediction</td>
  <td>Bellman Expectation Equation</td>
  <td>Iterative Policy Evaluation</td>
</tr>
<tr>
  <td>Control</td>
  <td>Bellman Expectation Equation + Greedy Policy Improvement</td>
  <td>Policy Iteration</td>
</tr>
<tr>
  <td>Control</td>
  <td>Bellman Optimality Equation</td>
  <td>Value Iteration</td>
</tr>
</tbody>
</table>

<h1>Extensions to Dynamic Programming</h1>

<h2>Asynchronous Dynamic Programming</h2>

<ul>
<li>Asynchronous DP backs up states individually, in any order.</li>
<li>For each selected state, apply the appropriate backup can significantly reduce computation.</li>
<li>Guaranteed to converge if all states continue to be selected</li>
</ul>

asynchronous dynamic programming 有以下几种简单的做法:

<h3>In-place dynamic programming</h3>

In-place value iteration only stores one copy of value function.

$$v(s) leftarrow max_{a in A} (R_s^a + gamma sum_{s^{prime} in S} P_{ss^prime}^a v(s^prime)) $$

<h3>Prioritised sweeping</h3>

使用 Bellman error 的大小来指导 state 更新选择的顺序, 如可以使用如下的 Bellman error:

$$|max_{a in A} (R_s^a + gamma sum_{s^{prime} in S} P_{ss^prime}^a v(s^prime)) -v(s)|$$

<h3>Real-time dynamic programming</h3>

使用 Use agent’s experience 来指导 state 更新选择的顺序.

观察到一个 experience $S_t, A_t, R_{t+1}$ 之后, 就对 $v(S_t)$ 做更新.

<h2>Full-width and sample backups</h2>

...

<h2>Approximate Dynamic Programming</h2>

...

<h1>Contraction Mapping</h1>

定义 算子 $T$ 是 满足 $gamma-constraction$ 的, 如果有: $|T(u) - T(v)|<em>{infty } leq gamma |u-v|</em>{infty}$ .

<blockquote>
  Theorem (Contraction Mapping Theorem)
  For any metric space V that is complete (i.e. closed) under an operator $T(v)$, where $T$ is a $gamma-contraction$,
  then, $T$ converges to a unique fixed point at a linear convergence rate of $gamma$.
</blockquote>

现在, 定义 V 为由 value functions 构成的空间(或者是 $|S|$ 维的向量空间), 在其上分别定义 Bellman expectation backup operator 和 Bellman optimality backup operator, 具体如下:

<ul>
<li>定义 Bellman expectation backup operator $T^pi(v) = R^pi + gamma P^pi v $, 其满足 $gamma-constraction$ 条件, 根据 Constraction Mapping Theorem, 容易得到 Policy Evaluation 和 Policy Iteration 的收敛性.</p></li>
<li><p>定义 Bellman optimality backup operator $T^ast(v) = max_{a in A} R^a + gamma P^a v $, 其满足 $gamma-constraction$ 条件, 根据 Constraction Mapping Theorem, 容易得到 Value Iteration 的收敛性.</p></li>
</ul>

<p><strong>参考资料</strong>: Reinforcement Learning Course by David Silver