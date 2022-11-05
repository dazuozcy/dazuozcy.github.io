---
Onelayout: post
title: "variadic template"
author: dazuo
date: 2020-07-03 10:19:00 +0800
categories: [C++]
tags: [variadic-template]
math: true
mermaid: true
---

# variadic templates

可变参数模板，是`C++11`新增的强大的特性之一，它对模板参数进行了高度泛化，能表示0到任意个数、任意类型的参数。

> 谈的是templates

- function template
- class template

> 变化的是template parameters

- 参数个数，利用参数个数逐一递减的特性，实现递归调用，基于function template
- 参数类型，利用参数个数逐一递减导致参数类型数量也逐一递减的特性实现递归继承或者递归复合，基于class template

可以方便地完成递归函数调用。

可以方便地完成递归类继承。

## 可变模板参数函数

```cpp
template <class... T>
void f(T... args) {    
    cout << sizeof...(args) << endl;  // 在模板里面使用`sizeof...(args)`可以打印模板参数的数量。
}

int main() {
	f();        //0
	f(1, 2);    //2
	f(1, 2.5, "");    //3    
}
```

由于可变模版参数的类型和个数是不固定的，所以我们可以传任意类型和个数的参数给函数`f`。这个例子只是简单的将可变模版参数的个数打印出来，如果需要将参数包中的每个参数打印出来的话就需要通过一些方法了。

展开可变模版参数函数的方法一般有两种：

- 通过递归函数来展开参数包

  通过递归函数展开参数包，需要提供一个参数包展开函数和一个递归终止函数，递归终止函数正是用来终止递归的。下面是例子

  ```cpp
  // 递归终止函数
  void print() {}
  
  // 参数包展开函数，每递归调用一次，参数包中的参数就少一个
  template <typename T, typename... Types>
  void print(const T& firstArg, const Types&... args) {  
  	cout << firstArg << endl;    
  	print(args...);
  }
  
  int main() {
      print(7.5, "hello", 666);
  }
  ```

- 通过逗号表达式来展开参数包

  递归函数展开参数包是一种标准做法，也比较好理解，但也有一个缺点,就是必须要一个重载的递归终止函数，即必须要有一个同名的终止函数来终止递归，这样可能会感觉稍有不便。

  有没有一种更简单的方式呢？其实还有一种方法可以不通过递归方式来展开参数包，这种方式需要借助逗号表达式和初始化列表。

  ```cpp
  template <class T>
  void printarg(T t) {
     cout << t << endl;
  }
  
  template <class ...Args>
  void expand(Args... args) {
     int arr[] = {(printarg(args), 0)...};
  }
  
  expand(1,2,3,4);
  ```

  `expand`函数中的逗号表达式`(printarg(args), 0)`先执行`printarg(args)`，再得到逗号表达式的结果`0`。

  同时还用到了`C++11`的另外一个特性——**初始化列表**，通过初始化列表来初始化一个变长数组, `{(printarg(args), 0)...}`将会展开成`((printarg(arg1),0), (printarg(arg2),0), (printarg(arg3),0), ...)`，最终会创建一个元素值都为`0`的数组int arr[sizeof...(Args)]。

  由于是逗号表达式，在创建数组的过程中会先执行逗号表达式前面的部分`printarg(args)`打印出参数，也就是说在构造`arr`数组的过程中就将参数包展开了，这个数组的目的纯粹是为了在数组构造的过程展开参数包。

  将上面的实现改进一下，少写最后的递归终止函数。如下所示：

  ```cpp
  template<class F, class ...Args>
  void expand(const F& f, Args...args) {
    std::initializer_list<int>{(f(args), 0)...};
  }
  
  expand([](int i){std::cout << i << std::endl;}, 1,2,3);
  ```





## 可变模板参数类

https://www.cnblogs.com/qicosmos/p/4325949.html



# 应用

项目中用到了很多第三方库，为了方便在程序运行故障时定界与定位，需要在调用第三方库接口时在接口前后记录日志，方便根据运行日志界定故障，分析原因。

以CPU上的oneDNN库为例，下面是AddN算子中调用的oneDNN的部分接口。

```cpp
auto desc = dnnl::binary::desc(dnnl::algorithm::binary_add, src0_mem_desc, src1_mem_desc, dst_mem_desc);
auto prim_desc = dnnl::binary::primitive_desc(desc, engine_);
```



最简单粗暴直接的实现方案就是在调用处前后直接加日志。如下所示：

```cpp
MS_LOG(DEBUG) << "begin to invoke constructor of dnnl::binary::desc";
auto desc = dnnl::binary::desc(dnnl::algorithm::binary_add, src0_mem_desc, src1_mem_desc, dst_mem_desc);
MS_LOG(DEBUG) << "end to invoke constructor of dnnl::binary::desc";

MS_LOG(DEBUG) << "begin to invoke constructor of dnnl::binary::primitive_desc";
auto prim_desc = dnnl::binary::primitive_desc(desc, engine_);
MS_LOG(DEBUG) << "end to invoke constructor of dnnl::binary::primitive_desc";
```



这种简单粗暴直接的方式有缺点：

- 每个oneDNN接口前后直接加记录日志的语句，oneDNN接口淹没在日志语句中，喧宾夺主，代码不优雅。
- 日志语句大同小异，复制粘贴容易产生copy-paste error。



所以利用C++11的新特性variadic template，设计如下接口，代替之前直接调用的接口。

```cpp
template <class T, class... Args>
auto CreateDesc(Args &&... args) {
  MS_LOG(DEBUG) << "begin to invoke constructor of " << demangle(typeid(T).name());
  auto desc = T(std::forward<Args>(args)...);
  MS_LOG(DEBUG) << "end to invoke constructor of " << demangle(typeid(T).name());
  return desc;
}
```

修改效果如下：

```diff
- auto desc = dnnl::binary::desc(dnnl::algorithm::binary_add, src0_mem_desc, src1_mem_desc, dst_mem_desc);
- auto prim_desc = dnnl::binary::primitive_desc(desc, engine_);
+ auto desc = CreateDesc<dnnl::binary::desc>(dnnl::algorithm::binary_add, src0_mem_desc, src1_mem_desc,,,);
+ auto prim_desc = CreateDesc<dnnl::binary::primitive_desc>(desc, engine_);
```

可以看出，相比简单粗暴直接方式，不易出错，且更加优雅。

