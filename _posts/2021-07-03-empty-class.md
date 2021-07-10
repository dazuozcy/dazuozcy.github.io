---
layout: post
title: "空class"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [C++]
tags: [空类，深拷贝，浅拷贝]
math: true
mermaid: true
---

空class有六个默认函数：
- 默认构造函数
- 复制构造函数
- 析构函数
- 赋值运算符
- 取址运算符
- 取址运算符const

```cpp
class Empty {
public:
  Empty();                           // 默认构造函数
  Empty(const Empty& );             // 拷贝构造函数
  ~Empty();                          // 析构函数
  Empty& operator=(const Empty&);  // 赋值运算符
  Empty* operator&();                // 取址运算符
  const Empty* operator&() const;    // 取址运算符 const};
}
```

### 深拷贝与浅拷贝区别
https://zhuanlan.zhihu.com/p/342851361

https://zhuanlan.zhihu.com/p/188311618

### 赋值函数，拷贝函数
### 手写深拷贝，动态分配内存
### 写一个拷贝构造函数？为什么引用传递而不是值传递
