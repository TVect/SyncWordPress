---
ID: 559
post_title: Flow-based generative models 小结
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/559
published: true
post_date: 2019-10-09 10:01:44
---
这边整理一下 Flow-based generative models.

主要想法是要找一个性质良好变换 $f$, 将变量映射到一个隐空间, 使得变换之后的分布是一个简单的易于处理的分布. 通过这种操作可以得到精确的 log likelihood 的表达, 进而可以通过 MLE 做训练.

下面涉及到三篇文章, <a href="https://arxiv.org/abs/1410.8516">NICE</a>, <a href="https://arxiv.org/abs/1605.08803">REAL NVP</a>,<a href="https://arxiv.org/abs/1807.03039">Glow</a> 都分别提出各种不同的变换形式 $f$, 以处理这个问题.

<h1>NICE: non-linear independent components estimation</h1>

文章基于想法 <code>a good representation is one in which the data has a distribution that is easy to model</code>. 学习了一个非线性变换，将数据变换到隐空间上，使得变换之后的数据服从某种特定的简单的分布. 一方面这个变换可以比较容易的求 Jacobian 和求逆，另一方面可以通过这种变换的组合构造更加复杂的非线性变换. 训练准则是极大化 exact log-likelihood.

对于一个分布 $p_X(x)$, 希望找到一个变换 $h = f(x)$, 将数据映射到一个新的空间, 在这个空间上的分布是可分解的: $p_H(h) = \prod_{d} p_{H_d}(h_d)$.

使用概率分布 basic transformation 的公式, 可知: $p_X(x) = p_H(f(x)) |det \frac{\partial f(x)}{\partial x}|$.

这篇 paper 中设计了变换$f$, 使其拥有性质: <code>"easy determinant of the Jacobian" and "easy inverse"</code>

<h2>coupling layer</h2>

<h3>General coupling layer</h3>

$$
\begin{aligned}
x=(x_{I_1}, x_{I_2}) &amp;\mapsto y=(y_{I_1}, y_{I_2}) &#92;
y_{I_1} &amp;= x_{I_1} &#92;
y_{I_2} &amp;= g(x_{I_2}; m(x_{I_1})) &#92;
\end{aligned}
$$

其中, $g : R^{D−d} × m(R^d) \rightarrow R^{D−d}$ 称为 <code>coupling law</code>, 其对于第一部分是一个可逆变换.

此时, 定义的变换的 Jacobian 为:

$$
 \frac{\partial y}{\partial x} = \begin{pmatrix}
I_d &amp; 0&#92; 
\frac{\partial y_{I_2}}{\partial x_{I_1}} &amp; \frac{\partial y_{I_2}}{\partial x_{I_2}}
\end{pmatrix}
$$

变换的逆变换为:
$$
\begin{aligned}
x_{I_1} &amp;= y_{I_1} &#92;
x_{I_2} &amp;= g^{-1}(y_{I_2}; m(y_{I_1})) &#92;
\end{aligned}
$$

<h3>Additive coupling layer</h3>

特别地，取上述 coupling law <code>g</code> 为: $g(x_{I_2}; m(x_{I_1})) = x_{I_2} + m(x_{I_1}) $, 此时有:

$$
\begin{aligned}
y_{I_2} &amp;= x_{I_2} + m(x_{I_1}) &#92;
x_{I_2} &amp;= y_{I_2} - m(y_{I_1})) &#92;
\end{aligned}
$$

使用这种 additive coupling law 之后，计算变换的逆变换就很简单了, 而且此时的 Jacobian 即为一个单位阵. 另外 coupling function m 的选择没有什么限制, 可以是一个 domain &amp; codomain 合适的神经网络.

<h3>Combining coupling layers</h3>

可以将多个 coupling 层堆叠在一起得到更加复杂的变换.

<blockquote>
  Since a coupling layer leaves part of its input unchanged, we need to exchange the role of the two subsets in the partition in alternating layers, so that the composition of
  two coupling layers modifies every dimension.
</blockquote>

特别地，对于Additive coupling layer，检查 combining 之后的 Jacobian，可以发现至少要 3 层才能保证每一维能影响到其他维度.

<h2>allowing rescaling</h2>

<blockquote>
  include a diagonal scaling matrix $S$ as the top layer, which multiplies the $i$-th ouput value by $S_{ii}$: $(x_i)<em>{i \le D} \rightarrow (S</em>{ii}x_i)_{i \le D}$. This allows the learner to give more weight (i.e. model more variation) on some dimensions and less in others.
</blockquote>

此时,  NICE criterion 形如: $ \log(p_X(x)) = \sum_{i=i}^D [\log(p_{H_i}(f_i(x))) + \log(|S_{ii}|)]$

<h2>prior distribution</h2>

we choose the prior distribution to be factorial, i.e.: $p_H(h) = \prod_{i=1}^D p_{H_d}(h_d)$.

其中的 $p_{H_d}(h_d)$ 可以选为一些标准分布, 比如 gaussian distribution, logistic distribution.

<h1>REAL NVP: real-valued non-volume preserving transformations</h1>

<h2>Coupling layers</h2>

<strong>affine coupling layer</strong>

$$
\begin{aligned}
y_{1:d} &amp;= x_{1:d} &#92;
y_{d+1:D} &amp;= x_{d+1:D} \bigodot  \exp(s(x_{1:d})) + t(x_{1:d})
\end{aligned}
$$

where $s$ and $t$ stand for scale and translation, and are functions from $R^d \rightarrow R^{D−d}$, and $\bigodot$ is the Hadamard product or element-wise product.

The Jacobian of this transformation is:

$$
\frac{\partial y}{\partial x} = \begin{pmatrix}
 I_d &amp; 0 &#92; 
 \frac{\partial{y_{d+1: D}}}{\partial {x_{1:d}}} &amp; diag(\exp[s(x_{1:d})]) 
\end{pmatrix}
$$

其行列式为: $\exp [\sum_j s(x_{1:d})_j]$

Since computing the Jacobian determinant of the coupling layer operation does not involve computing the Jacobian of $s$ or $t$, those functions can be arbitrarily complex. We will make them deep convolutional neural networks.

computing the inverse is no more complex than the forward propagation:

$$
\begin{aligned}
x_{1:d} &amp;= y_{1:d} &#92;
x_{d+1:D} &amp;= (y_{d+1:D} - t(x_{1:d})) \bigodot  \exp(-s(y_{1:d}))
\end{aligned}
$$

<h3>Masked convolution</h3>

Partitioning can be implemented using a binary mask b, and using the functional form for $y$:

$$ y = b \bigodot x + (1-b) \bigodot (x \bigodot exp(s(b \bigodot x)) + t(b \bigodot x))$$

<blockquote>
  We use two partitionings that exploit the local correlation structure of images: spatial checkerboard patterns, and channel-wise masking. The spatial checkerboard pattern mask has value 1 where the sum of spatial coordinates is odd, and 0 otherwise. The channel-wise mask b is 1 for the first half of the channel dimensions and 0 for the second half. For the models presented here, both s(·) and t(·) are rectified convolutional networks.
</blockquote>

<img src="https://www.tvect.cn/wp-content/uploads/2019/10/mask_schema.jpg" alt="" />

<h3>Combining coupling layers</h3>

Although coupling layers can be powerful, their forward transformation leaves some components unchanged. This difficulty can be overcome by composing coupling layers in an alternating pattern, such that the components that are left unchanged in one coupling layer are updated in the next

<h2>Multi-scale architecture</h2>

<img src="https://www.tvect.cn/wp-content/uploads/2019/10/d08fc734415ef6bbc281bd4c90da28bc.png" alt="" />

每个 $f^{(i)}$ 是一系列的 coupling-squeezing-coupling 操作.

对于 $ f^{(i \le L)} $ 有:

<blockquote>
  we first apply three coupling layers with alternating checkerboard masks, then perform a squeezing operation, and finally apply three more coupling layers with alternating channel-wise masking.
</blockquote>

对于 $f^{(L)}$ 有

<blockquote>
  For the final scale, we only apply four coupling layers with alternating checkerboard masks.
</blockquote>

<strong>squeezing operation:</strong> 
for each channel, it divides the image into subsquares of shape 2 × 2 × c, then reshapes them into subsquares of shape 1 × 1 × 4c. The squeezing operation transforms an s × s × c tensor into an s/2 × s/2 × 4c tensor

<h1>Glow: Generative Flow with Invertible 1×1 Convolutions</h1>

<h2>常见 generative models 及 flow-based generative model 的优点</h2>

<strong>generative models 简单分类:</strong>

<ul>
<li><strong>likelihood-based methods</strong>

<ol>
<li>Autoregressive models.
Those have the advantage of simplicity, but have as disadvantage that synthesis has limited parallelizability, since the computational length of synthesis is proportional to the dimensionality of the data; this is especially troublesome for large images or video.</p></li>
<li><p>Variational autoencoders (VAEs), which optimize a lower bound on the log-likelihood of the data.
Variational autoencoders have the advantage of parallelizability of training and synthesis, but can be comparatively challenging to optimize.</p></li>
<li><p>Flow-based generative models.</p></li>
</ol></li>
<li><p><strong>generative adversarial networks (GANs)</strong></p></li>
</ul>

<p><strong>merits of flow-based generative models</strong>

<ul>
<li>Exact latent-variable inference and log-likelihood evaluation.
In VAEs, one is able to infer only approximately the value of the latent variables that correspond to a datapoint. GAN’s have no encoder at all to infer the latents. In reversible generative models, this can be done exactly without approximation. Not only does this lead to accurate inference, it also enables optimization of the exact log-likelihood of the data, instead of a lower bound of it.</p></li>
<li><p>Efficient inference and efficient synthesis.
Autoregressive models, such as the Pixel- CNN, are also reversible, however synthesis from such models is difficult to parallelize, and typically inefficient on parallel hardware. Flow-based gener- ative models like Glow (and RealNVP) are efficient to parallelize for both inference and synthesis.</p></li>
<li><p>Useful latent space for downstream tasks.
The hidden layers of autoregressive models have unknown marginal distributions, making it much more difficult to perform valid manipulation of data. In GANs, datapoints can usually not be directly represented in a latent space, as they have no encoder and might not have full support over the data distribution. This is not the case for reversible generative models and VAEs, which allow for various applications such as interpolations between datapoints and meaningful modifications of existing datapoints.</p></li>
<li><p>Significant potential for memory savings.
Computing gradients in reversible neural networks requires an amount of memory that is constant instead of linear in their depth, as explained in the RevNet paper.</p></li>
</ul>

<h2>Proposed Generative Flow</h2>

<p><img src="https://www.tvect.cn/wp-content/uploads/2019/10/c31a49d4f9032e76e77ac3e6932a0620.png" alt="" />

<img src="https://www.tvect.cn/wp-content/uploads/2019/10/03460346b43806dd6b3c11921809f355.png" alt="" />

<h3>Actnorm: scale and bias layer with data dependent initialization</h3>

<blockquote>
  We propose an actnorm layer (for activation normalizaton), that performs an affine transformation of the activations using a scale and bias parameter per channel, similar to batch normalization. These parameters are initialized such that the post-actnorm activations per-channel have zero mean and unit variance given an initial minibatch of data. This is a form of data dependent initialization (Salimans and Kingma, 2016). After initialization, the scale and bias are treated as regular trainable parameters that are independent of the data.
</blockquote>

<h3>Invertible 1 × 1 convolution</h3>

NICE &amp; REAL NVP proposed a flow containing the equivalent of a permutation that reverses the ordering of the channels. We propose to replace this fixed permutation with a (learned) invertible 1 × 1 convolution, where the weight matrix is initialized as a random rotation matrix. Note that a 1 × 1 convolution with equal number of input and output channels is a generalization of a permutation operation.

The log-determinant of an invertible 1 × 1 convolution of a h × w × c tensor h with c × c weight matrix W is straightforward to compute:

$$
\log |det ( \frac{\partial conv2d(h; W)} {\partial h} )| = h · w · \log | det(W)|
$$

The cost of computing or differentiating $det(W)$ is $O(c^3)$, which is often comparable to the cost computing $conv2D(h; W)$ which is $O(h · w · c^2)$. We initialize the weights $W$ as a random rotation matrix, having a log-determinant of 0; after one SGD step these values start to diverge from 0.

<strong>LU Decomposition</strong>

This cost of computing $det(W)$ can be reduced from $O(c^3)$ to $O(c)$ by parameterizing $W$ directly in its LU decomposition: $W = PL(U + diag(s))$

where $P$ is a permutation matrix, $L$ is a lower triangular matrix with ones on the diagonal, $U$ is an upper triangular matrix with zeros on the diagonal, and $s$ is a vector.

In this parameterization, we initialize the parameters by first sampling a random rotation matrix $W$, then computing the corresponding value of $P$ (which remains fixed) and the corresponding initial values of $L$ and $U$ and $s$ (which are optimized).

<h3>Affine Coupling Layers</h3>

类似于 Real NVP 中的 coupling layers，不过这里只保留了 channel-wise masking, 不再使用 spatial checkerboard patterns.

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1410.8516">NICE: NON-LINEAR INDEPENDENT COMPONENTS ESTIMATION</a></p></li>
<li><p><a href="https://arxiv.org/abs/1605.08803">DENSITY ESTIMATION USING REAL NVP</a></p></li>
<li><p><a href="https://arxiv.org/abs/1807.03039">Glow: Generative Flow with Invertible 1×1 Convolutions</a></p></li>
<li><p><a href="https://zhuanlan.zhihu.com/p/41912710">细水长flow之NICE：流模型的基本概念与实现</a></p></li>
<li><p><a href="https://zhuanlan.zhihu.com/p/43048337">RealNVP与Glow：流模型的传承与升华</a></p></li>
</ul>