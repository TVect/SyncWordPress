---
ID: 224
post_title: State Space Models 之 HMM
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/224
published: true
post_date: 2019-03-15 15:25:29
---
<h1>Introduction</h1>

<blockquote>
  HMM can be interpreted as an extension of a mixture model in which the choice of mixture component for each observation is not selected independently but depends on the choice of component for the previous observation.
  
  The HMM is widely used in speech recognition, natural language modelling, on-line handwriting recognition, and for the analysis of biological sequences such as proteins and DNA.
</blockquote>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/758ee40569603466ba0c6a8efe39ba4e.png" alt="" />

<h1>HMM 中几个问题</h1>

<h2>Maximum likelihood for the HMM</h2>

给定观测数据 $X = {x_1, ..., x_N}$, 我们要确定 HMM 的参数以极大化似然函数. 似然函数可以表示为: $p(X | \theta) = \sum_{z} p(X, Z | \theta)$.

和 mixture model 一样, 直接做 MLE 会有困难, 这里用 EM Algorithm 进行求解.

<strong>E-step</strong>

take these old parameter values and find the posterior distribution of the latent variables $p(Z|X, \theta^{old})$.

<strong>M-step</strong>

use this posterior distribution, which is calculated in E-step, to evaluate the expectation of the logarithm of the complete-data likelihood function, as a function of the parameters $\theta$, to give the function $Q(\theta, \theta^{old}) = \sum_{z} p(Z | X, \theta^{old}) ln \, p(X, Z | \theta)$

引入符号 $\gamma(z_n) = p(z_n | X, \theta^{old})$, 和 $\xi(z_{n-1}, z_{n}) = p(z_{n-1}, z_n | X, \theta^{old})$

极大化 $Q(\theta, \theta^{old})$, 其中 $ \theta = &#123;\pi, A, \phi &#125;$, 可以得到:

$$ \pi_k = \frac {\gamma (z_{1,k})} {\sum_{j=1}^{K} \gamma(z_{1j})} $$

$$ A_{jk} = \frac {\sum_{n=2}^{N} \xi(z_{n-1, j}, z_{nk})} {\sum_{l=1}^{K} \sum_{n=2}^{N} \xi(z_{n-1, j}, z_{nl})}$$

对于 $\phi$ 的优化, 视具体的发射概率形式会有所不同.

<h2>forward-backward algorithm &amp; sum-product algorithm in PGM</h2>

现在要寻找一种高效的算法来求出 $\gamma(z_{nk}), \xi(z_{n-1, j}, z_{nk})$.

这可以从 <strong>PGM 中 sum-product algorithm</strong> 容易的得到, 或者使用 HMM 的分解表示推导.

定义: $ \alpha(z_n) = p(x_1, ..., x_n, z_n), \beta(z_n) = p(x_{n+1}, ..., x_N | z_n)$

容易得到递推公式如下：

$$ \alpha(z_n) = p(x_n | z_n) \sum_{z_{n-1}} \alpha(z_{n-1}) p(z_n | z_{n-1}) \quad where \ \alpha(z_1) = p(z_1) p(x_1 | z_1) $$

$$ \beta(z_n) = \sum_{z_{n+1}} \beta(z_{n+1}) p(x_{n+1} | z_{n+1}) p(z_{n+1} | z_{n}) \quad where \ \beta(z_N) = 1$$

从而可以得到 $\gamma(z_n), \xi(z_{n-1}, z_{n})$ 表达式如下:

$$ \gamma(z_n) = \frac {\alpha(z_n) \beta(z_n)} {p(X)} $$

$$ \xi(z_{n-1}, z_{n}) =  \frac {\alpha(z_{n-1}) p(x_n | z_n) p(z_n | z_{n-1}) \beta(z_n)} {p(X)} $$

$$ p(X) = \sum_{z_n} \alpha(z_n) \beta(z_n) $$

<h3>numerical accuracy problem: scaling factors</h3>

前面的 forward-backward algorithm 中 $\alpha(z_n)$ 会随着 $n$ 的增大迅速减少, 会有数值问题. 因为迭代中涉及到乘积求和，无法通过简单的 log 变换处理这里的数值问题.

这里引入了 re-scaled versions of $\alpha(z_n)$.

$$ \hat{\alpha}(z_n) = p(z_n | x_1, ..., x_n) = \frac {\alpha(z_n)} {p(x_1, ..., x_n)} = \frac {\alpha(z_n)} {\prod_{m=1}^n c_m}. \quad where \ c_m = p(x_n | x_1, ..., x_{n-1})$$

$$ c_n \hat{\alpha(z_n)} = p(x_n | z_n) \sum_{z_{n-1}} \hat{\alpha}(z_{n-1}) p(z_n | z_{n-1})$$

注意, $c_n$ 可以看做上式右边的 normalizer， 可以简单的通过对上式右边求和得到.

同样，对于 $\beta(z_n)$, 引入 $\hat{\beta}(z_n)$:

$$ \hat{\beta}(z_n) = \frac {p(x_{n+1}, ..., x_N | z_n)} {p(x_{n+1}, ..., x_N | x_1, ..., x_n)} = \frac {\beta(z_n)} {\prod_{m=n+1}^N c_m} $$

$$ c_{n+1} \hat{\beta}(z_n) = \sum_{z_{n+1}} \hat{\beta}(z_{n+1}) p(x_{n+1} | z_{n+1}) p(z_{n+1} | z_{n}) $$

最后, $\gamma(z_n), \xi(z_{n-1}, z_n)$ 计算公式如下:

$$ \gamma(z_n) = \hat{\alpha}(z_n) \hat{\beta}(z_n) $$

$$ \xi(z_{n-1}, z_n) = c_n \hat{\alpha}(z_{n-1}) p(x_n|z_n) p(z_n | z_{n-1}) \hat{\beta}(z_n) $$

<h2>viterbi algorithm &amp; max-product / max-sum algorithm in PGM</h2>

这里的解码算法对应着 PGM 中的 max-sum algorithm.

记 $w(z_n) = max p(x_1, ..., x_n, z_1, ..., z_n)$,

递推公式为: $ w(z_{n+1}) = ln \, p(x_{n+1} | z_{n+1}) + max_{z_n} (ln p(x_n | z_n) + w(z_n))$

<h1>番外篇</h1>

<h2>Belief Propagation in PGM</h2>

<h3>Sum-Product Algorithm in PGM</h3>

<h3>Max-Sum Algorithm in PGM</h3>

<h1>参考资料</h1>

<ul>
<li><p>[PRML Chapter 13.2 Hidden Markov Models]</p></li>
<li><p>[PRML Chapter 8 Graphical Models]</p></li>
</ul>