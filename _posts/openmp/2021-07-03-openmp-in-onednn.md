---
layout: post
title: "OpenMP in oneDNN"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [openMP]
tags: [openMP, oneDNN]
math: true
mermaid: true
---

本文介绍下`OpenMP`在`oneDNN`库中的使用。

# parallel()

`oneDNN`中CPU上多线程并行的基础是`parallel()`函数，将与`OpenMP`无关的内容删除以后，精简如下所示：

```cpp
// src/common/dnnl_thread_parallel_nd.hpp

inline int adjust_num_threads(int nthr, size_t work_amount) {
    if (nthr == 0) nthr = dnnl_get_current_num_threads();
#if DNNL_CPU_THREADING_RUNTIME == DNNL_RUNTIME_OMP
    return (work_amount == 1 || omp_in_parallel()) ? 1 : nthr;
#else
    return (int)std::min((size_t)nthr, work_amount);
#endif
}

/* general parallelization */
template <typename F>
void parallel(int nthr, F f) {
    nthr = adjust_num_threads(nthr, SIZE_MAX);
    if (nthr == 1) {
        f(0, 1);
        return;
    }
#pragma omp parallel num_threads(nthr)
    {
        int nthr_ = omp_get_num_threads();
        int ithr_ = omp_get_thread_num();
        assert(nthr_ == nthr);
        f(ithr_, nthr_);
    }
}
```

`omp_in_parallel()`：如果从并行域内部调用，则返回`true`/`非零`。如果是嵌套并行域里的串行域里调用，也是返回`true`/`非零`。

所以`adjust_num_threads()`根据工作量大小以及是否在并行域来调整分配的线程数量。



# `#pragma omp parallel`

**`#pragma omp parallel num_threads(N)` 与`#pragma omp parallel for num_threads(N)`的区别？**

### 代码片段1

```cpp
#include <iostream>
#include <omp.h>

int main()
{    
  #pragma omp parallel num_threads(4)
  {
    for (int i=0;i<4;i++) {
      for (int j=0;j<4;j++) {
        std::cout << "(" << i << "," << j <<") Thread num == " << omp_get_thread_num() << std::endl;
      }
    }
  }
}
```

对于上面的代码片段的效果是：

每个线程都会执行一遍花括号里的**双层`for`**循环。所以最终会打印出`4*(4*4)=64`条形如`(3,1) Thread num == 2`的条目。即总共打印64条，每个线程打印16条。



### 代码片段2

```cpp
#include <iostream>
#include <omp.h>

int main()
{    
  #pragma omp parallel for num_threads(4)
    for (int i=0;i<4;i++) {
      for (int j=0;j<4;j++) {
        std::cout << "(" << i << "," << j <<") Thread num == " << omp_get_thread_num() << std::endl;
      }
    }
}
```

对于上面的代码片段的效果是：

只会将最外层（索引为`i`）的`for`循环进行并行化，即有4个线程，每个线程都执行一遍索引为`j`的for循环，也就算打印4个条目。



### 代码片段3

```diff
#include <iostream>
#include <omp.h>

int main()
{    
-   #pragma omp parallel for num_threads(4)
+   #pragma omp parallel num_threads(4)
    for (int i=0;i<4;i++) {
      for (int j=0;j<4;j++) {
        std::cout << "(" << i << "," << j <<") Thread num == " << omp_get_thread_num() << std::endl;
      }
    }
}
```

如果将`#pragma omp parallel`后的关键字`for`去掉，如代码片段3所示，则和代码片段1的效果是一样的，即总共打印`16`条，每个线程打印`4`条。



# #pragma omp barrier

```cpp
// src/common/dnnl_thread.hpp

inline void dnnl_thr_barrier() {
#pragma omp barrier
}
```



# #pragma omp parallel for simd collapse(k)

```cpp
// src/cpu/rnn/ref_rnn.hpp

template <typename gates_t, typename acc_t>
// The loop body needs to be put in a function as some versions of icc have
// an issue with lambdas & macros inside omp simd loops
inline void body_loop(int i, int k, const gates_t *ws_gates, acc_t *diff_bias,
        const rnn_utils::rnn_conf_t &rnn) {
    for (int j = 0; j < rnn.mb; j++)
        diff_bias[i * rnn.dhc + k] += ws_gates[j * rnn.scratch_gates_ld + i * rnn.dhc + k];
}

template <typename gates_t, typename acc_t>
void gates_reduction(const rnn_utils::rnn_conf_t &rnn, const gates_t *ws_gates_, acc_t *diff_bias_) {
#pragma omp parallel for simd collapse(2)
    for (int i = 0; i < rnn.n_gates; i++)
        for (int k = 0; k < rnn.dhc; k++)
            body_loop(i, k, ws_gates_, diff_bias_, rnn);
}
```

## simd

矢量化。



## collapse(k)

如果不加`collapse(k)`原语，则只会对最外层`for`循环进行并行化。加`collapse(k)`，则可以并行化嵌套`for`循环，`k`表示希望并行的嵌套`for`循环的层次。



# schedule

```cpp
// src/cpu/ref_shuffle.cpp

#pragma omp parallel for collapse(3) schedule(static)
    for_(dim_t mb = 0; mb < MB; ++mb)
        for_(dim_t cb = 0; cb < C; cb += blksize)
        for (dim_t sp = 0; sp < SP; ++sp) {
            const dim_t off = mb * stride_mb + sp * blksize;
            const dim_t output_off = off + cb * SP;
            PRAGMA_OMP_SIMD()
                for (dim_t cc = 0; cc < nstl::min(blksize, C - cb); ++cc) {
                    const dim_t input_c = rev_transposed_[cb + cc];
                    const dim_t input_off = off + input_c / blksize * SP * blksize
                        + input_c % blksize;
                    output[output_off + cc] = input[input_off];
                }
        }
```

schedule(static): 在编译时完成调度。



