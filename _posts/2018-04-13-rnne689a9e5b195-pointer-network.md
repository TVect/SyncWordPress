---
ID: 165
post_title: Pointer Network
author: chin340823
post_excerpt: ""
layout: post
permalink: 'http://www.tvect.cc/2018/04/13/rnn%e6%89%a9%e5%b1%95-pointer-network/'
published: true
post_date: 2018-04-13 14:22:58
---
<h1>概述</h1>

在传统的seq2seq的方法中，输出部分的字典长度需要提前固定住，比如在NMT中，decoder部分的字典大小是固定的。

所以，这些传统的方法不能直接用于输出字典依赖于输入序列长度的情况。

本文的主要贡献包括：

<ol>
<li>提出了简单有效的Pointer Network结构，可以处理可变长度字典的情况。</li>
<li>将Pointer Network应用到了三个问题上，学到的模型可以很好的扩展到test phrase数据点比train phrase 阶段数据点要多的情况。、</li>
<li>Pointer Network的模型可以学到 a competitive small scale TSP approximate solver，表明纯粹的数据驱动的方法可以在computationally intractable的问题上得到近似解。</li>
</ol>

<h1>模型</h1>

<img src="http://www.tvect.cc/wp-content/uploads/2018/04/ptrnet-01.png" alt="" />

<h2>Attention vs. Pointer Network</h2>

<h3>Content Based Input Attention</h3>

传统的 attention based seq2seq的基本公式如下：

<img src="http://www.tvect.cc/wp-content/uploads/2018/04/attention-01.png" alt="" />

<h3>Ptr-Net</h3>

前面的attention机制中，我们用 softmax(u) 对 encoder state 进行加权，为decoder阶段产生extra information。而在 ptr-net 中，我们使用 softmax(u) 作为指向输入元素的指针，输出直接拷贝相应的输入元素。

具体公式如下：

<img src="http://www.tvect.cc/wp-content/uploads/2018/04/ptrnet-02.png" alt="" />

<h1>Examples</h1>

<img src="http://www.tvect.cc/wp-content/uploads/2018/04/examples.png" alt="" />

<ul>
<li><strong>convex hulls</strong>
<img src="http://www.tvect.cc/wp-content/uploads/2018/04/convex-hulls-01.png" alt="" /></p></li>
<li><p><strong>Delaunay triangulations</strong></p></li>
<li><p><strong>Travelling Salesman Problem (TSP)</strong>
<img src="http://www.tvect.cc/wp-content/uploads/2018/04/tsp-01.png" alt="" /></p></li>
</ul>

<h1>Pointer Network in NLP</h1>

<h2>Summarization：</h2>

<p>同时结合了传统的attention based seq2seq 和 pointer network，进行加权之后生成最后的summary。

<img src="http://www.tvect.cc/wp-content/uploads/2018/04/ptr-summary.png" alt="" />

<h1>参考资料</h1>

<a href="https://arxiv.org/abs/1506.03134" title="Pointer Network">Pointer Network</a>

<a href="https://arxiv.org/abs/1704.04368" title="Get To The">Get To The Point: Summarization with Pointer-Generator Networks </a>