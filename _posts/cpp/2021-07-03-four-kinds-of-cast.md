---
layout: post
title: "四种类型转换"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [C++]
tags: [类型转换]
math: true
mermaid: true
---

# `static_cast`
主要用于基本类型之间和具有继承关系的类型之间的转换。这种转换一般会更改变量的内部表示方式，因此，static_cast应用于指针类型转换没有太大意义.
```cpp
例： //基本类型转换 
int i=0; 
double d = static_cast<double>(i); //相当于 double d = (double)i; 
//转换继承类的对象为基类对象 
class Base{}; 
class Derived : public Base{}; 
Derived d; 
Base b = static_cast<Base>(d); //相当于 Base b = (Base)d;
```


# `dynamic_cast`
- 它与static_cast相对，是动态转换。
- 这种转换是在运行时进行转换分析的，并非在编译时进行，明显区别于其他三种类型转换。 
- 该函数只能在继承类对象的指针之间或引用之间进行类型转换。进行转换时，会根据当前运行时类型信息，判断类型对象之间的转换是否合法。dynamic_cast的指针转换失败，可通过是否为nullptr检测，引用转换失败则抛出一个bad_cast异常。

```cpp
例： class Base{}; 
class Derived : public Base{}; 
//派生类指针转换为基类指针 
Derived *pd = new Derived; 
Base *pb = dynamic_cast<Base*>(pd); 
 if (!pb) 
cout << "类型转换失败" << endl; 
 
//没有继承关系，但被转换类有虚函数 
class A(virtual ~A();) //有虚函数 
class B{}: 
A* pa = new A; 
B* pb = dynamic_cast<B*>(pa);
```

# `const_cast`
- 该函数用于去除指针变量的常量属性，将它转换为一个对应指针类型的普通变量。反过来，也可以将一个非常量的指针变量转换为一个常指针变量。
- 这种转换是在编译期间做出的类型更改。

```cpp
例： const int* pci = 0; 
int* pk = const_cast<int*>(pci); //相当于int* pk = (int*)pci; 
 
const A* pca = new A; 
A* pa = const_cast<A*>(pca); //相当于A* pa = (A*)pca;
```
出于安全性考虑，const_cast无法将非指针的常量转换为普通变量。

# `reinterpret_cast`
- 该函数将一个类型的指针转换为另一个类型的指针. 
- 这种转换不用修改指针变量值存放格式(不改变指针变量值),只需在编译时重新解释指针的类型就可做到. 
- reinterpret_cast可以将指针值转换为一个整型数,但不能用于非指针类型的转换。
```cpp
例: //基本类型指针的类型转换 
double d=9.2; 
double* pd = &d; 
int *pi = reinterpret_cast<int*>(pd); //相当于int *pi = (int*)pd; 
//不相关的类的指针的类型转换 
class A{}; 
class B{}; 
A* pa = new A; 
B* pb = reinterpret_cast<B*>(pa); //相当于B* pb = (B*)pa; 
//指针转换为整数 
long l = reinterpret_cast<long>(pi); //相当于long l = (long)pi;
```

### 类指针如何用C++转换类别，如A* a如何转换到B*，是否所有的指针都可用dynamic_cast进行转换。