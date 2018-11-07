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

估计概率语言模型参数的时候, 作为分母的 partition functions 要在整个 vocabulary 上求和, 计算量会很大. 常用的处理方法有 NCE, Negative Sampling 等.

这里对 NCE 的原理, 实现等做了一些整理.

<!--more-->

<h1>NCE 原理</h1>

NCE 的基本想法是对观测数据和人工噪声样本做对比区分，从而发现真实样本中的规律。

<blockquote>
  The idea is to perform nonlinear logistic regression to discriminate between the observed data and some artificially generated noise, using the model log-density function in the regression nonlinearity. We show that this leads to a consistent (convergent) estimator of the parameters, and analyze the asymptotic variance. In particular, the method is shown to directly work for unnormalized models, i.e. models where the density function does not integrate to one. The normalization constant can be estimated just like any other parameter.
</blockquote>

<h2>Definitions</h2>

记数据分布为 $p_d$, 噪声分布为 $p_n$,

令 $ U = &#123;u_1; u_2; ...; u_{Td+Tn} &#125;$, 其中 $ Td $ 为数据样本的个数, $ Tn $ 为从噪声分布中采样的样本个数.

用 $ C_t $ 表示和数据点 $u_t$ 对应的类别标签. $ C_t = 1 $ 代表数据来源于数据分布, $ C_t = 0 $ 代表数据来源于噪声分布.

现在建模 $ p(.|C=1) = p_m(.|\theta)$, 且假设存在着一个 $ \theta^* $, 使得 $ p_d(.) = p_m(.;\theta^*)$.

所以先验分布为:

<div class="katex math multi-line no-emojify">p(C=1) = \frac {T_d} {T_d + T_n} \\

p(C=1) = \frac {T_d} {T_d + T_n}
</div>

似然为:

<div class="katex math multi-line no-emojify">p(u|C=1) = p_m(u; \theta) \\

p(u|C=0) = p_n(u)
</div>

进而, 后验分布为:

<div class="katex math multi-line no-emojify">p(C=1 | u; \theta) = \frac {p_m(u; \theta)} {p_m(u; \theta) + vp_n(u)} \\

p(C=0 | u; \theta) = \frac {vp_n(u)} {p_m(u; \theta) + vp_n(u)} \\

v = \frac {T_n} {T_d} = \frac {P(C=0)} {P(C=1)}
</div>

<h2>The NCE Estimator</h2>

记未归一化的模型为 $p_m^0(.; \alpha)$, 归一化因子为 $ Z(\alpha) = \int p_m^0(u; \alpha) du $, 归一化因子需要做积分, 很多时候很难计算.

Noise Contrastive Estimation (NCE) 是未归一化模型的一种估计方法. 它的基本想法是把 $ Z $, 或者等价的 $ c = ln \frac{1}{Z}$ 视为模型参数的一部分 (不再是当做一个 partition function).

<div class="katex math multi-line no-emojify">ln \, p_m(.; \theta) = ln \, p_m^0(.; \alpha) + c \\

\theta = (\alpha, c)
</div>

下面需要证明的是, 对这个 unnormalized model 做优化得到的最优解 $\theta^* = &#40;\alpha^*, c^*&#41;$, 自然的满足了归一化约束, 最终的 $c$ 自然的成为了归一化因子的近似.

<div class="katex math multi-line no-emojify">\begin{aligned}
J_T(\theta) &amp;= \frac{1}{T_d} [\sum \limits_{t=1}^{T_d} ln \, p(C=1 | u; \theta)+ \sum \limits_{t=1}^{T_n} ln \, p(C=0 | u; \theta)] \\
&amp;= \frac{1}{T_d} \sum \limits_{t=1}^{T_d} ln \, p(C=1 | u; \theta) + \frac {v} {T_n}\sum \limits_{t=1}^{T_n} ln \, p(C=0 | u; \theta) \\\\
\theta_T^{*} &amp;= arg \max J_T(\theta)
\end{aligned}
</div>

下面有说明的是, <strong>在 $T$ 充分大时, 最优解 $\theta_T^{*}$ 自然满足归一化约束.</strong>

实际上当 <code>$ T $</code> 充分大的时候, 有:

<div class="katex math multi-line no-emojify">J(\theta) = E_{p_d}ln \, p(C=1 | u; \theta) + vE_{p_n} ln \, p(C=0 | u; \theta)
</div>

令 $f=v \frac {p_m}{p_d}$, 将上式视为 $ f $ 的函数有:

<div class="katex math multi-line no-emojify">\begin{aligned}
J(f) &amp;= \int p_d(u) ln \, p(C=1 | u; \theta) du + v\int p_n(u) ln \, p(C=0 | u; \theta) du \\
&amp;= \int p_d(u) ln \, \frac {1}{1+f} du + v \int p_n(u) ln \, \frac {f}{1+f} du \\
&amp;= -\int p_d(u) ln \, (1+f) du + v \int p_n(u) (ln \, f - ln \, (1+f)) du
\end{aligned}
</div>

根据变分法中相关知识, 在 $ J(f) $ 取得极大值时有:

<div class="katex math multi-line no-emojify">\frac {\partial J(f) } {\partial f} = - \frac {p_d(u)} {1 +f} + v \, p_n (\frac {1} {f} - \frac {1} {1+f}) = 0\\

f = v \frac {p_n} {p_d}
</div>

从而在 $ T $ 足够大时, 最优解 $ \theta^* $ 满足 $ f(\theta^*) =  v \frac {p_n} {p_d} $,

即 $ p_m(\theta^*) = p_d $.

(注意, 前面已经假设存在着一个 $ \theta^* $, 使得 $ p_d(.) = p_m(.;\theta^*) $)

<strong>Tips</strong>: 另外的证明方式可以参见 <a href="https://spaces.ac.cn/archives/5617">博客: “噪声对比估计”杂谈：曲径通幽之妙</a>

<h1>NCE 和 Negative Sampling</h1>

在语言模型的例子中，NCE 具体化为:

<div class="katex math multi-line no-emojify">p(1|c,w) = \frac {u_{\theta}(w|c)}{u_{\theta}(w|c) + k p_n(w)} \\
p(0|c,w) = \frac {k p_n(w)}{u_{\theta}(w|c) + k p_n(w)}
</div>

令 <code>$ k = |V| $</code> 且 <code>$ q(w)$</code> 为均匀分布, 有:

<div class="katex math multi-line no-emojify">p(1|c,w) = \frac {u_{\theta}(w|c)}{u_{\theta}(w|c) + |V| \frac {1}{|V|}} = \frac {u_{\theta}(w|c)} {u_{\theta}(w|c) + 1}\\
p(0|c,w) = \frac {|V| \frac {1}{|V|}}{u_{\theta}(w|c) + |V| \frac {1}{|V|}} = \frac {1}{u_{\theta}(w|c) + 1}
</div>

再结合 NCE 的优化函数，对比之下，即可知道 negative sampling 可视为 NCE 的特例.

<h1>案例</h1>

<h2>tensorflow 中 nce_loss 实现代码</h2>

设计思路可以参考 <a href="http://www.aclweb.org/anthology/N16-1145">文章: Simple, Fast Noise-Contrastive Estimation for Large RNN Vocabularies</a>, <a href="https://arxiv.org/abs/1206.6426">文章: A fast and simple algorithm for training neural probabilistic language model</a>,

<ul>
<li><strong>关于 normalizer</strong></li>
</ul>

<a href="https://arxiv.org/abs/1206.6426">文章: A fast and simple algorithm for training neural probabilistic language model</a>中有详细提到 normalizing constants 的处理。

首先从公式中可以看出，在训练词向量的时候应该为每一个 context 学习一个 normalizing constant。 文章中最早也是这么做的，为训练集中每一个 context 学习一个 normalizing constant, 然后把这些 normalizing constant 存在一个 hashtab 中. 但是在 context size 比较大的时候，会比较麻烦。令人震惊的是，文中说他们发现即使把所有的 normalizing constants 固定为 1，也并不会对模型的 performance 有影响。

<a href="http://ruder.io/word-embeddings-softmax/index.html">博客: On word embeddings - Part 2: Approximating the Softmax</a> 和 <a href="http://www.aclweb.org/anthology/N16-1145">文章: Simple, Fast Noise-Contrastive Estimation for Large RNN Vocabularies</a> 也对这块有类似的说明，并且说，让模型自己去学习 normalizing constant 的话，学到的 normalizing constant 也会接近于 1，且方差很小。

<ul>
<li><strong>关于代码实现</strong></li>
</ul>

参见后续的博客.

<h1>参考资料</h1>

<ul>
<li><p><a href="http://www.aclweb.org/anthology/N16-1145">文章: Simple, Fast Noise-Contrastive Estimation for Large RNN Vocabularies</a></p></li>
<li><p><a href="https://arxiv.org/abs/1206.6426">文章: A fast and simple algorithm for training neural probabilistic language model</a></p></li>
<li><p><a href="http://ruder.io/word-embeddings-softmax/index.html">博客: On word embeddings - Part 2: Approximating the Softmax</a></p></li>
<li><p><a href="https://spaces.ac.cn/archives/5617">博客: “噪声对比估计”杂谈：曲径通幽之妙</a></p></li>
<li><p><a href="http://www.tensorflow.org/extras/candidate_sampling.pdf">candidate_sampling</a></p></li>
</ul>