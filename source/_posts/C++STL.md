---
title: c++STL
comments: false
tags:
  - C++
excerpt: 千磨万击还坚韧，任尔东西南北风
toc: false
date: 2025-03-20 20:03:05
cover: 'https://images7.alphacoders.com/135/thumb-1920-1354305.jpeg'
---
# c++STL
### STL的诞生
- C++的[**面向对象**](https://github.com/xyz2781790037/CPP/blob/main/C%2B%2B%E6%A0%B8%E5%BF%83/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.md)和**泛型编程**思想，目的就是**复用性的提升**
- 大多情况下，数据结构和算法都未能有一套标准,导致被迫从事大量重复工作
- 为了建立数据结构和算法的一套标准,诞生了**STL**
### STL基本概念
- STL(Standard Template Library,**标准模板库**)
- STL 从广义上分为: **容器(container) 算法(algorithm) 迭代器(iterator)**
- **容器和算法**之间通过**迭代器**进行无缝连接。
- STL 几乎所有的代码都采用了模板类或者模板函数
### STL六大组件
STL大体分为六大组件，分别是:**容器、算法、迭代器、仿函数、适配器、空间配置器**
### 容器
容器是用来存储和管理数据的结构，STL 提供了多种不同类型的容器，每种容器都有其特定的应用场景。按照数据存储方式，可以分为 **序列式容器** 和 **关联式容器**。

#### **序列式容器**

用于 **顺序存储数据**，数据的存储顺序由用户决定，常见的有：

- [**vector**（动态数组）](https://github.com/xyz2781790037/CPP/blob/main/C%2B%2B%E6%A0%B8%E5%BF%83/vector.md)：支持快速随机访问，插入删除效率较低（尾部除外）。
- [**deque**（双端队列）](https://github.com/xyz2781790037/CPP/blob/main/C%2B%2B%E6%A0%B8%E5%BF%83/deque.md)：支持头尾插入删除，适合需要频繁在两端操作的数据结构。
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

| 迭代器类型         | 支持操作            |
| ------------------ |  ------------------------------- |
| **输入迭代器**     |  只能读取一次，`++it`            |
| **输出迭代器**     | 只能写入一次，`++it`            |
| **前向迭代器**     |  ++it                            |
| **双向迭代器**     |  `++it`, `--it`                  |
| **随机访问迭代器** |  `+`, `-`, `it[n]`->`*(it  + n)` |

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

**仿函数** 是指 **重载了 `operator()` 的类或结构体对象**，它的行为类似于函数，但具有 **类的特性**。仿函数在 **C++ STL** 中主要用于 **自定义排序、条件判断、计算等场景**，并且可以携带状态信息（成员变量）。

仿函数本质上是一个 **重载了 `operator()` 的类**，这样它的对象可以像函数一样被调用

```c++
#include <iostream>
class Add{
public:
    int operator()(int a,int b){
        return a + b;
    }
};
int main(){
    Add add;
    std::cout << add(1,2) << std::endl;// add.operator()(1, 2)
    return 0;
}
```

#### STL中的常见仿函数

**算术仿函数**、**关系仿函数**、**逻辑仿函数**和**自定义仿函数**,定义在 `<functional>` 头文件中。

（1）算术仿函数

```c++
#include <iostream>
#include <functional>  // 引入 STL 算术仿函数

int main() {
    std::plus<int> add;  // 加法仿函数
    std::minus<int> sub; // 减法仿函数
    std::multiplies<int> mul; // 乘法仿函数
    std::divides<int> div; // 除法仿函数
    std::modulus<int> mod; // 取模仿函数
    std::negate<int> neg;  // 取负仿函数

    std::cout << "加法: " << add(10, 5) << std::endl; // 10 + 5 = 15
    std::cout << "取负: " << neg(7) << std::endl;    // -7

    return 0;
}

```

（2）关系仿函数

```c++
#include <iostream>
#include <functional>

int main() {
    std::greater<int> g;  // 大于
    std::less<int> l;     // 小于
    std::greater_equal<int> ge;  // 大于等于
    std::less_equal<int> le;     // 小于等于
    std::equal_to<int> eq;       // 等于
    std::not_equal_to<int> ne;   // 不等于

    std::cout << "5 > 3: " << g(5, 3) << std::endl;  // 1 (true)
    std::cout << "5 == 3: " << eq(5, 3) << std::endl; // 0 (false)

    return 0;
}

```

（3）逻辑仿函数

```c++
#include <iostream>
#include <functional>

int main() {
    std::logical_and<bool> land;  // 逻辑与
    std::logical_or<bool> lor;    // 逻辑或
    std::logical_not<bool> lnot;  // 逻辑非

    std::cout << "true && false: " << land(true, false) << std::endl; // 0 (false)
    std::cout << "!true: " << lnot(true) << std::endl; // 0 (false)

    return 0;
}

```

### 适配器

**STL 适配器**是 C++ 标准库中的一种工具，它们用来改变或扩展容器、迭代器和函数对象的功能，而不需要改变原始的容器、迭代器或仿函数本身。通过适配器，我们可以以不同的方式使用容器、迭代器和函数对象，从而更方便地满足特定需求。

在 STL 中，主要有三种类型的适配器：

1. **容器适配器**
2. **迭代器适配器**
3. **函数对象适配器**

#### 容器适配器

容器适配器实际上是对容器的**接口**进行修改或封装，但它们依然依赖于其他基础容器（如 `std::deque`、`std::vector`、`std::list`）来存储数据。它们并没有重新实现数据存储结构，而是提供了简化或者特定的数据操作方式,比如：

- `std::stack` 是一个容器适配器，它封装了一个底层容器（通常是 `std::deque` 或 `std::vector`）并只暴露 `push`、`pop` 和 `top` 等接口。你不能直接访问 `std::stack` 中的元素，所有操作都受到限制以保持栈的 LIFO（后进先出）特性。

区别：

- **暴露的接口不同**：容器适配器提供的接口是经过简化或特定限制的，只能进行某些操作
- **封装的容器不同**：容器适配器内部依赖于标准容器，如 `std::deque` 或 `std::vector`，但它只暴露出适合某一场景的操作接口  `std::stack<int, std::vector<int>> stack_using_vector;`
- **没有单独的存储结构**：容器适配器通常不拥有自己的存储结构，它们是对标准容器的一种封装。

#### 迭代器适配器

迭代器适配器也是对迭代器的一种封装或扩展，通常是为了改变迭代器的行为。迭代器本身提供了容器元素的访问方式，而迭代器适配器通过对迭代器的封装，提供了额外的功能或行为。

##### **迭代器适配器的特点**：

- **改变迭代器的遍历方式**：迭代器适配器可以改变迭代器的方向（例如反向遍历）或增强原始迭代器的功能。
- **底层容器无关**：迭代器适配器不会改变底层容器，它只改变迭代器本身的行为或使用方式。

##### **迭代器适配器与普通迭代器的区别**

迭代器适配器和普通迭代器的主要区别在于，**它们并不直接用于遍历容器，而是用来改变容器元素访问的方式或行为**：

1. **普通迭代器**：用于直接访问容器中的元素，按顺序进行遍历。
2. **迭代器适配器**：通过封装普通迭代器，改变其行为。比如 `std::reverse_iterator` 会使得迭代器反向遍历，`std::insert_iterator` 用来在遍历过程中插入元素。

#### 函数对象(仿函数)适配器

将现有的函数对象（Functors）或函数适配到新的接口或新的上下文。它使得函数对象或函数指针能够被更加灵活地使用，通常用于将一个函数对象转化成更通用的形式，或者是修改其行为。

函数对象适配器本质上**是将函数（或者函数指针）转化为一个函数对象。这个适配器包装了原始函数，并提供了一个`operator()`来调用这个函数。**

使用主要体现在 **函数对象的封装** 和 **使其适应不同的使用场景**

##### 函数对象的封装

函数对象适配器主要的作用之一是 **封装** 函数或函数指针，使其变得更加灵活和通用。简单地说，封装意味着将一个函数（或函数指针）转化为一个可以作为对象传递、存储和调用的形式。

```c++
struct Compare {
    bool operator()(int a, int b) {
        return a < b;  // 比较两个整数
    }
};
int main(){
    Compare com
    vector<int> v = {1,3,2,5,4};
    sort(v.begin(),v.end(),com);
    for(int num : v){
        cout << num << endl;
    }
    return 0;
}
```

**使其适应不同的使用场景**

###### 通过 `std::function` 将普通函数转化为可调用对象，适配 STL 算法。

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

bool isLessThan(int a, int b) {
    return a < b;
}
int main() {
    std::vector<int> vec = {5, 2, 8, 3, 1};
    
    // 使用 std::function 将普通函数适配为可调用对象
    std::function<bool(int, int)> compareFunc = isLessThan;
    
    std::sort(vec.begin(), vec.end(), compareFunc);
    for (int num : vec) {
        std::cout << num << " ";
    }

    return 0;
}
```

### 空间配置器

**空间配置器（Allocator）** 是一个用于管理内存分配和释放的机制，它为 STL 容器提供了一个统一的接口，以便容器能够根据需要分配和释放内存。通过自定义空间配置器，用户可以更精细地控制内存的管理方式，从而满足不同的内存管理需求。

在 STL 中，所有的容器默认使用 `std::allocator` 作为其空间配置器，`std::allocator` 使用 `new` 和 `delete` 来进行内存的分配和释放。

#### `std::allocator` 的基本实现

`std::allocator` 是标准库提供的默认空间配置器，它提供了用于分配和释放内存的一组接口：

- `allocate()`: 分配内存。
- `deallocate()`: 释放内存。
- `construct()`: 在已分配的内存上构造对象。
- `destroy()`: 销毁对象。

#### `std::allocator` 的基本接口

#### 1. `allocate`

用于分配一块内存，返回一个指向该内存块的指针。

```c++
T* allocate(std::size_t n);
```

`n` 是请求分配的元素数量。返回的指针指向一个足够大的内存块，可以容纳 `n` 个类型为 `T` 的对象。

#### 2. `deallocate`

释放之前通过 `allocate` 分配的内存。

```c++
void deallocate(T* p, std::size_t n);
```

`p` 是指向要释放的内存的指针，`n` 是分配的元素数量。

#### 3. `construct`

在指定的内存位置上构造对象。对于 `new` 和 `delete` 操作，这就是一种手动构造对象的方式。

```c++
void construct(T* p, Args&&... args);
```

在 `p` 指向的内存位置构造一个类型为 `T` 的对象，使用传递给 `construct` 的参数。

#### 4. `destroy`

销毁指定内存上的对象。对于动态分配的对象，这相当于手动调用 `delete`。

```c++
void destroy(T* p);
```