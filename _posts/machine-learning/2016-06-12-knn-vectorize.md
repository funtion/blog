---
layout: post
date: 2016-06-12 15:30
status: public
tags: [k-nearest-neighbors, vectorize, python, numpy, algorithm]
title: 'K-Nearest Neighbors 的 vectorize 方法'
categories: [Machine-Learning]
---

K-Nearest Neighbors (KNN) 是一个常用的分类算法，但是两两之间计算距离是比较耗费时间的，通过向量化(vectorize) 的方法，可以极大的加速计算的过程。

## 直接计算
最直接的算法是通过两个循环来计算每一组距离，代码如下：
```python
import numpy as np
def brute_force(X_train, X):
    num_test = X.shape[0]
    num_train = X_train.shape[0]
    dists = np.zeros((num_test, num_train))
    for i in xrange(num_test):
        for j in xrange(num_train):
            dists[i, j] = np.linalg.norm(X[i] - X_train[j])
    return dists
```
其中每一个数据是一个`D` 维的向量， `X_train` 是一个`num_train * D` 的矩阵，表示训练数据，`X` 是一个 `num_test *  D` 的矩阵，表示测试数据，而要计算的距离为欧式距离。
## 去掉内层循环
通过 Numpy 的官方[文档](http://docs.scipy.org/doc/numpy-1.10.1/reference/generated/numpy.linalg.norm.html#numpy.linalg.norm) 可以发现，`numpy.linalg.norm` 支持 `axis` 参数，因此可以直接去掉内层的循环，将代码改写如下：
```python
import numpy as np
def one_loop(X_train, X):
    num_test = X.shape[0]
    num_train = X_train.shape[0]
    dists = np.zeros((num_test, num_train))
    for i in xrange(num_test):
        dists[i, :] = np.linalg.norm(X_train - X[i],axis=1)
    return dists
```
## 去掉外循环：第一次尝试
去掉内循环的方法无法直接用于去掉外层的循环，这时候需要进一步开阔思路。既然我们要求的是每一个测试数据，每一个训练数据的组合在每一个维度上的差值，那么可以把数据映射到三维空间，`diff [i, j, k]`表示第`i`个测试数据与第`j`个训练数据的组合在第`k`维度的差别，然后再映射回两维空间，借助 Numpy 的 broadcast 机制，可以将代码改写为：
```python
import numpy as np
def first_try(X_train, X):
    num_test = X.shape[0]
    num_train = X_train.shape[0]
    diff = np.subtract(X[:,np.newaxis, :], X_train)
    dists = np.sqrt(np.multiply(diff,diff).sum(2)).reshape([num_test,num_train])
    return dists
```
然而运行起来会发现，当数据量比较大的时候产生了 `Memoey Error`，因为这个方法开辟了过大的数组。

## 去掉外循环：第二次尝试
既然如此，能不能不开辟中间过程中的临时三维数组来计算呢？这就需要进一步观察一下要计算的公式。
```mathjax
\begin{equation}
\begin{split} 
dists[i, j] &= \sqrt{\sum_k{(X[i, k] - X\_train[j, k] )^2} } \\
              &= \sqrt{\sum_k{\left(X[i, k]^2 - 2 \cdot X[i, k] \cdot X\_train[j, k] +X\_train[j, k] ^ 2 \right) } } \\
              &= \sqrt{\sum_k{X[i, k]^2} - 2\sum_k{ (X[i, k] \cdot X\_train[j, k])} + \sum_k{X\_train[j, k] ^ 2  } }
\end{split}
\end{equation}
```
可以发现，需要的结果可以分成三部分来计算，代码如下:
```python
import numpy as np
def second_try(X_train, X):
    num_test = X.shape[0]
    num_train = X_train.shape[0]
    A = np.square(X).sum(axis = 1)
    B = np.dot(X, X_train.T)
    C = np.square(X_train).sum(axis = 1)
    dists = np.asarray(np.sqrt(np.matrix(A).T-2*B+C))
    return dists
```
## 运行结果
向量化在性能改进上的作用是显著的。选用`num_train=5000`, `num_test=500`, `D=3072`的一个数据，在型号为`Intel i5-3210m`的 CPU 上进行测试，得到结果如下：
> 直接计算： 59.594000 秒
> 完全向量化计算： 0.808000 秒

减少了大约 `98.7%`的计算时间。