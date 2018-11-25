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

MC learns from complete episodes: no bootstrapping.

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

TD learns from incomplete episodes, by bootstrapping.

<strong>Simplest temporal-difference learning algorithm: TD(0)</strong>

<ul>
<li>Update value $V(S_t)$ toward estimated return $R_{t+1} + \gamma V(S_{t+1})$</li>
</ul>

$$V(S_t) \leftarrow V(S_t) + \alpha (R_{t+1} + \gamma V(S_{t+1}) − V(S_t))$$

<ul>
<li>$R_{t+1} + \gamma V(S_{t+1}) $ is called the TD target</li>
<li>$\delta_t = R_{t+1} + \gamma V(S_{t+1}) − V(S_t))$ is called the TD error</li>
</ul>

<strong>Advantages and Disadvantages of MC vs. TD</strong>

<ul>
<li>TD can learn before knowing the final outcome
TD can learn online after every step, MC must wait until end of episode before return is known</p></li>
<li><p>TD can learn without the final outcome
TD can learn from incomplete sequences, MC can only learn from complete sequences
TD works in continuing (non-terminating) environments, MC only works for episodic (terminating) environments</p></li>
<li><p>bias vs. variance
Return $G_t = R_{t+1} + \gamma R_{t+2} + ... $ is <strong>unbiased</strong> estimate of $v_\pi(S_t)$
True TD target $R_{t+1} + \gamma v_\pi(S_{t+1})$ is <strong>unbiased</strong> estimate of $v_\pi(S_t)$
TD target $R_{t+1} + \gamma V(S_{t+1})$ is <strong>biased</strong> estimate of $v_\pi(S_t)$

<ul>
<li>MC has high variance, zero bias
Good convergence properties (even with function approximation)
Not very sensitive to initial value
Very simple to understand and use</p></li>
<li><p>TD has low variance, some bias
Usually more efficient than MC
TD(0) converges to $v_\pi(s)$ (but not always with function approximation)
More sensitive to initial value</p></li>
</ul></li>
<li><p>Markov property

<ul>
<li>TD exploits Markov property
Usually more efficient in Markov environments</p></li>
<li><p>MC does not exploit Markov property
Usually more effective in non-Markov environments</p></li>
</ul></li>
</ul>

<h2>Unified View of Reinforcement Learning</h2>

<p><img src="https://i.loli.net/2018/11/25/5bfa7a7365f12.png" width="600" align=center />

<h1>TD(λ)</h1>

<h2>n-Step TD</h2>

Define the n-step return

$$G_tR_{t+1} + \gamma R_{t+2} + ... + \gamma^{n−1}R_{t+n} + \gamma^n V(S_{t+n})$$

n-step temporal-difference learning:

$$V(S_t) \leftarrow V(S_t) + \alpha (G_t^n − V(S_t))$$

<h2>Forward View of TD(λ)</h2>

The $\lambda$-return $G_t^\lambda$ combines all n-step returns $G_t^n$