---
layout: post
title: "二进制优化"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [LLVM]
tags: [binary optimization]
math: true
mermaid: true
---

二进制优化工具[BOLT](https://github.com/facebookincubator/BOLT)是`Facebook`开发的基于`LLVM`的工具，它利用反馈数据(`perf`采样或插桩)优化二进制布局以提高应用程序性能。

BOLT对应用的优化作用主要来自与BB块重排、函数重排、冷热分区。

# BB块

BB块(Basic Block)是编译器对代码进行编译分析的最小单元。编译器把程序拆分为许多BB块进行分析。

BB块组成了一个程序控制流图的节点，具有一下特征：

- 只有一个入口，即程序中不会有任何其他地方能通过jump等跳转类指令进入到此BB块。
- 只有一个出口，即程序中只有最后一条指令能触发进入到其他BB块。

所以，BB块是一个程序顺序执行时跳转的基本单元。

```c
function foo(int x) {
	if (x > 0) {
		... // B1
	} else {
		... // B2
	}
}
function bar() {
	foo(... /* > 0 */); // gets inlined
}
function baz() {
	foo(... /* < 0 */); // gets inlined
}
```

使用objdump工具可以得到二进制中的汇编指令，按照：

- 根据jump、call、ret等指令得到BB块的出口。
- 根据jump、call、ret等指令的目标或函数的开头得到BB块的入口。



# BB块重排

BB块重拍目的是优化函数内部的控制流图，使得函数在完成相同功能的前提下执行最少的判断和跳转，降低不必要的性能损失。



# 函数重排

函数重排目的是优化函数的调用，通过对指令和函数的相对位置重新排布，使二进制代码局部性更优。

该优化主要提升了I-TLB的性能，I-cache也会有小幅提升。

操作系统以页为单位将可执行文件加载进内存，CPU以cacheline为单位将内存中的代码段加载进Cache。当程序发生函数调用时，若跳转的目标地址超出cache甚至页表范围，则会引发一次cache miss或TLB miss，导致CPU空闲(需要等待取指令)。

函数重排，即调整函数的相对位置，尽量使得目标地址在一个cache或一个页表范围内来减少I-cache miss和I-TLB miss，以提升程序性能。



# 冷热分区

把频繁调用的函数代码段放在一起，称之为热函数区。那些不常用的函数代码段放在一起，称之为冷函数区。相对函数重排，是更大粒度的重排。



# 反馈数据

只有获取到函数或者BB块的运行次数即跳转信息，作为依据，才能实现函数或者BB重排。

当前获取反馈数据的两种方式：

- 插桩

  BOLT在二进制中所有BB块或函数跳转的地方插入计数指令，统计整个程序的实际执行情况。插桩方式比较准确，但是非常影响程序性能。

- perf

  利用perf工具采样。获取的反馈数据精确度较低。



# 参考文献

[BOLT: A Practical Binary Optimizer for Data Centers and Beyond](https://arxiv.org/abs/1807.06735)

