---
ID: 674
post_title: xgboost 阅读笔记
author: Chin
post_excerpt: >
  本文是很早之前阅读 XGBoost A
  Scalable Tree Boosting System
  的笔记。以前是在
  网易云笔记上面，这里只是简单的搬运过来。
layout: post
permalink: 'http://blog.tvect.cc/2018/08/28/xgboost-%e9%98%85%e8%af%bb%e7%ac%94%e8%ae%b0/'
published: true
post_date: 2018-08-28 15:52:26
---
本文是很早之前阅读 XGBoost A Scalable Tree Boosting System 的笔记。以前是在 <a href="http://note.youdao.com/noteshare?id=e08d817ffc9c77c52be776a413fc641d&amp;sub=C35545F459634CFFA9CE7D227BF9FFD0">网易云笔记</a>上面，这里只是简单的搬运过来。

<h1>概述</h1>

We propose a novel sparsity-aware algorithm for sparse data and weighted quantile sketch for approximate tree learning. More importantly, we provide insights on cache access patterns, data compression and sharding to build a scalable tree boosting system.

<strong>优点&amp;创新点</strong>

<ul>
<li>单机版本的xgboost运行速度比现存的实现方案快10+倍，在分布式或者内存有限的情况下，数据量可以很好的扩展到10亿级别的样本。</li>
<li>提出了新颖的处理稀疏数据的办法。</li>
<li>提出了一个理论上证明的加权分位数算法(weighted quantile sketch procedure), 可用于近似的分裂点寻找。</li>
<li>propose an effective cache-aware block structure for out-of-core tree learning</li>
</ul>

<h1>算法模型相关</h1>

<h2>模型概述</h2>

<ul>
<li><strong>目标函数</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-01.png" width="600" align=center /></p></li>
<li><p><strong>树的定义</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-02.png" width="400" align=center /></p></li>
<li><p><strong>树的复杂度</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-03.png" width="350" align=center /></p></li>
<li><p><strong>重写目标函数</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-04.png" width="500" align=center /></p></li>
<li><p><strong>loss reduction after a split</strong>
<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-05.png" width="600" align=center /></p></li>
<li><p><strong>其他防止过拟合的方法</strong>

<ul>
<li>Shrinkage
scales newly added weights by a factor η after each step of tree boosting.
reduces the influence of each individual tree and leaves space for future trees to improve the model.</p></li>
<li><p>Column Subsampling
using column sub-sampling prevents over-fitting even more so than the traditional row sub-sampling.
the usage of column sub-samples also speeds up computations of the parallel algorithm.</p></li>
</ul></li>
</ul>

<h2>分裂点寻找</h2>

<p>One of the key problems in tree learning is to find the best split

<h3>Exact Greedy Algorithm</h3>

枚举所有特征上所有可能的分裂点。

为了有效的计算，一般需要对每个特征按特征取值大小进行排序，排序后再按顺序遍历累加。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-06.png" width="500" align=center />

案例：scikit-learn, R’s gbm, 单机版本的XGBoost

<h3>Approximate Algorithm</h3>

Exact Greedy Algorithm很有用，但是无法试用于分布式环境或者是数据量太大无法放到内存的情况。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-07.png" width="500" align=center />

上面算法中，先根据特征分布的分位数信息得到候选的分裂点（具体可以看下面的Weighted Quantile Sketch），再根据聚合统计量选出最优的分裂点。

具体的，有两个版本的算法：

<ul>
<li>Global
只在树构造的init阶段产生候选分类点，接下来每次分裂都使用同样的candidate</p></li>
<li><p>Local
在每次分裂之后，都重新产生candidate</p></li>
</ul>

<p>global的方法需要的proposal steps更少，但是它需要产生的分裂点更多，因为它没有为每次分裂调整candidate.

在更深的树的情况下，local的方法可能更为合适。

案例：在xgboost的所有的环境下（单机或者是分布式）都支持approximate algorithm

<h2>Weighted Quantile Sketch</h2>

特征的分位数信息经常可以用来产生该特征候选分裂点。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-08.png" width="500" align=center />

上面的 epsilon 是一个控制精度的参数，基本上可视为有1/epsilon 个候选点.

另外，上面的每个点都用了二阶导数进行加权。因为损失函数可以视为加权的平方损失，label为g/h，权重为h.

<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-09.png" width="400" align=center />

然而，对于大的数据集没有trivial的方法找到符合上述准则的split point。

为了解决这个问题，文章中提出了a novel distributed weighted quantile sketch algorithm，可以处理加权的数据，并且给出了一个理论证明。

<h2>Sparsity-aware Split Finding</h2>

实际问题中，输入数据经常是稀疏的，为了处理这个问题，xgboost给每个树节点加上了一个默认分裂方向。

<img src="http://blog.tvect.cc/wp-content/uploads/2018/08/xgboost-10.png" width="500" align=center />

<h1>系统设计相关</h1>

<ul>
<li>Column Block for Parallel Learning</li>
<li>Cache-aware Access</li>
<li>Blocks for Out-of-core Computation</li>
</ul>

没有仔细看了。。。。。。

<h1>参考文献</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1603.02754">XGBoost: A Scalable Tree Boosting System</a></p></li>
<li><p>Introduction to Boosted Trees. Tianqi Chen 2014</p></li>
</ul>