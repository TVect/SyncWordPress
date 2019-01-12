---
ID: 75
post_title: >
  Language Understanding 文章汇总 –
  Contextual LU
author: chin340823
post_excerpt: ""
layout: post
permalink: https://blog.tvect.cn/?p=75
published: true
post_date: 2018-10-08 22:03:53
---
[toc]

草稿版本，后续待整理

<!--more-->

<h2>Context Sensitive SLU Using Role Dependent LSTM Layers (Hori+ 2015)</h2>

<ol>
<li>建模了 dialog context.

每个 utterance 使用 LSTM 建模, 前一个 utterance 经过 LSTM 之后的隐状态会当做下一个 utterance 输入 LSTM 时的初始化状态.</p></li>
<li><p>使用两个不同的 LSTM 来分别建模client utterance 和 agent utterance.

但是在role转换的时候前一个 LSTM 的隐状态还是会用作后一个 LSTM 的初始化.</p></li>
</ol>

<h2>End-to-End Memory Networks with Knowledge Carryover for Multi-Turn SLU (Chen+, 2016)</h2>

<p>借鉴 end-to-end memory network 的想法, 用 slot tagging 上 (拼接intent 和slot作为标签??)

<h2>Sequential Dialogue Context Modeling for SLU</h2>

Sequential Dialogue Encoder Network

<pre><code>1. 用 BiGRU_m 得到 context utterance 的向量表示 m1, m2, ..., m_{t-1}

2. 用 BiGRU_c 得到 current utterance 的向量表示 c

3. 每个 m_i 和 c 拼接后丢到 FF 中, 得到向量表示 g_i

4. 用 BiGRU_s 处理 g_1, ..., g_{t-1}, 得到最终的 dialogue context encoding.
</code></pre>

相比于 End-to-End Memory Networks, 这里的做法似乎能比较好的建模 context utterance 中的前后时序关系.

<h2>How Time Matters: Learning Time-Decay Attention for Contextual SLU in Dialogues</h2>

two attention types (content-aware and time-aware)

two attention levels (sentence-level and role-level)

和下面哪一篇是同一批人做的，结构类似。这一篇主要关注在探索各种不同的 time-decay attention function，最后提出一个 universal time-decay attention function. 
下面一篇主要是learning flexible contextsensitive time-decay functions in an end-to-end fashion.

<img src="https://note.youdao.com/yws/public/resource/c7323f66627543384b1ad855b31c56a1/xmlnote/83D5E9ADFF594F3B94BD162D2C11EC47/14115" alt="" />

基本流程公式如下:

<img src="https://note.youdao.com/yws/public/resource/c7323f66627543384b1ad855b31c56a1/xmlnote/2618C76EBE4549BC8065B88ACE9B8336/14113" alt="" />

上面的公式（3）中给出了三种 history summary vector 的公式,  包括原始形式，sentence-attention形式，和role-attention的形式.

另外，公式（3）中输入 x 是 intent 和 attribute 的 one-hot 编码,并不是原始的句子。所以模型需要requires the ground truth annotations for history utterances for training and testing.
在sentence-level attention 当中, 每个history vector在喂到模型之前，先要乘以它相应的时间感知权重系数。

在role-level attention当中, 每个 role vector 的对应系数权重为 当前role对应的history utterences中的 maximum attention value.

时间感知的attention机制
<img src="https://note.youdao.com/yws/public/resource/c7323f66627543384b1ad855b31c56a1/xmlnote/D4EAE52929D14350A343CC6CB1FD6921/14114" alt="" />

上面分别给出了Convex Time-Decay Attention，Linear Time-Decay Attention，Concave Time-Decay Attention，以及Universal Time-Decay Attention。
注意，所有这些 attention weights 最终都要进行一下归一化操作。

<h2>Learning Context-sensitive time-decay attention for role-based dialogue modeling</h2>

<img src="https://note.youdao.com/yws/public/resource/c7323f66627543384b1ad855b31c56a1/xmlnote/A7B088DA75294164A00D5FC6C4F814D4/14121" alt="" />

<h3>Role-Based Context-Sensitive Attention</h3>

在前文 time-decay attention 的基础上，进一步 encode context-sensitive characteristics into the attention mechanisms。
<img src="https://note.youdao.com/yws/public/resource/c7323f66627543384b1ad855b31c56a1/xmlnote/878D36051FFC42BD82FCAD1C38984429/14125" alt="" />

将 speaker-specific contextual encoding 和 the feature of the current utterance 一起丢到一个全连接层，预测 universal time-decay function 中的一些参数。

注意其中某些输出要当做指数幂，为了确保为正数，比如 D0。（12）式为引入的两个 constraint，第一项保证输出尽量是正的，第二项保证输出值绝对值尽量要小。

整个过程同样的也是 end-to-end 的训练。

这两篇文章中有很多 content attention 和 time-decay attention的对比结合等，但是 content attention是怎么做的，怎么结合的貌似没怎么看的清楚 ？？？？？？