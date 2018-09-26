---
ID: 753
post_title: >
  A Network-based End-to-End Trainable
  Task-oriented Dialogue System
  阅读笔记
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://blog.tvect.cc/2018/09/26/a-network-based-end-to-end-trainable-task-oriented-dialogue-system-%e9%98%85%e8%af%bb%e7%ac%94%e8%ae%b0/'
published: true
post_date: 2018-09-26 17:54:19
---
[toc]

<!--more-->

笔记梗概如下，待完善细节

<h1>概述</h1>

数据: delexicalised. 每个词属于 a word, a delexicalised slot name or a delexicalisedslot value

<h1>主要部件</h1>

<h2>Intent Network</h2>

LSTM or CNN

<h2>Belief Trackers</h2>

目标：
1. maintain a multinomial distribution for each informable slot.
2. maintain a binary distribution for each requestable slot.

每个 slot 有一个 tracker, 每个 tracker 是一个 Jordan RNN, tracker 的输入特征是通过对当前的 user input 和前一步的 machine response 分别做 CNN 得到的。

<strong>FAQ</strong>:
- Vs是什么？
    1. 是前面出现过的所有可能的slot value
    2. 还是KB里面已经订好的slot value

<pre><code>之前DSTC中用的是1.
这里如果是1会导致输出维度不一样，在后面的policy net中似乎不好做输入？
</code></pre>

<h2>Database Operator</h2>

对每个 slot, 根据 belief tracker 的结构取其最可能的取值 value, 用这个 value 去进行数据库查询, 构造出一个 DB entities 上的 0-1 真值向量。

<blockquote>
  This query is then applied to the DB to create a binary truth value vector xt over DB entities where a 1 indicates that the corresponding entity is consistent with the query (and hence it is consistent with the most likely belief state)
</blockquote>

在真值向量非空的时候, 会维护一个 pointer, 指向 匹配的实体中 随机的一个。如果当前指向的实体不再满足匹配条件，就更新 pointer, 反之保持 pointer 不变.

pointer 指向的实体会用来构造最终的系统 response.

<h2>Policy Network</h2>

输出为一个表示 system action 的向量.
输入由三部分处理得到：Intent Network 输出的向量表示, Belief Tracker 输出的 belief state, Database Operator 输出的真值向量.

<ul>
<li>belief state 处理</li>
</ul>

因为 generation net 只关心产生合适的句子形式, 所以slot中特定value的个体概率大小并不重要, 所以可以构造一个 summary belief vector. 它包含三部分: the summed value probabilities, the probability that the user said they "don’t care" about this slot and the probability that the slot has not been mentioned.

<strong>FAQ:</strong> the summed value probabilities 和 the probability that the slot has not been mentioned 不会重叠了吗?

<ul>
<li>true vector 处理</li>
</ul>

基于和前面同样的原因, 只需要关系匹配的实体个数, 不需要关心具体的取值. 所以可以 true vector 压缩为 6维的 one-hot 表示, 分别表示(no match, 1 match, ... , more than 5 matches)

<h2>Generation Network</h2>

Condition on Policy Network 的输出, 用 language model 做 language generation, 产生 template-like sentences.

最句子模板产生之后, 可以将 the generic tokens 替换为对应的 actual values. 包含替换 delexicalised slots, 替换 delexicalised values

在 generation network 中也可以引入attention机制, 形成 <strong>Attentive Generation Network</strong>

<h1>Wizard-of-Oz Data Collection</h1>

为了处理 task-oriented dialog system 对 in-domain data 需求的问题, 文中提出了一个新颖的众包版本的 the Wizard-of-Oz (WOZ) paradigm, 用来收集 domain-specific corpora

user和wizard只要完成单轮对话，完成之后会要求对其他的对话来完成又一轮。

user 需要根据任务需求和对话历史输入一个合适的sentence

wizard 需要根据对话信息填写 belief state 信息表格, 填写完表格, system 会给出符合查询条件的entity. 基于会话历史信息和system query的实体结果, wizard 需要输入一个合适的 response.

<h1>Empirical Experiments</h1>

<h2>Training</h2>

分为两步:

<ol>
<li>首先, 训练 Belief Tracker, 使用交叉熵损失函数.</p></li>
<li><p>固定 Belief Tracker, 训练模型其他的部分.

损失函数是对 最后generation network的输出 做 cross entropy 得到的.

<p>训练中使用了 early stopping, gradient clipping</p></li>
</ol>

<h2>Decoding</h2>

<ul>
<li>decoding using the average likelihood term</li>
<li>weighted decoding strategy</li>
</ul>

<h2>Evaluation</h2>

<ul>
<li>Tracker performance</li>
<li>Corpus-based evaluation</li>
<li>Human evaluation</li>
</ul>

<h1>番外</h1>

<h2>A Convolutional Neural Network for Modelling Sentences</h2>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://ziyaochen.github.io/2018/07/13/NDM/">文献阅读笔记-博客</a></p></li>
<li><p><a href="https://github.com/shawnwun/NNDIAL">github上源代码</a></p></li>
<li><p><a href="http://www.aclweb.org/anthology/W/W14/W14-4340.pdf">Word-Based Dialog State Tracking with Recurrent Neural Networks</a></p></li>
</ul>