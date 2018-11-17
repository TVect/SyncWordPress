---
ID: 876
post_title: 'Reinforcement Learning Course by David Silver 笔记 &#8211; Lecture01'
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/876
published: true
post_date: 2018-11-17 21:54:36
---
[toc]

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

<img src="https://pic2.zhimg.com/80/v2-2e404ba62b53818c20d1082375903ad1_hd.png" alt="image" />

At each step t 
- the agent:
    - Executes action $A_t$
    - Receives observation $O_t$
    - Receives scalar reward $R_t$
- The environment:
    - Receives action $A_t$
    - Emits observation $O_{t+1}$
    - Emits scalar reward $R_{t+1}$

t increments at env. step

<h2>History and State</h2>

The <strong>history</strong> is the sequence of observations, actions, rewards

$ H_t = O_1, R_1, A_1, ..., A_{t-1}, O_t, R_t $

i.e. all observable variables up to time t.

<strong>State</strong> is the information used to determine what happens next. Formally, state is a function of the history: $S_t = f(H_t)$

有点像 task-oriented dialog system 中的每一轮会话 nlu 等等的记录和 dst 的关系?

<h3>三种不同的 State 概念</h3>

<ul>
<li><strong>Environment State</strong></li>
</ul>

The <strong>environment state</strong> $S_t^e$ is the environment’s private representation, i.e. whatever data the environment uses to pick the next observation or reward.

The environment state is not usually visible to the agent. Even if $S_t^e$ is visible, it may contain irrelevant information

<ul>
<li><strong>Agent State</strong></li>
</ul>

The <strong>agent state</strong> $S_t^a$ is the agent’s internal representation, i.e. whatever information the agent uses to pick the next action, i.e. it is the information used by reinforcement learning algorithms.

It can be any function of history: $S_t = f(H_t)$

<ul>
<li><strong>Information State</strong></li>
</ul>

An <strong>information state</strong> (a.k.a. <strong>Markov state</strong>) contains all useful information from the history.

<blockquote>
  Definition
  A state $S_t$ is Markov if and only if $P[S_{t+1} | St] = P[S_{t+1} | S_1, ..., S_t]$
</blockquote>

<h3>Fully Observable Environments</h3>

<h3>Partially Observable Environments</h3>

<h1>Inside An RL Agent</h1>

<h2>Components</h2>

<h3>Policy: agent’s behaviour function</h3>

<h3>Value function: how good is each state and/or action</h3>

<h3>Model: agent’s representation of the environment</h3>

<h2>RL Agent Taxonomy</h2>

<h1>Problems within Reinforcement Learning</h1>