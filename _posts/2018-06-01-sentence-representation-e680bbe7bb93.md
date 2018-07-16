---
ID: 299
post_title: Sentence Representation 总结
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://blog.tvect.cc/2018/06/01/sentence-representation-%e6%80%bb%e7%bb%93/'
published: true
post_date: 2018-06-01 11:49:21
---
<h1>概述</h1>

提出了一个新颖的方法，基于会话数据无监督的学习句子级别的语义相似性。直观的说，如果句子的回答分布相似，则它们在语义上是相似的。

文章中考虑了:
1. 单独从 unsupervised conversational input-response pairs 学习到的句子向量表示, 在相关的评测任务上表现良好。
2. 将 unsupervised conversational input-response prediction 和 supervised training on Natural Language Inference (NLI) data 一起进行 multi-task training, 得到的句子向量表示, 效果会有进一步提升。

<h1>Approach</h1>

<h2>Conversational Learning Task using Input-Response Prediction</h2>

具体来说, 这里的 会话预测任务 从要一个正确的 response 和 K-1 个随机采样的responses中 判别出正确的response, 即是要建模 $$P(y|x)$$

<h2>模型结构</h2>

如下图所示, 模型将 input sentence 和 response sentence 编码成固定长度的向量 u 和 v, 再计算内积和 softmax。通过最大化正确的 response 的 loglikelihood 进行参数学习。

注意到, 为了 model the differences in meaning between inputs and responses, 在 response 一侧得到的表示在和 input 的表示做内积之前，经过了附加的一个 FF network。
当然，在input encoder 这一侧也可以加一个 FF network, 但文章中提到, 早期的试验表明只在一个encoder 之后加上FF network, 就已经足够。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/06/InputResponseEncoderModel-00-1024x718.png" alt="" />

在具体的做sentence embedding部分，即图中的encoder部分, 文章中尝试了如下两种结构。

<h3>deep averaging network (DAN)</h3>

DAN 的基本做法是将 word-level embeddings 做平均, 再丢到一个 DNN 里面。

<strong>缺少时序信息？？？</strong>

在文章中, 考虑的是将 word 和 bigram 一起去做编码, word 和 bigram 的 embedding 在训练中进行学习。
另外, 做平均的时候, 是将 input embedding 求和之后会除以 sqrt(n), 其中 n 是序列的长度。这样做的目的是：

<blockquote>
  We want our input embeddings to be sensitive to length. However, we also want to ensure that for short sequences the relative differences in the representations are not dominated by sentence length effects
</blockquote>

<h3>Transformer</h3>

Transformer 结构来源于 google 2017 年的 《Attention is all you need.》

原始的 Transformer 结构中有 encoder 和 decoder 部分, 这里只用到了 encoder 部分。

因为 Transformer encoder 的输出也是一个变长的序列, 这里也通过做平均的方式得到固定长度的表示。

<blockquote>
  Intuitively, this is similar to building a bag-of-words representation, except that the words have had a chance to interact with their contexts through the attention operations in the encoding layers
</blockquote>

文章 《Universal Sentence Encoder》中也提到了Transformer做平均的方法，具体是求和之后除以句子长度的平方根，类似于 DAN 的做法。

Transformer细节可以参考 <a href="http://blog.tvect.cc/2018/05/15/attention-is-all-you-need/">Attention is all you need 笔记</a>

<h3>两种 Encoder 结构的对比</h3>

文章 《Universal Sentence Encoder》也是使用 DAN 和 Transformer 结构做 sentence encoding。其中提到了两者在准确率和计算资源消耗方面的对比。

<ul>
<li>the transformer based encoder achieves the best overall transfer task performance. However, this comes at the cost of compute time and memory usage scaling dramatically with sentence length.</p></li>
<li><p>The primary advantage of the DAN encoder is that compute time is linear in the length of the input sequence. And the results demonstrate that DANs achieve strong baseline performance on text classification tasks.</p></li>
</ul>

<p>基本上，transformer 的准确率方面表现的更好, 而 DAN 在计算时间, 内存需求, 资源消耗方面更好。

<h2>Multitask Encoder</h2>

文章中有尝试多任务的模型。
考虑了 conversational input-response prediction 和 natural language inference (NLI), 共享这两个任务的 encoder。

图示如下：
<img src="http://blog.tvect.cc/wp-content/uploads/2018/06/SentRep-multitask-01-1.png" alt="" />

<h1>一些实验结果</h1>

<h2>Response Prediction 实验 和 SNLI 实验</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/06/expriment-01-1024x273.png" alt="" />

左图是只做 Response Prediction 任务的结果, 可以看到, 对于不同的 N, transformer encoder 表现的都比 DAN 要好。
之后展示的试验结果都是 transformer encoder 的结果, 因为其效果更好。

右图是利用 Response Prediction (Reddit dataset 数据) 和 SNLI 数据做 multitask learning 的实验。 结果展示的是 multitask model 和 一些 baseline modle 在 SNLI 分类上的效果。

multitask model 的准确率略低。可能是因为该模型与之前工作有以下两点不同:

<ul>
<li>multitask model 是从头开始学习所有的参数, 包括 word embedding 参数。而其他的一些使用了预训练的词向量。主要是因为 Reddit dataset 比较大, 并不是很必要使用预训练的词向量。</p></li>
<li><p>multitask model 同时学了两个任务, 可能会导致 balancing performance between them。</p></li>
</ul>

<p>但是, 在下面的 STS 上, multitask model 表现的会更好。文章中猜想, 可能是因为 Reddit data 扮演了一部分正则化的作用, 防止得到的句子表示对SNLI任务过拟合, 从而可以提升其他任务上的 transfer performance。

<h2>STS Benchmark</h2>

Semantic Textual Similarity (STS) benchmark 包含了三个类别(captions,news and forums)共8628个 sentence pairs。这些 sentence pairs 都有一个人工标注的, 取值从0-5的, 语义相似度得分。

试验中考虑了两种试验方案:
1. 取 multitask model 得到的 sentence representation 直接做内积评分, 算相似度。
2. 使用一个附加的 transformation matrix 对得到的 sentence representation 做变换之后, 再做内积评分, 算相似度。这个 transformation matrix 使用了 STS 的数据进行训练。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/06/expriment-03-1024x780.png" alt="" />

上图的 Table 3 是以上两种实验方案和一些 baselines 的结果对比, Table 4 是考虑上述方案在 STS 数据三个类别的效果。Figure 8 展示的是不同数量的 SNLI 数据，对 STS 评测效果的影响。

可以看到：
1. untuned Reddit model is competitive and outperforms many neural representation models. 从 Reddit model 得到的向量表示, 能够将语义上相似的句子映射到embedding space 也保持相近。
2. Adapted multitask model 在 STS 上基本获得了当前最好的效果, 和人工特征的模型有一比。
3. 从 Figure 8 可以看到, 即使只用 10% 的数据, 相关系数得分就有了很大提升, 在使用 40%+ 的监督数据之后, 评分曲线基本平坦了。

<h2>CQA Subtask B</h2>

这个任务当中, 给定原始问题Q, 和top 10 相关的问题, 目标是要根据相似度对这些相关问题进行排序。度量指标采用的是 Mean Average precision(MAP)。

和 STS experiments 一样，这里也使用了 cosine 相似度。这里并没有做任何 tuning, 得到的结果已经很好了。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/06/expriment-04-300x172.png" alt="" />

<h2>其他</h2>

《Universal Sentence Encoder》中也做了一系列实验，包括：

<ol>
<li>对 Transformer 和 DAN 做编码时准确率，计算时间，内存使用等情况进行了对比分析。</p></li>
<li><p>对 Transformer 和 DAN, 又分别考虑了 加上 word level transfer 与 不加 transfer learning 的情况。其中 word level transfer 尝试了使用 CNN 和 DAN 两种情况。</p></li>
</ol>

<blockquote>
  <p>The sentence level embeddings surpass the performance of transfer learning using word level embeddings alone. Models that make use of sentence and word level transfer achieve the best overall performance.
</blockquote>

<h1>参考资料</h1>

<ul>
<li><a href="http://www.sohu.com/a/232972682_129720">通用句子语义编码器，谷歌在语义文本相似性上的探索</a></p></li>
<li><p><a href="https://arxiv.org/abs/1804.07754">Learning Semantic Textual Similarity from Conversations</a>

<blockquote>
  Yinfei Yang, Steve Yuan, Daniel Cer, Sheng-yi Kong, Noah Constant, Petr Pilar, Heming Ge, Yun-Hsuan Sung, Brian Strope, Ray Kurzweil
</blockquote></li>
<li><a href="https://arxiv.org/abs/1803.11175">Universal Sentence Encoder</a>

<blockquote>
  Daniel Cer, Yinfei Yang, Sheng-yi Kong, Nan Hua, Nicole Limtiaco, Rhomni St. John, Noah Constant, Mario Guajardo-Cespedes, Steve Yuan, Chris Tar, Yun-Hsuan Sung, Brian Strope, Ray Kurzweil
</blockquote></li>
</ul>