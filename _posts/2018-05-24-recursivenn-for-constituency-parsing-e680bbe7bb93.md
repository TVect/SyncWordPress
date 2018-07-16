---
ID: 281
post_title: >
  RecursiveNN for Constituency Parsing
  总结
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://blog.tvect.cc/2018/05/24/recursivenn-for-constituency-parsing-%e6%80%bb%e7%bb%93/'
published: true
post_date: 2018-05-24 17:07:45
---
<h1>概述</h1>

<h1>模型</h1>

<h2>Standard Recursive Neural Network</h2>

<h3>Weight-Tied</h3>

最基本的递归神经网络结构如下：

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/rvnn-1024x354.png" alt="" />

上述模型结构图中表示的任务, 是使用 node 的向量表示来预测 node 表示的短语的情感极性。

具体到 Structure Prediction 的任务中, 图示如下：

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/rvnn_constituent.png" alt="" />

在 Recursive Neural Networks for Structure Prediction 的任务中, 可以通过 RNN 得到 2 个 node 合并之后的向量表示, 同时可以基于得到的向量表示计算一个得分, 表示多大程度上两个子节点应该合并为一个父节点。根据每个 node 的得分将得分最高的 node 进行合并。继续整个流程, 基于当前已合并的节点, 计算节点合并的得分...直到合并到 root 节点。

所以, 使用这种贪婪的方式, Recursive Neural Networks 可以同时学到短语的向量表示和句子的句法结构。

<h3>Weight-Untied</h3>

** Model: Compositional Vector Grammars(CVGs) = syntactically untied RNN (SU-RNN) + PCFG **

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/cvgs-1024x773.png" alt="" />

<h4>SU-RNN</h4>

下面的 MV-RNN 模型考虑的合成函数依赖于参与组合的词，可能会比较难以捕获相似POS tags或相似句法类别上的句法相似性，因为没有这些相似POS tags上不共享权重。

而在 SU-RNN 中, 对于不同的句法环境(POS tags), 使用不同的合成函数或者合成矩阵。而在相同的句法环境(POS tags)中, 保持同样的合成函数或者合成矩阵。具体的, 会根据不同的 discrete syntactic categories of the children to choose the composition matrix。

<h4>PCFG</h4>

考虑到 beam search 中的每个候选得分都需要进行 matrix-vector product 操作, 很费时。所以, 可以先用 PCFG 去除 unlikely candidate tree, 只考虑 subset of trees, 提升速度。

对于 PCFG 中的 CFG 部分, 一般是由领域相关的专家给出的, 例如英语专家规定英语的 CFG. 而PCFG 中的 p 是从语料中统计而来. 运用最大似然估计

整个 inference 过程包含两步。
1. 使用基本的PCFG, 运行CKY动态规划算法, 保存最后得到的 top-k parsing tree。
2. 只在第一步得到的 top-k 的 parsing tree 上, 运行完整的 CVGS 模型的 beam search。

训练的过程, 同样有两步, 类似上面 inference 过程的两步。

<h2>Matrix-Vector Recursive Neural Network</h2>

考虑到某些 word 的作用会像一个 operator, 比如 "very good" 中的 "very". 所以, 用一个 vector 和一个 matrix 在一起表示一个 word, 其中的 matrix 表示 word 的 operator 方面的属性。

具体操作时, 会先把一个词的 matrix 和另一个词的 vector 相乘, 将得到的两个 vector 拼接之后再做通常的线性变换。

相比于 Standard Weight-Tied Recursive Neural Network, 这种方式得到的 composition function 相比之下会更加 powerful。

模型结构图如下：

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/mvrnn-1024x196.png" alt="" />

<h2>Recursive Neural Tensor Network</h2>

MV-RNN 的参数量太大了，特别是在 vocabulary size 很大的时候。

RNTN 使用了固定参数数目的单个的 powerful 的组合函数 (通过使用很多个双线性变换)。对于不同的 node, RNTN 都使用的是同样的一个基于 tensor 的 composition function。

模型结构图如下：

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/rntn-1024x442.png" alt="" />

其中，使用了很多个的双线性变换，这些双线性变换可以帮助直接建立input words之间的联系。

<h2>Tree LSTM</h2>

传统的 LSTM 架构只考虑了严格的序列信息传递, 下面的两种扩展 Child-Sum Tree-LSTM 和 N-ary Tree-LSTM, 可以利用来自多个子节点的更加丰富的信息。另外, Tree-LSTM 可能会对每个 Child Node 都有一个 forget gate, 可以选择性的整合这些 Children 的信息。

<h3>Child-Sum Tree-LSTMs</h3>

Child-Sum Tree-LSTMs 的计算公式如下。注意到, 在 tree 的结构为简单的 chain 的时候, 下面的计算公式退化为标准的 LSTM 计算公式。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/dependency-tree-lstm.png" alt="" />

<h4>Dependency Tree-LSTMs</h4>

<blockquote>
  Child-Sum Tree-LSTM unit conditions its components on the sum of child hidden states, it is well suited for trees with high branching factor or whose children are unordered. For example, it is a good choice for dependency trees, where the number of dependents of a head can be highly variable.
  We refer to a Child-Sum Tree-LSTM applied to a dependency tree as a Dependency Tree-LSTM.
</blockquote>

在 Dependency Tree-LSTM 中, Tree 中每个节点都对应着 input sentence 中的一个 head word。

<h3>N-ary Tree-LSTMs</h3>

<blockquote>
  The N-ary Tree-LSTM can be used on tree structures where the branching factor is at most N and where children are ordered,
</blockquote>

N-ary Tree-LSTMs 的计算公式如下。注意到, 在 tree 的结构为简单的 chain 的时候, 下面的计算公式退化为标准的 LSTM 计算公式。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/05/nary-tree-lstm-1024x768.png" alt="" />

<h4>Constituency Tree-LSTMs</h4>

在 Constituency Tree-LSTM 中, Tree 中每个叶子节点对应着 input sentence 中的一个 word。

<h1>参考资料</h1>

<ul>
<li><a href="https://nlp.stanford.edu/pubs/SocherEtAl_EMNLP2013.pdf">Recursive Deep Models for Semantic Compositionality Over a Sentiment Treebank</a>

<blockquote>
  Richard Socher, Alex Perelygin, Jean Y. Wu, Jason Chuang,
  Christopher D. Manning, Andrew Y. Ng and Christopher Potts
</blockquote>

提出了Stanford Sentiment Treebank, 和模型Recursive Neural Tensor Network</p></li>
<li><p><a href="https://nlp.stanford.edu/pubs/SocherBauerManningNg_ACL2013.pdf">Parsing with Compositional Vector Grammars</a>

<blockquote>
  Richard Socher John Bauer Christopher D. Manning Andrew Y. Ng
</blockquote>

提出了CVGs,结合了SU-RNN和PCFG</p></li>
<li><p><a href="https://arxiv.org/abs/1503.00075">Improved Semantic Representations From Tree Structured LSTM</a>

<blockquote>
  Kai Sheng Tai, Richard Socher, Christopher D. Manning
</blockquote></li>
<li><p><a href="http://web.stanford.edu/class/cs224n/archive/WWW_1617/lectures/cs224n-2017-lecture14-TreeRNNs.pdf">Lecture 14: Tree Recursive Neural Networks and Constituency Parsing</a></p></li>
</ul>