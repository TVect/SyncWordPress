---
ID: 53
post_title: >
  Reasoning About Entailment With Neural
  Attention 阅读笔记
author: chin340823
post_excerpt: ""
layout: post
permalink: 'http://www.tvect.cc/2017/12/04/reasoning-about-entailment-with-neural-attention-%e9%98%85%e8%af%bb%e7%ac%94%e8%ae%b0/'
published: true
post_date: 2017-12-04 22:23:59
---
[toc]

<h1>概述</h1>

It is the first generic end-to-end differentiable system that achieves state-of-the-art accuracy on a textual entailment dataset

<h2>Contributions</h2>

<ol>
<li>提出了基于 LSTM 的神经网络模型，一次读入两个句子确定导致关系，而不是将每个句子直接独立地嵌入到语义空间中；</li>
<li>用 word-word 注意力机制来扩展模型</li>
<li>给出了关于 RTE 任务的神经注意力的详细的定性分析</li>
</ol>

<img src="https://note.youdao.com/yws/public/resource/c495aa3a55f2207889f98c9bd5ce0e28/xmlnote/B54D50F715824890A55B688E8B0A27ED/3636" alt="网络模型" title="网络模型" />

<h1>模型</h1>

<h2>1. 基本模型A: 使用LSTM处理RTE问题</h2>

在之前的工作中，Bowman使用LSTM独立的编码premise和hypothesis，再将它们concat，送到MLP做分类。这表明，LSTM可以学到适合做RTE的句子表示。

与Bowman不同，本文使用的是<strong>Conditional Encoding</strong>。

premise输入到左边的LSTM中，hypothesis输入到右边的LSTM中(和前一个LSTM参数不共享)，这个LSTM的memory state使用左边的LSTM的最后一个cell state做初始化。

也就是说，hypothesis LSTM is conditioned on premise LSTM.

最后加上softmax层做三分类(Entailment, Neutral, Contradiction)

<h2>2. 模型B: LSTM with neural attention</h2>

这个模型的想法是将hypothesis的句子表示与premise建立注意力机制，而不是将hypothesis的每个单词都与premise做alignment。

从上图中标记B的地方也可以看出，attention仅仅依赖于hypothesis的last hidden state。

<img src="https://note.youdao.com/yws/public/resource/c495aa3a55f2207889f98c9bd5ce0e28/xmlnote/9DFFE5F3877A4C4991CFEF9818EE8513/4276" alt="模型B-公式" title="模型B-公式" />

<h2>3. 模型C: LSTM with word-by-word attention</h2>

模型对hypothesis和premise每个单词做了alignment，所以这里称为word-by-word attention。

从模型架构图中标记C的地方也可以看出，hypothesis中的每个词都与premise中对应的词进行了alignment。

<img src="https://note.youdao.com/yws/public/resource/c495aa3a55f2207889f98c9bd5ce0e28/xmlnote/742D9C1A09B64E80A6D814818235FC38/4279" alt="模型C-公式" title="模型C-公式" />

<h2>4. LSTM with two-way attention</h2>

使用两个相同的LSTM(结构和权重都相同)，一个基于premise对hypothesis进行表示，另一个基于hypothesis对premise进行表示(仅仅交换了hypothesis和premise的位置)，将得到的结果做拼接用于分类。

<h1>试验结果</h1>

<img src="https://note.youdao.com/yws/public/resource/c495aa3a55f2207889f98c9bd5ce0e28/xmlnote/5715D52B3DA64791A519597E568AADE9/4319" alt="实验结果" title="实验结果" />

<h2>attention</h2>

<img src="https://note.youdao.com/yws/public/resource/c495aa3a55f2207889f98c9bd5ce0e28/xmlnote/217F4A5B29314F09967383ED5101FF33/4324" alt="attention结果" title="attention结果" />

<h2>Word-by-word Attention</h2>

<img src="https://note.youdao.com/yws/public/resource/c495aa3a55f2207889f98c9bd5ce0e28/xmlnote/894359F3DF17415191477DE0CAFBCC4B/4326" alt="wbw结果" title="wbw结果" />

<h1>参考资料</h1>

<ul>
<li><p>代码尝试
<a href="https://github.com/shyamupa/snli-entailment.git" title="https://github.com/shyamupa/snli-entailment.git">https://github.com/shyamupa/snli-entailment.git</a>
严重依赖于初始化, 经常会落入局部极小, 有时候能跳出局部极小值, 达到比较好的解。</p></li>
<li><p>原始论文
<a href="http://arxiv.org/pdf/1509.06664" title="http://arxiv.org/pdf/1509.06664">http://arxiv.org/pdf/1509.06664</a></p></li>
<li><p>参考博客
<a href="http://www.jianshu.com/p/8799afd9b7c7" title="http://www.jianshu.com/p/8799afd9b7c7">http://www.jianshu.com/p/8799afd9b7c7</a>
<a href="https://zhuanlan.zhihu.com/p/21292638" title="https://zhuanlan.zhihu.com/p/21292638">https://zhuanlan.zhihu.com/p/21292638</a></p></li>
</ul>