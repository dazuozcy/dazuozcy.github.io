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

C++从C发展而来，保留了C的绝大部分特性作为子集。

# **设计理念**

C++的设计理念是同时提供：
- 将内置操作和内置类型直接映射到硬件，从而提供高效的内存利用和高效的底层操作
- 灵活且低开销的抽象机制，使得用户自定义类型在各方面与内置类型相当。

C++支持4种程序设计风格（编程范式）：
- 过程式程序设计
- 数据抽象：接口的设计与实现细节隐藏。
- 面向对象程序设计，`C`也可以通过一些技术模拟`OOP`，《Object-Oriented Programming With ANSI-C》，但这并不是语言本身的一部分。
- 泛型程序设计


# **应用**

# **语义**
C++对某些关键词的语义进行了扩展。如`struct`：在`C`中，`struct`仅仅表示一堆数据的聚合，没有`class-like features`: 没有方法，没有构造函数，没有基类。·C++·继承了`struct`关键字，然后扩展了语义。

https://www.geeksforgeeks.org/difference-between-c-and-c/