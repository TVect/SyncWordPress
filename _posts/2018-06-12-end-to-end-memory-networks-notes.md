---
ID: 341
post_title: End-To-End Memory Networks notes
author: Chin
post_excerpt: ""
layout: post
permalink: >
  http://www.tvect.cc/2018/06/12/end-to-end-memory-networks-notes/
published: true
post_date: 2018-06-12 17:24:21
---
<h1>概述</h1>

<strong>Two grand challenges</strong>：

<ul>
<li>make multiple computational steps in the service of answering a question or completing a task</li>
<li>describe long term dependencies in sequential data.</li>
</ul>

文章中提出了一个新颖的 RNN 结构，在产生一个输出之前，可以对外存记忆循环读取多次。

<ul>
<li>与之前的Memory Network相比：
之前的 Memory Network 不好通过反向传播进行简单的训练, 而且在每一层它都需要标注数据。而这里的模型可以视为一个 continuous form of the Memory Network, 可以实现 end-to-end 训练。</li>
<li>与 RNNsearch 相比：
这里的模型增加了 multiple hops。后面试验表明在 long-term memory 上的 multiple hops, 对于获取好的性能是很必要的。</li>
</ul>

<h1>模型</h1>

<h2>Single Layer</h2>

将要保存到 memory 中的 input 记为 $$x_{1}, x_{2}, x_{3}, ...$$, query 记为 q.

<h3>Input memory representation</h3>

最简单的情况下, 通过 embedding matrix A, 将 input $$x_{1}, x_{2}, x_{3}, ...$$ 转化为 memory vector $$m_{1}, m_{2}, m_{3}, ...$$ 。通过 embedding matrix B, 将 query q 转化为另一个向量表示 u.

接下来，通过内积和 softmax 操作, 计算 u 和 m 之间的匹配度: $$ p_{i} = Softmax(u^{T}m_{i})$$

<h3>Output memory representation</h3>

每个输入 $$x_{i}$$ 还对应着一个 output vector $$c_{i}$$. 最简单的方法, 可以通过另一个 embedding matrix C 得到（与前面的memory vector类似）.

最后通过$$p_{i}$$ 和 $$c_{i}$$ 做 weighted sum, 得到输出表示：$$o = \\sum_{i} p_{i} c_{i}$$

<h3>Generating the final prediction</h3>

利用前面得到的 output vector o 和 input embedding u, 得到最终的预测：$$\\hat {a} = Softmax(W(o + u))$$, 其中 $$W \\in R^{V*d}$$

<img src="http://www.tvect.cc/wp-content/uploads/2018/06/end2end-memnet-1024x509.png" alt="" />

<h2>Multiple Layers</h2>

通过将 memory layers 堆叠起来， 可以支持 multiple hops 操作。
具体如下：

<ul>
<li>第k+1层的输入$$u^{k+1}$$ 由第k层的输出 $$u^{k}$$ 和 第k层的输入 $$o^{k}$$ 共同决定。
$$ u^{k+1} = u^{k} + o^{k} $$</li>
<li>每一层都有自己的 embedding matrix $$A_{k}, C_{k}$$,
可以通过施加一些限制，来达到减少参数，简化训练的目的。</li>
<li>在最后一层，同样也结合该层的input 和 output，来产生response
$$\\hat {a} = Softmax(Wu^{K+1}) = Softmax(W (o_{K} + u_{K}))$$</li>
</ul>

文章中尝试了两种形式的权重共享：

<ol>
<li><strong>Adjacent</strong>
$$A^{k+1} = C^{k}$$
$$W^{T} = C^{K}$$
$$B = A^{1}$$</li>
<li><strong>Layer-wise (RNN-like)</strong>
$$A^{1} = A^{2} = ... = A^{K}$$
$$C^{1} = C^{2} = ... = C^{K}$$
同时在不同hop之间更新u中，增加了linear mapping: $$u^{k+1} = H u^{k} + o^{k}$$</li>
</ol>

总体上，模型很类似于之前的 Memory Network，只是这里用 continuous weighting 替代了 hard max operations.

<h1>Synthetic Question and Answering Experiments</h1>

之前的 Memory Network 结构，在训练的时候需要标注出 第一次相关的statements是哪个，第二次相关的statements是哪一个。
而本文的模型不需要，它只要最终的答案即可进行端到端的训练。

<h2>模型中一些细节</h2>

<h3>句子表示</h3>

尝试了两种形式的句子表示:

<ol>
<li>bag-of-words (BoW) representation
对于句子 $$x_{i} = {x_{i1}; x_{i2}; ... ; x_{in} }$$, 得到的相应的 memory vector 为：$$m_{i} = \\sum_{j} A x_{ij}$$, 得到的相应的 output vector 为： $$c_{i} = \\sum_{j} C x_{ij}$$.
对于 query q, 得到的表示为：$$u = \\sum_{i} B q_{i}$$
<strong>这种方法无法捕获词的顺序的信息</strong>。</p></li>
<li><p><strong>position encoding</strong>
$$ m_{i} = \\sum_{j} l_{j} \\cdot A x_{ij} $$
其中 $$ \\cdot $$ 表示元素乘法操作
$$ l_{j} $$是一个列向量，满足 $$ l_{kj} = (1-j/J)-(k/d)(1-2j/J) $$, $$ J $$是 sentence中 word 的个数, $$ d $$是 embedding 的维度。</p></li>
</ol>

<p>通过这种 position encoding，可以在句子表示中加入词序信息。同样的方法可以用在其他的句子表示部分。

<h3>Temporal Encoding</h3>

...

<h3>Learning time invariance by injecting random noise</h3>

...

<h3>linear start (LS) training</h3>

...

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1503.08895" title="End-To-End Memory Network">End-To-End Memory Networks</a></p></li>
<li><p><a href="https://blog.csdn.net/u014300008/article/details/52794821">CSDN 笔记</a></p></li>
<li><p><a href="https://zhuanlan.zhihu.com/p/32257642">论文笔记 - Memory Networks 系列</a></p></li>
</ul>