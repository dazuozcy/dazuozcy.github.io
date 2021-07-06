---
layout: post
title: "面向对象的特性"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [C++]
tags: [面向对象]
math: true
mermaid: true
---

封装，继承，多态

## 类的访问权限：`private`, `protected`, `public`
- `public`：可以被该类中的函数、友元函数、子类的函数访问，也可以由该类的对象访问；
- `protected`：可以被该类中的函数、友元函数、子类的函数访问，但不可以由该类的对象访问；
- `private`：可以被该类中的函数、友元函数访问，但不可以由子类的函数、该类的对象访问。


## `C++`里`struct`与`class`区别
成员的默认访问控制权限不同，在`struct`中是`public`，在`class`中是`private`.
何时用`struct`,何时用`class`：
- use struct for POD(plain-old-data) structures without any class-like features.
- Use class when you make use of feathers such as private or protected members, non-default constructors and operators, etc.
