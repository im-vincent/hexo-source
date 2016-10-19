title: NumPy
date: 2015-09-30 09:49:55
tags: [python]
---

> [量化分析师的Python日记](https://uqer.io/community/share/54ca15f9f9f06c276f651a56 "Title")

```
import numpy as np
np.version.full_version
'1.9.2'
```

#### 数组

通过"type"函数查看a的类型，这里显示a是一个array

```
a = np.arange(20)
print a

[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19]

type(a)

numpy.ndarray
```

reshape重新构造数组,构造一个4*5的二维数组（4行5列)

```
a = a.reshape(4, 5)
print a

[[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]
 [15 16 17 18 19]]

```

"ndim"查看维度；"shape"查看各维度的大小；"size"查看全部的元素个数，等于各维度大小的乘积；"dtype"可查看元素类型；"dsize"查看元素占位（bytes）大小。



创建数组

```
raw = [0,1,2,3,4]
a = np.array(raw)
a
array([0, 1, 2, 3, 4])


raw = [[0,1,2,3,4], [5,6,7,8,9]]
b = np.array(raw)
b
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])
       
       
d = (4, 5)
np.zeros(d)
array([[ 0.,  0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.,  0.]])
#4*5的全零矩阵

np.random.rand(5)
array([ 0.93807818,  0.45307847,  0.90732828,  0.36099623,  0.71981451])
#(0, 1)区间的随机数数组：

a = np.arange(20).reshape(4,5)
print "a:"
print a
print "sum of all elements in a: " + str(a.sum())
print "maximum element in a: " + str(a.max())
print "minimum element in a: " + str(a.min())
print "maximum element in each row of a: " + str(a.max(axis=1))
print "minimum element in each column of a: " + str(a.min(axis=0))
a:
[[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]
 [15 16 17 18 19]]
sum of all elements in a: 190
maximum element in a: 19
minimum element in a: 0
maximum element in each row of a: [ 4  9 14 19]
minimum element in each column of a: [0 1 2 3 4]
#axis(0列,1行)
```

#### 矩阵
科学计算中大量使用到矩阵运算，除了数组，NumPy同时提供了矩阵对象（matrix）。矩阵对象和数组的主要有两点差别：一是矩阵是二维的，而数组的可以是任意正整数维；二是矩阵的'*'操作符进行的是矩阵乘法，乘号左侧的矩阵列和乘号右侧的矩阵行要相等，而在数组中'*'操作符进行的是每一元素的对应相乘，乘号两侧的数组每一维大小需要一致。数组可以通过asmatrix或者mat转换为矩阵，或者直接生成也可以：

```
a = np.arange(20).reshape(4, 5)
a = np.asmatrix(a)
print type(a)

b = np.matrix('1.0 2.0; 3.0 4.0')
print type(b)

<class 'numpy.matrixlib.defmatrix.matrix'>
<class 'numpy.matrixlib.defmatrix.matrix'>
```

#### 数组元素访问
数组和矩阵元素的访问可通过下标进行，以下均以二维数组（或矩阵）为例：

```
a = np.array([[3.2, 1.5], [2.5, 4]])
print a[0][1]
print a[0, 1]

1.5
1.5
```

利用':'可以访问到某一维的全部数据，例如取矩阵中的指定列：

```
a = np.arange(20).reshape(4, 5)
print "a:"
print a
print "the 2nd and 4th column of a:"
print a[:,[1,3]]

a:
[[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]
 [15 16 17 18 19]]
the 2nd and 4th column of a:
[[ 1  3]
 [ 6  8]
 [11 13]
 [16 18]]
```


稍微复杂一些，我们尝试取出满足某些条件的元素，这在数据的处理中十分常见，通常用在单行单列上。下面这个例子是将第一列大于5的元素（10和15）对应的第三列元素（12和17）取出来

```
a[:, 2][a[:, 0] > 5]

#array([12, 17])
```

可使用where函数查找特定值在数组中的位置：

```
loc = numpy.where(a==11)
print loc
print a[loc[0][0], loc[1][0]]

(array([2]), array([1]))
11
```
