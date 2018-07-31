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

<h2>Pre-trained biLM architecture</h2>

<hr />

<h1>参考资料</h1>

<ul>
<li><a href="https://arxiv.org/abs/1802.05365">Deep contextualized word representations
</a></li>
</ul>