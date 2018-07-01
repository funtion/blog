---
layout: post
date: 2016-12-11 17:22
status: public
tags: [EM, algorithm, probility, statistics, maximum-likelihood, 算法, 统计, 概率, 最大似然]
title: '闲话 EM 算法'
categories: [Machine-Learning]
---

## 1. 从一枚硬币开始

我们总是希望对未知的世界有更多的认识。而现在，我的手里有一枚硬币，可是我不知道它是否是均匀的。我想要知道它投出去以后正面朝上的概率是多少。于是，我抛了 10 次硬币，8 次朝上。我猜想，硬币正面朝上的概率是 80 %。

为什么这样是合理的呢？

我们知道，如果一面硬币正面朝上的概率是 $\theta$，抛掷 n 次，出现  x次正面的概率为

$$
p(x|\theta, n) = C_n^x \theta ^ x (1-\theta)^{(n - x)}
$$


现在 n，m 是已知的，而 $\theta​$ 是我们想知道的。我们相信，概率大的事情更可能发生，于是一个自然的想法是找到一个 $\theta​$ 使得事件发生的概率最大。对 $p(x|\theta, n )​$ 求导可得：
$$
\frac{\partial p}{\partial \theta} = 0 \Rightarrow \theta = \frac{x}{n} = 0.8
$$

## 2. 重复，再重复
但是只进行一次上面的实验，还不足以确定的认可硬币正面朝上的概率。于是我们又把上面的实验进行了很多次，每一次试验中，正面朝上的数量都不尽相同。也就是说，我们得到了正面朝上次数的一个序列 X。

$$
X = {x_1, x_2, \cdots, x_K}
$$

根据上面的思路，我们想要找到一个$\theta$ 使得 $p(|x\theta, n) = \prod_{k=1}^{K}p(x_k|\theta, n)$ 最大。由于里面是乘积的形式，一个常用的技巧是对这个式子求对数，即：

$$
\ln p(X|\theta, n) = \sum_{k=1}^{K} \lbrace C_n^{x_k} + x_k\ln \theta + (n - x_k)\ln(1-\theta) \rbrace
$$

求导可得

$$
\frac{\partial \ln p(X|\theta, n) }{\partial\theta} = \frac{\sum_{k=1}^{K}x_k}{\theta} - \frac{kn - \sum_{k=1}^{K}x_k}{1-\theta} = 0  \Rightarrow \theta = \frac{\sum_{k=1}^{K}x_k}{kn}
$$

这仍然与知觉相符合，即总的正面朝上的概率除以总的抛掷次数。

我们把以上的函数叫做似然函数，方法称为最大似然法 (Maximum likelihood)。
## 3. 如果有很多硬币呢？

看起来我们已经找到了解决这类问题的通用方法。然而事实并非总能如我们想象的美好。现在我们考虑一个更复杂的情况：多枚硬币。

同样的，每次我会从中选出一枚，抛掷 n 次，并记下正面朝上的次数。但是由于硬币外观都一样，并不知道每次实验是用的哪一个。我们把每个硬币被选到的概率记作 $\pi_0, pi_1, \cdots, \pi_M$。在这样的情况下，是否还能计算出最可能的参数呢？

按照之前的方法，我们可以得到：

$$
\ln p(X|\theta,\pi,n) = \sum_{k=1}^{K}\ln \lbrace  \sum_{m=1}^M \pi_m p(x_k|\theta_m,n)  \rbrace
$$

由于对数里面是求和的形式，不能再用上面的方法计算了。

## 4. 引入中间变量

> 任何问题都可以引入中间层来解决。--《程序员的自我修养》

再来仔细观察一下这个新问题，不难发现问题的关键在于多了一个硬币选择的过程。即我们不知道每次选择的是哪个硬币。那么不如先假设我们知道选择的情况。于是我们引入随机变量 Z。为了便于计算，Z 是一个 M 维的01向量且只包含一个1，及 $z \in \lbrace (1, 0, 0, \cdots, 0), (0, 1, 0, \cdots, 0), \cdots, (0, 0, \cdots, 0, 1)  \rbrace$ 。哪一位是1就表示选中的哪个硬币。

那么

$$
p(z) = \prod_{m=1}^{M} \pi_m ^{z_m}
$$

$$
p(x|z) = \prod_{m=1}^Mp(x_k|\theta_m,n)^{z_{m}}
$$

我们就得到了
$$
\ln p(X, Z|\theta,\pi,n) = \sum_{k=1}^{K} \sum_{m=1}^M\ln\lbrace \pi_m^{z_{km}}p(x_k|\theta_m,n)^{z_{km}} \rbrace
$$
这样就可以继续计算下去了。

我们把$(X, Z)$称为一个完整的观察。但是我们观察到的数据只有不完整的$X$。

## 5. Expectation Maximization (EM)

虽然我们的观测数据里面没有 $Z$，但是我们可以根据 $X$ 和 $\Theta$ 得到 $Z$ 的一个分布。

$$
p(Z|X, \Theta) = \frac{p(X|Z,\Theta)p(Z|\Theta)}{\sum_Z p(X|Z,\Theta)p(Z,\Theta)}
$$

其中 $\Theta$ 表示其他所有的参数。

这样我们就能得到似然函数的**期望**

$$
\sum_Z p(Z|X, \Theta) \ln p(X, Z|\Theta)
$$

一切看起来很顺利，但是我们还面临着最后一个问题。只有知道了$\Theta$，我们才能求出 $Z$ 的分布。而我们想要的却是 $\Theta$ 本身。解决这样循环依赖，通常的一个办法，是先从一个地方进行打破这个依赖，然后逐步迭代找到满意的答案。

也就是说按下面的步骤进行计算：

1. 随机选择参数 $\Theta$
2. 写出
    $Q(\Theta, \Theta^{old}) = \sum_Z p(Z|X, \Theta^{old}) \ln p(X, Z|\Theta)$
3. 算出新的 $\Theta$ 使得 $Q$ 最大
4. 重复2，3直到收敛

根据 Jensen 不等式，上面的过程最终会收敛。

## 6. 总结
> Probability theory is nothing but common sense reduced to calculation. -- Laplace