---
ID: 222
post_title: notes on state space models
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/222
published: true
post_date: 2019-03-14 23:45:16
---
<h1>Motivation</h1>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/758ee40569603466ba0c6a8efe39ba4e.png" alt="" />

<h2>Sequential Data</h2>

这里考虑序列数据建模的问题.

在一般情况下, 观测变量序列的概率分布可以有通用的分解方式: $ p(x_1, ..., x_N) = \prod_{n=1}^{N} p(x_n | x_1, ..., x_{n-1})$

这种表达方式, 参数量太大, 需要考虑一些限制假设.

<strong>Markov Chain</strong>:

最简单的办法是引入 Markov Assumption, 得到如图 Figure 13.3 的 Markov Chain, 此时: $ p(x_1, ..., x_N) = p(x_1) \prod_{n=2}^{N} p(x_n | x_{n-1})$

<strong>从 Markov Chain 到 State Space Models</strong>:

Markov Chain 可以用 限制参数数量 的模型来描述数据, 但是其中 observed data 受限于了 Markov Assumption.

为了能够让 observed data 不再受限于 Markov Assumption, 同时可以让模型可以用 a limited number of free parameters 来表示, 可以考虑为每一个观测变量引入一个相应的隐变量, 得到如图 Figure 13.5 所示的 State Space Models.

模型的联合分布为: $ p(x_1, ..., x_N, z_1, ..., z_N) = p(z_1) [\prod_{n=1}^{N} p(z_n | z_{n-1})] \prod_{n=1}^{N} p(x_n | z_n) $.

在这个模型中:

<ol>
<li>隐变量序列之间有 Markov 性.</p></li>
<li><p>观测变量序列不假设有 Markov 性, 实际上观测序列上并没有与 Markov Chain 对应的条件独立性, 也即当前观测依赖于之前所有的观测.</p></li>
</ol>

<p>具体的有两种重要的序列数据模型可以用 Figure 13.5 来描述. 一种是 Hidden Markov Models, 对应着隐状态为离散变量的情况. 另一种是 Linear Dynamic Models, 对应着隐状态为连续变量的情况.

下面分别说明.

<h1>HMM</h1>

参见 <a href="https://www.tvect.cn/archives/224">State Space Models 之 HMM</a>

<h1>LDS</h1>

<h1>参考资料</h1>

<ul>
<li><a href="">PRML Chapter 13 Sequential Data</a></li>
</ul>