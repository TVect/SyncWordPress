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

<h1>Value Iteration</h1>

<h1>Extensions to Dynamic Programming</h1>

<h1>Contraction Mapping</h1>