---
layout: post
title: "构造函数与析构函数"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [C++]
tags: [构造函数, 析构函数]
math: true
mermaid: true
---

# 介绍

需要一个机制来建立一个工作环境（构造函数）和一个逆操作来释放运行期获得的资源（析构函数）。以下摘自1979 年的实验记录本：

> - “new 函数”为成员函数创建运行的环境
> - “delete 函数”则执行相反的操作



## 构造函数

### 为什么需要构造函数
在C里面，经常需要给变量赋初值，也就是变量的初始化。如果变量比较多，一般会有个`init()`函数将所有的变量都初始化一遍。

在C++里，一个类对象也有或多或少的成员变量，它们也需要被初始化。但是如果使用像`init()`这样的函数对类对象提供初始化功能既不优雅也容易出错。这种方式没有规定一个对象必须进行初始化，程序员可能忘记初始化，或者初始化了多次。

一种更好的方法是允许程序员声明一个函数，它显式表明自己是专门完成对象初始化任务的。所以C++中引入了这种本质上就是构造一个给定类型的值的函数`构造函数`。

### 作用
构造函数主要在创建对象时完成对对象属性的初始化。构造函数一般有三方面的作用：
- 给创建的对象建立一个标识符；
- 为对象数据成员开辟内存空间；
- 完成对象数据成员的初始化。


http://events.jianshu.io/p/d0e4d43fa77d
https://www.yanbinghu.com/2019/08/11/25996.html


## 析构函数



### 构造函数初始化列表与构造函数体内复制的区别

### 构造函数是否可以放到private里面

### 平凡构造函数，非平凡构造函数

### 移动构造函数和拷贝构造函数对比

### 析构函数不加virtual会发生什么，原理