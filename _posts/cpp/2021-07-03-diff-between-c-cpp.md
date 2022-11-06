---
layout: post
title: "C++ vs C"
author: dazuo
date: 2020-07-04 20:19:00 +0800
categories: [C++]
tags: [C++]
math: true
mermaid: true
---

C++之父Bjarne Stroustrup设计C++是为了回答这样的问题：如何直接而高效地操作硬件（如内存管理，进程调度，设备驱动），同时又支持高效的高层次抽象（抽象在代码中体现为函数、类、模板、概念和别名）？

C++在1980年代仅仅是一个基于C和Simula语言功能的组合，在当时的计算机上作为**系统编程**的相对简单的解决方案，经过多年的发展，已经成长为一个远比当年更复杂和有效的工具，应用极其广泛。

C++一直关注的两个问题

- **语言结构到硬件设备的直接映射**
- **零开销抽象**
  - What you don’t use, you don’t pay for .（你不用的东西，你就不需要付出代价）
  - What you do use, you couldn’t hand-code any better.（你使用的东西，你手工写代码也不会更好）

也就成了C++的设计理念。

# **设计理念**

C++的设计理念是同时提供：
- 将内置操作和内置类型直接映射到硬件，从而提供高效的内存利用和高效的底层操作。
- 灵活且低开销的抽象机制，使得用户自定义类型在各方面与内置类型相当。

C++支持4种程序设计风格（编程范式）：
- 过程式程序设计
- 数据抽象：接口的设计与实现细节隐藏。
- 面向对象程序设计，`C`也可以通过一些技术模拟`OOP`，《Object-Oriented Programming With ANSI-C》，但这并不是语言本身的一部分。
- 泛型程序设计

C++ 是一门**偏向系统编程**的通用编程语言，它是

- 更好的C
- 支持数据抽象
- 支持面向对象编程
- 支持泛型编程



在C语言中，“**数据**”和“**处理数据的操作(函数)**”是分开来声明的，也就是说，语言本身并没有支持“数据和函数”之间的**关联性**。



# **应用**

[One of the revolutionary features of C++ over traditional languages is its support for exception handling.](https://www.codeproject.com/Articles/2126/How-a-C-compiler-implements-exception-handling)

# **语义**
C++对某些关键词的语义进行了扩展。如`struct`：在`C`中，`struct`仅仅表示一堆数据的聚合，没有`class-like features`: 没有方法，没有构造函数，没有基类。·C++·继承了`struct`关键字，然后扩展了语义。

https://www.geeksforgeeks.org/difference-between-c-and-c/