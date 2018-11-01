---
ID: 779
post_title: Noise Contrastive Estimation 笔记
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/779
published: true
post_date: 2018-11-01 10:45:38
---
[toc]

NCE 相关的笔记整理, 包括了 NCE 的原理, 有效性证明, 代码实现说明 等。

<!--more-->

<h1>概述</h1>

<h1>原理</h1>

<h1>NCE 和 Negative Sampling</h1>

<h1>案例</h1>

<h2>tensorflow 中 nce_loss 实现代码</h2>

设计思路可以参考 [文章: Simple, Fast Noise-Contrastive Estimation for Large RNN Vocabularies], [文章: A fast and simple algorithm for training neural probabilistic language model],

<ul>
<li><strong>关于 normalizer</strong></li>
</ul>

[文章: A fast and simple algorithm for training neural probabilistic language model]中有详细提到 normalizing constants 的处理。

首先从公式中可以看出，在训练词向量的时候应该为每一个 context 学习一个 normalizing constant。 文章中最早也是这么做的，为训练集中每一个 context 学习一个 normalizing constant, 然后把这些 normalizing constant 存在一个 hashtab 中. 但是在 context size 比较大的时候，会比较麻烦。令人震惊的是，文中说他们发现即使把所有的 normalizing constants 固定为 1，也并不会对模型的 performance 有影响。

[博客: On word embeddings - Part 2: Approximating the Softmax] 和 [文章: Simple, Fast Noise-Contrastive Estimation for Large RNN Vocabularies] 也对这块有类似的说明，并且说，让模型自己去学习 normalizing constant 的话，学到的 normalizing constant 也会接近于 1，且方差很小。

<ul>
<li><strong>关于 Candidate Sampling</strong></li>
</ul>

<h1>参考资料</h1>

<ul>
<li><p><a href="http://www.aclweb.org/anthology/N16-1145">文章: Simple, Fast Noise-Contrastive Estimation for Large RNN Vocabularies</a></p></li>
<li><p><a href="https://arxiv.org/abs/1206.6426">文章: A fast and simple algorithm for training neural probabilistic language model</a></p></li>
<li><p><a href="http://ruder.io/word-embeddings-softmax/index.html">博客: On word embeddings - Part 2: Approximating the Softmax</a></p></li>
<li><p><a href="http://www.tensorflow.org/extras/candidate_sampling.pdf">candidate sampling</a></p></li>
<li><p><a href="https://spaces.ac.cn/archives/5617">博客: “噪声对比估计”杂谈：曲径通幽之妙</a></p></li>
</ul>