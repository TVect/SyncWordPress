---
ID: 105
post_title: tensorflow中crf.py代码注释
author: Chin
post_excerpt: ""
layout: post
permalink: 'http://www.tvect.cc/2018/01/26/tensorflow%e4%b8%adcrf-py%e4%bb%a3%e7%a0%81%e6%b3%a8%e9%87%8a/'
published: true
post_date: 2018-01-26 15:53:23
---
tensorflow中有crf layer的实现, 下面是根据自己的理解对代码做的注释

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-python">def _lengths_to_masks(lengths, max_length):
  tiled_ranges = array_ops.tile(array_ops.expand_dims(math_ops.range(max_length), 0), [array_ops.shape(lengths)[0], 1])
  lengths = array_ops.expand_dims(lengths, 1)
  # mask [lengths.shape[0], max_length], 每一行前面几个为 1.0
  masks = math_ops.to_float(math_ops.to_int64(tiled_ranges) &lt; math_ops.to_int64(lengths))
  return masks

def crf_sequence_score(inputs, tag_indices, sequence_lengths, transition_params):
  # Compute the scores of the given tag sequence.
  unary_scores = crf_unary_score(tag_indices, sequence_lengths, inputs)
  binary_scores = crf_binary_score(tag_indices, sequence_lengths, transition_params)
  sequence_scores = unary_scores + binary_scores
  return sequence_scores

def crf_log_norm(inputs, sequence_lengths, transition_params):
  # Split up the first and rest of the inputs in preparation for the forward algorithm.
  # first_input shape为[batch_size, 1, num_tag] --&gt; [batch_size, num_tag]
  first_input = array_ops.slice(inputs, [0, 0, 0], [-1, 1, -1])
  first_input = array_ops.squeeze(first_input, [1])
  # rest_of_input shape为[batch_size, sequence_length-1, num_tags]
  rest_of_input = array_ops.slice(inputs, [0, 1, 0], [-1, -1, -1])

  # Compute the alpha values in the forward algorithm in order to get the partition function.
  forward_cell = CrfForwardRnnCell(transition_params)
  _, alphas = rnn.dynamic_rnn(cell=forward_cell, inputs=rest_of_input, sequence_length=sequence_lengths - 1, initial_state=first_input, dtype=dtypes.float32)
  log_norm = math_ops.reduce_logsumexp(alphas, [1])
  return log_norm

def crf_log_likelihood(inputs,`tag_indices, sequence_lengths, transition_params=None):
  # Get shape information.
  num_tags = inputs.get_shape()[2].value

  # Get the transition matrix if not provided.
  if transition_params is None:
    transition_params = vs.get_variable("transitions", [num_tags, num_tags])
  # 这样子构造transition_params, 某种程度上强加了在每个step的transition_params都相同的限制
  # 参考李航的书上,其实应该没有这种限制的.

  sequence_scores = crf_sequence_score(inputs, tag_indices, sequence_lengths,
                                       transition_params)
  log_norm = crf_log_norm(inputs, sequence_lengths, transition_params)

  # Normalize the scores to get the log-likelihood.
  # exp(sequence_scores)对应概率的分子部分, exp(log_norm)对应概率的分母部分
  log_likelihood = sequence_scores - log_norm
  return log_likelihood, transition_params

def crf_unary_score(tag_indices, sequence_lengths, inputs):
  batch_size = array_ops.shape(inputs)[0]
  max_seq_len = array_ops.shape(inputs)[1]
  num_tags = array_ops.shape(inputs)[2]

  flattened_inputs = array_ops.reshape(inputs, [-1])

  offsets = array_ops.expand_dims(
      math_ops.range(batch_size) * max_seq_len * num_tags, 1)
  offsets += array_ops.expand_dims(math_ops.range(max_seq_len) * num_tags, 0)
  flattened_tag_indices = array_ops.reshape(offsets + tag_indices, [-1])

  # 根据tag_indices对inputs进行gather, 获取tag对应的 emission score
  unary_scores = array_ops.reshape(
      array_ops.gather(flattened_inputs, flattened_tag_indices),
      [batch_size, max_seq_len])
  masks = _lengths_to_masks(sequence_lengths, array_ops.shape(tag_indices)[1])

  # masks shape为[batch_size, max_length], 每一个只有前面几个元素为1.0
  # unary_scores shape为[batch_size], 元素为 tag index对应激活的emit score沿着Sequence方向求和 
  unary_scores = math_ops.reduce_sum(unary_scores * masks, 1)
  return unary_scores

def crf_binary_score(tag_indices, sequence_lengths, transition_params):
  # Get shape information.
  num_tags = transition_params.get_shape()[0]
  num_transitions = array_ops.shape(tag_indices)[1] - 1

  # start_tag 表示 [[tag0, ..., tag(n-1)], ...], 
  # end tag 表示 [[tag1, tag2, ... tag(n)], ...]
  start_tag_indices = array_ops.slice(tag_indices, [0, 0],
                                      [-1, num_transitions])
  end_tag_indices = array_ops.slice(tag_indices, [0, 1], [-1, num_transitions])

  # Encode the indices in a flattened representation.
  # flattened_trans_indices 元素为 [[tag0*num_tag+tag1, ..., tag(n-1)*num_tags+tag(n)], ...]
  flattened_transition_indices = start_tag_indices * num_tags + end_tag_indices
  flattened_transition_params = array_ops.reshape(transition_params, [-1])

  # Get the binary scores based on the flattened representation.
  # binary_scores shape为[batch_size, num_transactions]
  binary_scores = array_ops.gather(flattened_transition_params,
                                   flattened_transition_indices)

  masks = _lengths_to_masks(sequence_lengths, array_ops.shape(tag_indices)[1])
  # 需要truncate, 和前面truncate操作对应
  truncated_masks = array_ops.slice(masks, [0, 1], [-1, -1])
  # binary_scores shape为[batch_size], 元素为每个sequence中对应trans_score求和
  binary_scores = math_ops.reduce_sum(binary_scores * truncated_masks, 1)
  return binary_scores

class CrfForwardRnnCell(core_rnn_cell.RNNCell):
  """Computes the alpha values in a linear-chain CRF.
  See http://www.cs.columbia.edu/~mcollins/fb.pdf for reference.
  """
  def __init__(self, transition_params):
    # _transition_params shape为 [1, num_tags, num_tags]
    self._transition_params = array_ops.expand_dims(transition_params, 0)
    self._num_tags = transition_params.get_shape()[0].value

  @property
  def state_size(self):
    return self._num_tags

  @property
  def output_size(self):
    return self._num_tags

  def __call__(self, inputs, state, scope=None):
    # state shape 为[batch_size, num_tags, 1]
    state = array_ops.expand_dims(state, 2)

    # 广播操作: shape:[batch_size, num_tags, 1] + shape:[1, num_tags, num_tags] 
    # transition_scores shape为: [batch_size, num_tags, num_tags]
    transition_scores = state + self._transition_params
    # 这样做的细节问题？？？？？？？？？？？？？？？？
    # 因为 + 在exp作用之后, 可以变为exp(input(step, i))作用在每个对应的sumexp上.
    # 对每一步做logsumexp之后, 得到的就是到当前步骤的所有path的score之和
    new_alphas = inputs + math_ops.reduce_logsumexp(transition_scores, [1])
    # new_alphas[i] 表示了到当前step, 且当前step的tag为i时, 所有路径的score之和 即为: log (sum(exp(x1), exp(x2), exp(x3), ...)))
    return new_alphas, new_alphas

def viterbi_decode(score, transition_params):
  trellis = np.zeros_like(score)
  backpointers = np.zeros_like(score, dtype=np.int32)
  trellis[0] = score[0]

  for t in range(1, score.shape[0]):
    # shape:[num_tags, 1] + shape:[num_tags, num_tags]  -&gt;  shape:[num_tags, num_tags]
    # v shape:[num_tags, num_tags] 记录了t-1的tag的全路径score + 转移到t时刻的tag的转移score
    v = np.expand_dims(trellis[t - 1], 1) + transition_params
    # 更新trellis为 t时刻tag的全路径score
    trellis[t] = score[t] + np.max(v, 0)
    # backpointers 记录t时刻tag的最佳前面路径
    backpointers[t] = np.argmax(v, 0)

  viterbi = [np.argmax(trellis[-1])]
  for bp in reversed(backpointers[1:]):
    viterbi.append(bp[viterbi[-1]])
  viterbi.reverse()

  viterbi_score = np.max(trellis[-1])
  return viterbi, viterbi_score
</code></pre>