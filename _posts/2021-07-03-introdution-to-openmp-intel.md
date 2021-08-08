---
layout: post
title: "Introduction to openMP"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [openMP]
tags: [openMP]
math: true
mermaid: true
---

这是一门非常优秀的OpenMP入门教程，讲解人也曾参与过OpenMP的开发。

# 1. Introduction
课程是`简洁的lectures + 简短的execise`的方式，讲究边学边练边掌握。

课程由五大模块组成，每个模块由一系列单元和讨论组成：
- Getting Started with OpenMP
- The Core Features of OpenMP
- Working with OpenMP
- Advanced OpenMP Topics
- Recapitulation

总共27个lectures.

# 2. Introduction to Parallel Programing Part1
摩尔定律说集成电路上可以容纳的晶体管数量大约每18个月便会增加一倍。芯片上更多的晶体管数量带来了更优秀的性能，作者认为以前程序的性能来自于芯片硬件。
你可以随心所欲的写你的软件，不用考虑性能，把性能留给了芯片硬件。更先进（晶体管数量更多）的芯片，更优秀的性能。

但是有一点不得不考虑，那就是功耗。根据Intel的研究，功耗和性能有这样一个拟合关系`power = perf^1.74`。

![image](../img/openMP/introduction-to-openmp-intel/comparison-between-two-archs.png){: width="1086" height="542"}


| 单核架构             | 多核架构                  |
|---------------------|:---------------------------------:|
|Capacitance = C <br> Voltage = V <br>Frequency = F <br>Power = CV^2F | Capacitance = 2.2C <br> Voltage = 0.6V <br>Frequency = 0.5F <br>Power = 0.396CV^2F | 

对比可以看出多核架构使我们能够以更低的频率完成相同的工作，同时节省大量的功耗。


# 3. Introduction to Parallel Programing Part2

## 并发与并行

| Concurrency             | Parallelism                  |
|---------------------|:---------------------------------:|
| A condition of a system in which multiple tasks are **logically** active at one time.| A condition of a system in which multiple tasks are **actually** active at one time. | 

|Concurrent Applications   |Parallel Applications|
|--------------------------|---------------------|
|An application for which computations logically execute simultaneously due to the semantics of the application. <br> The problem is fundamentally concurrent. | An application for which the computations actually execute simutanenously in oder to compute a problem in less time. <br> The problem doesn't inherently require concurrency...you can state it sequenttially.|

## OpenMP
- An API for writing multithreaded applications.
- A set of compiler directives and library routines for parallel applications programmers.
- Greatlt simplifies writing multi-threaded(MT) programs in Fortran, C and C++.

### OpenMP Solutuion Stack

![OpenMP solution stack](../img/openMP/introduction-to-openmp-intel/openmp-solution-stack.png?raw=true){: width="1086" height="542"}

### OpenMP Core syntax
- Most of the constructs in OpenMP are compiler directives.
- `#pragma omp construct [clause [clause]...]`
- `#pragma omp parallel num_threads(4)`
- `#include <omp.h>`
- Most OpenMP constructs apply to a "Structured Block".
- "Structured Block": A block of one or more statements with one point of entry at the top and one point of exit at the bottom.


# 4. The Boring Bits: Using an OpenMP Compiler(Hello World)
```bash
gcc -fopenmp foo.c

export OMP_NUM_THREADS=4

./a.out
```
## Exercise
Verify that your OMP environment works, write a multithreaded program that prints "Hello World".

# 5. Discussion1-Hello World and How Threads Work

```c
#include <stdio.h>
#include <omp.h>

int main()
{
  #pragma omp parallel
  {
    int ID = omp_get_thread_num();
    printf("hello(%d)", ID);
    printf(" world(%d) \n", ID);
    return 0;
  }
}
```
`#pragma omp parallel` asks for the default num of threads.
`omp_get_thread_num()` gets a unique identifier for each thread. range [0,N].

## Shared Memory Computer
Any computer composed of multiple processing elements that share an address space. There are two classes:
### Symmetric Multiprocessor(SMP)
A shared address space with "equal-time" access for each processor, and the OS treats every processor the same way.

### Non Uniform Address Space Multiprocessor(NUMA)
Different memory regions have different access costs...think of memory segmented into "Near" and "Far" momory.

## OpenMP Overview
- OpenMP is a multi-threading, shared address model.
- Threads communicate by sharing variables.
- Unintended sharing of data causes race conditions.
- Race conditions: when the program's outcome changes as the threads are scheduled differently.
- To control race conditioins, use synchronization to protect data conflicts.
- Synchronization is expensive.

# 6. Creating Threads(The Pi Program)

## fork-join-parallellism

![fork-join-parallellism](../img/openMP/introduction-to-openmp-intel/openmp-fork-join-parallellism.png?raw=true){: width="542" height="242"}

上图中蓝色背景的部分就称为**并行域**， 里面是**a team of threads**.
总体就是在某个时刻fork若干个线程, 在另外某个时刻join到一起。

下面是一个简单的例子
![openmp-execution-model-example](../img/openMP/introduction-to-openmp-intel/openmp-execution-model-example.png?raw=true){: width="542" height="242"}

## Exercise
把下面这个串行版本的计算`Pi`值的程序改成并行版本
```c
void calc_pi_serial()
{
    long num_steps = 0x20000000;
    double sum = 0.0;
    double step = 1.0 / (double)num_steps;

    double start = omp_get_wtime( );    
    for (long i = 0; i < num_steps; i++) {
        double x = (i + 0.5) * step;
        sum += 4.0 / (1.0 + x * x);
    }    
    double pi = sum * step;
    
    printf("pi: %.16g in %.16g secs\n", pi, omp_get_wtime() - start);
    // will print "pi: 3.141592653589428 in 5.664520263002487 secs"
}
```

# 7. Discussion2-The Simple Pi Program and Why the Performance is so Poor

# 8. Synchronization(Pi Program Revisited)

# 9. Discussion3-Synchronization Overhead and Eliminating False Sharing

# 10. Parallel Loops(Making the Pi Program Simple) Part1

# 11. Parallel Loops(Making the Pi Program Simple) Part2

# 12. Discussion4-Pi Program Wrap-Up

# 13. Barriers...and then More Workshare Constructs

# 14. Locks in OpenMP

# 15. OpenMP Runtime Library Routines

# 16. Environment Variablesin OpenMP

# 17. Data Environment

# 18. Discussion5-Debugging OpenMP Programs

# 19. Skill Practice: Linked Lists and OpenMP

# 20. Discussion6-Different Ways to Traverse Linked Lists

# 21. Tasks(Linked List the Easy Way)

# 22. Discussion7-Understanding Tasks

# 23. The Scary Stuff: Memory Model, Atomics, and Flush(Pairwise Synch)

# 24. Discussion8-The Pitfalls of Pairwise Synchronization

# 25. Threadprivate Data and How to Support libraries(Pi Again)

# 26. Discussion9-Random Number Generators

# 27. Recapitulation