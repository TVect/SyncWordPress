---
ID: 929
post_title: >
  RL Course by D.Silver 笔记 04 –
  Model-Free Prediction
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/929
published: true
post_date: 2018-11-25 18:10:44
---
[toc]

之前使用 dynamic programming 求解 MDP 问题，它的前提是要知道 MDP 的 transitions and rewards.

这里的 Model-Free Prediction 是指在不知道 MDP 的 transitions and rewards 的前提上, 估计 value function.

<!--more-->

<h1>Monte-Carlo Learning</h1>

<strong>Goal</strong>: learn $v_\pi$ from episodes of experience under policy $\pi$:

$$ S_1, A_1, R_2, ...,  S_k \sim \pi $$

Monte-Carlo policy evaluation uses empirical mean return instead of expected return.

<h2>First-Visit / Every-Visit Monte-Carlo Policy Evaluation</h2>

对 total return 关于不同的 state 求平均即可.

<h2>Incremental Monte-Carlo Updates</h2>

<ul>
<li>Update $V(s)$ incrementally after episode $ S_1, A_1, R_2, ..., S_T$</li>
<li>For each state $S_t$ with return $G_t$</li>
</ul>

$$ N(S_t) \leftarrow N(S_t) + 1 $$

$$ V(S_t) \leftarrow V(S_t) + \frac{1}{N(S_t)}(G_t - V(S_t))$$

<ul>
<li>In non-stationary problems, it can be useful to track a running mean, i.e. forget old episodes.</li>
</ul>

$$ V(S_t) \leftarrow V(S_t) + \alpha (G_t - V(S_t))$$

<h1>Temporal-Difference Learning</h1>

<h1>TD(λ)</h1>