---
layout: post
title: "Introduction to Parallel Computing-From Algorithms to Programming on State-of-the-Art"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [Parallel Computing]
tags: [Parallel Computing]
math: true
mermaid: true
---


# 1. Why Do We Need Parallel Programming
## 1.1 Why-Every Computer Is a Parallel Computer
Nowadays, all computers are essentially parallel. The parallelism is found on all levels of a modern computer’s architecture:
- 处理器架构层级。 比如SIMD.
- 多核处理器层级。
- many servers contain several multi-core processors.
- even consumer-level computers contain graphic processors capable of running hundreds or even thousands of threads in parallel.

**Four levels of parallelism**:
 - cluster of PCs → MPI
 - multi/many-cores → OpenMP
 - SIMD → intrinsics for vector instructions (SSE, AVX, …)
 - pipelining → needs non dependent instructions

### There are many reasons for making modern computers parallel
- 处理器和内存的频率不可能无限增长，至少以现在的工艺做不到。
- 随着频率增加，功耗也随之增加，能量效率随之下降。但是如果以较低的处理器速度来并行处理计算，对频率提升的需求就可以避免。

## 1.2 How—There Are Three Prevailing Types of Parallelism
### ① **shared memory systems**, i.e., systems with multiple processing units attached to a single memory. 基于Thread model.

### ② **distributed systems**, i.e., systems consisting of many computer units, each with its own processing unit and its physical memory, that are connected with fast interconnection networks. 基于message passing model

### ③ **graphic processor units** used as co-processors for solving general-purpose numerically intensive problems. 基于stream-based model.

## 1.3 What—Time-Consuming Computations Can Be Sped up

- The classical n-body problem
    ```
    Given the position and momentum of each member of a group of bodies at an
    initial instant, compute their positions and velocities for all future instances.
    ```

举了The classical n-body problem的例子，当n=2的时候，可以求出解析解，当n>2的时候，只能求出数值解。

不幸的是，实际场景中n通常非常大。n非常大的情况下，求数值解的计算量就非常大，耗时就非常可观。

这种情况下该怎么办？并行计算(paralell computing)就应运而生。

但是为了成功应用并行计算，需要先回答几个问题：
 - 如何将一个数值求解方法切分成子任务?
 - 每个处理器应该处理哪个子任务?
 - 每个处理器该如何与其他处理器协作？
 - How will we code all of these answers in the form of a parallel program, a program capable of running on the parallel computer and exploiting its resources.

# 2. Overview of Parallel Systems
某个程序在拥有p个计算单元的计算机C(p)上的执行时间：
- Tpar： 并行执行时间
- Tseq： 串行执行时间

满足公式：`Tpar ≤ Tseq ≤ p·Tpar`

