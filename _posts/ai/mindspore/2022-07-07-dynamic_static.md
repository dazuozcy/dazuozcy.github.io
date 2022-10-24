---
layout: post
title: "动静态图"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [MindSpore]
tags: [动静态图]
math: true
mermaid: true
---

目前主流的深度学习框架有静态图(Graph)和动态图(**PyNative**)两种执行模式。

- 静态图模式下，程序在编译执行时，首先生成神经网络的图结构，然后再执行图中涉及的计算操作。因此，在静态图模式下，编译器可以通过使用图优化等技术来获得更好的执行性能，有助于规模部署和跨平台运行。

- 动态图模式下，程序按照代码的编写顺序逐行执行，在执行正向过程中根据反向传播的原理，动态生成反向执行图。这种模式下，编译器将神经网络中的各个算子逐一下发到设备进行计算操作，方便用户编写和调试神经网络模型。

  

在MindSpore中，动态图模式又称为PyNative模式，静态图模式又被称为Graph模式。

PyNative模式的正向执行过程是按照Python的语法，逐语句执行。每一条Python语句执行完，都能够得到该条语句的执行结果。因此，在PyNative模式下，用户可以逐指令，或者在特定的指令位置调试网络脚本。

Graph模式下，MindSpore通过源码转换的方式，将Python源码转换成IR，再基于IR进行相关的图优化，变成最终在硬件上执行的图。

|              | 优点                                 | 缺点                                                         |
| ------------ | ------------------------------------ | ------------------------------------------------------------ |
| PyNative模式 | 支持任意Python语法，方便调试         | 性能较差                                                     |
| Graph模式    | 可以针对全局图进行图优化与，性能较好 | [支持的Python语法有限](https://www.mindspore.cn/docs/zh-CN/r1.8/note/static_graph_syntax_support.html)，调试较不方便 |



# 执行方式

## PyNative模式

在动态图下，执行正向过程完全是按照Python的语法执行的，而反向传播过程是基于Tensor实现的。因此，我们在执行正向过程中，将所有应用于Tensor的操作记录下来，并针对每个计算操作求取其反向，然后将所有反向过程串联起来形成整体反向传播图，最终将反向图在设备上执行并计算出梯度。

- 正向按照python语法执行python语句，通过pybind调用后端C++算子kenel接口
- 反向在正向执行过程中进行构图，执行完正向后按照Cell粒度构图后生成最终的反向整图
- 在生成过程中，将Cell粒度的图进行缓存，在非首个step时可以避免重复构图
- 执行反向图



## Graph模式

- 正向生成整图，经过编译优化后形成最终的正向图
- 反向生成整图，经过编译优化后形成最终的反向图
- 若出现条件逻辑或者动态shape，多次编译生成多张正向和反向图
- 执行正向图
- 执行反向图



MindSpore为了动静统一，解决两种模式不兼容的问题，针对动态图和静态图模式：

- 统一API，在两种模式下使用相同的API
- 统一动静态图的底层微分机制



# 参考

[PyNative 调试](https://mindspore.cn/tutorials/experts/zh-CN/master/debug/pynative_debug.html?)

[动静态图](https://mindspore.cn/tutorials/zh-CN/r1.8/advanced/pynative_graph/mode.html?)

