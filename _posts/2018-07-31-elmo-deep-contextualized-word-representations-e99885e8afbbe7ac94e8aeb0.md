---
ID: 608
post_title: 'ELMo: Deep contextualized word representations 阅读笔记'
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://blog.tvect.cc/2018/07/31/elmo-deep-contextualized-word-representations-%e9%98%85%e8%af%bb%e7%ac%94%e8%ae%b0/'
published: true
post_date: 2018-07-31 22:28:14
---
[toc]

总体上的感觉是，<strong>使用 预训练的语言模型 替代/增强 预训练的词向量</strong>，更好的建模了词的语义语法特性和不同上下文环境中的多义性。

<h1>概述</h1>

好的词向量模型应该能够
1. 表达词的复杂特性，比如语义和语法的特性，
2. 另外也需要能够刻画这些特性在不同语言上下文中的变化

传统的词向量模型当中，学到的向量表示更多的是一种上下文独立的表示。过去一般会使用 subword information 来增强词向量表示，或者是为每个word sense 学习一个向量表示。这篇文章中的方法既有使用Character CNN来捕获 subword information，也有学习到 multi sense information。

文章中通过学习一个 deep bidirectional language model (biLM)，将词向量表示为 biLM 的内部状态的函数（也不仅仅是简单的取最后一层的表示）。从而，学到的每个 word representation 不再仅仅是一个 word 的函数，而是整个 sentence 的函数。

文章中将这种词向量表示称为 ELMo representation, 意为 Embeddings from Language Models.

最后，这种从 biLM 得到的词向量表示可以很方便的加到现有的任务模型上，并且得到很大程度上的提升，这些任务包括question answering, textual entailment 和 sentiment analysis 等。

<hr />

<h1>模型结构</h1>

<h2>Bidirectional language models</h2>

给定 N 个 tokens 的序列 $$(t_{1}, ..., t_{n})$$, biLM 是要极大化log likelihood:

$$
\\sum \\limits_{k=1}^{N}(log \\, p(t_{k} | t_{1}, ..., t_{k-1}; \\theta_{x}, \\theta^{\\rightarrow}_{lstm}, \\theta_{s}) + log \\, p(t_{k} | t_{k+1}, ..., t_{N}; \\theta_{x}, \\theta^{\\leftarrow}_{lstm}, \\theta_{s}))
$$

其中, 在forward 和 backword 的 language model 中共享 token representation ($$\\theta_{x}$$), 和 Softmax layer ($$\\theta_{s}$$), 同时保持各自独立的 lstm 参数 ($$\\theta^{\\rightarrow}_{lstm}, \\theta^{\\leftarrow}_{lstm}$$)

<h2>ELMo</h2>

ELMo 是一个 biLM 中间层表示的任务特定的组合。

对于每个 token，L 层的 biLM 会得到 2L+1 个向量表示

$$
R_{k} = \\{x_{k}, h^{\\rightarrow}_{k, j}, h^{\\leftarrow}_{k, j} | j=1, 2, ..., L\\}=\\{h_{k, j} | j=0, 1, ..., L\\}
$$

对于不同的任务，ELMo 表示为

$$
ELMo_{k}^{task} = E(R_{k}; \\theta^{task}) = \\gamma^{task} \\sum \\limits_{j=0}^{L} s_{j}^{task} h_{k, j}
$$

其中，$$s^{task}$$ 为 softmax-normalized weights. 
$$\\gamma^{task}$$ 为 scalar parameter that allows the task model to scale the entire ELMo vector, 其在实际中能帮助训练.

另外，考虑到 biLM 每一层的激活有着不同的分布，有时候在做加权之前做layer normalization 也会有用.

<h2>Using biLMs for supervised NLP tasks</h2>

假设现在已有一个 pre-trained 的 biLM，和一个任务特定的监督模型，可以很方便的利用 biLM 来提升监督模型的性能。

比如，现在的监督任务模型使用通常的 word embedding $$x_{emb}$$ 作为输入，经过几层 RNN，每个词得到一个相应的输出 $$h_{rnn}$$。
为了利用pre-trained biLM，将 biLM 的参数固定住，为每个词计算相应的 $$ELMo^{task}$$ (参数 $$s^{task}$$ 和 $$\\gamma^{task}$$ 可以和监督任务模型进行联合训练得到).
可以有几种方式使用 $$ELMo^{task}$$:

<ul>
<li>将 EMLo 加在 RNN 的输入层，即为 $$[x_{emb}, ELMo^{task}]$$</li>
<li>将 EMLo 加在 RNN 的输出层，即为 $$[x_{emb}, ELMo^{task}]$$</li>
<li>将 ELMo 同时加在 RNN 的输入层和输出层.</li>
</ul>

最后，在使用 ELMo 的过程中，加上一些 dropout 或者 regularize，可能会更好. (<strong>加在 $$s^{task},\\gamma^{task}$$ 上??? 毕竟biLM 参数不是固定了吗???</strong>)

<blockquote>
  Finally, we found it beneficial to add a moderate amount of dropout to ELMo and in some cases to regularize the ELMo weights by adding w^2 to the loss. This imposes an inductive bias on the ELMo weights to stay close to an average of all biLM layers.
</blockquote>

<h2>Pre-trained biLM architecture</h2>

在这篇文章中实际使用的 biLM 结构中，先对每个 word 中的 character 做 convolution 得到 word 的表示，具体的使用的 conv 共有 32+ 32+ 64+ 128 + ...+ 1024 = 2048 filters， 后面接两个highway layer，再做线性投射到 512 dim，后面接 2 层 lstm. 最后为每个 word 提供了 3 层的表示 (传统的 word embedding 的方式只提供了 1 层的表示).

具体的模型细节可以参看参考资料中代码实现.

在 biLM 预训练完毕之后，就可以为特定的任务计算向量表示了。文章中提到在有些 case 当中，用domain specific 的数据对 biLM 做 fine tuning 会带来 perplexity 的显著下降和下游任务性能的提升。

<hr />

<h1>实验结果分析</h1>

<ul>
<li><p>相比于之前直接使用 top layer 的表示，使用 deep contextual representation (即使用不同 layer 表示的组合) 更能提升下游任务的性能。</p></li>
<li><p>语法信息在 lower layer 有更好的表示，而语义信息更多的是在 top layer 被捕获。</p></li>
<li><p>在下游任务中使用 ELMo，有助于在 training size 比较小的训练集获得比较好的 performance. 另外，也可以减少达到 state-of-the-art performance 所需要的参数更新次数。</p></li>
</ul>

<hr />

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1802.05365">Deep contextualized word representations
</a></p></li>
<li><p><a href="https://github.com/allenai/bilm-tf">代码实现 tensorflow 版: bilm-tf</a></p></li>
<li><p><a href="https://github.com/allenai/allennlp">allennlp实现</a></p></li>
</ul>