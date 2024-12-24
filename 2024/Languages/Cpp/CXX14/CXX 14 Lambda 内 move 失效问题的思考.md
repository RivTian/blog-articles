
#CXX14 #Lambda

最近在学习 C++ Move 时，有看到这样一个代码需求：在 lambda 中，将一个捕获参数 move 给另外一个变量。 看似一个很简单常规的操作，然而这个 move 动作却没有生效。

具体代码如下

```cpp
 std::vector<int> vec = {1,2,3};  
 ​  
 auto func = [=](){  
     auto vec2 = std::move(vec);  
     std::cout << vec.size() << std::endl; // 输出：3  
     std::cout << vec2.size() << std::endl; // 输出：3  
 };
```

代码可在 [wandbox](https://wandbox.org/permlink/et6ZTBhwpz5SH6w0) 运行。

我们期望的是，将对变量 `vec` 调用 std::move 后，数据将会移动至变量 `vec2`, 此时 `vec` 里面应该没有数据了。但是通过打印 `vec.size()` 发现 vec 中的数据并没有按预期移走。

这也就意味着，构造 vec2 时并没有按预期调用移动构造函数，而是调用了拷贝构造函数。

为什么会造成这个问题呢, 我们需要结合 `std::move` 和 `lambda` 的原理看下。（最终的解决方案可以直接看 [这里](#%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)）

# std::move 的本质

> std::move() 位于 `#include <utilty>` 中，但一般无需特地引入，`iostream`，`string` 等头文件会包含

对于 std::move，有两点需要注意：

1. std::move 中到底做了什么事情
    
2. std::move 是否可以保证数据一定能移动成功
    

对于第二点来说，答案显然是不能。这也是本文的问题所在。那么 std::move 实际上是做了什么事情呢？

对于 std::move，其实现大致如下：

```cpp
 template<typename T>  
 decltype(auto) move(T&& param)  
 {  
     using ReturnType = remove_reference_t<T>&&;  
     return static_cast<ReturnType>(param);  
 }
```

从代码可以看出，std::move 本质上是调用了 static_cast 做了一层强制转换，强制转换的目标类型是 `remove_reference_t<T>&&`，remove_reference_t 是为了去除类型本身的引用，例如左值引用。总结来说，std::move 本质上是将对象强制转换为了右值引用。

那么，为什么我们通常使用 std::move 实现移动语义，可以将一个对象的数据移给另外一个对象？

这是因为 std::move 配合了移动构造函数使用，本质上是移动构造函数起了作用。移动构造函数的一般定义如下：

```cpp
 class A  
 {  
 public:  
     A (A &&);  
 }
```

可以看到移动构造函数的参数就是个右值引用 `A&&`，因此 `A a = std::move(b);`, 本质上是先将 b 强制转化了右值引用 `A&&`, 然后触发了移动构造函数，在移动构造函数中，完成了对象 b 的数据到对象 a 的移动。

那么，在哪些情况下，`A a = std::move(b);` 会失效呢？ 显然是，当 std::move 强转后的类型不是 `A&&`，这样就不会命中移动构造函数。

例如：

```cpp
 const std::string str = "123";  
 std::string str2(std::move(str));
```

这个时候，对 str 对象调用 `std::move`，强转出来的类型将会是 `const string&&`, 这样移动构造函数就不会起作用了，但是这个类型却可以令复制构造函数生效。

结合本文最初的问题，在 lambda 中 move 没有生效，显然也是 std::move 强转的类型不是 `std::vector<int>&&`, 才导致了没有 move 成功。

那么，为什么会出现这个问题呢，我们需要理解下 lambda 的工作原理。

# lambda 闭包原理

对于 c++ 的 lambda，编译器会将 lambda 转化为一个独一无二的闭包类。而 lambda 对象最终会转化成这个闭包类的对象。 对于本文最初的这个 lambda 来说，最终实际上转化成了这么一个类型

```cpp
// 转换前  
 auto func = [=](){  
     auto vec2 = std::move(vec);  
 };  
 ​  
 // 转换后  
 class ClosureFunc{  
 public:  
     void operator() const{  
         auto vec2 = std::move(vec);  
     };  
 ​  
 private:  
     std::vector<int> vec;  
 };  
 ​  
 ClosureFunc func; 
```

这里需要注意, lambda 的默认行为是，**生成的闭包类的 `operator()` 默认被 const 修饰**。

那么这里问题就来了，当调用 `operator()` 时, 该闭包类所有的成员变量也是被 const 修饰的，此时对成员变量调用 `std::move` 将会引发上文中提到的，**强转出来的类型将会是 `const string&&` 问题**。因此，移动构造函数将不会被匹配到。

我们最初的问题 lambda 中 std::move 失效的问题，也是因为这个原因。这也很符合 const 函数的语义: const 函数是不能修改成员变量的值。

# 解决方案

那么，这个应该怎么解决呢？答案是 `mutable`。即在 lambda 尾部声明一个 mutable，如下：

```cpp
 auto func = [=]() mutable{  
     auto vec2 = std::move(vec);  
 };
```

这样编译器生成的闭包类的 `operator()` 将会不带 const 了。我们的 std::move 也可以正常转换，实现移动语义了。

```cpp
 std::vector<int> vec = {1,2,3};  
 ​  
 auto func = [=]() mutable{  
     auto vec2 = std::move(vec);  
     std::cout <<vec.size() << std::endl; // 输出：0  
     std::cout <<vec2.size() << std::endl; // 输出：3  
 };
```

代码可以在 [wandbox](https://wandbox.org/permlink/ox4SBxNrAi8M8ZLp) 运行。

# 参考

- [Lambda 表达式 - cppreference](https://zh.cppreference.com/w/cpp/language/lambda)
    
- Effective Modern c++
    
- [关于 C++ 右值及 std::move() 的疑问？](https://www.zhihu.com/question/50652989)