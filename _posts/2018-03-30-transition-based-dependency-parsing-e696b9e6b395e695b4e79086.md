---
ID: 122
post_title: >
  Transition Based Dependency Parsing
  方法整理
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://blog.tvect.cc/2018/03/30/transition-based-dependency-parsing-%e6%96%b9%e6%b3%95%e6%95%b4%e7%90%86/'
published: true
post_date: 2018-03-30 13:50:12
---
[toc]

<!--more-->

<hr />

<h1>Dependency Parsing简介</h1>

将句子分析成一颗依存句法树，描述出各个词语之间的依存关系。

<h2>基本概念</h2>

<ul>
<li>依存树</li>
</ul>

<blockquote>
  a dependency tree is a directed graph that satisfies the following constraints:
  1. There is a single designated root node that has no incoming arcs.
  2. With the exception of the root node, each vertex has exactly one incoming arc.
  3. There is a unique path from the root node to each vertex in V.
</blockquote>

<ul>
<li>投射性：</li>
</ul>

<blockquote>
  An arc from a head to a dependent is said to be projective if there is a path from the head to every word that lies between the head and the dependent in the sentence.
  
  A dependency tree is then said to be projective if all the arcs that make it up are projective.
  
  A dependency tree is projective if it can be drawn with no crossing edges.
</blockquote>

<h1>两类基本的Dependency Parsing算法</h1>

<h2>Transition Based Method</h2>

<blockquote>
  motivated by a stack-based approach called shift-reduce parsing originally developed for analyzing programming languages
</blockquote>

<h3>transition system</h3>

<h4>arc standard transition system</h4>

<ul>
<li>LEFTARC
Assert a head-dependent relation between the word at the top of stack and the word directly beneath it; remove the lower word from the stack.</li>
<li>RIGHTARC
Assert a head-dependent relation between the second word on the stack and the word at the top; remove the word at the top of the stack;</li>
<li>SHIFT
Remove the word from the front of the input buffer and push it onto the stack.</li>
</ul>

<h4>arc eager transition system</h4>

<ul>
<li>LEFTARC
Assert a head-dependent relation between the word at the front of the input buffer and the word at the top of the stack; pop the stack.</li>
<li>RIGHTARC
Assert a head-dependent relation between the word on the top of the stack and the word at front of the input buffer; shift the word at the front of the input buffer to the stack.</li>
<li>SHIFT
Remove the word from the front of the input buffer and push it onto the stack.</li>
<li>REDUCE
Pop the stack.</li>
</ul>

<h4>arc standard vs. arc eager</h4>

<ol>
<li>arc standard中的transition操作只会给stack顶部的元素添加依赖关系，而且一旦添加了head-dependent的依赖关系之后，dependent元素会从stack中移除，从而后续不能再添加与该元素有关的依存关系。</li>
<li>所以在arc standard的体系中，一个词在它的所有dependent被发现之前，是不可以被指定head的。这种限制会导致由 parse tree 构造训练样本的时候需要多做一些考虑。另外，如果一个词越是要等很长时间才能指定head，那么就越有可能会出错。</li>
<li>arc-eager 是 arc-standard 的一个替代，它对LEFTARC，RIGHTARC操作进行了些许修改，同时增加新的REDUCE操作。这种体系可以为一个词尽早的指定head。</li>
</ol>

<h3>算法框架</h3>

<h4>基本的Greedy的算法框架</h4>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/dependency-01.png" alt="dependency" />

<h4>带Beam Search的算法框架</h4>

为了能够使用Beam Search, 需要有一个到当前位置为止的action list的评分函数或者概率分布函数。
这样的评分函数或者概率分布函数有各种不同的定义方法。
比如可以用action list中每一步的得分求和来作为整个action list的得分。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/dependency-02.png" alt="" />

<h4>ORACLE的学习</h4>

学习一个分类器，可以使用SVM，多类逻辑回归等。

<strong>常见特征</strong>：
- individual features：
    stack顶部附近的word，pos
    buffer头部附近的word，pos
    以及这些word的associated dependency relation等，
- combining features：
    individual features的某些连接组合

<h3>样例</h3>

使用arc standard transition system
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/arc-standard.png" alt="" />

使用arc eager transition system
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/arc-eager.png" alt="" />

<h3>Transition Based方法的优缺点</h3>

<ol>
<li>efficiency，基本上只要扫一遍sentence，整个过程会产生 2*len(sentence) 个 transitions.</li>
<li>transition Based的方法很灵活，我们可以切换新的transition system而不改变整个算法流程和底层的parsing 算法。我们可以有用于各种各样的处理不同问题的transition system，比如做POS，做non-projective 依存分析，语义角色标注等等。</li>
<li>在greedy的算法中，当oracle给出错误的transition之后，是没有回退并尝试新的transition的。
另外最后只返回一个确定的 transition sequences.</li>
<li>基本的transition based的方法只能产生projective trees</li>
</ol>

<h2>Graph Based Method</h2>

利用了有向加权图的maximum spanning tree(MST)算法。

<h3>样例</h3>

下面是一个使用Chu-Liu Edmonds algorithm寻找最大生成树(parse tree) 的例子。
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/graph-method.png" alt="" />

<h3>特征和训练</h3>

<strong>常见特征</strong>：

<blockquote>
  <ol>
  <li>Wordforms, lemmas, and parts of speech of the headword and its dependent.</li>
  <li>Corresponding features derived from the contexts before, after and between the words.</li>
  <li>Pre-trained word embeddings such as those discussed in Chapter 3.</li>
  <li>The dependency relation itself.</li>
  <li>The direction of the relation (to the right or left).</li>
  <li>The distance from the head to the dependent.</li>
  </ol>
</blockquote>

<strong>模型训练</strong>：
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/graph-model.png" alt="" />

使用inference-based learning combined with the perceptron learning rule. 
我们先用初始的权重parse a sentence，如果parse tree不正确，会做权重更新。
找到incorrect parse中出现而在reference parse的不出现的feature，以一定的学习率降低这些feature相应的权重。

<h3>Graph Based方法的优缺点</h3>

特点：
1. graph based的方法可以产生non-projective trees.
2. 经验上来说，transition based的方法在依存关系比较短的时候有很高的准确率，但是随着head和dependent之间距离的增大，它的准确率会显著下降。
相比之下，transition based的方法使用了greedy local decisions，而Graph Based的方法尝试通过给整棵树打分来避免这种问题。

<h2>Evaluation</h2>

<ul>
<li>labeled attachment score (LAS)：只考虑每个词head的准确率</li>
<li>unlabeled attachment score (UAS)：考虑每个词的head和label的准确率</li>
</ul>

<h1>Syntactic Processing Using the Generalized Perceptron and Beam Search</h1>

<blockquote>
  Yue Zhang, Stephen Clark @Cambridge
</blockquote>

<h2>基本框架</h2>

<h3>基本模型</h3>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/base-framework.png" alt="" />

<hr />

<h3>BeamSearch</h3>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/beamsearch-framework.png" alt="" />

<hr />

<h3>Generalized Perceptron Algorithm</h3>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/generalized-perceptron.png" alt="" />

<hr />

文章中还提到，在实际中，我们会使用 Averaged Perceptron Algorithm（权值平均），和early update strategy（当在解码过程中发现candidates已经不正确了，可以停止解码，根据当前partial的解码序列和partial的金标准序列进行训练）

<h2>Conclude</h2>

提出了一个通用的框架，包含了一个广义的感知机算法和一个 beam-seach解码算法

当把这个框架应用具体的问题的时候，state item的具体结构，解码器中涉及到一些函数都需要被实例化。

<h1>A Fast and Accurate Dependency Parser using Neural Networks</h1>

<blockquote>
  Danqi Chen, Christopher D. Manning @stanford
</blockquote>

第一个比较成功的将神经网络用语依存分析

使用的传统的transition based方法的时候，特征稀疏，特征的不完备性，特征的计算量比较大（在句法解析的过程中，超过95%的时间都花在了特征计算上）

<h2>模型结构</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/danqichen.png" alt="" />

<h3>特点</h3>

<ul>
<li>使用了POS and label embeddings：
尽管POS和label都比较小，但是不同的pos或者label之间仍然存在着一些语义相似性。</li>
<li>使用了Cube activation function：
据作者说，cube函数展开有三次项，有助于捕获三个元素之间的交互作用。</li>
<li>目标函数：
cross-entropy loss plus a l2-regularization term</li>
</ul>

<h1>Structured Training for Neural Network Transition-Based Parsing</h1>

<blockquote>
  David Weiss, Chris Alberti, Michael Collins, Slav Petrov@google
</blockquote>

<h2>基本结构</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/weiss.png" alt="" />

<h3>特点</h3>

<strong>在Chen方法上的改进</strong>：
1. 对于POS tags 和 labels 可以使用更小的嵌入维度
2. 激活函数使用Relu, 取代Cube
3. 使用更深的网络, 有2个隐藏层

<strong>创新点</strong>：
- <strong>添加一个附加的output layer</strong>

通过学习parsing actions的概率分布来对网络的隐藏层进行预训练. 之后固定住隐藏层表示, 通过structured perceptron学习一个附加的final output layer. 准确率可以得到一定的提升.
结合了Collins本人在之前做dependency parsing用到的perceptron algorithm和early update的方法。

待优化的目标函数如下：
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/weiss-loss.png" alt="" />

上面使用了预训练过程得到的向量来作为特征向量，参与到感知机算法的训练中。

<ul>
<li><strong>语料补充</strong>
通过自动parsing得到的高质量的数据对已有的gold data进行补充, 可以进一步提升准确率.
tri-training: 使用两个不同的parser来处理未标注的语料，会把两个parser产生相同的parse tree的sentence加入到训练数据中。</li>
</ul>

<h1>A Neural Probabilistic Structured-Prediction Model for Transition-Based Dependency Parsing</h1>

<blockquote>
  Hao Zhou, Yue Zhang, Shujian Huang, Jiajun Chen
</blockquote>

基本想法：结合Chen的neural network和前面提到的general framework

<h2>尝试1</h2>

<strong>直接使用neural network的输出来取代general framework中的线性评分函数。</strong>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/zhou-try01.png" alt="" />
效果不好，可能是因为序列中每一步的action是概率相关的，使用每一步action的概率相乘去近似整个序列的概率可能不太准确。

<h2>尝试2</h2>

<strong>直接建模整个action sequences的概率分布</strong>

定义整个action sequences的概率分布如下：
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/zhou-try02-1.png" alt="" />

训练的目标函数如下：
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/zhou-try02-2.png" alt="" />

使用上面的目标函数，在计算梯度的时候，会涉及到大量的可能的action序列。
我们使用contrastive learning来近似计算上面的Z
<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/zhou-try02-3.png" alt="" />

在新的目标函数中，我们只考虑惩罚那些位于Beam中的负样本，也即概率相对高的负样本。
至此，整个算法流程如下：

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/zhou-try02-4.png" alt="" />

<h2>Conclusion</h2>

maximizes the likelihood of action sequences instead of individual actions

<h1>Globally Normalized Transition-Based Neural Networks</h1>

<blockquote>
  Daniel Andor, Chris Alberti, David Weiss, Aliaksei Severyn, Alessandro Presta, Kuzman Ganchev, Slav Petrov and Michael Collins @google
</blockquote>

提出了一个globally normalized transition-based neural network model，在POS tagging， dependency parsing，sentence compression等任务上有state of the art的表现。

模型的核心是一个an incremental transition based parser. 对于不同的问题，我们需要调整transition system和输入特征。

<h2>Transition System</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/google-01.png" alt="" />

<hr />

<h2>Global vs. Local Normalization</h2>

<h3>Local Normalization</h3>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/google-02.png" alt="" />

<hr />

<h3>Global Normalization</h3>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/google-03.png" alt="" />

<hr />

<h2>Training</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/google-04.png" alt="" />

对于Local Normalization而言，归一化项的求和数目比较少，求梯度等计算比较容易。

相比之下global normalization中，归一化项求和数量很多，为了让整个过程容易训练，同样使用了BeamSearch 和 early update，只考虑哪些Beam中path而不是所有可能的path。

<h2>结果对比</h2>

<img src="http://blog.tvect.cc/wp-content/uploads/2018/03/result.png" alt="" />

<h1>其他</h1>

全局标准化+BeamSearch的想法，应该也可以类似的用在language model的问题上。比如，在NMT的解码阶段，尝试直接对partial sentence的概率进行建模，而不是对每一步的word的概率进行建模，再使用BeamSearch，应该也会有更好的效果。

<h1>参考文献</h1>

<ul>
<li><strong>Syntactic Processing Using the Generalized Perceptron and Beam Search</strong></li>
<li><strong>A Fast and Accurate Dependency Parser using Neural Networks</strong></li>
<li><strong>Structured Training for Neural Network Transition-Based Parsing</strong></li>
<li><strong>A Neural Probabilistic Structured-Prediction Model for Transition-Based Dependency Parsing</strong></li>
<li><strong>Globally Normalized Transition-Based Neural Networks</strong></li>
</ul>