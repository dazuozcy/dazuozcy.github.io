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

> 如果一个程序员了解底层实现模型，他就能够写出效率较高的代码，自信心也比较高。
>
> ​																																						——《深度探索C++对象模型》

封装，继承，多态



# 封装

ADT(Abstract Data Type, 抽象数据类型)。

“一个ADT或class hierarchy的数据封装”比“在C程序中程序性地使用全局数据”好。



C++在布局及存取时间上主要的额外负担是由`virtual`引起的，包括：

- virtual function机制。用以支持一个有效率的“运行期绑定”。
- virtual base class。用以实现“多次出现在继承体系中的base class，有一个单一而被共享的实例”。



# 继承

继承机制是面向对象程序设计使代码**可复用**的最重要的手段。

继承包含2部分：

- 接口(interface)
- 实现(implementation)

```cpp
class Person {
public:
    virtual void Eat() const = 0;  // 纯虚成员函数
    virtual void Say(const std::string& msg);  // 普通虚成员函数
    int Name() const; // 非虚成员函数
};

class Student: public Person {
    ...
};

class Teacher: public Person {
    ...
};
```

继承接口和实现主要包括三种方式：

- 纯虚函数（只继承接口），继承的仅是基类成员函数的接口，必须在派生类中重写该函数的实现。
- 普通虚函数（继承接口和实现，可override），继承的是基类成员函数的接口和缺省的实现。由派生类自行选择是否重写该函数。
- 非虚成员函数（继承接口和实现，不可override），继承的是基类成员函数的接口和强制的实现。派生类中不能重新实现该接口。



## 底层原理

## 虚继承

在虚继承的情况下，base class不管在继承链中被派生多少次，永远只会存在一个实例。





## 虚函数作用，什么是纯虚函数，虚函数可以是静态的吗

## 静态函数，虚函数区别

## 虚函数的底层机制，虚函数指针何时生成，在多继承下怎样。虚函数表，存在哪

## 数组对象怎么析构，怎么释放

## 虚表布局，菱形继承

## 类的内存布局



# 多态

> **polymorphism** - providing a single interface to entities of different types.[virtual](https://stroustrup.com/glossary.html#Gvirtual) [function](https://stroustrup.com/glossary.html#Gfunction)s provide dynamic (run-time) polymorphism through an interface provided by a [base class](https://stroustrup.com/glossary.html#Gbase-class). [Overloaded function](https://stroustrup.com/glossary.html#Goverloaded-function)s and [template](https://stroustrup.com/glossary.html#Gtemplate)s provide [static](https://stroustrup.com/glossary.html#Gstatic) (compile-time) polymorphism.
>
> ​																																								 ——TC++PL 12.2.6, 13.6.1, D&E 2.9.

> 为不同类型的实体提供单一接口。虚函数通过基类提供的接口来提供动态(运行时)多态。模板和重载函数提供静态（编译期）多态。

- 动态多态（运行时多态），面向对象编程，底层机制是虚函数。
- 静态多态（编译时多态），泛型编程，底层机制是模板或者重载函数。

## 虚函数实现动态多态的原理

多态的表现形式：通过指针或者引用。

#### 指针

**派生类指针**赋给**基类指针**；

通过**基类指针**调用基类或者派生类中的同名**虚函数**时：

- 若该指针指向一个**基类**的对象，那么被调用的是**基类**的虚函数；
- 若该指针指向一个**派生类**的对象，那么被调用的是**派生类**的虚函数。

#### 引用

派生类的对象可以赋给基类引用；

通过**基类引用**调用基类和派生类中的同名**虚函数**时:

- 若该引用引用的是一个**基类**的对象，那么被调用是**基类**的虚函数；
- 若该引用引用的是一个**派生类**的对象，那么被 调用的是**派生类**的虚函数。





# 类的访问权限：`private`, `protected`, `public`

- `public`：可以被该类中的函数、友元函数、子类的函数访问，也可以由该类的对象访问；
- `protected`：可以被该类中的函数、友元函数、子类的函数访问，但不可以由该类的对象访问；
- `private`：可以被该类中的函数、友元函数访问，但不可以由子类的函数、该类的对象访问。



# `C++`里`struct`与`class`区别

成员的默认访问控制权限不同，在`struct`中是`public`，在`class`中是`private`.
何时用`struct`,何时用`class`：

- use struct for POD(plain-old-data) structures without any class-like features.
- Use class when you make use of feathers such as private or protected members, non-default constructors and operators, etc.
