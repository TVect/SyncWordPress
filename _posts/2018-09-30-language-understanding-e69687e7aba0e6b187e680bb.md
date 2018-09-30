---
ID: 776
post_title: >
  Language Understanding 文章汇总 –
  pipeline
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/776
published: true
post_date: 2018-09-30 07:44:54
---
[toc]

初步整理的 Language Understanding 的资料, 细节图片等待进一步补充.

<!--more-->

<h1>Pipeline</h1>

<h2>Domain/Intent Classification</h2>

<h3>Recurrent Neural Network and LSTM Models for Lexical Utterance Classification</h3>

普通的使用 LSTM 做分类

文章中对 unknown word 做了一些处理, 使用 <strong>word hash (subword? character?)</strong>

<h3>Sequential Short-Text Classification with RNN and CNN</h3>

<ol>
<li>使用 LSTM+Pooling 或者是 CNN+Pooling 获取短文本的表示</li>
<li>文中说的是两层的ANN做分类,也可以从卷积的角度来看.

即做了两层的因果卷积,第一层的因果卷积filter size为 d1+1,

第二层因果卷积的filter size为 d2+1</p></li>
</ol>

<h2>Slot Filling</h2>

<h3>Leveraging Sentence-level Information with Encoder LSTM for Semantic Slot Filling</h3>

<p>考虑了整个句子的信息来做slot filling.

encoder-labeler LSTM 先用 encoder lstm 把整个句子编码为一个向量, 再用这个向量作为另一个用作标注的 lstm 的 initial state. 从而在做 labeling 的时候可以考虑到整个句子的信息.

<ul>
<li><strong>labeler LSTM(W)</strong>

简单的 LSTM+Softmax，考虑了输入句子中word前后上下文关系</p></li>
<li><p><strong>labeler LSTM (W+L)</strong>

在简单的 LSTM+Softmax基础上加上了输出标签到下一个状态输入的连接，显式的考虑了前后label之前的依赖关系. 最后使用 BeamSearch 做解码</p></li>
<li><p><strong>encoder-labeler LSTM</strong>

包括 <strong>encoder-labeler LSTM(W)</strong> 和 <strong>encoder-labeler LSTM(W+L)</strong> 两种形式, 分别对应着前面两个labeler LSTM 对应的变种. 差别在于这里使用了encoder的输出作为labeler LSTM的initial state. 这意味着 labeler LSTM 会参考整个句子的信息做标注.

与 encoder-decoder 模型相比, encoder-labeler LSTM 将同一个句子输入了两次.

另外, 在 encoder-labeler LSTM 中 encoder LSTM 和 labeler LSTM 是两个不同的网络, 但是它们使用的是用一个 embedding matrix.</p></li>
</ul>

<h3>Neural Models for Sequence Chunking</h3>

<p>Semantic slot filling 之类的任务常常被当做序列标注问题来处理.

当前很多的处理这种问题的 DNN 方法是基于 IOB 标注的. 但是这些做法会有以下一些问题:

<ol>
<li>we don’t have an explicit model to learn and identify the scope of chunks in a sentence, instead we infer them implicitly (by IOB labels).

Hence the learned model might not be able to fully utilize the training data which could result in poor performance.</p></li>
<li><p>some neural networks like RNN or LSTM have the ability to encode context information but don’t treat each
chunk as a complete unit</p></li>
</ol>

<p>本文使用 Neural Sequence chunking 的方法来处理上述的两个缺陷, 先做 Segmentation, 显式的确定 chunk 的范围, 再把 chunk 当做一个整体来做 Labeling.

<ul>
<li>MODEL 1

IOB labels 做 segmentation

先用 BiLSTM 做序列标注, 为每个 word 一个 IOB 的标签, 从而可以得到句子中的 chunk.

用 chunk 中 word 对应的 hidden state 做平均, 得到 chunk 的表示. 基于 chunk 的表示, 在 chunk 级别上预测 chunk label.</p></li>
<li><p>MODEL 2

IOB labels 做 segmentation

使用了 encoder-decoder 的结构, encoder 部分会预测 IOB 标签, decoder 部分会预测 chunk label.

在做了 encoder 之后, 可以得到句子的 chunk. decoder 部分使用 chunk 作为输入, 而非 word.

<ol>
<li>使用 encoder LSTM 输出的[hT, h1]作为 decoder LSTM 的 initial state</p></li>
<li><p>decoder LSTM 每个时间步的输入由三部分组成:

Cx 由 Chunk 内的 word 做 CNNMax 得到

Cy 由 Chunk 内的 word 对应的 encoder 隐状态做 average 得到

Cw 由 context word embeddings of the chunk 构成.</p></li>
</ol></li>
<li><p>MODEL 3

pointer network 做 segmentation

与 MODEL 2 类似, 区别在于这里使用了 pointer network 来识别 chunk, 而不是用 IOB 来识别 chunk.

MODEL 3 中会贪婪的做 segmentation 和 labeling. 会先结合 encoder hidden state, the ending point candidate word embedding, current beginning word embedding, decoder hidden state, 以及 chunk length embedding 给出 endpoint candidate 作为 chunk 终止字符的概率.

后面的 decoder hidden state 更新, 和 chunk labeling 和 MODEL 2 类似.</p></li>
</ul>

<p>损失函数为 segmentation and labeling 两部分的 crossentropy 之和.

试验在 text chunking 和 slot filling 上都有 state-of-the-art 的效果, 试验结果还说明了使用 Pointer network 比使用 IOB 效果要好.

<h2>Multi-task</h2>