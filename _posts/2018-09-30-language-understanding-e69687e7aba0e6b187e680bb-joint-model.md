---
ID: 71
post_title: 'Language Understanding 文章汇总 &#8211; joint model'
author: chin340823
post_excerpt: ""
layout: post
permalink: https://blog.tvect.cn/?p=71
published: true
post_date: 2018-09-30 18:02:51
---
[toc]

<!--more-->

<h1>Joint-Model</h1>

<h2>Attention-Based RNN for Joint Intent Detection and Slot Filling</h2>

<h2>Multi-Domain Joint Semantic Frame Parsing using Bi-directional RNN-LSTM</h2>

为了联合的建模domain, intent,slots, 这里在每句话后面加上了<EOS>,并为它赋予标签domain-intent

文章中比较了四种设定下的试验结果:

<ol>
<li>SD-Sep</li>
</ol>

For each domain, a separate intent detection and slot filling model was trained, resulting in 2 × jDj classifiers, where jDj is the number of domains. Optimum parameters were found on the development set for each experiment and used for computing performance on the test set. The output of all the classifiers were joined for overall error rates.

<ol start="2">
<li>SD-Joint</li>
</ol>

For each domain, a single model that estimates both intent and sequence of slots was used, resulting in jDj classifiers.

<ol start="3">
<li>MD-Sep</li>
</ol>

An intent detection model and a slot filling model were trained using data from all the domains, resulting in 2 classifiers. The output of intent detection was merged with the output of slot filling for computing overall template error rates.

<ol start="4">
<li>MD-Joint:</li>
</ol>

A single classifier for estimating the full semantic frame that includes domain, intent, and slots for each utterance was trained using all the data

结果：

In both single-domain and multi-domain settings, intent detection accuracy improves with joint training (although small), but slot filling degrades.

<h2>Slot-Gated Modeling for Joint Slot Filling and Intent Prediction</h2>

之前的模型在做joint model的时候, slot filling 和 intent prediction 部分会有着独立的权重, 本文考虑到 slot 和 intent 之间有很强的关联性, 引入了 slot gate 机制, 旨在显式的建模 slots 和 intent vectors之间的关系.

<h1>Contextual LU</h1>

<h2>Context Sensitive Spoken Language Understanding Using Role Dependent LSTM Layers (Hori+ 2015)</h2>

<ol>
<li>建模了 dialog context.

每个 utterance 使用 LSTM 建模, 前一个 utterance 经过 LSTM 之后的隐状态会当做下一个 utterance 输入 LSTM 时的初始化状态.</p></li>
<li><p>使用两个不同的 LSTM 来分别建模client utterance 和 agent utterance.

但是在role转换的时候前一个 LSTM 的隐状态还是会用作后一个 LSTM 的初始化.</p></li>
</ol>

<h2>End-to-End Memory Networks with Knowledge Carryover for Multi-Turn Spoken Language Understanding (Chen+, 2016)</h2>

<p>借鉴 end-to-end memory network 的想法, 用 slot tagging 上 (拼接intent 和slot作为标签??)