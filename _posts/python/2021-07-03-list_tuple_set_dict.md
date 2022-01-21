---
layout: post
title: "list vs. tuple vs. set vs. dict"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [Python]
tags: [Python]
math: true
mermaid: true
---

# `list`

类似于C/C++中的动态数组，但是list里面的元素可以是不同类型

有序，可重复，可修改，非同构

## 创建

```python
list1 = []  # 创建空列表
list2 = [1, 'hello', 3.14, 'world'] # 创建非同构列表
list3 = list((1, '2', 3.14)) # tuple转list
```

## 读/写

```python
list1 = [1, 'hello', 3.14, 'world']

# 索引
print (list1[1])  # 输出'hello'

# 切片
print (list1[1:3])  # 输出'hello'

# pop
list1.pop() # 删除列表的最后一个元素
print (list1) # [1, 'hello', 3.14]

list1.append('python')
print (list1) # [1, 'hello', 3.14, 'python']
```



# `tuple`

有序，可重复，不可修改，非同构

## 创建

```python
tuple1 = ()  # 创建空元组
tuple2 = (1, 'hello', 3.14, 'world') # 创建非同构元组
tuple3 = tuple([1, '2', 3.14]) # list转tuple
```

## 读

```python
tuple1 = (1, 'hello', 3.14, 'world')

# 索引
print (tuple1[1])  # 输出'hello'

# 切片
print (tuple1[1:3])  # 输出'hello'

tuple1[0]='bad' # error, 不支持修改内容
```



# `set`

跟集合的数学概念一致。

无序，不重复，可修改，非同构。

## 创建

```python
set1 = {1, 'hello', 3.14, 'world'} # 创建非同构集合
set2 = {(1, '2', 3.14)} # 只有一个元素：(1, '2', 3.14)
set3 = {[1, '2', 3.14]} # TypeError: unhashable type: 'list'
```

## 读/写

```python
set1 = {1, 'hello', 3.14, 'world'} # 创建非同构集合
print (set1[0])  # error, 无序，不支持索引，切片
print (set1.pop()) # 随机弹出一个元素，这里弹出的是1
print (set1) # 输出 {'hello', 3.14, 'world'}
set1.add('python') # 添加一个元素
```



# `dict`

无序，键不重复，可修改，非同构

## 创建

```python
dict1 = {}  # 创建空字典
dict2 = ('key1':'value1', 2: 3.14) # 创建非同构字典
dict3 = dict({1:"apple", 2:'orange', 3:"cherry"])
```

## 读/写

```python
dict1 = ('key1':'value1', 2: 3.14)
print (dict1.keys())  # 输出dict_keys(['key1', 2])
print (dict1.values())  # 输出dict_values(['value1', 3.14])
print (dict1['key1']) # 输出'value1'
dict1['key1'] = 'replaced'
print (dict1)  # 输出{'key1':'value1', 2: 3.14}

dict1.update({'python':666})
print (dict1) # 输出{'key1':'value1', 2:3.14, 'python':666}

dict1.pop('python')
print (dict1) # 输出{'key1':'value1', 2:3.14}
```





能用tuple就不要用list，因为tuple中数据不可修改，更安全。
