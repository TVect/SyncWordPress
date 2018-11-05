---
ID: 833
post_title: Gaussian Processes 初步 笔记
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/833
published: true
post_date: 2018-11-02 17:32:15
---
[toc]

整理的一些 Gaussian Processes 的初步知识

<!--more-->

<h1>基本概念</h1>

<strong>Gaussian process</strong> 定义为一个 函数 $y(x)$ 的分布, 该分布满足对于定义域上的任意有限点集 $x_1,..., x_N$, 函数在这些点的联合概率分布是高斯分布。

高斯过程可以看做是多维高斯分布在无限维上的扩展。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/gp-768x519.png" alt="" />

<hr />

<h1>Gaussian Processes for Regression</h1>

<h2>Noiseless GP regression</h2>

观测数据为: $ D=&#123; (x_i, y_i) , i=1,2,...,N &#125; $, 其中 $ f_i = f(x_i) $.

给定测试数据集 $ \bold X_* \in R^{N_* \times  D} $, 现在有预测相应的输出 $\bold f_*$.

<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/noiseless-768x183.png" alt="" />

<h2>Noisy GP regression</h2>

考虑观测值带有噪声的情况, $ y = f(x) + \epsilon $, 其中 $f$ 是一个隐含的无噪声的函数, 噪声 $ \epsilon \sim N(0, \sigma^2_y) $, 且假定噪声之间是独立的.

<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/noise-gp-768x582.png" alt="" />

<ul>
<li><strong>案例</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/regression-demo.png" alt="" /></li>
</ul>

<h1>Gaussian Processes for Classification</h1>

定义模型为 $ p(y_i|x_i) = \sigma (y_if(x_i)) $, 其中, $ \sigma $ 为 sigmoid 函数. 注意, $f \sim GP(0, k)$, 但是 $y$ 不再服从高斯过程.

<div class="katex math multi-line no-emojify">p(y_* | \bold y_N) = \int p(y_*|f_*)p(f_*|\bold y_N) df_*
</div>

其中,$ \bold y_N = {y_1, y_2, ..., y_N}$. 上式右边积分中, 第一项已经知道是一个 sigmoid 函数.

上面整个的积分没有解析解, 所以可以考虑作近似, 比如 sampling methods 或者是 analytical approximation. 因为有对 sigmoid 函数和高斯分布做卷积的近似公式, 所以这里试图把第二项也表示为一个高斯分布, 或者是用一个高斯分布近似.

<div class="katex math multi-line no-emojify"> \begin{aligned}
p(f_*|\bold y_N) &amp;= \int p(f_*, \bold f_N | \bold y_N) d \bold f_N \\
&amp;= \frac{1}{p(\bold y_N)} \int p(f_*, \bold f_N) p(\bold y_N | f_*, \bold f_N) d \bold f_N \\
&amp;= \frac{1}{p(\bold y_N)} \int p(f_* | \bold f_N) p(\bold f_N)p(\bold y_N | f_*, \bold f_N) d \bold f_N \\
&amp;= \int p(f_* | \bold f_N) p(\bold f_N | \bold y_N) d \bold f_N
\end{aligned}
</div>

因为 $f \sim GP(0, k)$, 上面积分号中第一项可以利用 GP for regression 中类似的方式得出.

<div class="katex math multi-line no-emojify">p(f_* | \bold f_N) = N(f_* | k^T_*K^{-1} \bold y_N, k_{**}-k^T_*K^{-1}k_*)
</div>

上面积分号中第二项可以利用 Laplace Approximation 得到一个近似的高斯分布.

<div class="katex math multi-line no-emojify">p(\bold f_N | \bold y_N) = \frac {p(\bold y_N| \bold f_N) p(\bold f_N)}{Z}
</div>

<div class="katex math multi-line no-emojify">p(\bold f_N | \bold y_N) \sim q(\bold f_N | \bold y_N) = N(\bold f_{*N}, (K^{-1} + W)^{-1}) \\
\bold f_{*N} = K \triangledown log \, p(\bold y_N | \bold f_{*N}) \\
W = - \triangledown \triangledown log \, p(\bold y_N | \bold f_{N})
</div>

上面的 $ \bold f_{*N} $ 需要迭代求解.

最终有:

<div class="katex math multi-line no-emojify">E[f_*|\bold y_N] = K^T (\bold y_N - \bold \sigma_N) \\
var[f_*|\bold y_N] = k_{**} - k_*^T(W_N^{-1} + K)^{-1}k_*
</div>

<ul>
<li><strong>案例</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/classification-demo-768x348.png" alt="" /></li>
</ul>

<h1>相关知识</h1>

<h2>Gaussian basics</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/gaussian_basics.png" alt="" />

<h2>Laplace Approximation</h2>

给定一个分布 $p(z)$, Laplace Approximation 的目标是要找一个近似的高斯分布 $q(z)$, 其均值在 $p(z)$ 的 mode 处, 其方差通过对 $ ln p(z) $ 在 mode 处做 Taylor expansion 得到.

<ul>
<li><p><strong>原理</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/laplace-approximation.png" alt="" /></p></li>
<li><p><strong>案例</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/11/laplace-demo.png" alt="" /></p></li>
</ul>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1505.02965">文章 Gaussian Processes: A Quick Introduction</a> by M. Ebden, 2008</p></li>
<li><p>PRML Chapter 6.4 Gaussian Processes</p></li>
<li><p>PRML Chapter 4.4 Laplace Approximation</p></li>
<li><p>MLAPP Chapter 15</p></li>
</ul>