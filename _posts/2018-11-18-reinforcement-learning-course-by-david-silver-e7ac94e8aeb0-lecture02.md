---
ID: 883
post_title: 'Reinforcement Learning Course by David Silver 笔记 &#8211; Lecture02'
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/883
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
  A <strong>Markov Process</strong> (or <strong>Markov Chain</strong>) is a tuple $\left \langle S, P \right \rangle$
  1. $S$ is a (finite) set of states
  2. $P$ is a state transition probability matrix: $P_{ss'} = P [S_{t+1} = s' | S_t = s]$
</blockquote>

<h1>Markov Reward Processes</h1>

<h1>Markov Decision Processes</h1>

<h1>Extensions to MDPs</h1>