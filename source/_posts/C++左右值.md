---
title: c++左右值
comments: false
tags:
  - C++
excerpt: 闲居清风亭，左右清风来。
toc: false
date: 2025-03-20 09:37:01
cover: 'https://images8.alphacoders.com/133/1339755.jpeg'
---
# C++中的左值和右值
## 一、左值和右值的概念
#### 左值
左值是一个表示数据的表达式，它代表一个具名的内存位置，程序可以获取其地址，可以通过地址访问它们，是可被引用的数据对象。左值又可分为可修改的的左值和不可修改的左值。
- 可修改的的左值
左值其实就是指可出现在赋值语句左边的表达式，它们的值是可以被修改的(**最初的概念**);
```C++
int a;
a = 10;
 
int* ptr = &a;
*ptr = 100;
```
- 不可修改的左值
随着关键字const的引入，这个左值的概念发生了变化 。因为虽然不能对const变量赋值，但是可以获取其地址。
```c++
const int a = 10;
const int* ptrA = &a; //可以通过地址访问它
a = 100;              //错误
```
#### 右值
右值代表一个临时的值，不能被取地址，不能被修改，是不能出现在赋值语句的左边的。
```C++
int func() {
	return 1;
}
 
void test01() {
 
	int a = 1;
    int b = 2;
	int c = a + b;
    
	int d = func();
};
```
面的a,b,c,d这几个变量都是左值，但是a+b和func()都是右值。 我们知道是无法写成如下这样的：
```
a + b = 3;
func() = 3;
```
## 二. 左值引用和右值引用
#### 1. 左值引用 (Lvalue Reference)
左值引用是指引用一个左值（即具有持久性和可修改性的对象）。左值是指那些有地址的对象，比如变量名、数组元素等。
```c++
int a = 5;
int &ref = a;  // ref 是 a 的左值引用
```
- 左值引用可以绑定到一个左值。
- 左值引用通常用于传递可修改的对象（传递引用可以避免复制，提高效率）。
#### 2. 右值引用 (Rvalue Reference)
右值引用是指引用一个右值（即临时对象或将要销毁的对象）。右值是没有持久性的对象，比如临时变量、字面量、函数返回值等。右值引用的作用就是允许一个对象的资源“转移”到另一个对象，而不是复制它。
```c++
int &&rref = 10;  // rref 是 10 的右值引用
```
- 右值引用可以绑定到一个右值。
- 右值引用通常用于实现移动语义和完美转发，这是 C++11 引入的一项非常重要的特性，目的是减少不必要的对象拷贝，从而提高程序的性能。
#### 左值与右值的区别
- 左值：具有持久性，能出现在赋值语句的左侧，通常是变量、数组元素、解引用等。
- 右值：没有持久性，不能出现在赋值语句的左侧，通常是临时对象、字面量、返回值等。
## 移动语义
移动语义就是可以将对象的资源所有权从一个对象转移到另一个对象，而不进行资源的复制的技术。
移动语义得以实现，这其中便离不开右值引用的支持。
让编译器知道什么时候需要的是拷贝，什么时候不需要，这就是右值引用发挥作用的地方了。简单来说，就是编译器会根据会根据传进来的参数是左值引用还是右值引用来决定调用拷贝构造函数还是移动构造函数(移动构造函数里将资源从一个对象转移到另一个对象，而不用复制)。
```c++
#include <iostream>
#include <string>

class MyClass {
public:
    std::string data; // 假设我们有一个简单的成员变量，存储字符串

    // 默认构造函数
    MyClass(const std::string& str) : data(str) {
        std::cout << "MyClass constructor: " << data << std::endl;
    }

    // 移动构造函数
    MyClass(MyClass&& other) noexcept : data(std::move(other.data)) {
        std::cout << "MyClass move constructor\n";
    }

    // 移动赋值操作符
    MyClass& operator=(MyClass&& other) noexcept {
        std::cout << "MyClass move assignment operator\n";
        if (this != &other) {
            data = std::move(other.data); // 通过移动将资源转移到当前对象
        }
        return *this;
    }

    // 打印 data
    void print() const {
        std::cout << "Data: " << data << std::endl;
    }
};

int main() {
    MyClass obj1("Hello, world!"); // 调用默认构造函数
    obj1.print();

    // 移动构造一个新对象
    MyClass obj2 = std::move(obj1); // 调用移动构造函数
    obj2.print();

    // 移动赋值操作符
    MyClass obj3("Temporary Data");
    obj3.print();
    obj3 = std::move(obj2); // 调用移动赋值操作符
    obj3.print();

    return 0;
}
```
```bush
MyClass constructor: Hello, world!
Data: Hello, world!
MyClass move constructor
Data: Hello, world!
MyClass constructor: Temporary Data
Data: Temporary Data
MyClass move assignment operator
Data: Hello, world!
```
1. 右值引用让编译器知道何时可使用移动语义
2. 编译移动构造函数，使其提供所需的方法。也就是怎么让资源从一个对象转移到另一个对象，而不用复制
## 完美转发
完美转发（Perfect Forwarding）是 C++11 引入的一种技术，它允许我们在不丢失类型信息的情况下将函数参数转发给另一个函数。完美转发确保了无论传递给函数的是**左值**还是**右值**，都能够正确地转发到目标函数中，而不会进行不必要的拷贝或失去原始类型的特性。

完美转发的关键在于**转发引用（Forwarding Reference）**，也叫做**万能引用（Universal Reference），**通过 T&& 来实现。
#### 引用折叠
在 C++ 中，**引用不能引用引用**（例如 int& & 是非法的），但在模板和 decltype 等场景下，我们可能会遇到**多重引用**，这时 C++ 需要确定最终的**引用**类型，这就是引用折叠规则的作用。
> 引用折叠规则：
> & + & -> &
> & + && -> &
>&& + & -> &
>&& + && -> &&
- 任何左值引用（&）+ 任何引用（& 或 &&）= 左值引用（&）
- 只有右值引用（&&）+ 右值引用（&&）= 右值引用（&&）
#### forward
std::forward 是 C++11 引入的一个标准库函数，专门用于完美转发（Perfect Forwarding），它的主要作用是在模板中保持参数的原始值类别（左值或右值），确保参数不会被意外降级为左值，从而避免性能问题或语义错误。
**std::forward 的核心作用就是让参数以“合适”的方式传递给其他函数，而不改变它原来的值类别（左值或右值）。**
```c++
#include <iostream>
#include <utility>  // std::forward

void process(int& x) { std::cout << "Lvalue reference\n"; }
void process(int&& x) { std::cout << "Rvalue reference\n"; }

template <typename T>
void wrapper(T&& arg) { 
    process(std::forward<T>(arg));  // 通过 std::forward 传递参数
}

int main() {
    int a = 42;
    wrapper(a);  // 左值传递
    wrapper(42); // 右值传递
}
//Lvalue reference
//Rvalue reference

```

```
wrapper(a);
```

- `T` 被推导为 `int&`，所以 `T&&` 变成 **`int& &&`，根据引用折叠规则，它变成 `int&`**。
- `std::forward<int&>(arg)` → `static_cast<int&>(arg)`，保持 `arg` 作为左值。
- `process(int&)` 被调用。

```
wrapper(42);
```

- `T` 被推导为 `int`，所以 `T&&` **保持为 `int&&`**。
- `std::forward<int>(arg)` → `static_cast<int&&>(arg)`，保持 `arg` 作为右值。
- `process(int&&)` 被调用。

如果不使用 `std::forward`

`wrapper(42);` 传递 `arg` 时，**`arg` 作为一个具名变量，被视为左值**，即使 `T` 是 `int&&`。

结果：**右值 `42` 变成左值，调用 `process(int&)`，破坏了移动语义**。

#### **`std::forward` 与 `std::move` 的区别**

##### **`std::move`**

- `std::move(arg)` **无条件地** 将 `arg` **转换为右值**（`T&&`）。
- 适用于**希望明确转移所有权**的场景（如移动构造函数）。

##### **`std::forward`**

- `std::forward<T>(arg)` **只有当 `T` 是右值引用（`T&&`）时，才会将 `arg` 作为右值传递**。
- 适用于**泛型代码，确保参数的原始值类别不变**。
