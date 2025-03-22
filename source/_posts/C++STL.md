---
title: c++STL
comments: false
tags:
  - C++
excerpt: 千磨万击还坚韧，任尔东西南北风
toc: false
date: 2025-03-20 20:03:05
cover: 'https://images8.alphacoders.com/133/1339755.jpeg'
---
# c++STL
### STL的诞生
- C++的**面向对象**和**泛型编程**思想，目的就是**复用性的提升**
- 大多情况下，数据结构和算法都未能有一套标准,导致被迫从事大量重复工作
- 为了建立数据结构和算法的一套标准,诞生了**STL**
### STL基本概念
- STL(Standard Template Library,**标准模板库**)
- STL 从广义上分为: **容器(container) 算法(algorithm) 迭代器(iterator)**
- **容器和算法**之间通过**迭代器**进行无缝连接。
- STL 几乎所有的代码都采用了模板类或者模板函数
### STL六大组件
STL大体分为六大组件，分别是:**容器、算法、迭代器、仿函数、适配器（配接器）、空间配置器**
### 容器
容器是用来存储和管理数据的结构，STL 提供了多种不同类型的容器，每种容器都有其特定的应用场景。按照数据存储方式，可以分为 **序列式容器** 和 **关联式容器**。

#### **序列式容器**

用于 **顺序存储数据**，数据的存储顺序由用户决定，常见的有：

- **vector**（动态数组）：支持快速随机访问，插入删除效率较低（尾部除外）。
- **deque**（双端队列）：支持头尾插入删除，适合需要频繁在两端操作的数据结构。
- **list**（双向链表）：支持高效插入删除（不支持随机访问）。
- **forward_list**（单向链表）：比 `list` 轻量级，仅支持单向遍历。
- **array**（固定大小数组）：C++11引入，类似 `vector` 但大小固定。
- **stack**（栈）：后进先出（LIFO）。
- **queue**（队列）：先进先出（FIFO）。

#### **关联式容器**

用于 **按特定规则存储数据**，通常是基于平衡二叉树（`std::set` 和 `std::map`）或哈希表（`std::unordered_set` 和 `std::unordered_map`）。

- **set**（集合）：自动排序，元素唯一。
- **multiset**（多重集合）：允许重复元素，自动排序。
- **map**（映射表）：键值对存储，自动排序，键唯一。
- **multimap**（多重映射）：允许键重复，自动排序。
- **unordered_set**（无序集合）：基于哈希表，元素唯一，查找更快（O(1)）。
- **unordered_multiset**（无序多重集合）：允许重复元素，基于哈希表。
- **unordered_map**（无序映射）：哈希表实现，键唯一。
- **unordered_multimap**（无序多重映射）：允许键重复，哈希表实现。

### 算法

`#include <algorithm>`

### **2.1 非修改序列操作**

- `find(begin, end, value)`: 查找值
- `count(begin, end, value)`: 统计值出现的次数
- `for_each(begin, end, func)`: 对每个元素执行 `func`

### **2.2 修改序列操作**

- `copy(src_begin, src_end, dest_begin)`: 复制
- `replace(begin, end, old_value, new_value)`: 替换
- `remove(begin, end, value)`: 删除（逻辑删除，返回新的 `end` 迭代器）

### **2.3 排序与查找**

- `sort(begin, end)`: 排序（默认 `operator<` 比较）
- `stable_sort(begin, end)`: 稳定排序
- `binary_search(begin, end, value)`: 二分查找（要求数据有序）
- `lower_bound(begin, end, value)`: 返回第一个大于等于 `value` 的位置
- `upper_bound(begin, end, value)`: 返回第一个大于 `value` 的位置

### **2.4 其他**

- `reverse(begin, end)`: 逆序
- `unique(begin, end)`: 去重
- `next_permutation(begin, end)`: 计算下一个字典序排列

### 迭代器

#### **迭代器的基本概念**

**迭代器类似于指针**，但它不仅仅局限于数组，还能适用于各种 STL 容器。

常见迭代器操作：

- `it = container.begin();` → 指向容器的第一个元素
- `it = container.end();` → 指向容器末尾的**下一个位置**
- `++it` → 迭代器向前移动
- `--it` → 迭代器向后移动（仅适用于双向迭代器）
- `*it` → 访问迭代器当前指向的元素

#### 迭代器类型

**常规迭代器** (`iterator`)：可以修改容器中的元素。`begin()`

**常量迭代器** (`const_iterator`)：不能修改容器中的元素，只能读取。`cbegin()`

**反向迭代器** (`reverse_iterator`)：从容器的末尾向前遍历。`rbegin()`

**常量反向迭代器** (`const_reverse_iterator`)：与 `reverse_iterator` 类似，但不能修改元素。`crbegin()`

| 迭代器类型         | 适用容器                 | 支持操作             |
| ------------------ | ------------------------ | -------------------- |
| **输入迭代器**     | istream_iterator         | 只能读取一次，`++it` |
| **输出迭代器**     | ostream_iterator         | 只能写入一次，`++it` |
| **前向迭代器**     | forward_list             | ++it                 |
| **双向迭代器**     | list`, `set`, `map       | ++it`, `--it         |
| **随机访问迭代器** | vector`, `array`, `deque | +`, `-`, `it[n]      |

#### **迭代器的基本作用**

迭代器（Iterator）是**一种对象**，用于遍历和操作 STL 容器（如 `vector`, `list`, `array`, `map` 等）中的元素。迭代器的主要作用包括：

1. **遍历容器**（代替 `for` 循环和索引）
   `for(vector<int>::iterator c = works.begin();c != works.end();c++)` ->`for(int c：works)`
2. **提供统一的访问方式**（适用于各种容器）
3. **支持 STL 算法**（如 `sort()`, `find()`, `reverse()`）
4. **提高代码的通用性和可读性**
5. **避免直接使用指针操作，减少错误**

```c++
std::vector<int> v = {1, 2, 3};
int* p = &v[0];
v.push_back(4);
std::cout << *p;
```

### 仿函数

