---
layout: post
date: 2017-02-17 22:51
status: public
tags: [logistic, regression, sigmoid, bayesian, probility, softmax]
title: '理解 Logistic 回归的原理'
categories: [Machine-Learning]
---

我们总是感叹 logistic 回归的公式的优雅与简洁:

$$
P(Y) =  \frac{1}{1 + e^{-X^TW}} = Sigmoid(X^TW)
$$

可是他是怎么想到用这样一个公式，去建模分类问题的概率，并取得很好的成果？

这个神奇的 Sigmoid 函数到底从何而来？

使用贝叶斯概率，可以对此有一个优雅的解释。

首先来看贝叶斯公式

$$
P(A_i|B) = \frac{P(B|A_i)*P(A_i)}{P(B)} =  \frac{P(B|A_i)*P(A_i)}{\sum_z P(B|A_z)P(A_z)}
$$

那么对于一个2分类问题，就是

$$
P(c_1|X) = \frac{P(X|c_1)*P(c_1)}{P(X|c_1)*P(c_1)+P(X|c_2)*P(c_2)}
$$

如果我们把分子变成 $1$, 就得到
$$
P(c_1|X) = \frac{1}{1 + \frac{P(X|c_2)*P(c_2)}{P(X|c_1)*P(c_1)}}
$$
仔细观察一下这个式子，发现只有分母的第二项含有 $X$ 。如果我们想建立一个最简单的线性模型来表示它，那么第一个想法便是

$$
P(c_1|X) = \frac{1}{1 + X^TW} ，其中 X^TW = \frac{P(X|c_2)*P(c_2)}{P(X|c_1)*P(c_1)}
$$

可惜这样是不正确的。因为概率乘除得到的结果总是一个正数，而 $X^TW$ 的计算结果可以是任意一个实数。那么，怎么把一个实数映射到正整数呢？

$$
y = e^x
$$

当然绝对值函数也是一个满足要求的选择，但是它不单调，会给后来的计算带来很多的问题。

于是我们就有了

$$
P(c_1|X) = \frac{1}{1 + e^{X^TW}} ，其中 X^TW = \ln{\frac{P(X|c_2)*P(c_2)}{P(X|c_1)*P(c_1)}}
$$

当然这里还遗留了一个小问题，就是随着 $WX$ 的增大，$$ P(c_1 \| x) $$的值是变小的。为了更加符合直观感受，通常我们用的是$y = e^{-x}$ 作为变换。这样就得到在最开始给出的公式。

进一步的，从

$$
P(A_i|B) =   \frac{P(B|A_i) * P(A_i)}{\sum_z P(B|A_z)P(A_z)}
$$

也不难得理解多分类问题的 Softmax 公式

$$
P(c_j|X) = \frac{e^{X^TW_j}}{\sum_k e^{X^TW_k}}
$$
