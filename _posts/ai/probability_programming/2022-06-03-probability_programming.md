---
layout: post
title: "probability_programming"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [probability_programming]
tags: [AI]
math: true
mermaid: true
---

# Edward2

随机变量`rv`带有一个概率分布，它是一个`TF`分布实例，用于控制随机变量的方法，例如`log_prob`和`sample`.

默认情况下，实例化一个随机变量rv会创建一个采样操作以形成Tensor rv.value ~ rv.distribution.sample()。默认样本数(可通过rv的sample shape参数控制)为1。如果提供了可选的value参数，则不会创建采样操作。



[Facebook的时间序列预测算法Prophet：Forecasting at scale](https://zhuanlan.zhihu.com/p/492992712)



[深度学习vs概率编程](https://zhuanlan.zhihu.com/p/234931176)

**将确定的权重值变成分布**

深度学习模型具有强大的拟合能力，而贝叶斯理论具有很好的可解释能力，将两者结合，通过设置网络权重为分布、引入隐空间分布等，可以对分布进行采样前向传播，由此引入了**不确定性**，因此，增强了模型的**鲁棒性**和**可解释性**。



# probability vs likelihood

假设有一个参数为 $\theta$ 的概率模型，$p(x|\theta)$ 有两个名称：

- $x$ 的概率(给定 $\theta$ )
- $\theta$ 的可能性(假设观察到 $x$)

似然度是 $\theta$ 的函数。下面是几个简单的用途：

如果你观察 $x$，并想估计产生它的 $\theta$，最大似然原理就是说要选择最大似然 $\theta$，也就是使 $p(x|\theta)$ 最大化的$\theta$。

这与最大后验或`MAP`形成对比，后者是使 $p(\theta | x)$ 最大化的 $\theta$。由于 $x$ 是固定的，这相当于最大化 $p(\theta) p(x | \theta)$，即 $\theta$ 的先验概率与 $\theta$ 的似然的乘积。
