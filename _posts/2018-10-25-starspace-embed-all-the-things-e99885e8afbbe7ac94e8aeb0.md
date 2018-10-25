---
ID: 822
post_title: 'StarSpace &#8211; Embed All The Things 阅读笔记'
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/822
published: true
post_date: 2018-10-25 22:28:20
---
[toc]

之前和 rasa_nlu 贡献者沟通, TA 有提到 Embedding intent classifier 可以用来做 multiple intents. 所以有翻出来其中提到的 paper 和相关的代码。

文档中提到, embedding intent classifier 参考了 StarSpace: Embed All The Things 的做法, 把 user inputs 和 intent labels 都映射到同一个 space, 进而可以做 classification 之类的任务.

下面是对文章 StarSpace: Embed All The Things 的一些笔记和rasa_nlu中相应代码的说明。

<!--more-->

<h1>StarSpace 的笔记</h1>

<h2>Model</h2>

<h2>应用场景</h2>

<ul>
<li>Multiclass Classification (e.g. Text Classification)</p></li>
<li><p>Multilabel Classification</p></li>
<li><p>Collaborative Filtering-based Recommendation</p></li>
<li><p>Collaborative Filtering-based Recommendation with
out-of-sample user extension</p></li>
<li><p>Content-based Recommendation</p></li>
<li><p>Multi-Relational Knowledge Graphs (e.g. Link Prediction)</p></li>
<li><p>Information Retrieval (e.g. Document Search) and Document Embeddings</p></li>
<li><p>Learning Word Embeddings</p></li>
<li><p>Learning Sentence Embeddings</p></li>
<li><p>Multi-Task Learning</p></li>
</ul>

<h1>rasa_nlu 中使用案例</h1>

<blockquote>
  <p>The embedding intent classifier embeds user inputs and intent labels into the same space. Supervised embeddings are trained by maximizing similarity between them. This algorithm is based on the starspace idea from: https://arxiv.org/abs/1709.03856. However, in this implementation the mu parameter is treated differently and additional hidden layers are added together with dropout. This algorithm also provides similarity rankings of the labels that did not “win”.
</blockquote>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1709.03856">论文原文 StarSpace: Embed All The Things</a></p></li>
<li><p><a href="https://github.com/facebookresearch/StarSpace">github 代码地址: </a></p></li>
<li><p><a href="https://github.com/RasaHQ/rasa_nlu/blob/0.12.3/rasa_nlu/classifiers/embedding_intent_classifier.py#L37">rasa_nlu 中 EmbeddingIntentClassifier 实现代码</a></p></li>
</ul>