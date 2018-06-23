---
ID: 387
post_title: numpy ndarray类型中view/copy
author: chin340823
post_excerpt: ""
layout: post
permalink: 'http://www.tvect.cc/2018/06/23/numpy-ndarray%e7%b1%bb%e5%9e%8b%e4%b8%adviewcopy/'
published: true
post_date: 2018-06-23 12:43:01
---
在numpy的reshape和ravel函数说明中指出在必要时会返回对象的拷贝。
numpy.reshape()中指出：

<pre class="prism-highlight line-numbers" data-start="1"><code class="language-null">This will be a new view object if possible; otherwise, it will be a copy.  
Note there is no guarantee of the *memory layout* (C- or Fortran- contiguous) of the returned array.
</code></pre>

numpy.ravel()中指出：

<pre class="prism-highlight line-numbers" data-start="1"><code class="language-null">A 1-D array, containing the elements of the input, is returned.  A copy is made only if needed.
</code></pre>

下面就自己的理解做一下解释。
</br>

<h1>ndarray的<a href="http://old.sebug.net/paper/books/scipydoc/numpy_intro.html">内存结构</a></h1>

<img src="http://obn75nm65.bkt.clouddn.com/ndarry-memory.png" alt="ndarray内存结构" title="numpy ndarray" />
数据存储区域保存着数组中所有元素的二进制数据，dtype对象则知道如何将元素的二进制数据转换为可用的值。数组的维数、大小等信息都保存在ndarray数组对象的数据结构中。

strides中保存的是当每个轴的下标增加1时，数据存储区中的指针所增加的字节数。例如图中的strides为12,4，即第0轴的下标增加1时，数据的地址增加12个字节：即a[1,0]的地址比a[0,0]的地址要高12个字节，正好是3个单精度浮点数的总字节数；第1轴下标增加1时，数据的地址增加4个字节，正好是单精度浮点数的字节数。

具体来说，<strong>在展示数组的时候会先根据data的起始地址和strides[0]的偏移大小，找到行首元素的位置，按照strides[1]的偏移大小，取出指定个数的元素，把他们作为一行展示出来</strong>。

另外，元素在数据存储区中的排列格式有两种：C语言格式和Fortan语言格式。在C语言中，多维数组的第0轴是最上位的，即第0轴的下标增加1时，元素的地址增加的字节数最多；而Fortan语言的多维数组的第0轴是最下位的，即第0轴的下标增加1时，地址只增加一个元素的字节数。在NumPy中，元素在内存中的排列缺省是以C语言格式存储的，如果你希望改为Fortan格式的话，只需要给数组传递order="F"参数。

</br>

<h1>案例分析：</h1>

<h3>案例1：基本函数</h3>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-python">&gt;&gt;&gt; c = a.view()  
&gt;&gt;&gt; c is a  
False  
&gt;&gt;&gt; c.base is a      #c是a持有数据的镜像  
True  
&gt;&gt;&gt; c.flags.owndata  
False  
&gt;&gt;&gt;  
&gt;&gt;&gt; c.shape = 2,6    # a的形状没变  
&gt;&gt;&gt; a.shape  
(3, 4)  
&gt;&gt;&gt; c[0,4] = 1234        #a的数据改变了  
&gt;&gt;&gt; a  
array([[   0,    1,    2,    3],  
       [1234,    5,    6,    7],  
       [   8,    9,   10,   11]])  
</code></pre>

<h3>案例2：reshape返回view还是copy？</h3>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-python">&gt;&gt;&gt; arr = np.arange(12).reshape((3,4))
&gt;&gt;&gt; print arr.strides, arr[::2][::2].strides    # 返回view
(16L, 4L) (32L, 8L)
&gt;&gt;&gt; print arr.T.strides, arr.T[::2][::2].strides    # 返回view
(4L, 16L) (8L, 32L)
&gt;&gt;&gt; print arr.T[::2][::2].base is arr.base
True
&gt;&gt;&gt; brr = arr.T.reshape((12,))    # 返回的是copy，因为此时已经无法在不修改数据存储区域的基础上，仅仅通过修改dims，strides等信息，来构造新的结构了
&gt;&gt;&gt; print brr.strides
(4L, )
&gt;&gt;&gt; print np.may_share_memory(arr, brr)
False
&gt;&gt;&gt; print brr.base is arr.base
False
</code></pre>

<h3>案例3：flatten和ravel操作对比</h3>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-python">&gt;&gt;&gt; arr = np.arange(12).reshape((3,4))
&gt;&gt;&gt; print np.may_share_memory(arr.flatten(), arr)    #flatten：return a copy of the array collapsed into one dimension
False
&gt;&gt;&gt; print np.may_share_memory(arr.ravel(), arr)    #ravel：返回view
True
&gt;&gt;&gt; print np.may_share_memory(arr.T.ravel(), arr)    #ravel：返回copy
False
</code></pre>

解释：
成员函数.ravel()，等价于numpy.ravel()，其说明如下：

<pre class="prism-highlight line-numbers" data-start="1"><code class="language-null">Return a contiguous flattened array.
A 1-D array, containing the elements of the input, is returned.  A copy is made only if needed.
</code></pre>

相比之下，flatten会一直返回对象的拷贝，而ravel一般会返回view，但会在必要的时候返回数据的copy.
在上面的例子中arr.T.ravel()无法在不修改数据存储区域的基础上，仅仅通过修改dims，strides等信息，来构造新的结构，所以会返回一个数据的拷贝。

<h3>案例4：resize操作成功还是失败？</h3>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-python">&gt;&gt;&gt; arr = np.arange(8)
&gt;&gt;&gt; arr.resize(12)    # 注意成员函数resize，numpy通用函数resize的区别
&gt;&gt;&gt; arr.resize((2,4)); print arr    # resize操作成功
[[0 1 2 3]
 [4 5 6 7]]
&gt;&gt;&gt; arr = np.arange(8); arr.resize(12); brr = arr; arr.resize((2,4))    # 失败
ValueError: cannot resize an array that references or is referenced
by another array in this way.  Use the resize function
&gt;&gt;&gt; arr = np.arange(8); arr.resize(12); brr = arr; np.resize(arr, (2,4))    # 成功
</code></pre>

解释：
成员函数会抛出异常ValueError

<pre class="prism-highlight line-numbers" data-start="1"><code class="language-null">    If `a` does not own its own data or references or views to it exist, and the data memory must be changed.
</code></pre>

而通用函数numpy.resize(), 不会抛出上述异常。因为成员函数ndarray.resize()是对array进行原地修改的，而numpy.resize()则不会对原对象做修改，只是返回一个新的指定shape的array
另外，numpy.resize()和numpy.resize()还有如下不同：

<pre class="prism-highlight line-numbers" data-start="1"><code class="language-null">If the new array is larger than the original array, then the new array is filled with repeated copies of `a`.  
Note that this behavior is different from a.resize(new_shape) which fills with zeros instead of repeated copies of `a`.
</code></pre>