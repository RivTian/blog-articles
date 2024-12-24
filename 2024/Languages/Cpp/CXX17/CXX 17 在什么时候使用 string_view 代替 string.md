
![](https://img2023.cnblogs.com/blog/1849509/202312/1849509-20231211102449649-1203884209.png)


## 前言

C++17增加了**`std::string_view`，**它在很多情况会优于使用**`std::string` 。**

尤其是用做函数形参的时候，使用`std::string_view`基本一定优于老式的`const std::string&`这种写法。

## 了解`std::string_view`

在讲述它的优越性之前，我们应该先介绍一下它。

首先，它顾名思义，就是一个字符串视图，它和`std::string`不一样，不是一个可变的字符串类型。

它只是用来指代“字符串”（这个字符串可以指代的东西很多）的，并**不拥有所有权，自然也不可变**。

通常实现只保有**两个**数据成员：一个指向字符串的指针，一个表示字符串的的`size`（通常是`size_t`类型）。

**通常`64`位系统下大小为16个字节。**

## **了解`std::string`**

```cpp
class string {//简单示例，实际不可能如此
public:
    // all 83 member functions
private:
    char* m_data;
    size_type m_size;
    size_type m_capacity;
    std::array<char, 16> m_sso;
};
```

对于 64 位系统，每个字符串`std::string`有 24 个字节的“开销”（`size`，`capacity`，`data`），另外还有 16 个字节用于 `SSO` 缓冲区。

加起来也就是**40。**

## **实际使用**

### **老式写法**

```cpp
void func(const std::string&s){
    std::cout << s << '\n';
}
```

看起来没有任何问题，但其实在很多传参调用的情况下，开销是很大的。

```cpp
std::string s{"乐呵"};
func("乐呵");
func(s);
```

你觉得上面哪个调用，**谁的开销更大**？

是 `func("乐呵");`，这里我们传入的是字符串字面量，它和`std::string`不是一个类型，这里实际上需要调用`std::string`的转换构造函数，在当前构造出一个临时的`std::string`对象，也就是一个纯右值表达式。

`const std::string&`可以接纯右值表达式，没问题，并且延长临时对象的生存期，可以在函数局部使用。

```cpp
const char* p = "乐呵";
func(p);//传指针也和上面说的差不多。
```

另外，使用`const std::string&`还更容易造成一些bug，比如：

```cpp
 const std::string& f(const std::string& str) {
    return str;
}
int main() {
    auto& ret = f("哈哈");
    std::cout << ret << '\n';
}
```

`const std::string& str`接纯右值表达式是没问题。但是它最后还想返回这个对象的引用，就不对了。

在函数调用中**绑定到函数形参的临时量**，存在到含这次函数调用的全表达式结尾为止：**如果函数返回一个生命长于全表达式的引用，那么它会成为悬垂引用。**



### **使用`std::string_view`**

```cpp
void func(std::string_view s){
    std::cout << s << '\n';
}

int main(){
    std::string s{"乐呵"};
    const char* p = "乐呵";
    func("乐呵");
    func(s);
    func(p);
}
```

`std::string`有一个到`std::string_view`的[转换函数](https://zh.cppreference.com/w/cpp/string/basic_string/operator_basic_string_view)，其他的都是正常走[std::string_view的构造函数](https://zh.cppreference.com/w/cpp/string/basic_string_view/basic_string_view)。

`std::string_view` 只是一个视图，用来指代原字符串的，保有一个`size`和一个指针即可。

新增加的库基本上不会再以`const std::string&`这种作为形参，比如`std::format`，`std::vformat`。

## 总结

如果你能用`std::string_view`，那么请使用。至少在用作接口的时候，一定是。