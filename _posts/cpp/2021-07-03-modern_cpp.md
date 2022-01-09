---
layout: post
title: "Modern C++"
author: dazuo
date: 2020-07-03 10:19:00 +0800
categories: [C++]
tags: [智能指针]
math: true
mermaid: true
---



**`const`**——支持接口和符号常量的不变性。

**虚函数**——提供运行期多态。

**引用**——支持运算符重载和简化参数传递。

**运算符和函数重载**——除了算法和逻辑运算符外，还包括：允许用户定义 `=`（赋值）、`()`（调用；支持函数对象（[§4.3.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#431-lambda-表达式)））、`[]`（下标访问）和 `->`（智能指针）。

**类型安全链接**——消除许多来自不同翻译单元中不一致声明的错误。

**抽象类**——提供纯接口。

**模板**——在经历了多年使用宏进行泛型编程的痛苦之后，更好地支持泛型编程。

**异常**——试图给混乱的错误处理带来某种秩序；RAII（[§2.2.2](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/02.md#222-标准库组件)）便是为此目标而设计的。

**`dynamic_cast`** 和 **`typeid`**——一种非常简单的运行期反射形式（“运行期类型识别”，又名 RTTI）。

**`namespace`**——允许程序员在编写由几个独立部分组成的较大程序时避免名称冲突。

**条件语句内的声明**——让写法更紧凑和限制变量作用域。

**具名类型转换**——（`static_cast`、`reinterpret_cast` 和 `const_cast`）：消除了 C 风格的类型转换中的二义性，并使显式类型转换更加显眼。

**`bool`**：一种被证明非常有用和流行的布尔类型；C和C++曾经使用整数作为布尔变量和常量。



- 内存模型——一个高效的为现代硬件设计的底层抽象，作为描述并发的基础（[§4.1.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#411-内存模型)）
- `auto` 和 `decltype`——避免类型名称的不必要重复（[§4.2.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#421-auto-和-decltype)）
- 范围 `for`——对范围的简单顺序遍历（[§4.2.2](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#422-范围-for)）
- 移动语义和右值引用——减少数据拷贝（[§4.2.3](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#423-移动语义)）
- 统一初始化—— 对所有类型都（几乎）完全一致的初始化语法和语义（[§4.2.5](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#425-统一初始化)）
- `nullptr`——给空指针一个名字（[§4.2.6](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#426-nullptr)）
- `constexpr` 函数——在编译期进行求值的函数（[§4.2.7](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#427-constexpr-函数)）
- 用户定义字面量——为用户自定义类型提供字面量支持（[§4.2.8](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#428-用户定义字面量)）
- 原始字符串字面量——不需要转义字符的字面量，主要用在正则表达式中（[§4.2.9](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#429-原始字符串字面量)）
- 属性——将任意信息同一个名字关联（[§4.2.10](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#4210-属性)）
- lambda 表达式——匿名函数对象（[§4.3.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#431-lambda-表达式)）
- 变参模板——可以处理任意个任意类型的参数的模板（[§4.3.2](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#432-变参模板)）
- 模板别名——能够重命名模板并为新名称绑定一些模板参数（[§4.3.3](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#433-别名)）
- `noexcept`——确保函数不会抛出异常的方法（[§4.5.3](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#453-noexcept-规约)）
- `override` 和 `final`——用于管理大型类层次结构的明确语法
- `static_assert`——编译期断言
- `long long`——更长的整数类型
- 默认成员初始化器——给数据成员一个默认值，这个默认值可以被构造函数中的初始化所取代
- `enum class`——枚举值带有作用域的强类型枚举



- `unique_ptr` 和 `shared_ptr`——依赖 RAII（[§2.2.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/02.md#221-语言特性)）的资源管理指针（[§4.2.4](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#424-资源管理指针)）
- 内存模型和 `atomic` 变量（[§4.1.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#411-内存模型)）
- `thread`、`mutex`、`condition_variable` 等——为基本的系统层级的并发提供了类型安全、可移植的支持（[§4.1.2](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#412-线程和锁)）
- `future`、`promise` 和 `packaged_task`，等——稍稍更高级的并发（[§4.1.3](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#413-期值future)）
- `tuple`——匿名的简单复合类型（[§4.3.4](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#434-tuple)）
- 类型特征（type trait）——类型的可测试属性，用于元编程（[§4.5.1](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#451-实现技巧)）
- 正则表达式匹配（[§4.6](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#46-c11标准库组件)）
- 随机数——带有许多生成器（引擎）和多种分布（[§4.6](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#46-c11标准库组件)）
- 时间——`time_point` 和 `duration`（[§4.6](https://gitee.com/bexsder/Cxx_HOPL4_zh/blob/main/04.md#46-c11标准库组件)）
- `unordered_map` 等——哈希表
- `forward_list`——单向链表
- `array`——具有固定常量大小的数组，并且会记住自己的大小
- emplace 运算——在容器内直接构建对象，避免拷贝
- `exception_ptr`——允许在线程之间传递异常
