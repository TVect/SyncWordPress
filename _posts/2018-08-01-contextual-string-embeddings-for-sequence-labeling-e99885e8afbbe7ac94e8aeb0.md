---
ID: 620
post_title: >
  Contextual String Embeddings for
  Sequence Labeling 阅读笔记
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://blog.tvect.cc/2018/08/01/contextual-string-embeddings-for-sequence-labeling-%e9%98%85%e8%af%bb%e7%ac%94%e8%ae%b0/'
published: true
post_date: 2018-08-01 08:02:42
---
[toc]

<h1>概述</h1>

当前的 word embedding 方法大概分为三类：

<ol>
<li>Classical word embeddings
pre-trained over very large corpora and shown to capture latent syntactic and semantic similarities.</li>
<li>Character-level features
not pre-trained, but trained on task data to capture task-specific subword features.</li>
<li>Contextualized word embeddings
capture word semantics in context to address the polysemous and context-dependent nature of words.</li>
</ol>

这篇文章中训练了一个 character language model，利用其中的 internal states，为每个 word 产生了一个 embedding. 文章中将这种 embedding 称为 contextual string embeddings，声称是结合了上述三种方法中优点特性：

<ol>
<li>pre-train on large unlabeled corpora</li>
<li>capture word meaning in context and therefore produce different embeddings for polysemous words depending on their usage</li>
<li>model words and context fundamentally as sequences of characters, to both better handle rare and misspelled words as well as model subword structures such as prefixes and endings.</li>
</ol>

文章中有利用 contextual string embeddings 在四种序列标注任务上做了实验，结果都有提升，特别是在 NER 任务上有了显著提升。

<h1>模型结构</h1>

<h2>Recurrent Network States</h2>

普通的基于 LSTM 的 character 级别的 language model，包括了一个前向的和一个后向的 language model.

记 forward language model 中得到的hidden state 和 cell state 为 $$h_{t}^{f}, c_{t}^{f}$$.
记 backward language model 中得到的hidden state 和 cell state 为 $$h_{t}^{b}, c_{t}^{b}$$.

<h2>Extracting Word Representations</h2>

<h2>Sequence Labeling Architecture</h2>

<h1>实验和结论</h1>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://drive.google.com/file/d/17yVpFA7MmXaQFTe-HDpZuqw9fJlmzg56/view">论文: Contextual String Embeddings for Sequence Labeling</a></p></li>
<li><p><a href="https://github.com/zalandoresearch/flair">Flair embeddings 代码实现</a></p></li>
</ul>