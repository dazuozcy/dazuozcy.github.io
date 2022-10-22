---
layout: post
title: "图缓存"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [性能优化]
tags: [图缓存]
math: true
mermaid: true
---

`PyNative`模式下，模型是一个算子一个算子下发执行。每个`step`里每个算子都要先编译再执行。非动态`shape`场景下(大部分情况)，某个算子在`step`之间除了输入数据不同，其他都一样。这种情况下，第二个`step`开始的算子编译就属于重复工作，没有必要了。通过缓存机制，就可以节省第二个`step`开始的算子编译时间。

