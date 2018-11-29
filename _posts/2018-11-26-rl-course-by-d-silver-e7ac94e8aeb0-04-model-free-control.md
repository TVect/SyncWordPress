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

<h2>Sarsa</h2>

<strong>Sarsa Algorithm for On-Policy Control</strong>:

Every time-step:

<ul>
<li><strong>Policy evaluation</strong> Sarsa, $Q \approx  q_\pi$</li>
<li><strong>Policy improvement</strong> $\epsilon$-greedy policy improvement</li>
</ul>

<strong>Convergence of Sarsa</strong>:

<blockquote>
  Theorem
  Sarsa converges to the optimal action-value function, $Q(s, a) \rightarrow q_\ast(s, a)$, under the following conditions:
  
  <ul>
  <li>GLIE sequence of policies $π_t(a|s)$</li>
  <li>Robbins-Monro sequence of step-sizes $\alpha_t$
  $ \sum_{t=1}^\infty \alpha_t = \infty $
  $ \sum_{t=1}^\infty \alpha_t^2 &lt; \infty $</li>
  </ul>
</blockquote>

<strong>n-Step Sarsa</strong>:

<ul>
<li>Define the n-step Q-return:</li>
</ul>

$$q_t^{(n)} = R_{t+1} + \gamma R_{t+2} + ... + \gamma^{n−1}R_{t+n} + \gamma^{n}Q(S_{t+n})$$

<ul>
<li>n-step Sarsa updates $Q(s, a)$ towards the n-step Q-return:</li>
</ul>

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (q_t^{(n)} − Q(S_t, A_t))$$

<h2>Sarsa(λ)</h2>

尝试对 1-step return, 2-step return, ..., n-step return, ... 做平均.

<h3>Forward View Sarsa(λ)</h3>

更新公式为: $ Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (q_t^\lambda − Q(S_t, A_t))$

其中, $q_t^\lambda$ 各个时间步 return 的加权平均: $q_t^\lambda = (1 − \lambda) \sum_{n=1}^\infty \lambda^{n-1} q_t^{n}$.

<h3>Backward View Sarsa(λ)</h3>

<ul>
<li>Sarsa(λ) has one eligibility trace for each state-action pair:</li>
</ul>

$$
\begin{aligned}
E_0(s, a) &amp;= 0 &#92;
E_t(s, a) &amp;= \gamma \lambda E_{t−1}(s, a) + 1(S_t=s, A_t=a)
\end{aligned}
$$

<ul>
<li>$Q(s, a)$ is updated for every state s and action a, in proportion to TD-error $\delta_t$ and eligibility trace $E_t(s, a)$:</li>
</ul>

$$
\begin{aligned}
\delta_t &amp;= R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) − Q(S_t, A_t) &#92;
Q(s, a) &amp; \leftarrow Q(s, a) + \alpha \delta_t E_t(s, a)
\end{aligned}
$$

<strong>Sarsa(λ) Algorithm 伪代码</strong>:

<img src="https://i.loli.net/2018/11/27/5bfd4b1e85861.png" width="600" align=center />

<h1>Off-Policy Learning</h1>

Evaluate target policy $\pi(a|s)$ to compute $v_\pi(s)$ or $q_\pi(s, a)$, While following behaviour policy $u(a|s)$.

Why is this important?

<ul>
<li>Learn from observing humans or other agents</li>
<li>Re-use experience generated from old policies</li>
<li>Learn about optimal policy while following exploratory policy</li>
<li>Learn about multiple policies while following one policy</li>
</ul>

<h2>Importance Sampling</h2>

现在要关于 P(X) 求期望, 但很难直接对 P(X)进行采样, 可以通过引入一个 proposal distribution Q(X), 转化为关于 Q(X) 求期望的问题.

$$
E_{X \sim P} [f(X)] = \sum P(X) f(X) = \sum Q(X) \frac {P(X)} {Q(X)} f(X) = E_{X \sim Q} [\frac {P(X)}{Q(X)}f(X)]
$$

<strong>Importance Sampling for Off-Policy Monte-Carlo</strong>

<ul>
<li>基本想法:
Use returns generated from $u$ to evaluate $\pi$, weight return $G_t$ according to similarity between policies.</li>
</ul>

$$
G_t^{\pi/u} = \frac {\pi(A_t|S_t)} {u(A_t|S_t)} \frac {\pi(A_{t+1}|S_{t+1})} {u(A_{t+1}|S_{t+1})} ... \frac {\pi(A_T|S_T)} {u(A_T|S_T)} G_t
$$

$$ V(S_t) \leftarrow V(S_t) + \alpha (G_t^{\pi/u} − V(S_t)) $$

<ul>
<li>备注:
Cannot use if $u$ is zero when $\pi$ is non-zero.
Importance sampling can dramatically increase variance.</li>
</ul>

<strong>Importance Sampling for Off-Policy TD</strong>

<ul>
<li>基本想法:
Use TD targets generated from $u$ to evaluate $\pi$, Weight TD target by importance sampling.</li>
</ul>

$$
V(S_t) \leftarrow V(S_t) + \alpha (\frac {\pi(A_t|S_t)}{u(A_t|S_t)} (R_{t+1} + \gamma V(S_{t+1})) − V(S_t))
$$

<ul>
<li>备注:
Much lower variance than Monte-Carlo importance sampling.
Policies only need to be similar over a single step.</li>
</ul>

<h2>Q-Learning</h2>

consider off-policy learning of action-values $Q(s, a)$ without importance sampling

<ul>
<li>Next action is chosen using behaviour policy $A_{t+1} \sim u(·|S_t)$</li>
<li>But we consider alternative successor action $A_0 \sim \pi(·|S_t)$</li>
<li>And update $Q(S_t, A_t)$ towards value of alternative action:</li>
</ul>

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (R_{t+1} + \gamma Q(S_{t+1}, A^\prime) − Q(S_t, A_t))$$

We now allow both behaviour and target policies to improve.

<ul>
<li>The target policy $\pi$ is greedy w.r.t. $Q(s, a)$: $\pi(S_{t+1}) = \arg \max_{a^\prime} Q(S_{t+1}, a^\prime)$</p></li>
<li><p>The behaviour policy $u$ is e.g. $\epsilon$-greedy w.r.t. $Q(s, a)$.</p></li>
<li><p>The Q-learning target then simplifies:</p></li>
</ul>

<p>$$ 
\begin{aligned}
R_{t+1} + \gamma Q(S_{t+1}, A^\prime) &amp;=  R_{t+1} + \gamma Q(S_{t+1},\arg \max_{a^\prime} Q(S_{t+1}, a^\prime)) &#92;
&amp; = R_{t+1} + \max_{a^\prime} \gamma Q(S_{t+1}, a^\prime)
\end{aligned}
$$

<ul>
<li>此时，更新公式为:</li>
</ul>

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (R_{t+1} + \gamma \max_{a^\prime} Q(S_{t+1}, a^\prime) − Q(S_t, A_t))$$

<strong>Q-learning 伪代码</strong>:

<img src="https://i.loli.net/2018/11/29/5c00002e0d6f9.png" width="600" align=center />

<blockquote>
  Theorem
  Q-learning control converges to the optimal action-value function, $Q(s, a) \rightarrow q_\ast(s, a)$
</blockquote>

<h1>Summary</h1>

<ul>
<li><strong>backup diagram</strong>
<img src="https://i.loli.net/2018/11/29/5bfffd951798b.png" width="600" align=center /></p></li>
<li><p><strong>update equation</strong>
<img src="https://i.loli.net/2018/11/29/5bfffdf94fc17.png" width="600" align=center /></p></li>
</ul>

<hr />

<p><strong>参考资料</strong>: Reinforcement Learning Course by David Silver