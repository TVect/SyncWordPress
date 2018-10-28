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

假定有一系列的 entities，这些 entities 由来自固定长度字典中不同的 features 集合来描述。StarSpace 模型可以用来学习这些 entities 的表示。在得到不同 entities 的表示之后，不同类型的 entities 之间甚至都可以进行比较。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/10/starspace-768x573.png" alt="" />

<h2>应用场景</h2>

<ul>
<li><strong>Multiclass Classification (e.g. Text Classification)</strong></li>
</ul>

positive pair 来源于训练数据集中的标注数据，negative entities 是从 possible labels 中采样得到的。

<ul>
<li><strong>Multilabel Classification</strong></li>
</ul>

和 Multiclass 的情况类似，只是这里每个 document 可能会对应着多个 positive labels.

<ul>
<li><strong>Collaborative Filtering-based Recommendation</strong></li>
</ul>

训练数据由一系列 users 组成, 每个 user 通过一系列该 user 感兴趣的 items 构成.

positive pair 是通过选定一个 user, 和该 user 感兴趣的一个 item 构成. negative entities 是从所有可能的 items 中采样得到。

<ul>
<li><strong>Collaborative Filtering-based Recommendation with out-of-sample user extension</strong></li>
</ul>

前面的协同过滤的方法不能很好的扩展到 new user, 因为它是为每个 user Id 学习了一个 embedding.

<blockquote>
  The positive pair generator instead picks a user, selects $a$ as all the items they like except one, and $b$ as the left out item. 
  That is, the model learns to estimate if a user would like an item by modeling the user not as a single embedding based on their ID, but by representing the user as the sum of embeddings of items they like.
</blockquote>

<ul>
<li><strong>Content-based Recommendation</strong></li>
</ul>

有一系列的 user, 每个 user 由一系列的 items 描述得到, 每一个 item 是由字典中的一系列特征描述得到. 与前面方法不一样的是，这里不是把每个 item 当作一个独立的 feature.

比如，在 document recommendation 任务当中，user 通过所有他喜欢的 documents 来描述，而每个 documents 又由 bag-of-words 来描述.

同样的，$a$ 可以选为 all the items they like except one, and $b$ as the left out item.

<ul>
<li><strong>Multi-Relational Knowledge Graphs (e.g. Link Prediction)</strong></li>
</ul>

给定三元组 (h, r, t)，这里是要学习 h, r, t 的表示.

可以把 h, r, t 的每个实例定义为字典里的一个特征. 在应用过程当中可以把 $a$ 当作由 h, r 构成的, 也可以把 $a$ 当作是仅由 h 构成的.

<ul>
<li><strong>Information Retrieval (e.g. Document Search) and Document Embeddings</strong></li>
</ul>

训练数据是由  (search keywords, relevant document) 构成. 所以，可以把 $a$ 选为 search keywords, $b$ 选为 relevant document, $b^-$ 选为 other irrelevant documents.

另外，如果没有形如上面的标注数据，只有 documents. 可以选择 $a$ 为某 document 中任意 keywords, $b$ 选为 remaining words.

<ul>
<li><strong>Learning Word Embeddings</strong></li>
</ul>

可以选 $a$ 为 a window of words, $b$ 为 the middle word.

<ul>
<li><strong>Learning Sentence Embeddings</strong></li>
</ul>

选择 $a, b$ 是来自于同一篇 document 中的两个 sentences (如果 document 过长, 可以限制选择的句子间的 distance 在一定的范围之内), $b^-$ 是来源于另外 document 里面的 sentence.

<ul>
<li><strong>Multi-Task Learning</strong></li>
</ul>

上面提到的这些任务中，如果公用了同样的 dictionary，那么就可以结合在一起，同时训练。

比如，<strong>supervised classification</strong> + <strong>unsupervised word or sentence embedding</strong>

<h1>rasa_nlu 中使用案例</h1>

以下参考的版本为 rasa-nlu==0.12.3

<blockquote>
  The embedding intent classifier embeds user inputs and intent labels into the same space. Supervised embeddings are trained by maximizing similarity between them. This algorithm is based on the starspace idea from: https://arxiv.org/abs/1709.03856. However, in this implementation the mu parameter is treated differently and additional hidden layers are added together with dropout. This algorithm also provides similarity rankings of the labels that did not “win”.
</blockquote>

当前的 rasa_nlu.classifier.embedding_intent_classifier.EmbeddingIntentClassifier 似乎还没有很好的处理 multiple intents 的问题.

代码中会根据 intent_split_symbol 将 intent 拆分为子意图, 进一步根据 one-hot 表示得到 intent 的表示. 但在整个预测过程当中还是只针对所有出现过的意图 (组合的意图) 来计算相似度, 给出 intent_ranking.

比如说, 如果训练样本中只出现了意图 intentA_intentB, 没有单独出现 intentA 和 intentB. 那么, 在预测过程当中, 只会计算当前 utterence 和 intentA_intentB 的相似度, 而不会单独计算 utterence 与 intentA 和 intentB 的相似度. 从而最后输出的 intent_rank 中不会有 intentA 或 intentB 的排名. 这可能并不是我想要的 multiple intents.

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1709.03856">论文原文 StarSpace: Embed All The Things</a></p></li>
<li><p><a href="https://github.com/facebookresearch/StarSpace">github 代码地址: </a></p></li>
<li><p><a href="https://github.com/RasaHQ/rasa_nlu/blob/0.12.3/rasa_nlu/classifiers/embedding_intent_classifier.py#L37">rasa_nlu 中 EmbeddingIntentClassifier 实现代码</a></p></li>
</ul>