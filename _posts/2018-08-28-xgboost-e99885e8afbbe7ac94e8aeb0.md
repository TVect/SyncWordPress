---
ID: 65
post_title: xgboost 阅读笔记
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/65
published: true
post_date: 2018-08-28 15:52:26
---
[toc]

本文是很早之前阅读 XGBoost A Scalable Tree Boosting System 的笔记。以前是在  <a href="http://note.youdao.com/noteshare?id=e08d817ffc9c77c52be776a413fc641d&amp;sub=C35545F459634CFFA9CE7D227BF9FFD0">网易云笔记</a>   上面。

<!--more-->

<h1>概述</h1>

We propose a novel sparsity-aware algorithm for sparse data and weighted quantile sketch for approximate tree learning. More importantly, we provide insights on cache access patterns, data compression and sharding to build a scalable tree boosting system.

<h2>优点&amp;创新点</h2>

<ul>
<li>单机版本的xgboost运行速度比现存的实现方案快10+倍，在分布式或者内存有限的情况下，数据量可以很好的扩展到10亿级别的样本。</p></li>
<li><p>提出了新颖的处理稀疏数据的办法。</p></li>
<li><p>提出了一个理论上证明的加权分位数算法(weighted quantile sketch procedure), 可用于近似的分裂点寻找。</p></li>
<li><p>propose an effective cache-aware block structure for out-of-core tree learning</p></li>
</ul>

<h1>算法模型相关</h1>

<h2>模型概述</h2>

<p><strong>树的定义</strong>

$f_t(x) = w_{q(x)} \quad w \in \mathbb{R}^T, q: \mathbb{R}^m \rightarrow [1,...,T]$

<strong>树的复杂度</strong>

$ \Omega(f_t) = \gamma T + \frac{1}{2} \lambda \sum_{j=1}^T  w_i^2$

其中, $T$ 表示这棵树 $f_t$ 的叶子结点的个数, $w_i$ 表示叶子结点上的取值.

<strong>目标函数</strong>

用 $\hat{y_i}^{t-1}$ 表示第 $i$ 个样本在 $t-1$ 轮迭代时的预测值, 目标是要找到 $f_t$ 使得下面的 loss 最小.

$$ \mathit{L}^t = \sum_{i=1}^n l(y_i, \hat{y_i}^{t-1} + f_t(x_i)) + \Omega(f_t) $$

对上式右侧做二阶 Taylor 展开, 有:

$$ \mathit{L}^t \simeq \sum_{i=1}^n [l(y_i, \hat{y_i}^{t-1}) + g_i f_t(x_i) + \frac{1}{2} h_i f_t^2(x)] + \Omega(f_t) $$

其中, $g_i = \partial_{\hat{y}^{t-1}} l(y_i, \hat{y}^{t-1})$, $h_i = \partial_{\hat{y}^{t-1}}^2 l(y_i, \hat{y}^{t-1})$ 分别为 loss 的一阶导数和二阶导数.

<strong>重写目标函数</strong>

假设决策树的结构已知，可以进一步重写目标函数, 进而得到在损失函数最小时每个叶子结点上的预测值, 以及对应的损失函数最小值.

$$
\begin{aligned}
\tilde{\mathit{L}}^t &amp;= \sum_{i=1}^n [l(y_i, \hat{y_i}^{t-1}) + g_i f_t(x_i) + \frac{1}{2} h_i f_t^2(x)] + \gamma T + \frac{1}{2} \lambda \sum_{j=1}^T  w_i^2 &#92;
&amp;= \sum_{j=1}^T [(\sum_{i \in I_j} g_i) w_j + \frac{1}{2}(\sum_{i \in I_j}h_i + \lambda) w_j^2] + \gamma T
\end{aligned}
$$

其中, $I_j =  [i | q(x_i) = j]$, 表示落到 第 $j$ 个叶子结点中的所有样本集合.

上式式关于 $w_j$ 的二次函数, 可以方便的求出关于 $w_j$ 的最优解和此时 $L$ 的值, 即:

$$
\begin{aligned}
w_j^* &amp;= - \frac{\sum_{i \in I_j} g_i}{\sum_{i \in I_j} h_i + \lambda} &#92;
\tilde{\mathit{L}}^t &amp;= -\frac{1}{2} \sum_{j=1}^T \frac{(\sum_{i \in I_j} g_i)^2}{\sum_{i \in I_j} h_i + \lambda} + \gamma T
\end{aligned}
$$

<strong>每一棵树的构造方式</strong>

基于上面给出的损失函数最小值, 可以计算分裂前后的损失函数的差值(loss reduction after a split).

具体来说, 假设某个结点有 $I$ 个样本, 现考虑将其分裂为左右另个结点, 分别包括 $I_L$ 和 $I_R$ 个样本. 分裂前后的 loss reduction 计算如下:

$$
\begin{aligned}
\mathit{L}<em>{split} &amp;= \mathit{L}</em>{before} - \mathit{L}<em>{after} &#92;
&amp;= \frac{1}{2} [\frac{(\sum</em>{i \in I_L} g_i)^2}{\sum_{i \in I_L} h_i + \lambda} + \frac{(\sum_{i \in I_R} g_i)^2}{\sum_{i \in I_R} h_i + \lambda} - \frac{(\sum_{i \in I} g_i)^2}{\sum_{i \in I} h_i + \lambda}] - \gamma
\end{aligned}
$$

XGBoost 会采用最大化这个差值作为准则来进行决策树的构建，通过遍历所有特征的所有取值，寻找使得 $L_{split}$ 最大时对应的分裂方式。
此外，由于损失函数前后存在差值一定为正的限制，此时 $\gamma$ 也起到了一定的预剪枝效果。

<h2>一些防止过拟合的方法</h2>

<ul>
<li><strong>Shrinkage</strong>
scales newly added weights by a factor η after each step of tree boosting.
reduces the influence of each individual tree and leaves space for future trees to improve the model.</p></li>
<li><p><strong>Column Subsampling</strong>
using column sub-sampling prevents over-fitting even more so than the traditional row sub-sampling.
the usage of column sub-samples also speeds up computations of the parallel algorithm.</p></li>
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
<li>Local
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

<h1>gbdt vs. xgboost</h1>

<ol>
<li><p>GBDT是机器学习算法，XGBoost是该算法的工程实现。</p></li>
<li><p>在使用CART作为基分类器时，XGBoost显式地加入了正则项来控制模型的复杂度，有利于防止过拟合，从而提高模型的泛化能力。</p></li>
<li><p>GBDT在模型训练时只使用了代价函数的一阶导数信息，XGBoost对代价函数进行二阶泰勒展开，可以同时使用一阶和二阶导数。</p></li>
<li><p>传统的GBDT采用CART作为基分类器，XGBoost支持多种类型的基分类器，比如线性分类器。</p></li>
<li><p>传统的GBDT在每轮迭代时使用全部的数据，XGBoost则采用了与随机森林相似的策略，支持对数据进行采样。</p></li>
<li><p>传统的GBDT没有设计对缺失值进行处理，XGBoost能够自动学习出缺失值的处理策略。</p></li>
</ol>

<h1>参考文献</h1>

<ul>
<li><p><a href="https://arxiv.org/abs/1603.02754">XGBoost: A Scalable Tree Boosting System</a></p></li>
<li><p>Introduction to Boosted Trees. Tianqi Chen 2014</p></li>
</ul>