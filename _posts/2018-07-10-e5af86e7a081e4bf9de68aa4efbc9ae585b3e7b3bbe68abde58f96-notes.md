---
ID: 45
post_title: 密码保护：关系抽取 notes
author: chin340823
post_excerpt: ""
layout: post
permalink: https://blog.tvect.cn/?p=45
published: true
post_date: 2018-07-10 22:30:05
---
[toc]

<!--more-->

<hr />

<h1>概述</h1>

<h1>基本方法</h1>

<h2>Pipeline 方法</h2>

<h3><a href="https://arxiv.org/pdf/1504.06580.pdf">Classifying Relations by Ranking with CNN</a></h3>

word embedding + position embedding + CNN --> seq * n_dim
在seq这一维度上做 max pooling, 与每一个label的表示做内积, 使用margin based的ranking-loss做损失函数。

<h3><a href="http://www.aclweb.org/anthology/P/P16/P16-2034.pdf">Attention-Based BiLSTM for Relation Classification</a></h3>

word embedding + position indicator作为输入，使用bilstm + attention得到sentence的表示，用softmax cross entropy做损失函数

<h3><a href="http://www.thunlp.org/~lzy/publications/acl2016_cnnatt.pdf">Relation Classification via Multi-Level Attention CNNs</a></h3>

word embedding + position embedding 作为初始的 input
求每一个entity与每个word的attention得分，再由word的两个attention得分得到新的word 表示
对前面的输出R, 使用att pooling取代max pooling:引入一个weighting matrix U,求出一个 word<em>label 的 att matrix Ap.最终的输出向量为wo=max(R</em>Ap).
使用的损失函数为: margin-based pairwise loss

<h2>Joint Model 方法</h2>

<h3><a href="https://arxiv.org/pdf/1601.00770v1.pdf">End-to-End Relation Extraction using LSTMs on Sequences and Tree Structures</a></h3>

<h2>Distant Supervision 方法</h2>

<h3>Distant Supervision for Relation Extraction with Sentence-level Attention and Entity Descriptions</h3>

<h1>参考资料</h1>

<ul>
<li>Pipeline 方法

<ul>
<li><a href="https://arxiv.org/pdf/1504.06580.pdf">Classifying Relations by Ranking with CNN</a></p></li>
<li><p><a href="http://www.aclweb.org/anthology/P/P16/P16-2034.pdf">Attention-Based BiLSTM for Relation Classification</a></p></li>
<li><p><a href="http://www.thunlp.org/~lzy/publications/acl2016_cnnatt.pdf">Relation Classification via Multi-Level Attention CNNs</a></p></li>
</ul></li>
<li><p>Joint Model 方法

<ul>
<li><a href="https://arxiv.org/pdf/1601.00770v1.pdf">End-to-End Relation Extraction using LSTMs on Sequences and Tree Structures</a></li>
</ul></li>
<li>Distant Supervision 方法

<ul>
<li><a href="http://www.nlpr.ia.ac.cn/cip/~liukang/liukangPageFile/AAAI2017.pdf">Distant Supervision for Relation Extraction with Sentence-level Attention and Entity Descriptions</a></li>
</ul></li>
</ul>