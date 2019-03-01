---
ID: 213
post_title: >
  Local Minima in Deep Learning
  相关文献汇总
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/213
published: true
post_date: 2019-02-28 17:53:19
---
这里整理了一些关于 Deep Learning 中 local minima 的文章.

主要是关于如何避免在 Deep Learning 中可能遇到的 local minima 的问题, 如何找到 global minima 等.

<!--more-->

<h2><a href="https://arxiv.org/abs/1805.08671">Adding One Neuron Can Eliminate All Bad Local Minima</a> (2018.05)</h2>

文章中主要考虑的是 binary classification problem with a special
type of smoothed hinge loss functions.

文章中指出, for any neural network, by adding a special neuron (e.g., exponential neuron) to the network and adding a quadratic regularizer of this neuron, the new loss function has no bad local minimum.

<ul>
<li>博客: <a href="https://rossum.ai/blog/does-adding-one-neuron-help-real-world-networks/">Does Adding One Neuron Help Real World Networks?</a>

给出了一些使用 exponential neural 的例子.</p></li>
</ul>

<h2><a href="https://arxiv.org/abs/1901.00279">Elimination of All Bad Local Minima in Deep Learning</a> (2019.01)</h2>

<p>对 <a href="">Adding One Neuron Can Eliminate All Bad Local Minima</a>文章的结果进行了推广, 弱化了假设条件, 使得添加 exponential neuron 的方式适用于更多不同的问题(或者 loss 函数).

<blockquote>
  In this paper, we prove, without any strong assumption, that adding one neuron per output unit can eliminate all suboptimal local minima for multi-class classification, binary classification, and regression with an arbitrary loss function.
</blockquote>

另外, 文章中也提到了这种消除 bad local minima 方法的局限性和缺点，即这种方法并不能保证我们能找到 $\tilde{L}(\theta, a, W, b)$ 的局部极小(对应着 $L(\theta)$ 的全局极小.)

<ol>
<li>消除 local minima 并不表示在多项式时间内找到global optima.</li>
</ol>

<blockquote>
  eliminating all suboptimal local minima is not sufficient to guarantee the global optimality in polynomial time without taking advantage of additional assumptions or structures of neural networks. (because of the possible discrete nature of the space of $\theta$, and of bad saddle points )
</blockquote>

<ol start="2">
<li>对于原本 $L(\theta)$ 的局部极小 $\theta_0$, 新构造的损失函数 $\tilde{L}(\theta, a, W, b)$ 在 $\theta_0$ 处可能不是局部极小 (diverge in terms of $(\theta_0, a, b, W)$). 也就是说, 这种方法可能反而会错过 $L(\theta)$ 的局部极小.</li>
</ol>

<blockquote>
  an iterative optimization algorithm can converge to a suboptimal local minimum $\theta$ of $L$ (i.e., a local minimum that is not a global minimum), and diverge in terms of $(θ, a, b, W)$ of $\tilde{L}$.
</blockquote>