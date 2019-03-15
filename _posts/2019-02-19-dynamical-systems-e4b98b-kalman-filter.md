---
ID: 210
post_title: State Space Models 之 LDS
author: chin340823
post_excerpt: ""
layout: post
permalink: https://www.tvect.cn/archives/210
published: true
post_date: 2019-02-19 17:53:55
---
这篇文章会对 Linear Dynamic Systems 做一些总结.

<h1>Introduction</h1>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/758ee40569603466ba0c6a8efe39ba4e.png" alt="" />

LDS 与 HMM 类似, 不过这里的隐状态是连续变量.

<h1>linear-Gaussian state space model</h1>

这里先考虑一类特殊情况, linear-Gaussian state space model that the latent variables $z_n$, as well as the observed variables $x_n$, are multivariate Gaussian distributions whose means are linear functions of the states of their parents in the graph.

模型具体表示如下：

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/828b65764b43d9abd7fd36c0a8a0fd44.png" alt="" />

<strong>Decoding in LDS vs. HMM</strong>

<blockquote>
  Because the linear dynamical system is a linear-Gaussian model, the joint distribution over all variables, as well as all marginals and conditionals, will be Gaussian. It follows that the sequence of individually most probable latent variable values is the same as the most probable latent sequence. There is thus no need to consider the analogue of the Viterbi algorithm for the linear dynamical system.
</blockquote>

<h2>Inference in LDS</h2>

<strong>Problem 1: finding the posterior marginal for a node $z_n$ given observations from $x_1$ up to $x_n$</strong>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/b458bc44b134864dd2972adce9fda7ad.png" alt="" />

这里的前向递推公式也叫 <strong>Kalman filter</strong>

递推公式的一些解释:

<blockquote>
  we can view the quantity $A u_{n−1}$ as the prediction of the mean over zn obtained by simply taking the mean over $z_{n−1}$ and projecting it forward one step using the transition probability matrix $A$. This predicted mean would give a predicted observation for $x_n$ given by $C A z_{n−1}$ obtained by applying the emission probability matrix $C$ to the predicted hidden state mean. We can view the update equation for the mean of the hidden variable distribution as taking the predicted mean $A u_{n−1}$ and then adding a correction that is proportional to the error $x_n − C A z_{n−1}$ between the predicted observation and the actual observation. The coefficient of this correction is given by the Kalman gain matrix. Thus we can view the Kalman filter as a process of making successive predictions and then correcting these predictions in the light of the new observations.
</blockquote>

案例: Kalman filter 用于 tracking 的一个图示如下

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/4e9ecbe2ec8826186a5f515949500340.png" alt="" />

<strong>Problem 2: finding the marginal for a node $z_n$ given all observations $x_1$ to $x_N$</strong>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/7147b1ee1ff17c6309c883e3b7b5b552.png" alt="" />

这里的发现递推公式也叫 <strong>Kalman Smoothing</strong>

<strong>Problem 3: finding the pairwise posterior marginals</strong>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/88f2edf51472429d51d8b86fdeb7a2c6.png" alt="" />

<h2>Learning in LDS</h2>

<img src="https://www.tvect.cn/wp-content/uploads/2019/03/610e69c70a87b85d43ed778230aacee3.png" alt="" />

<h1>参考资料</h1>

<ul>
<li><p><a href="">PRML Chapter 13.3  Linear Dynamical Systems</a></p></li>
<li><p><a href="https://github.com/roboticcam/machine-learning-notes/blob/master/dynamic_model.pdf">Dynamic Model by Yida Xu</a></p></li>
</ul>