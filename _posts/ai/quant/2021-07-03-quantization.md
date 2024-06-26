---
layout: post
title: "量化"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [AI]
tags: [量化]
math: true
mermaid: true
---

量化根据是否需要重训练，分为：

- **感知量化训练**，优点：模型准确率更好，适用于对模型压缩率和准确率要求较高的场景。

- **训练后量化**，优点：简单易用，适用于追求高易用性和缺乏训练资源的场景。



# 训练后量化

`Post-training Quantization`, **PAT**，或者**离线量化**。

在模型训练结束后进行的量化，对训练后模型中的权重由浮点数(如float32)量化到低比特整数(如int8/int4)，并通过少量校准数据基于推理过程对数据(`activation`)进行校准量化，从而加速模型推理速度。

通常，训练后的模型权重已经确定，因此可根据权重的数值离线计算得到权重的量化参数。

而通常数据是在线输入的，无法准确获取数据的数值范围，通常需要一个较小的有代表性的数据集来模拟在线数据的分布，利用数据集执行前向推理，得到对应的中间结果，并根据这些浮点结果离线计算出数据的量化参数。



量化根据对象的不同，分为：

- 权重(weight)量化
- 数据(activation)量化



## 权重量化

是指根据权重的数值分布情况，将权重处理到低比特。

训练后量化场景通过离线量化的方式，直接从用户推理模型中读取权重数据，然后调用量化算法对权重进行量化，并把量化后的数据回写到模型中。

推理时，在内存初始化阶段，对网络模型中的权重进行反量化操作变成float进行正常的推理。



## 数据量化

是指根据数据的数值分布情况，将输入数据(activation)处理到低比特。每一层的数据分布是未知且巨大的，只能在前向过程中确定，因此数据量化是基于推理或者训练过程的。

在训练后量化场景中，通过在线量化的方式，通过修改用户推理模型，在待量化层位置插入旁路量化节点，采集待量化层的输入数据，然后校准得到数据量化因子scale, offset。在推理时一般使用少量数据集，代表所有数据集的分布。



## 校准

在训练后量化场景中，做前向推理获取数据量化因子的过程。



## 量化算法

权重量化

- `ARQ`算法(`Adaptive Range Quantization`)
- `NUQ`算法(`Non Uniform Quantization`)

数据量化

- `IFMR`算法(`Input Feature Map Reconstruction`)
- `HFMG`算法(`Histogram Feature Map Glutton`)

### `ARQ`算法

对权重直接量化的方法。有2种方式：`channel_wise`和`non-channel_wise`。

- `channel_wise`：所有filter一起分析数据分布，进行量化，公用一个量化因子。

- `non channel_wise`：每个filter独立做数据分析，进行量化，有独立的量化因子。

### `NUQ`算法

是对权重进行非等间距量化的一种方法。

均匀量化将权重数据从32bit量化到8bit，NUQ在此基础上挑选部分量化台阶。

### `IFMR`算法

在某个数据分布下，通过搜索的方式确定最佳量化方式。

- 1。将浮点数截断到[clip_min, clip_max]范围。
- 2。将浮点数据量化到int范围。

### `HFMG`算法

通过直方图的方式来记录激活数据的数据分布，通过搜索的方式确定最佳量化截断位置。

- 1。根据输入数据创建直方图。
- 2。若有多个batch的数据，则对每个batch的数据创建一个直方图，然后进行直方图合并。
- 3。基于创建的直方图，根据搜索的方式确定数据的截断点。



# **量化感知训练**

`Quantization Aware Training`, **QAT**，或者**在线量化**。

MindSpore的感知量化训练是一种伪量化的过程，它在模型中**需要记录量化参数的地方**插入**伪量化节点**，其作用有二：

- 统计流经算子节点的数据的最大最小值，找到输入、权重等待量化数据的分布；
- 模拟量化模型在量化到低比特时引入的精度损失(来自rounding和clamping操作)，将该损失作用到网络模型中，传递给损失函数，让优化器在训练过程中对该损失值进行优化，从而提高模型对量化效应的适应能力，获得更高的量化模型精度。



一般对于权重的量化使用对称量化，对于激活的量化使用非对称量化。原因是对于权重而言，一般是对称的，如`[-6.0, 6.0]`，采用对称量化有更少的计算量。对于激活而言，其值分布在一侧，比如ReLU输出就是`[0.0, +∞)`，如果采用对称量化，则会比较浪费。



## 对称量化

原始float数据与量化后的INT8数据之间的转换：
$$
scale = \frac{2 \times max(\vert x_{min} \vert, x_{max})}{Q_{max}-Q_{min}}
$$


$$
offset = 0
$$

$$
int = round( \frac{float}{scale})
$$

$$
float = scale \times int
$$




## 非对称量化

原始float数据与量化后的INT8数据之间的转换：
$$
scale = \frac{x_{max}-x_{min}}{Q_{max}-Q_{min}}
$$

$$
offset = Q_{min} - round(\frac{x_{min}}{scale})
$$

$$
float = scale \times (uint8 + offset)
$$

$$
uint8 = round(\frac{float}{scale}) - offset
$$
MindSpore中的实现参见:

- `mindspore/python/mindspore/compression/quant/quant_utils.py`
- `mindspore/ccsrc/plugin/device/gpu/kernel/cuda_impl/cuda_ops/fake_quant_perlayer_impl.cu`
- `mindspore/mindspore/ccsrc/plugin/device/gpu/kernel/cuda_impl/cuda_ops/fake_quant_perchannel_impl.cu`
- `mindspore/python/mindspore/ops/_op_impl/_custom_op/fake_quant_perlayer.py`
- `mindspore/python/mindspore/ops/_op_impl/_custom_op/fake_quant_perchannel.py`
- `mindspore/ccsrc/plugin/device/gpu/kernel/cuda_impl/cuda_ops/minmax_update_impl.cu`
- `mindspore/python/mindspore/ops/_op_impl/_custom_op/minmax_update_perlayer.py`





# 通道稀疏





# 张量分解







# 模型部署优化

主要为算子融合，是指通过数学等价，将模型中的多个算子运算，融合成单算子运算，以减少实际前向过程中的运算量。

- Conv+Bn融合，融合后的Bn层会被删除。
- Bn+Mul融合，融合后的Mul层会被删除。
- Bn+Add融合，融合后的Add层会被删除。



# INT4部署时遇到的问题

## 部分Conv用INT4量化后，性能比INT8劣化

INT8量化时，Dequant(INT32->Fp16)和Quant(Fp16->INT8)可以融合成Requant算子。

但是INT4量化时，先做mad(INT32)，然后Dequant(INT32->Fp16)和Quant(Fp16->INT4)，由于硬件没有INT32->INT4的指令，所以无法将Dequant和Quant融合成Requant算子，导致INT4相比INT8，vector计算量增大很多，所以部分Conv性能反而是负收益。
