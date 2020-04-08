---
title: 信息学竞赛中的STL容器进阶
categories:
  - STL
tags:
  - null
date: 2020-04-07 18:02:56
---

## 概述

在信息学竞赛中，使用STL是高效率的：一方面，STL的高复用性使编写代码的难度大大降低，加快了开发效率；另一方面，STL的许多算法和容器，在编译器的优化下，特别是在允许使用`O2`编译优化的竞赛场景下，运行效率一般更高。

本文将着重介绍STL容器在信息学竞赛中的应用。

**容器**是模板类和算法的泛型集合，使程序员轻松实现常见的数据结构。 容器可以分为三类：**序列容器**，**关联容器**和**无序关联容器**，每种容器都旨在支持一组不同的操作。

容器管理了为其元素分配的存储空间，并提供成员函数以访问它们，或直接，或通过迭代器（类似于指针的对象）。

大多数容器拥有至少几个相同的成员函数，并且共享功能。 哪种容器最适合特定应用场景，不仅取决于所提供的功能，还取决于其针对不同工作负载的效率。

## 迭代器

### 迭代器



![range-rbegin-rend.svg](http://upload.cppreference.com/mwiki/images/3/39/range-rbegin-rend.svg)

### 迭代器类型

1. 输入迭代器：

   可**读**迭代器指向的元素值。

   输入迭代器上的算法必须是**单趟**算法：一旦自增输入迭代器，则所有其先前的副本都可能失效。

   典型的例子是[`istream_iterator`](https://en.cppreference.com/w/cpp/iterator/istream_iterator)，就像`cin`读字符流。

2. 输出迭代器：

   可**写**迭代器指向的元素值。

   输出迭代器上的算法必须是**单趟**算法：一旦自增输出迭代器，则所有其先前的副本都可能失效。

   典型的例子是[`ostream_iterator`](https://zh.cppreference.com/w/cpp/iterator/ostream_iterator)，就像`cout`写字符流。

3. 向前迭代器：

   可**读写**迭代器指向的元素值。

   不同于输入迭代器和输出迭代器，向前迭代器可以用于**多趟**算法[^1]。

4. 双向迭代器：

   可**双向移动**，即自增和自减的向前迭代器。

5. 随机访问迭代器

   可在常数时间内指向**任何元素**的双向迭代器。

   许多算法要求容器支持随机访问迭代器，比如`std::sort`和`std::binary_search`。

6. 连续迭代器

   连续迭代器指向的逻辑相邻元素在内存中物理上相邻。

   下列标准库类型支持连续迭代器<font color="#008000">（C++11 前）</font>：

   - [`array::iterator`](https://zh.cppreference.com/w/cpp/container/array)。
   - [`vector::iterator`](https://zh.cppreference.com/w/cpp/container/vector)，对于除`bool`以外的`value_type`。

<style>
    td{
        text-align:center;
    }
</style>
<table>
   <tr>
      <td colspan=5>迭代器类型</td>
      <td>支持的操作</td>
   </tr>
   <tr>
      <td rowspan=6>连续迭代器</td>
      <td rowspan=5>随机访问迭代器</td>
      <td rowspan=4>双向迭代器</td>
      <td rowspan=3>正向迭代器</td>
      <td rowspan=2>输入迭代器</td>
      <td>读入</td>
   </tr>
   <tr>
      <td>自增（非多趟）</td>
   </tr>
   <tr>
      <td></td>
      <td>自增（多趟）</td>
   </tr>
   <tr>
      <td colspan=2></td>
      <td>递减</td>
   </tr>
   <tr>
      <td colspan=3></td>
      <td>随机访问</td>
   </tr>
   <tr>
      <td colspan=4></td>
      <td>连续存储</td>
   </tr>
   <tr>
      <td colspan=6>属于以上类别之一并且还满足输出迭代器要求的迭代器称为可变迭代器。</td>
   </tr>
   <tr>
      <td rowspan=2>输出迭代器</td>
      <td rowspan=2 colspan=4></td>
      <td>写</td>
   </tr>
   <tr>
      <td>自增（非多趟）</td>
   </tr>
</table>

## 容器

### 容器与迭代器

对于元素类型为`T`的容器类型`C`的实例`a, b`：

- `C::iterator`：`C`的迭代器类型，必须是正向迭代器。
- `C::const_iterator`：`C`的`const`迭代器类型，是只读的正向迭代器，不可修改迭代器指向的元素。
- `C::value_type`：为`T`。
- 默认构造`C()`，创建一个空的`C`对象。
- 拷贝构造`C(a)``C b = a`，创建一个`a`的拷贝。
- 赋值`a = b`，销毁`a`并将`b`中的每一个元素复制到`a`。
- `a.begin()``a.cbegin()`：返回容器首的迭代器。在`a`非`const`限定时，前者返回`iterator`类型的迭代器，否则返回`const_iterator`类型的迭代器；后者始终返回`const_iterator`类型的迭代器。
- `a.end()``a.cend()`：返回容器的尾后迭代器。在`a`非`const`限定时，前者返回`iterator`类型的迭代器，否则返回`const_iterator`类型的迭代器；后者始终返回`const_iterator`类型的迭代器。
- `a == b``a != b`：当且仅当`a`和`b`的长度相同，且`a`中每一个元素均与`b`中相应的元素相等时，前者为真。后者等价于`!(a == b)`。
- `a.swap(b)``swap(a, b)`：交换`a`和`b`。
- `a.size()`：返回元素个数，等价于`a.end() - a.begin()`。
- `a.empty()`：当且仅当容器为空时为真，等价于`a.begin() == a.end()`。

### 序列容器

序列容器实现了可顺序访问的数据结构，强调各元素在容器内的顺序。包含`std::vector`，`std::deque`和`std::list`等。



#### `std::array`



#### `std::vector`



#### `std::deque`



#### `std::queue`



#### `forward_list`



#### `std::list`



### 容器适配器

容器适配器为顺序容器提供了不同的接口。

#### `std::stack`



#### `std::queue`



#### `std::priority_queue`



### 关联容器

关联容器实现了可以快速查找（对数复杂度）的排序数据结构。

#### `Compare`具名规则

键的唯一性是通过相等关系确定的，$a$ 与$b$相等，当且仅当$!comp(a, b)$且$!comp(b, a)$，其中$comp$函数为`Compare`所定义的排序规则。

#### `std::set`

```c++
template <
	typename Key,
	typename Compare = std::less<Key>,
	typename Allocator = std::allocator<Key>
> class set;
```

`std::set`是一个关联容器，其中包含`Key`类型的**唯一对象**的有序集合。 使用`Compare`对键进行排序。搜索、删除和插入操作具有对数复杂度。通常使用红黑树实现。

成员函数：

- `rbegin()``crbegin()`：

#### `std::map`

```c++
template <
	typename Key,
	typename T,
	typename Compare = std::less<Key>
	typename Allocator = std::allocator<std::pair<const Key, T> >
> class map;
```

`std::map`是一个有序的关联容器，包含具有唯一键的键值对。使用`Compare`对键进行排序。搜索、删除和插入操作具有对数复杂度。通常使用红黑树实现。

#### `std::multiset`

与`std::set`相比，`std::multiset`允许多个键拥有相等的值。自C++11起，等价的元素按照插入顺序进行排序。

成员函数基本相同，在此不加赘述。

#### `multimap`

### 无序关联容器

无序关联容器实现了可以快速查找（均摊常数，最坏线性）的无序（哈希）数据结构。

#### `unordered_set`



#### `unordered_map`



#### `unordered_multiset`



#### `unordered_multimap`



### 容器适配器

### 迭代器无效

只读方法永远不会使迭代器或引用无效。 修改容器内容的方法可能会使迭代器和/或引用无效，如下表所概述。



## 注释与参考

1. https://en.cppreference.com/w/cpp/container

[^1]:关于多趟保证，请参阅https://zh.cppreference.com/w/cpp/named_req/ForwardIterator#.E5.A4.9A.E8.B6.9F.E4.BF.9D.E8.AF.81