---
layout: post
title: "图缓存"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [MindSpore]
tags: [图缓存]
math: true
mermaid: true
---

`PyNative`模式下，模型是一个算子一个算子下发执行。每个`step`里每个算子都要先编译再执行。非动态`shape`场景下(大部分情况)，某个算子在`step`之间除了输入数据不同，其他都一样。这种情况下，第二个`step`开始的算子编译就属于重复工作，没有必要了。通过缓存机制，就可以节省第二个`step`开始的算子编译时间。



增加日志打印，如下所示。

```diff
diff --git a/mindspore/ccsrc/runtime/pynative/op_compiler.cc b/mindspore/ccsrc/runtime/pynative/op_compiler.cc
index 9db48047fe..a68a86fe5a 100644
--- a/mindspore/ccsrc/runtime/pynative/op_compiler.cc
+++ b/mindspore/ccsrc/runtime/pynative/op_compiler.cc
@@ -69,7 +69,10 @@ OpCompilerInfoPtr OpCompiler::Compile(const session::BackendOpRunInfoPtr &op_run
     const auto &op_compiler_info = iter->second;
     MS_EXCEPTION_IF_NULL(op_compiler_info);
     *single_op_cache_hit = true;
+    MS_LOG(ERROR) << "Cache Hitted: " << graph_info;
     return iter->second;
+  } else {
+    MS_LOG(ERROR) << "Cache Missed: " << graph_info;
   }
   *single_op_cache_hit = false;
   // Generate kernel graph.

```



执行`pytest -s tests/st/pynative/test_pynative_lenet.py --disable-warnings`，前2个epoch的输出日志如下所示。

可以看出第一个epoch的每个算子日志中都出现了`Cache Missed`字样，因为是第一次编译。第二个epoch开始，日志中都出现了`Cache Hitted`。

```verilog
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.022.511 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_Conv2D[const vector][32, 1, 32, 32]43_[const vector][6, 1, 5, 5]43_(5, 5)16[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.023.712 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 6, 28, 28]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.024.327 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MaxPool[const vector][32, 6, 28, 28]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.025.029 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_Conv2D[const vector][32, 6, 14, 14]4343DefaultFormat_[const vector][16, 6, 5, 5]43_(5, 5)116[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.025.641 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 16, 10, 10]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.026.223 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MaxPool[const vector][32, 16, 10, 10]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.026.858 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_Reshape[const vector][32, 16, 5, 5]4343DefaultFormat_[output](32, -1)[tensor, shape]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.027.599 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MatMul[const vector][32, 400]4343DefaultFormat_[const vector][120, 400]43_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.028.182 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_BiasAdd[const vector][32, 120]4343DefaultFormat_[const vector][120]43_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.028.814 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 120]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.029.373 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MatMul[const vector][32, 120]4343DefaultFormat_[const vector][84, 120]43_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.029.920 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_BiasAdd[const vector][32, 84]4343DefaultFormat_[const vector][84]43_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.030.499 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 84]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.031.066 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MatMul[const vector][32, 84]4343DefaultFormat_[const vector][10, 84]43_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.031.620 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_BiasAdd[const vector][32, 10]4343DefaultFormat_[const vector][10]43_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.076.359 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_OneHot[const vector][32]34_[const vector][]43_[const vector][]43_[output]10[indices, depth, on_value, off_value]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.077.465 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_SoftmaxCrossEntropyWithLogits[const vector][32, 10]4343DefaultFormat_[const vector][32, 10]4343DefaultFormat_
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.078.543 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReduceSum[const vector][32]4343DefaultFormat_[y]false[input_x, axis]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.079.355 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_RealDiv[const vector][]4343DefaultFormat_[const vector][]43_[output][x, y]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.095.103 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_Conv2D[const vector][32, 1, 32, 32]4343DefaultFormat_[const vector][6, 1, 5, 5]4343DefaultFormat_(5, 5)16[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.156.199 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 6, 28, 28]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.162.271 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MaxPool[const vector][32, 6, 28, 28]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.165.779 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_Conv2D[const vector][32, 6, 14, 14]4343DefaultFormat_[const vector][16, 6, 5, 5]4343DefaultFormat_(5, 5)116[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.174.816 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 16, 10, 10]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.176.891 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MaxPool[const vector][32, 16, 10, 10]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.178.429 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_Reshape[const vector][32, 16, 5, 5]4343DefaultFormat_[output](32, -1)[tensor, shape]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.187.785 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MatMul[const vector][32, 400]4343DefaultFormat_[const vector][120, 400]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.200.955 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_BiasAdd[const vector][32, 120]4343DefaultFormat_[const vector][120]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.204.070 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 120]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.205.273 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MatMul[const vector][32, 120]4343DefaultFormat_[const vector][84, 120]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.208.087 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_BiasAdd[const vector][32, 84]4343DefaultFormat_[const vector][84]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.209.490 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReLU[const vector][32, 84]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.210.611 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_MatMul[const vector][32, 84]4343DefaultFormat_[const vector][10, 84]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.213.333 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_BiasAdd[const vector][32, 10]4343DefaultFormat_[const vector][10]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.217.555 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_OneHot[const vector][32]3434DefaultFormat_[const vector][]4343DefaultFormat_[const vector][]4343DefaultFormat_[output]10[indices, depth, on_value, off_value]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.224.591 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_SoftmaxCrossEntropyWithLogits[const vector][32, 10]4343DefaultFormat_[const vector][32, 10]4343DefaultFormat_
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.235.916 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_ReduceSum[const vector][32]4343DefaultFormat_[y]false[input_x, axis]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.273.076 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:75] Compile] Cache Missed: CPU_RealDiv[const vector][]4343DefaultFormat_[const vector][]4343DefaultFormat_[output][x, y]
======epoch:  0  loss:  2.3276513  cost time:  0.42531776428222656
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.434.218 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_Conv2D[const vector][32, 1, 32, 32]4343DefaultFormat_[const vector][6, 1, 5, 5]4343DefaultFormat_(5, 5)16[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.434.541 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 6, 28, 28]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.434.917 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MaxPool[const vector][32, 6, 28, 28]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.435.200 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_Conv2D[const vector][32, 6, 14, 14]4343DefaultFormat_[const vector][16, 6, 5, 5]4343DefaultFormat_(5, 5)116[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.435.547 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 16, 10, 10]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.435.886 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MaxPool[const vector][32, 16, 10, 10]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.436.291 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_Reshape[const vector][32, 16, 5, 5]4343DefaultFormat_[output](32, -1)[tensor, shape]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.436.656 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MatMul[const vector][32, 400]4343DefaultFormat_[const vector][120, 400]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.436.905 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_BiasAdd[const vector][32, 120]4343DefaultFormat_[const vector][120]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.437.220 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 120]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.437.564 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MatMul[const vector][32, 120]4343DefaultFormat_[const vector][84, 120]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.437.809 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_BiasAdd[const vector][32, 84]4343DefaultFormat_[const vector][84]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.438.121 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 84]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.438.453 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MatMul[const vector][32, 84]4343DefaultFormat_[const vector][10, 84]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.438.627 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_BiasAdd[const vector][32, 10]4343DefaultFormat_[const vector][10]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.439.051 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_OneHot[const vector][32]3434DefaultFormat_[const vector][]4343DefaultFormat_[const vector][]4343DefaultFormat_[output]10[indices, depth, on_value, off_value]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.439.225 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_SoftmaxCrossEntropyWithLogits[const vector][32, 10]4343DefaultFormat_[const vector][32, 10]4343DefaultFormat_
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.439.983 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReduceSum[const vector][32]4343DefaultFormat_[y]false[input_x, axis]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.440.181 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_RealDiv[const vector][]4343DefaultFormat_[const vector][]4343DefaultFormat_[output][x, y]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.441.372 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_Conv2D[const vector][32, 1, 32, 32]4343DefaultFormat_[const vector][6, 1, 5, 5]4343DefaultFormat_(5, 5)16[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.441.643 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 6, 28, 28]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.442.342 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MaxPool[const vector][32, 6, 28, 28]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.443.698 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_Conv2D[const vector][32, 6, 14, 14]4343DefaultFormat_[const vector][16, 6, 5, 5]4343DefaultFormat_(5, 5)116[x, w](0, 0, 0, 0)2NCHW(0, 0, 0, 0)1(1, 1, 1, 1)1(1, 1, 1, 1)[output]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.444.007 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 16, 10, 10]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.444.635 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MaxPool[const vector][32, 16, 10, 10]4343DefaultFormat_2[output](1, 1, 2, 2)NCHW(1, 1, 2, 2)[x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.445.278 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_Reshape[const vector][32, 16, 5, 5]4343DefaultFormat_[output](32, -1)[tensor, shape]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.445.564 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MatMul[const vector][32, 400]4343DefaultFormat_[const vector][120, 400]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.445.777 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_BiasAdd[const vector][32, 120]4343DefaultFormat_[const vector][120]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.446.055 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 120]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.446.440 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MatMul[const vector][32, 120]4343DefaultFormat_[const vector][84, 120]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.446.631 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_BiasAdd[const vector][32, 84]4343DefaultFormat_[const vector][84]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.446.890 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReLU[const vector][32, 84]4343DefaultFormat_[output][x]
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.447.255 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_MatMul[const vector][32, 84]4343DefaultFormat_[const vector][10, 84]4343DefaultFormat_[output]false[x1, x2]truefalsetrue
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.447.427 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_BiasAdd[const vector][32, 10]4343DefaultFormat_[const vector][10]4343DefaultFormat_[output]NCHW[x, b]NCHW
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.447.760 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_OneHot[const vector][32]3434DefaultFormat_[const vector][]4343DefaultFormat_[const vector][]4343DefaultFormat_[output]10[indices, depth, on_value, off_value]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.447.917 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_SoftmaxCrossEntropyWithLogits[const vector][32, 10]4343DefaultFormat_[const vector][32, 10]4343DefaultFormat_
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.448.799 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_ReduceSum[const vector][32]4343DefaultFormat_[y]false[input_x, axis]-1
[ERROR] DEVICE(20547,7f3e6fbd1740,python):2022-10-22-19:14:33.448.994 [mindspore/ccsrc/runtime/pynative/op_compiler.cc:72] Compile] Cache Hitted: CPU_RealDiv[const vector][]4343DefaultFormat_[const vector][]4343DefaultFormat_[output][x, y]
======epoch:  1  loss:  2.2348325  cost time:  0.025225400924682617
```

