---
ID: 210
post_title: State Space Models 之 LDS
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/210
published: true
post_date: 2019-02-19 17:53:55
---
这篇文章会对 Linear Dynamic Systems 做一些总结.

<h1>Introduction</h1>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/758ee40569603466ba0c6a8efe39ba4e.png" alt="" />

LDS 与 HMM 类似, 不过这里的隐状态是连续变量.

这里先考虑一类特殊情况, linear-Gaussian state space model that the latent variables $z_n$, as well as the observed variables $x_n$, are multivariate Gaussian distributions whose means are linear functions of the states of their parents in the graph.

模型具体表示如下：

<h1>参考资料</h1>

<ul>
<li><p>PRML Chapter 13.3  Linear Dynamical Systems</p></li>
<li><p><a href="https://github.com/roboticcam/machine-learning-notes/blob/master/dynamic_model.pdf">Dynamic Model by Yida Xu</a></p></li>
</ul>