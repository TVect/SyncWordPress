---
post_title: numpy 数组对象小结
layout: post
published: true
---
[toc]

# NumPy数组对象
## 简介
NumPy中的ndarray是一个多维数组对象，该对象由两部分组成：
    实际的数据；
    描述这些数据的元数据。
大部分的数组操作仅仅修改元数据部分，而不改变底层的实际数据。

## 优点
NumPy数组在数值运算方面的效率优于Python提供的list容器。 
使用NumPy可以在代码中省去很多循环语句，因此其代码比等价的Python代码更为简洁。


# 创建多维数组
numpy.array([numpy.arange(2), numpy.arange(2)])


# 自定义数据类型
numpy.dtype([(,name', str_, 40), ('numitems', int32), ('price',float32)])


# 数组的切片和索引
numpy.arange(24).reshape((2,3,4))[0,...]
### Tips: ... 
    多个冒号可以用一个省略号...来代替,表示遍历剩下的维度


# 改变数组的维度
## .reshape((x, y))

## 用元组设置维度
b.shape = (2, 3)

## .resize((x, y))
和reshape函数的功能一样，但resize会直接修改所操作的数组

## .transponse()

## 展平操作 
    .ravel(): return a view of the original array
    .flatten(): return a copy of the original array

### Tips: ravel & flatten
    The difference is that flatten always returns a copy and ravel returns a view of the original array whenever possible. This isn't visible in the printed output, but if you modify  the array returned by ravel, it may modify the entries in the original array. If you modify the entries in an array returned from flatten this will never happen. ravel will often be faster since no memory is copied, but you have to be more careful about modifying the array it returns.

# 数组的组合
水平组合：np.hstack()
垂直组合：np.vstack()
深度组合：np.dstack()
列组合：np.column_stack()    而对于二维数组, column_stack与hstack的效果是相同的
行组合：np.row_stack()    对于二维数组, row_stack与vstack的效果是相同的
连接函数：np.concatenate((a1, a2, ...), axis=0)    可以用concatenate实现hstack, vstack类似的效果

# 数组的分割
水平分割：np.hsplit()
垂直分割：np.vsplit()
深度分割：np.dsplit()
split函数：numpy.split(ary, indices_or_sections, axis=0)

# 数组的属性
shape属性
dtype属性
ndim属性：数组的维数
size属性：数组中元素的总个数
itemsize属性：数组中每个元素在内存中所占的字节数
nbytes属性：整个数组所占用的存储空间，即为itemsize*size
T属性：数组的转置，对于一维数组，其T属性就是原数组
real属性：给出复数数组的实部
imag属性：给出复数数组的虚部
flat属性：

    flat属性将返回一个numpy.flatiter对象，这是获得flatiter对象的唯一方式——我们无法访问flatiter的构造函数。
    这个所谓的“扁平迭代器”可以让我们像遍历一维数组一样去遍历任意的多维数组。
    可以用flatiter对象直接获取一个或多个数组元素。
    flat属性是一个可赋值的属性。对flat属性赋值将导致整个数组的元素都被覆盖。

# 数组的转换
.tolist()：转换成列表
.astype()：在转换数组时可以指定类型
