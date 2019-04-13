---
ID: 283
post_title: 典型形式的Bayesian Optimization
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/283
published: true
post_date: 2019-04-13 21:59:17
---
[toc]

Bayesian Optimization 可以用来做 black-box derivative-free global optimization. 这篇博客整理记录了基本形式的 BayesOpt 的做法.

<!--more-->

<h1>Overview of Bayesian Optimization</h1>

Bayesian optimization (BayesOpt) is a class of machine-learning-based optimization methods focused on solving the problem $$ max_{x \in A} f(x) $$

where
1. The feasible set $A$ is a simple set, in which it is easy to assess membership.
2. The objective function $f$ is continuous. And $f$ is "expensive to evaluate" in the sense that the number of evaluations that may be performed is limited, typically to a few hundred.

因为BayesOpt 可以 optimize expensive black-box derivative-free functions, 所以可以用 BayesOpt 去做 Machine Learning 中超参数的调优.
(常见的 GridSearch, RandomSearch 之类的参数搜索调参方式会独立评估每一个参数组合, BayesOpt 会借鉴之前参数组合的效果, 来选择下一步要探索的参数组合, 相比之下不会像前面提到的两种方式那样显得盲目.)

<h1>Components of Bayesian Optimization</h1>

典型的 Bayesin Optimization 方法包括两个部分:
1. surrogate for the objective/a method for statistical inference, typically Gaussian process (GP) regression;
2. an acquisition function for deciding where to sample, which is often expected improvement.

典型的 Bayesian Optimization 的算法伪码如下:

<img src="https://www.tvect.cn/wp-content/uploads/2019/04/043425d0a6f414fdbaca2ddb3a9a5781.png" alt="" />

下面分别对这两部分做一下介绍.

<h2>surrogate for the objective</h2>

常用 高斯过程回归 来建模 $P(y | x, D)$. 高斯过程的知识可以参考 <a href="https://www.tvect.cn/archives/85">Gaussian Processes 初步 笔记</a>

<h2>acquisition function</h2>

这里主要介绍  expected improvement 的方法.

<h3>Expected Improvement</h3>

假定到目前为止, 已经观测到的数据点为 $ (x_1, y_1), (x_2, y_2), ..., (x_n, y_n)$, 考虑要从中选取一个数据点 $x_i$, 其对应的 $y_i$ 是这些已观察中最大的, 把这个 $x_i$ 作为 final solution 返回.

令 $ f_n^* = max_{i&lt;=n} f(x_i)$, 假设现在新增加了一个观测 $(x, f(x))$, 可以把 Expected Improvement 自然的定义为:

$ EI_n[x] := E_n[f(x) - f_n^*] $

Here, $E_n[·] = E[· | x1:n; y1:n]$ indicates the expectation taken under the posterior distribution given evaluations of $f$ at $x_1, ..., x_n$ (也即是由前面的高斯过程回归的预测分布给出).

expected improvement algorithm 即选择 $x_{n+1} = max_x EI_n(x)$ 作为下一个待评估点.

<h1>参考资料</h1>

<ul>
<li><p><a href="https://app.sigopt.com/static/pdf/SigOpt_Bayesian_Optimization_Primer.pdf">Bayesian Optimization Primer</a></p></li>
<li><p><a href="https://arxiv.org/abs/1807.02811">A Tutorial on Bayesian Optimization</a></p></li>
</ul>