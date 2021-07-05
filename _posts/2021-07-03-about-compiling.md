---
layout: post
title: "编译过程与内存分区"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [编译]
tags: [编译]
math: true
mermaid: true
---

# 编译过程
## 预处理
插入`#include`的头文件、进行宏(`#define`)的替换、删除注释等操作，生成的文件名后缀为`.i`。
```shell
g++ -E main.cpp > main.i  #-E让g++只进行预处理操作
```

## 编译
将预处理过后的文件翻译成汇编文件，文件名后缀`.s`。
```shell
g++ -S main.i  #-S让g++只进行编译操作
```

## 汇编
将汇编语言文件翻译成机器码的二进制文件。
```shell
g++ -c main.s  #-c让g++单独进行汇编操作
```
## 链接
引入代码中使用到的库文件，生成可执行文件。
```shell
g++ main.o -o main #-E让g++只进行预处理操作
```


# 内存分区
全局区、堆区、栈区、常量区、代码区