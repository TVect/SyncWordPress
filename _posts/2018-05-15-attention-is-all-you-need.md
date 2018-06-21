---
ID: 228
post_title: Attention is all you need 笔记
author: chin340823
post_excerpt: ""
layout: post
permalink: >
  http://www.tvect.cc/2018/05/15/attention-is-all-you-need/
published: true
post_date: 2018-05-15 18:46:55
---
<strong>下面是 Attention is all you need 的翻译或者说笔记。</strong>
Seq2Seq 模型中使用 RNN 时，会有时序依赖，无法并行。同时 RNN 中要建立 long range dependencies，需要 O(range) 次计算。这篇文章和之前一篇 ConvS2S，都旨在解决上述问题。

ConvS2S使用CNN取代了RNN，而这篇文章则摒弃了RNN和CNN，直接使用attention建模。

<h1>概述</h1>

主流的 seqence transduction models 是基于复杂的循环或者卷积网络的，包含了 encoder 和 decoder 的部分，同时还通过 attention 机制关联了 encoder 和 decoder。
本文提出了一个新颖的简单的网络结构Transformer，只依赖于 attention 机制，完全免除了 CNN 或者 CNN 的结构。
机器翻译任务的实验表明，模型在质量上更加优越、并行性更好并且需要的训练时间显著减少。

<h1>模型结构</h1>

Transformer 同样也采用了 encoder-decoder 的架构，在 encoder 和 decoder 中使用了 stacked self-attention and point-wise, fully connected layers。

模型结构的图示如下：

<img src="http://www.tvect.cc/wp-content/uploads/2018/05/transformer-structure.png" alt="" />

<h2>Encoder and Decoder Stacks</h2>

<strong>Encoder部分</strong> 由6个相同的layer堆叠而成，其中，每个layer由两个sub-layer构成。第一个sub-layer使用 <strong>multi-head self-attention</strong> 机制实现，每个 block 中使用相同的 query, key 和 value, 均来源于前一个 encoder layer。第二个sub-layer则是一个简单的<strong>全连接前馈神经网络</strong>。在每个sub-layer上都使用了残差连接以及layer normalization，即每个sub-layer的输出为LayerNorm(x+Sublayer(x))。

<strong>Decoder部分</strong>同样由6个相同的layer堆叠而成。其中，每个layer由三个sub-layer构成。第一个是 <strong>masked multi-head self-attention</strong>, 与 encoder 部分类似, 但是做了 mask 操作, 过滤掉不合法的连接（当前的生成过程不应当依赖于将来生成的值）。第二个 sub-layer 是 <strong>encoder-decoder attention</strong>, 它对应的 query 来源于前一个 decoder layer, key 和 vaule 来源于 encoder 的output。第三个 sub-layer 是一个简单的<strong>全连接前馈神经网络</strong>, 和 encoder 部分一样。
类似于encoder，decoder的每个sub-layer上也使用了残差连接与layer normalization。

<h2>Attention</h2>

<img src="http://www.tvect.cc/wp-content/uploads/2018/05/transformer-attention.png" alt="" />

<h3>Scaled Dot-Product Attention</h3>

<strong>Inputs</strong>: a query q, and a set of key-value (k-v) pairs.
<strong>Output</strong>: weighted sum of values, where weight of each value is computed by an inner product of query and corresponding key.

query,keys, values, and output are all vectors.
queries and keys have same dimensionality $$d_{k}$$, value have dimensionality $$d_{v}$$

$$
Attention(q, K, V) = \\sum_{i} \\frac{exp(q*k_{i})}{(\\sum_{j} exp(q*k_{j}))} v_{i}
$$

当有多个 query 的时候, 可以把这些 query 向量堆叠成矩阵 $$Q$$, 进而有：

$$
Attention(Q, K, V) = softmax(QK^{T}) v_{i}
$$
$$
dimentionality: [|Q|, d_{k}] * [d_{k}, |K|] * [|K|, d_{v}] = [|Q|, d_{v}]
$$

<strong>Problem</strong>:
As $$d_{k}$$ gets large, the variance of $$q*k$$ increases,
$$\\rightarrow$$ some values inside the softmax get larger,
$$\\rightarrow$$ the softmax gets very peaked,
$$\\rightarrow$$ hence its gradient gets smaller.

<strong>Solution</strong>:
Scale by length of query/key vectors
$$
Attention(Q, K, V) = softmax(\\frac{QK^{T}}{\\sqrt{d_{k}}}) v_{i}
$$

<h3>Multi-Head Attention</h3>

先将Q, K, V映射到低纬空间，做h个这样的映射, 在低纬空间中做scaled dot-product attention, 将最后得到的 h 个结果拼接起来, 最后再做一个线性变换得到输出。

$$ MultiHead(Q, K, V) = Concat(head_{1}, ..., head_{k}) W^{O} $$

$$ where \\quad head_{i} = Attention(QW_{i}^{Q}, KW_{i}^{K}, VW_{i}^{V}) $$

$$ W_{i}^{Q} \\in R^{d_{model}*d_{k}},  W_{i}^{K} \\in R^{d_{model}*d_{k}}, W_{i}^{V} \\in R^{d_{model}*d_{v}}, W^{O} \\in R^{hd_{v}*d_{model}} $$

<h3>Applications of Attention in our Model</h3>

<ol>
<li>在encoder-decoder attention层，query来自于先前的解码层，key与value来自于encoder的输出。</p></li>
<li><p>在encoder上单独使用了self-attention。这里的key，value，query都来自于encoder中上一层的输出。</p></li>
<li><p>在decoder上也单独使用了self-attention。这里的key，value，query来自于decoder中当前时间步及之前的输出。为了避免信息向左流动，在scaled dot-product attention中增加了一个屏蔽层（mask），用以屏蔽掉那些softmax中的不合法连接（仅在训练时发挥作用）。</p></li>
</ol>

<h2>Position-wise Feed-Forward Networks</h2>

<p>前面提到的前馈神经网络有两层，用 RELU 做激活函数：
$$ FFN(x) = max(0, xW_{1} + b_{1}) W_{2} + b_{2} $$

每一层都有各自的权重矩阵, 同一层当中的权重矩阵在不同的 position 是共享的。
也可以把这种变换视为 kernel size 为 1 的两个卷积的叠加。

文章中具体使用的输入输出维度为 $$ d_{model} = 512, d_{ff} = 2018 $$.

<h2>Embeddings and Softmax</h2>

与其他序列转导模型类似，我们使用学习到的嵌入将输入词符和输出词符转换为维度为$$d_{model}$$ 的向量。 我们还使用普通的线性变换和 softmax 函数将解码器输出转换为预测的下一个词符的概率。
在我们的模型中，两个嵌入层之间和pre-softmax线性变换共享相同的权重矩阵。 在嵌入层中，我们将这些权重乘以 $$ \\sqrt{d_{model}} $$ ???

<h2>Positional Encoding</h2>

由于本文的模型不包含循环和卷积结构，为了让模型利用序列的顺序，我们必须注入序列中关于词符相对或者绝对位置的一些信息。

因为，如果不做位置编码的话，交换输入序列中相邻两个词的位置，会产生相同的 attention, 导致无法获取位置信息。

有多种位置编码可以选择，文章中使用的位置编码如下：

$$ PE(pos, 2i) = sin(\\frac{pos}{10000^{2i/d_{model}}}) $$

$$ PE(pos, 2i+1) = cos(\\frac{pos}{10000^{2i/d_{model}}}) $$

其中pos 是位置，i 是维度。

<h2>其他的Trick</h2>

<h3>label smoothing</h3>

<blockquote>
  First, it may result in over-fitting: if the model learns to assign full probability to the groundtruth label for each training example, it is not guaranteed to generalize.
  Second, it encourages the differences between the largest logit and all others to become large, and this, combined with the bounded gradient ∂l/∂zk , reduces the ability of the model to adapt. Intuitively, this happens because the model becomes too confident about its predictions.
  For a training example with ground-truth label y, we replace the label distribution q(k|x) = delta(k,y) with q(k|x) = (1 − epsilon) delta(k,y) + epsilon * u(k)。
  In our experiments, we used the uniform distribution u(k) = 1/K, so that q(k) = (1 − epsilon) delta(k,y) +  epsilon/K .
</blockquote>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1706.03762">原始论文: Attention is all you need</a></p></li>
<li><p><a href="https://kexue.fm/archives/4765">博客：一文读懂「Attention is All You Need」| 附代码实现</a></p></li>
<li><p><a href="https://arxiv.org/pdf/1512.00567.pdf">label-smoothing regularization (LSR)</a></p></li>
<li><p><a href="https://github.com/Kyubyong/transformer">代码实现：A TensorFlow Implementation of the Transformer</a></p></li>
</ul>