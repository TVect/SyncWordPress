---
ID: 868
post_title: tensorflow nce_loss 代码简单注释
author: Chin
post_excerpt: ""
layout: post
permalink: http://blog.tvect.cc/archives/868
published: true
post_date: 2018-11-07 11:06:22
---
前面的博客中介绍了NCE 的原理，这里给出 tensorflow nce_loss 代码的简单注释。

<!--more-->

<pre><code class="language-Python ">@tf_export("nn.nce_loss")
def nce_loss(...):
  logits, labels = _compute_sampled_logits(...)
  sampled_losses = sigmoid_cross_entropy_with_logits(
      labels=labels, logits=logits, name="sampled_losses")
  return _sum_rows(sampled_losses)
</code></pre>

重点部分在于 _compute_sampled_logits 方法:

<pre><code class="language-Python ">def _compute_sampled_logits(...):
    ...
    if sampled_values is None:
      # 采样分布为 approximately log-uniform or Zipfian distribution.
      # `P(class) = (log(class + 2) - log(class + 1)) / log(range_max + 1)`
      # 其中 class 是按词频排序好的.
      sampled_values = candidate_sampling_ops.log_uniform_candidate_sampler(...)
    sampled, true_expected_count, sampled_expected_count = (
        array_ops.stop_gradient(s) for s in sampled_values)
    ...
    # 计算 true_logits 和 sampled_logits 
    # 通过内积加上bias得到, 即一个简单的单层神经网络
    true_logits += true_b
    sampled_logits += sampled_b

    if remove_accidental_hits:
      acc_hits = candidate_sampling_ops.compute_accidental_hits(
          labels, sampled, num_true=num_true)
      acc_indices, acc_ids, acc_weights = acc_hits
      ...
      记录中samples 和 true_labels 中重复的部分, 下面的函数中会使得冲突的noise sample对应的logits 加上 -FLOAT.MAX, 即使概率变为0.
      ...
      sampled_logits += sparse_ops.sparse_to_dense(
          sparse_indices,
          sampled_logits_shape,
          acc_weights,
          default_value=0.0,
          validate_indices=False)

    # 减去 log (k * p_n * z_normalizer), 为了表示完整的后验概率.
    # 在做 Negative Sampling 的时候, 上面要减去的项为 0.
    if subtract_log_q:
      # Subtract log of Q(l), prior probability that l appears in sampled.
      true_logits -= math_ops.log(true_expected_count)
      sampled_logits -= math_ops.log(sampled_expected_count)
    ...
    return out_logits, out_labels
</code></pre>

关于 candidate sampling 更多更详细的内容可以参考 <a href="http://www.tensorflow.org/extras/candidate_sampling.pdf">candidate_sampling</a>