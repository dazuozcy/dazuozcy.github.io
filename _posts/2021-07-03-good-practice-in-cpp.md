---
layout: post
title: "C++里好的编码技巧"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [C++]
tags: [C++]
math: true
mermaid: true
---

```cpp
static auto Anyone = [](auto&& k, auto&&... args) { return ((args == k) || ...); };

string s="autumn";

//等价于 if(s=="spring" || s== "summer" || s == "autumn" || s == "winter")
if (Anyone(s, "spring", "summer", "autumn", "winter")) {
    ...    
}
```


[RAII妙用之计算函数耗时](https://zhuanlan.zhihu.com/p/139519294)

# 模板特化
- 允许某些形式的优化
- 减少代码膨胀

[Why Not Specialize Function Templates?](http://gotw.ca/publications/mill17.htm)

[深入理解特化与偏特化](https://sg-first.gitbooks.io/cpp-template-tutorial/content/jie_te_hua_yu_pian_te_hua.html)


# Expression Template，奇异递归模板模式CRTP
