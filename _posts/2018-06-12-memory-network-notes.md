---
ID: 322
post_title: Memory Network notes
author: Chin
post_excerpt: ""
layout: post
permalink: >
  http://blog.tvect.cc/2018/06/12/memory-network-notes/
published: true
post_date: 2018-06-12 17:24:11
---
<h1>概述</h1>

大部分的机器学习方法缺乏一个可以和 inference 部分无缝衔接的 简单的 可读写的 长时记忆模块。

比如，在传统的 RNN 中，使用 hidden states 等作为他们的记忆功能，但是这种方法产生的记忆太小了，它把信息压缩到了 dense vectors 当中, 无法精确的表达过去的信息。

<h1>Memory Network 的基本结构</h1>

memory network 由 memory m (an array of objects indexed by $$m_{i}$$) 和 四个单元 I(input feature map), G(generalization), O(output feature map), R(response) 组成：

<ul>
<li><strong>I: (input feature map)</strong> 把输入变成内部的特征表示
这一部分可以使用标准的预处理流程, 比如 parsing, coreference and entity resolution. 最后将输入编码成特征向量表示。</p></li>
<li><p><strong>G: (generalization)</strong> 根据输入更新 memory
这一部分最简单的做法就是直接把 $$I(x)$$ 放在一个 memory slot 里面： $$m_{H_{x}} = I(x)$$. 其中 $$H(.)$$ 是一个用于选择 slot 的函数。
有一些更复杂的 $$G$$ 的变种, 可以去更新早先已经保存的记忆。
如果 memory 很大, 需要重新去组织 memories, 这可以通过 slot choosing function 来实现。比如说, 可以通过设计或者训练, 让 $$H(x)$$ 根据 entity 或者 topic 去选择合适的 slot 去更新。
如果 memory 满了, 可以设计一种遗忘的机制。</p></li>
<li><p><strong>O: (output feature map)</strong> 根据新的输入和当前 memory 状态, 产生新的 output feature</p></li>
<li><p><strong>R: (response)</strong> 根据 output feature 产生需要的 response format</p></li>
</ul>

<p>模型结构图如下：
<img src="http://www.tvect.cc/wp-content/uploads/2018/06/memory-network-1024x494.png" alt="" />

<h2>A MemNN implementation for text</h2>

作者在文章中提出了一种最基本的 Memory Network 实现方式。

basic model的 I 就是一个简单的embedding lookup操作，也就是将原始文本转化为词向量的形式，而 G 模块则是直接将输入的向量存储在 memory 数组的下一个位置，不做其他操作，也就是直接写入新的记忆，对老的记忆不做修改。

主要的工作在O和R两个模块进行。 O模块根据输入的问题向量在所有的记忆中选择出topk相关的记忆，具体选择方式为，先选记忆中最相关的memory, 再根据选出的记忆继续选择一个最相关的 memory:

$$ o_{1} = O_{1}(x, m) = argmax \ S_{o}(x, m_{i})$$
$$o_{2} = O_{2}(x, m) = argmax \ S_{o}([x, m_{o_{1}}], m_{i})$$

选择出最相关的 top k 个memory slot 之后。将其作为R模块的输入，用于生成最终的答案。这里简单就是使用与上面相同的评分函数计算所有候选词与R输入的相关性，得分最高的词语就作为正确答案输出即可：
$$
r = argmax_{w \\in W} \ S_{R}([x, m_{o1}, m_{o2}], w)
$$

对于需要产生一个句子输出的情况下，也可以在 R 模块使用 RNN LM.

前面使用到的评分函数的形式为:
$$
s(x, y) = \phi_{x}(x)^{T} U^{T} U \phi_{y}(y)
$$

<h3>损失函数</h3>

最终的损失函数选为：
<img src="http://www.tvect.cc/wp-content/uploads/2018/06/memnet-loss-1024x226.jpg" alt="" />

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-null">(6) 有没有挑选出正确的第一句话
(7) 正确挑选出了第一句话后能不能正确挑出第二句话
(6)+(7) 合起来就是能不能挑选出正确的语境，用来训练 attention 参数
(8) 把正确的 supporting fact 作为输入，能不能挑选出正确的答案，来训练 response 参数
</code></pre>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1410.3916" title="Memory Networks">Memory Networks</a></p></li>
<li><p><a href="https://zhuanlan.zhihu.com/p/32257642" title="论文笔记 - Memory Networks 系列">论文笔记 - Memory Networks 系列</a></p></li>
<li><p><a href="https://zhuanlan.zhihu.com/p/29590286" title="记忆网络之Memory Networks">记忆网络之Memory Networks</a></p></li>
</ul>