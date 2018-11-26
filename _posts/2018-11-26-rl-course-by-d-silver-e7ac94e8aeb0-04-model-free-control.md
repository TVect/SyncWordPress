---
ID: 941
post_title: >
  RL Course by D.Silver 笔记 05 –
  Model-Free Control
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/941
published: true
post_date: 2018-11-26 23:00:38
---
[toc]

在前面 Model-Free Prediction 的基础上, 这里要做的是 Model-Free Control.

<!--more-->

<h1>On-Policy Monte-Carlo Control</h1>

<strong>Policy evaluation</strong>: Monte-Carlo policy evaluation $Q = q_\pi$, instead of state evaluation.

<strong>Policy improvement</strong>: $\epsilon$-greedy policy improvement, instead of greedy policy improvement

<strong>$\epsilon$-Greedy Policy Improvement 的有效性说明</strong>:

<blockquote>
  Theorem
  For any $\epsilon$-greedy policy $\pi$, the $\epsilon$-greedy policy $\pi^{\prime}$ with respect to
  $q_\pi$ is an improvement, $v_{\pi^{\prime}(s)} ≥ v_{\pi(s)}$.
</blockquote>

$$
\begin{aligned}
q_\pi(s, \pi^{\prime}(s)) &amp;= \sum_{a \in A} \pi^{\prime}(a|s) q_\pi(s, a) &#92;
&amp; = \frac {\epsilon}{m} \sum_{a \in A} q_\pi(s, a) + (1-\epsilon) \max_{a \in A} q_\pi(s, a) &#92;
&amp; \geq \frac {\epsilon}{m} \sum_{a \in A} q_\pi(s, a) + (1-\epsilon) \sum_{a \in A} \frac {\pi(a|s) - \epsilon/m}{1-\epsilon} q_\pi(s, a) &#92;
&amp; = \sum_{a \in A} \pi(a|s) q_\pi(s, a) = v_\pi(s)
\end{aligned}
$$

再利用 Lecture03 中 policy improvement theorem 中同样的技巧, 容易得到 $v_{\pi^{\prime}(s)} ≥ v_{\pi(s)}$.

<strong>GLIE</strong>

<blockquote>
  Definition
  Greedy in the Limit with Infinite Exploration (GLIE)
  All state-action pairs are explored infinitely many times: $\lim_{k \rightarrow \infty} N_k(s, a) = \infty $
  The policy converges on a greedy policy: $\lim_{k \rightarrow \infty} \pi_k(a|s) = 1(a = \arg\max_{a \in A} Q_k(s, a)) $
</blockquote>

For example, $\epsilon$-greedy is GLIE if $\epsilon$ reduces to zero at $\epsilon_k = \frac {1}{k}$

<strong>GLIE Monte-Carlo Control</strong>

<ul>
<li>Sample k-th episode using $\pi$: $ {S_1, A_1, R_2, ..., S_T} \sim \pi$</li>
<li>For each state $S_t$ and action $A_t$ in the episode,</li>
</ul>

$$
\begin{aligned}
N(S_t, A_t) &amp;\leftarrow N(S_t, A_t) + 1 &#92;
Q(S_t, A_t) &amp;\leftarrow Q(S_t, A_t) + \frac {1} {N(S_t, A_t)} (G_t − Q(S_t, A_t))
\end{aligned}
$$

<ul>
<li>Improve policy based on new action-value function</li>
</ul>

$$
\begin{aligned}
\epsilon &amp;\leftarrow 1/k &#92;
\pi &amp;\leftarrow \epsilon - greedy(Q)
\end{aligned}
$$

<blockquote>
  Theorem
  GLIE Monte-Carlo control converges to the optimal action-value function: $Q(s, a) \rightarrow q_\ast(s, a)$ .
</blockquote>

<h1>On-Policy Temporal-Difference Learning</h1>

<h1>Off-Policy Learning</h1>

<h1>Summary</h1>