---
ID: 89
post_title: 'RL Course by D.Silver 笔记 01 &#8211; Introduction'
author: chin340823
post_excerpt: ""
layout: post
permalink: https://blog.tvect.cn/?p=89
published: true
post_date: 2018-11-17 21:54:36
---
[toc]

<!--more-->

<h1>About Reinforcement Learning</h1>

强化学习与其他机器学习分支相比的不同点:

<ul>
<li>There is no supervisor, only a reward signal</p></li>
<li><p>Feedback is delayed, not instantaneous</p></li>
<li><p>Time really matters (sequential, non i.i.d data)</p></li>
<li><p>Agent’s actions affect the subsequent data it receives</p></li>
</ul>

<h1>The Reinforcement Learning Problem</h1>

<h2>Rewards</h2>

<p>Rewards $R_{t}$ 是一个标量值的反馈信号，它反映 agent 在 $t$ 时刻做得怎么样。agent 的目标就是要最大化 cumulative reward。

Reinforcement learning is based on the <strong>reward hypothesis</strong> : All goals can be described by the maximisation of expected cumulative reward

<h2>Agent and Environment</h2>

At each step t:

<ul>
<li>the agent:
executes action $A_t$, Receives observation $O_t$, Receives scalar reward $R_t$</li>
<li>The environment:
receives action $A_t$, emits observation $O_{t+1}$, emits scalar reward $R_{t+1}$</li>
</ul>

t increments at env. step

<h2>History and State</h2>

<ul>
<li><strong>history</strong>
The <strong>history</strong> is the sequence of observations, actions, rewards
$ H_t = O_1, R_1, A_1, ..., A_{t-1}, O_t, R_t $
i.e. all observable variables up to time t.</p></li>
<li><p><strong>State</strong>
<strong>State</strong> is the information used to determine what happens next. Formally, state is a function of the history: $S_t = f(H_t)$</p></li>
</ul>

<p>有点像 task-oriented dialog system 中的每一轮会话 nlu 等等的记录和 dst 的关系?

<h3>三种不同的 State 概念</h3>

<ul>
<li><strong>Environment State</strong>
The <strong>environment state</strong> $S_t^e$ is the environment’s private representation, i.e. whatever data the environment uses to pick the next observation or reward.
The environment state is not usually visible to the agent. Even if $S_t^e$ is visible, it may contain irrelevant information</p></li>
<li><p><strong>Agent State</strong>
The <strong>agent state</strong> $S_t^a$ is the agent’s internal representation, i.e. whatever information the agent uses to pick the next action, i.e. it is the information used by reinforcement learning algorithms.
It can be any function of history: $S_t = f(H_t)$</p></li>
<li><p><strong>Information State</strong>
An <strong>information state</strong> (a.k.a. <strong>Markov state</strong>) contains all useful information from the history.

<blockquote>
  Definition
  A state $S_t$ is Markov if and only if $P[S_{t+1} | St] = P[S_{t+1} | S_1, ..., S_t]$
</blockquote>

Once the state is known, the history may be thrown away, i.e. The state is a sufficient statistic of the future.
e.g. The environment state $S_t^e$ is Markov. The history $H_t$ is Markov</p></li>
</ul>

<h3>两种不同的 Environments</h3>

<ul>
<li><p>Fully Observable Environments
Agent <strong>directly</strong> observes environment state: $O_t = S_t^a = S_t^e$
Formally, this is a <strong>Markov decision process</strong> (MDP)</p></li>
<li><p>Partially Observable Environments
Agent <strong>indirectly</strong> observes environment. Now $S_t^a neq S_t^e$, and agent must construct its own state representation $S_t^a$.
Formally this is a <strong>partially observable Markov decision process</strong> (POMDP)</p></li>
</ul>

<h1>Inside An RL Agent</h1>

<h2>Components</h2>

<ul>
<li><p><strong>Policy</strong>: agent’s behaviour function</p></li>
<li><p><strong>Value function</strong>: how good is each state and/or action</p></li>
<li><p><strong>Model</strong>: agent’s representation of the environment
A model predicts what the environment will do next，incluing next state and next (immediate) reward.</p></li>
</ul>

<h2>RL Agent Taxonomy</h2>

<ul>
<li>Categorizing RL agents (1)</li>
</ul>

<table>
<thead>
<tr>
  <th>Taxonomy</th>
  <th></th>
  <th></th>
</tr>
</thead>
<tbody>
<tr>
  <td>Value Based</td>
  <td>No Policy (Implicit)</td>
  <td>Value Function</td>
</tr>
<tr>
  <td>Policy Based</td>
  <td>Policy</td>
  <td>No Value Function</td>
</tr>
<tr>
  <td>Actor Critic</td>
  <td>Policy</td>
  <td>Value Function</td>
</tr>
</tbody>
</table>

<ul>
<li>Categorizing RL agents (2)</li>
</ul>

<table>
<thead>
<tr>
  <th>Taxonomy</th>
  <th></th>
  <th></th>
</tr>
</thead>
<tbody>
<tr>
  <td>Model Free</td>
  <td>Policy and/or Value Function</td>
  <td>No Model</td>
</tr>
<tr>
  <td>Model Based</td>
  <td>Policy and/or Value Function</td>
  <td>Model</td>
</tr>
</tbody>
</table>

<h1>Problems within Reinforcement Learning</h1>

<h2><strong>Learning and Planning</strong></h2>

<ul>
<li><p><strong>Reinforcement Learning</strong>:
The environment is initially unknown. The agent interacts with the environment. The agent improves its policy.</p></li>
<li><p><strong>Planning</strong>:
A model of the environment is known. The agent performs computations with its model (without any external interaction). The agent improves its policy.</p></li>
</ul>

<h2>Exploration and Exploitation</h2>

<p><strong>Exploration</strong> finds more information about the environment.

<strong>Exploitation</strong> exploits known information to maximise reward.

It is usually important to explore as well as exploit.

<h2>Prediction and Control</h2>

<ul>
<li><strong>Prediction</strong>: evaluate the future
Given a policy</p></li>
<li><p><strong>Control</strong>: optimise the future
Find the best policy</p></li>
</ul>

<hr />

<p><strong>参考资料</strong>：Reinforcement Learning Course by David Silver