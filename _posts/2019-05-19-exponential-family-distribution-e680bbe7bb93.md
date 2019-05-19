---
ID: 364
post_title: exponential family distribution 小结
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/364
published: true
post_date: 2019-05-19 16:05:35
---
<h1>定义</h1>

The exponential family of distributions over $x$, given parameters $\eta$, is defined to be the set of distributions of the form:
$$p(x|\eta) = h(x) \exp(\eta^T t(x) - A(\eta))$$
where x may be scalar or vector, and may be discrete or continuous.

And these terms are called as follows:

<ol>
<li>$\eta$ is the natural parameter vector</li>
<li>$t(x)$ is the sufficient statistic vector</li>
<li>$h(x)$ is the base measure (a function of $x$)</li>
<li>$A(η)$ is the log-normalizer (a function of $η$)</li>
</ol>

An alternative, equivalent form often given is：$p(x|\eta) = h(x) g(\eta) \exp(\eta^T t(x))$

<h2>案例</h2>

<h3>Dirichlet Example</h3>

$ p(x|\alpha_1, ... , \alpha_k) = \frac{\Gamma(\sum_i^K \alpha_i)}{\prod_i^K \Gamma(\alpha_i)} \prod_{i=1}^{K} x_i^{\alpha_i - 1} = (\prod_{i=1}^{K} x_i^{-1}) e^{\sum_i^K \alpha_i \ln x_i - (\sum_i^K \ln \Gamma(\alpha_i) - \ln \Gamma(\sum_i^K \alpha_i))}$

Matching terms with the generic exponential family distribution form, we have:

<ul>
<li>$h(x) = \prod_{i=1}^{K} x_i^{-1}$</li>
<li>$t(x) = [\ln x_1, ... , \ln x_K]^T$</li>
<li>$\eta = [\alpha_1, ... , \alpha_K]^T$</li>
<li>$A(\eta) = \sum_i^K \ln \Gamma(\alpha_i) - \ln \Gamma(\sum_i^K \alpha_i)$</li>
</ul>

<h3>univariate Gaussian Example</h3>

$p(x | \mu, \sigma^2) = \frac{1}{\sqrt{2\pi\sigma^2}} \exp (-\frac{(x-\mu)^2}{2\sigma^2}) = \frac{1}{\sqrt{2\pi}} \exp (\frac{u}{\sigma^2} x - \frac{1}{2\sigma^2} x^2 - \frac{\mu^2}{2\sigma^2} - \ln \sigma) $

Matching terms with the generic exponential family distribution form, we have:

<ul>
<li>$h(x) = \frac{1}{\sqrt{2\pi}} $</li>
<li>$t(x) = [x, x^2]^T$</li>
<li>$\eta = [\frac{u}{\sigma^2}, \frac{1}{2\sigma^2}]^T$</li>
<li>$A(\eta) = \frac{\mu^2}{2\sigma^2} + \ln \sigma = -\frac{\eta_1^2}{4\eta_2} - \frac{1}{2}\ln(-2\eta_2)$</li>
</ul>

<h1>Moments of an exponential family</h1>

$$
\begin{aligned}
0 &amp;= \triangledown_{\eta} \int h(x) \exp(\eta^T t(x) - A(\eta)) dx &#92;
&amp;= \int h(x) \exp(\eta^T t(x) - A(\eta)) (t(x) - \triangledown_{\eta} A(\eta))dx &#92;
&amp;=  \mathbb{E} [t(x)] - \triangledown_{\eta} A(\eta) &#92;&#92;
\mathbb{E} [t(x)] &amp;= \triangledown_{\eta} A(\eta)
\end{aligned}
$$

特别地, 具体考虑 Dirichlet Distribution 的情况, 此时有

$$\mathbb{E} [\ln x_i] = \frac{\partial}{\partial \alpha_i} (\sum_i^K \ln \Gamma(\alpha_i) - \ln \Gamma(\sum_i^K \alpha_i)) = \frac{\partial \ln \Gamma(\alpha_i)}{\partial \alpha_i} - \frac{\partial \ln \Gamma(\sum_i \alpha_i)}{\partial \alpha_i} = \psi(\alpha_i) - \phi(\sum \alpha_i)$$

其中 $ \psi(.) $ 称为digamma function

<h1>Maximum likelihood estimation of an exponential family</h1>

假定现在有数据 $x_1, ..., x_n$, 要求参数 $\eta$ 的极大似然解.

$ L = \sum \ln p(x_i | \eta) = \sum_i (\eta^T t(x_i) - A(\eta) - \ln h(x_i)) = \eta^T \sum_i t(x_i) - \sum_i \ln h(x_i) - nA(\eta)$

$\triangledown_{\eta} L = \sum_i t(x_i) - n \triangledown_{\eta} A(\eta)$

$\triangledown_{\eta} A(\eta) = \frac{\sum_i t(x_i)}{n}$

<h1>Conjugate priors</h1>

Suppose the data come from an exponential family. Every exponential family has a conjugate prior:

$p(x_i | \eta) = h_l(x) \exp(\eta^T t(x) - A_l(\eta))$

$p(\eta | \lambda) = h_c(\eta) \exp (\lambda_1^T \eta + \lambda_2 (-A_l(\eta)) - A_c(\lambda))$

其中, 先验分布的 natural parameter $\lambda = [\lambda_1, \lambda_2]$, 维度为 $\dim(\eta) + 1$, sufficient statistics 为 $[\eta, - A_l(\eta)]$

此时, 可以求得后验分布为:

$
\begin{aligned}
p(\eta | x, \lambda) &amp; \propto p(\eta | \lambda) \prod_ip(x_i | \eta) &#92;
&amp;= h_c(\eta) \exp (\lambda_1^T \eta + \lambda_2 (-A_l(\eta)) - A_c(\lambda)) \prod_i [h_l(x) \exp(\eta^T t(x) - A_l(\eta))] &#92;
&amp; \propto h_c(\eta) \exp [(\lambda_1 + \sum_i^n t(x_i))^T\eta +(\lambda_2 + n)(-A_l(\eta))]
\end{aligned}
$

This is the same exponential family as the prior, with parameters:

$\hat{\lambda_1} = \lambda_1 + \sum_i^n t(x_i)$

$\hat{\lambda_2} = \lambda_2 + n$

<strong>一些其他表达形式的共轭</strong>

<ol>
<li>形式1 (参见 PRML Chapter 2.4)
$ p(x | \eta) = h(x) g(\eta) exp(\eta^T u(x)) $
$ p(\eta | \chi, \nu) = f(\chi, \nu)g(\eta)^{\nu} exp(\nu \eta^T \chi) $</p></li>
<li><p>形式2 (参见 Jordan PGM Chapter 9, Wikipedia)
$ p(x|\eta) = h(x) \exp(\eta^T t(x) - A_l(\eta)) $
$ p(\eta | \chi, \nu) = f(\chi, \nu) exp(\eta^T \chi - \nu A_l(\eta)) $</p></li>
</ol>

<h2>Coordinate ascent mean-field variational inference</h2>

<p><strong>Models with local and global hidden variables</strong>

The N observations are $x = x1, x_2, ..., x_N$.

The vector of global hidden variables is $\beta$, the N local hidden variables are $z = z_1, ..., z_N$.

The joint distribution factorizes into a global term and a product of local terms: $p(x, z, \beta | \alpha) = p(\beta | \alpha) \prod_{n=1}^N p(x_n, z_n | \beta)$

We assume that these <em>complete conditionals</em> distributions are in the exponential family：

$$
\begin{aligned}
p(\beta | x, z, \alpha) &amp;= h(\beta) \exp (\eta_g(x, z, \alpha)^T t(\beta) - A_g(\eta_g(x, z, \alpha))) &#92;
p(z_{n, j} | x_n, z_{n, -j}, \beta) &amp;= h(z_{n,j}) \exp (\eta_l(x_n, z_{n, -j}, \beta) t(z_{n,j}) - A_l(\eta_l(x_n, z_{n, -j}, \beta)))
\end{aligned}
$$

<strong>The mean-field variational family</strong>
我们目标是要近似后验分布$p(\beta, z | x)$, 现选择如下形式的 mean-field 变分分布来做近似.

$q(z, \beta) = q(\beta | \lambda) \prod_{n=1}^N \prod_{j=1}^J q(z_{n,j} | \phi_{n,j})$

上面只是给出了变分分布族的分解, 没有给出具体分布的形式. 这里可以进一步假定$q(\beta|\lambda), q(z_{n,j}|φ<em>{n,j})$ 分别与 $p(\beta|x, z), p(z</em>{n,j}|x_n, z_{n,−j}, \beta)$ 属于相同的指数家族分布, 即:

$$
\begin{aligned}
q(\beta | \lambda) &amp;= h(\beta) \exp{\lambda^T t(\beta) − A_g(\lambda)} &#92;
q(z_{n, j} | \phi_{n, j}) &amp;= h(z_{n,j}) \exp{\phi_{n,j}^T t(z_{n,j}) − A_l(\phi_{n,j})}
\end{aligned}
$$

现在需要优化 ELBO: $L(q) = \mathbb{E}_q [\log p(X, Z, \beta | \alpha)] − \mathbb{E}_q [\log q(Z, \beta)]$

<ol>
<li><strong>优化 $L(\lambda)$</strong></li>
</ol>

$$
\begin{aligned}
L(\lambda) &amp;= \mathbb{E}<em>q [\log p(\beta | X, Z, \alpha)] + \mathbb{E}_q [\log p(X, Z)]− \mathbb{E}_q [\log q(Z, \beta)] &#92;
&amp;= \mathbb{E}_q [\log p(\beta | X, Z, \alpha)] − \mathbb{E}_q [\log q(\beta)] + const &#92;
&amp;= \mathbb{E}_q [\log h(\beta) + (\eta_g(x, z, \alpha)^T t(\beta) - A_g(\eta_g(x, z, \alpha))] − \mathbb{E}_q [\log h(\beta) + (\lambda^T t(\beta) − A_g(\lambda))] + const &#92;
&amp;= \mathbb{E}</em>{q(z)} \eta_g(x, z, \alpha)^T \mathbb{E}<em>{q(\beta)}t(\beta) - \lambda^T \mathbb{E}</em>{q(\beta)} [t(\beta)] + A_g(\lambda) &#92;
&amp;= \mathbb{E}<em>{q(z)} \eta_g(x, z, \alpha)^T \triangledown</em>{\lambda} A_g(\lambda) - \lambda^T \triangledown_{\lambda} A_g(\lambda) + A_g(\lambda) &#92;&#92;
\triangledown_{\lambda} L(\lambda) &amp;= \mathbb{E}<em>{q(z)} \eta_g(x, z, \alpha)^T \triangledown</em>{\lambda}^2 A_g(\lambda) - \triangledown_{\lambda} A_g(\lambda) - \lambda^T \triangledown_{\lambda}^2 A_g(\lambda) + \triangledown_{\lambda} A_g(\lambda) &#92;
&amp;= \mathbb{E}<em>{q(z)} \eta_g(x, z, \alpha)^T \triangledown</em>{\lambda}^2 A_g(\lambda) - \lambda^T \triangledown_{\lambda}^2 A_g(\lambda) &#92;
\end{aligned}
$$

<ol start="2">
<li><strong>优化 $L(\phi_{n,j})$</strong></li>
</ol>

参考<a href="https://arxiv.org/abs/1206.7051v3">Stochastic Variational Inference</a> Page 10-11

<h1>参考资料</h1>

<ul>
<li><p>PRML Chapter 2.4 &amp; Chapter 10.4</p></li>
<li><p><a href="https://arxiv.org/abs/1206.7051v3">Stochastic Variational Inference</a></p></li>
<li><p><a href="http://www.cs.columbia.edu/~blei/fogm/2014F/lectures/exponential-family.pdf">The Exponential Family David M. Blei</a></p></li>
</ul>