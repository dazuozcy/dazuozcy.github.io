---
layout: post
title: "AdamWeightDecay fission"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [fission]
tags: [AdamWeightDecay, fission]
math: true
mermaid: true
---

- `Ascend`上`AdamWeightDecay`优化器是通过小算子组成而成，代码见附录。
- `Pynative`模式下正向过程完全是按照Python语法进行执行，对于某个小算子，就是从Python侧通过pybind11调用C++侧实现。所以多个小算子就会涉及到Python侧与C++侧的多次切换。

基于上面2点，有一种`Pynative`模式下网络训练性能优化的思路：在Python侧把`AdamWeightDecay`做成一个大算子，然后通过后端Fission pass把大算子拆分成对应的小算子。这样就大大减少了Python侧与C++侧切换次数，提升了性能（Bert Base 单step提升500ms）。



# 附录

[MindSpore AdamWeightDecay 优化器算子官网介绍](https://mindspore.cn/docs/zh-CN/r1.8/api_python/ops/mindspore.ops.AdamWeightDecay.html?highlight=AdamWeightDecay)

[Fission 代码](https://gitee.com/mindspore/mindspore/pulls/14247)

`AdamWeightDecay`小算子实现：

```cpp
# mindspore/python/mindspore/nn/optim/adam.py
@_adam_opt.register("Tensor", "Tensor", "Tensor", "Tensor", "Tensor", "Tensor", "Tensor", "Tensor",
                    "Tensor", "Bool", "Bool")
def _update_run_op(beta1, beta2, eps, lr, weight_decay, param, m, v, gradient, decay_flag, optim_filter):
    """
    Update parameters.

    Args:
        beta1 (Tensor): The exp decay rate for the 1st moment estimations. Should be in range (0.0, 1.0).
        beta2 (Tensor): The exp decay rate for the 2nd moment estimations. Should be in range (0.0, 1.0).
        eps (Tensor): Term added to the denominator to improve numerical stability. Should be greater than 0.
        lr (Tensor): Learning rate.
        weight_decay (numbers.Number): Weight decay. Should be equal to or greater than 0.
        param (Tensor): Parameters.
        m (Tensor): m value of parameters.
        v (Tensor): v value of parameters.
        gradient (Tensor): Gradient of parameters.
        decay_flag (bool): Applies weight decay or not.
        optim_filter (bool): Applies parameter update or not.

    Returns:
        Tensor, the new value of v after updating.
    """
    op_cast = P.Cast()
    if optim_filter:
        op_mul = P.Mul()
        op_square = P.Square()
        op_sqrt = P.Sqrt()
        op_cast = P.Cast()
        op_reshape = P.Reshape()
        op_shape = P.Shape()
        param_fp32 = op_cast(param, mstype.float32)
        m_fp32 = op_cast(m, mstype.float32)
        v_fp32 = op_cast(v, mstype.float32)
        gradient_fp32 = op_cast(gradient, mstype.float32)

        next_m = op_mul(beta1, m_fp32) + op_mul(op_cast(F.tuple_to_array((1.0,)), mstype.float32)
                                                - beta1, gradient_fp32)

        next_v = op_mul(beta2, v_fp32) + op_mul(op_cast(F.tuple_to_array((1.0,)), mstype.float32)
                                                - beta2, op_square(gradient_fp32))

        update = next_m / (eps + op_sqrt(next_v))
        if decay_flag:
            update = op_mul(weight_decay, param_fp32) + update

        update_with_lr = op_mul(lr, update)
        next_param = param_fp32 - op_reshape(update_with_lr, op_shape(param_fp32))

        next_param = F.depend(next_param, F.assign(param, op_cast(next_param, F.dtype(param))))
        next_param = F.depend(next_param, F.assign(m, op_cast(next_m, F.dtype(m))))
        next_param = F.depend(next_param, F.assign(v, op_cast(next_v, F.dtype(v))))

        return op_cast(next_param, F.dtype(param))
    return op_cast(gradient, F.dtype(param))
```

