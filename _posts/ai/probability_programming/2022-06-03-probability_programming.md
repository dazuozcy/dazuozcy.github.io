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



## 源码阅读

### `Normal`的`__init__`里没有`sample_shape`参数，这里为什么传`sample_shape`？

```python
beta = ed2.Normal(loc=0.0, scale=2.0, name='beta', sample_shape=3)
```

这里并不是直接调用了`Normal`分布的`__init__`接口，而是调用的`make_log_joint_fn`里的`interceptor`，

`rv_constructor`就是`make_random_variable`，根据名字知道其作用是构造随机变量。

在构造随机变量时，会把`sample_shape`从参数列表里弹出，然后调用`Normal`分布的`__init__`接口。

源码如下所示(`edward2/tensorflow/generated_random_variables.py`)：

```python
def make_random_variable(distribution_cls):
  """Factory function to make random variable given distribution class."""
  @traceable
  @functools.wraps(distribution_cls, assigned=("__module__", "__name__"))
  @expand_docstring(cls=distribution_cls.__name__,
                    doc=inspect.cleandoc(
                        distribution_cls.__init__.__doc__ if
                        distribution_cls.__init__.__doc__ is not None else ""))
  def func(*args, **kwargs):
    # pylint: disable=g-doc-args
    """Create a random variable for ${cls}.

    See ${cls} for more details.

    Returns:
      RandomVariable.

    #### Original Docstring for Distribution

    ${doc}
    """
    # pylint: enable=g-doc-args
    sample_shape = kwargs.pop("sample_shape", ())  # 弹出sample_shape参数
    value = kwargs.pop("value", None)
    return RandomVariable(distribution=distribution_cls(*args, **kwargs),
                          sample_shape=sample_shape,
                          value=value)
  return func
```



###  `object_y`传给`y_obs`后在哪里用？

```python
object_y = tf.convert_to_tensor(y, name='object_y', dtype=tf.float32)

def target(k, m, beta, delta, sigma):
    return log_joint(K=K, S=S, tau=tau, trend_indicator=0, k=k, m=m,
                     beta=beta, delta=delta, sigma=sigma, y_obs=object_y)

energy = -target(k, m, beta, delta, sigma)
```

`log_joint`的调用中`object_y`传给`y_obs`参数后，在哪里用的？

在`prophet_model`中，通过`name`参数捕获到`y_obs`(等号右边的)，将其作为随机变量`y_obs`(等号左边的)的`value`。

```python
def prophet_model(xxx):
    ...
    y_obs = ed2.Normal(name='y_obs', loc=Y_new, scale=sigma)
    return y_obs, {k, m, beta, delta, sigma}
```





`log_joint_func`中调用分布的`log_prob`，策略是如果分布有实现`_log_prob`接口，则直接调用`_log_prob`，否则再看有没有实现`_prob`接口，则调用`_prob`接口，并对结果求对数。



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
