---
layout: post
title: "时间序列预测"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [时间序列预测]
tags: [时间序列]
math: true
mermaid: true
---

# 时间序列

**时间序列**是按时间顺序出现的有序数列。数列就是**某一统计指标**的数值，比如：工厂订单数量、股票价格、网页访问量等。



# 时间序列预测

时间序列的**待预测变量**称为**观察值**。

时间序列预测是指：基于历史数据预测未来的观察值。预测分为2种：

- 点预测。预测某一时间点上的具体数值
- 区间预测。预测某一时间点上数值的区间。



时间序列**可预测**的前提：

- 了解时间序列观测值的影响因素
- 有可利用的历史数据
- 预测不会影响试图预测的结果



时间序列**可定量预测**的前提：

- 过去的一些模式会在未来延续下去。



# 术语

- 观察值

  时间序列需要预测的值。

- 外部变量

  影响观察值的其他变量。

- 误差

  无法被解释的量。

- 趋势

  当一个时间序列长期增长或下降时，表示该序列有趋势。

- 季节性

  当时间序列中的数据受到季节性因素的影响时，表示该序列具有季节性。季节性总是一个**已知并且固定的频率**。

- 周期性

  当时间序列存在**不固定频率**的上升和下降时，表示该序列有周期性。



# 时间序列的评估

时间序列不能做交叉验证，得做`back test`。在量化中又称为**回测**。原因是为了避免`data leak`，数据穿越。

有`R2`、`MASE`、`MSSE`等。

- $R^{2}$ 
  $$
  R^{2} = 1 - \frac{\frac{1}{n} \sum_{i=1}^{n}(y_{i} - \hat{y}_{i})^2}
                   {\frac{1}{n} \sum_{i=1}^{n}(y_{i} - \bar{y}_{i})^2}
  $$
  

​		其中，$\hat{y}_{i}$ 是预测值，$\bar{y}_{i}$ 是平均值。 $R^{2}$ 越趋近与1，表示模型越准。



- `MASE`(平均绝对标准误差)
  $$
  MASE = \frac{\frac{1}{T} \sum_{t=1}^{T} \vert\hat{y}_{t} - \hat{y}_{t} \vert}
              {\frac{1}{T-1} \sum_{t=2}^{T} \vert y_{t} - y_{t-1} \vert}
  $$



- `MSSE`(平均平方标准误差)
  $$
  MSSE = \frac{\frac{1}{T} \sum_{t=1}^{T} (\hat{y}_{t} - \hat{y}_{t})^{2}}
              {\frac{1}{T-1} \sum_{t=2}^{T} (y_{t} - y_{t-1})^{2}}
  $$



# 时间序列与机器学习

## 时间序列的产生

从概率上看，时间序列是随机过程在时间方向上的一次采样，这个采样只能随着时间往后，无法在空间上重复。

时间序列的建模就是期望通过这样的采样数据来学习模型，对后面的时间进行预测。

## 机器学习如何学习

从概率上看，有监督的学习的训练样本就是 $P(y \vert x)$ 在空间方向上的采样。这个采样可以不断重读，没有时间先后的概念。

机器学习的建模就是期望通过这样的采样数据来学习模型，对后面的样本进行预测。



- 机器学习中的建模一般需要假设样本独立同分布，依托**大数定律**拟合出参数。
- 时间序列中的建模一般需要假设样本平稳遍历性，依托**遍历定理**拟合出参数。



# Papers

- 有历史数据时，如何做预测
  - 入门：`Prophet`
  - 进阶：`DeepAR`
  - 最新：`Informer`
- 没有历史数据时，如何做预测
  - `Multi-modal Forecasting`
- 不同颗粒度的预测如何整合
  - 入门：`Reconciliation hierarcgical model`
  - 进阶：`structured regularization model`





# 参考

[时间序列入门](https://www.bilibili.com/video/BV1je4y1d7uH/?spm_id_from=pageDriver&vd_source=acf87e8d5611bbe9a378ee5a1b89c6ce)

