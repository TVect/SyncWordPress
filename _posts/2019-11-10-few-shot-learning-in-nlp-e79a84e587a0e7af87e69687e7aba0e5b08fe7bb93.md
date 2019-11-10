---
ID: 581
post_title: >
  few-shot learning in NLP
  的几篇文章小结
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/581
published: true
post_date: 2019-11-10 21:34:28
---
<!--more-->

<h1>Few-shot learning 简单回顾</h1>

<h2>问题定义</h2>

Few-shot Learning 是 Meta Learning 在监督学习领域的应用。
Meta Learning，又称为 learning to learn，在 meta training 阶段将数据集分解为不同的 meta task，去学习类别变化的情况下模型的泛化能力，在 meta testing 阶段，面对全新的类别，可以更好的完成分类。

<h2>数据形式</h2>

<strong>N-way K-shot 问题</strong>

few-shot 的训练集中包含了很多的类别，每个类别中有多个样本。在训练阶段，会在训练集中随机抽取 N 个类别，每个类别 K 个样本，构建一个 meta-task，作为模型的支撑集（support set）输入；再从这 N 个类中剩余的数据中抽取一批（batch）样本作为模型的预测对象（batch set）。即要求模型从 N*K 个数据中学会如何区分这 N 个类别。

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/meta-learning.jpg" alt="" />

<h2>基本方法</h2>

<ul>
<li>Model Based</p></li>
<li><p>Optimization Based</p></li>
<li><p><strong>Metric Based</strong></p></li>
</ul>

<p><strong>Match Net</strong>: <a href="https://arxiv.org/abs/1606.04080">Matching Networks for One Shot Learning</a>

<strong>Prototypical Net</strong>: <a href="https://arxiv.org/abs/1703.05175">Prototypical Networks for Few-shot Learning</a>

<strong>Relation Net</strong>: <a href="https://arxiv.org/abs/1711.06025">Learning to Compare: Relation Network for Few-Shot Learning</a>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/metric_methods-1024x497.jpg" alt="" />

<h1>Few-shot learning 在 NLP 中的几个案例</h1>

<h2>Intent Classification</h2>

<a href="https://arxiv.org/abs/1902.10482">Induction Networks for Few-Shot Text Classification</a>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/intent-classification-1024x460.jpg" alt="" />

<h3>网络结构</h3>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/intent-classification-components-1024x553.png" alt="" />

<h3>总体训练流程</h3>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/intent-classification-training.jpg" alt="" />

<h3>实验结果</h3>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/intent-classification-results-1024x239.png" alt="" />

<h2>Relation Extraction</h2>

<a href="http://nlp.csai.tsinghua.edu.cn/~lzy/publications/aaai2019_nre.pdf">Hybrid Attention-Based Prototypical Networks for Noisy Few-Shot Relation Classification</a>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/relation-extraction-1024x506.jpg" alt="" />

<h3>网络结构</h3>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/relation-extraction-components-1-1024x478.png" alt="" />

<h3>实验结果</h3>

<img src="https://www.tvect.cn/wp-content/uploads/2019/11/relation-extraction-results-1024x354.jpg" alt="" />

<h1>总结</h1>

<h2><strong>Sentence encoder</strong> : 编码每个句子为一个向量</h2>

<strong>方法备选</strong>：

<ul>
<li>LSTM + self attention</p></li>
<li><p>CNN + pooling</p></li>
</ul>

<h2><strong>Induction module</strong> : 从句子向量得到类别向量</h2>

<p><strong>方法备选</strong>：

<ul>
<li>avg</p></li>
<li><p>sum</p></li>
<li><p>self attention</p></li>
<li><p>instance-level attention
focus more attention on those query-related instances and reduce the effect of noise</p></li>
<li><p>dynamic routing induction
dynamic routing method makes our model generalize better
in the few-show text classification text</p></li>
</ul>

<h2><strong>relation module</strong> : 寻找合适的相似性度量方案</h2>

<p><strong>方法备选</strong>：

<ul>
<li><p>Feature-level Attention
每个 class 下的所有样本表示合在一起, 做多次卷积得到一个向量表示. 这个向量的每个维度就代表了相应的特征维度的权重.</p></li>
<li><p>Tensor layer
在 query vector 和 class vector 做内积时, 中间插入一个 tensor.</p></li>
</ul>

<h1>参考资料</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1805.07513">Diverse Few-Shot Text Classification with Multiple Metrics</a></p></li>
<li><p><a href="http://nlp.csai.tsinghua.edu.cn/~lzy/publications/aaai2019_nre.pdf">Hybrid Attention-Based Prototypical Networks for Noisy Few-Shot Relation Classification</a></p></li>
<li><p><a href="https://arxiv.org/abs/1902.10482">Induction Networks for Few-Shot Text Classification</a></p></li>
<li><p><a href="https://mp.weixin.qq.com/s/-73CC3JqnM7wxEqIWCejWQ">小样本学习综述</a></p></li>
<li><p><a href="https://lilianweng.github.io/lil-log/2018/11/30/meta-learning.html">Meta-Learning：Learning to learn fast</a></p></li>
</ul>