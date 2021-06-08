## C++中的lambda与函数对象

`lambda`表达式是C++11中引入的一项新技术，利用`lambda`表达式可以编写内嵌的匿名函数，用以替换独立函数或者函数对象，并且使代码更可读。但是从本质上来讲，lambda表达式只是一种语法糖，因为所有其能完成的工作都可以用其它稍微复杂的代码来实现。但是它简便的语法却给C++带来了深远的影响。

如果从广义上说，`lambda`表达式产生的是函数对象。函数对象的本质上是一个类而不是一个函数，在类中，对象重载了函数调用运算符()，从而使对象能够项函数一样被调用，我们称这些对象为函数对象（Function Object）或者仿函数（Functor）。相比lambda表达式，函数对象有自己独特的优势。下面我们开始具体讲解这两项黑科技。

## lambda表达式

先从一个简单的例子开始，我们定义一个输出字符串的`lambda`表达式，如下所示，表达式一般都是从方括号`[]`开始，然后结束于花括号`{}`：

```cpp
auto basic_lambda = [] {cout << "Hello Lambda" << endl;}; //定义简单的lambda表达式
basic_lambda(); //调用
```

下面分别是包含参数和返回类型的`lambda`表达式：

```cpp
auto add = [] (int a, int b)->int { return a + b;}; //返回类型需要用`->`符号指出
auto multiply = [](int a, int b) {return a * b;} //一般可以省略返回类型，通过自动推断就能得到返回类型
```

`lambda`表达式最前面的方括号提供了“闭包”功能。每当定义一个`lambda`表达式以后，编译器会自动生成一个 **匿名类** ，并且这个类重载了`()`运算符，我们将其称之为`闭包类型（closure type）`。在运行时，这个`lambda`表达式会返回一个匿名的闭包实例，并且该实例是一个右值。闭包的一个强大之处在于其可以通过传值或引用的方式捕捉其封装作用域内的变量，`lambda`表达式前面的方括号就是用来定义捕捉模式以及变量的`lambda`捕捉块，如下所示：

```cpp
int main() {
    int x = 10; // 定义作用域内的x，方便下面的lambda捕捉
    auto add_x = [x](int a) { return a + x;}; // 传值捕捉x
    auto multiply_x = [&x](int a) {return a * x;}; //引用捕捉x
}
```

当`lambda`捕捉块为空时，表示没有捕捉任何变量。对于传值方式捕捉的变量x，`lambda`表达式会在生成的匿名类中添加一个非静态的数据成员，由于闭包类重载`()`运算符是使用了const属性，所以不能在`lambda`表达式中修改传值方式捕捉的变量，但是如果把`lambda`标记为`mutable`，则可以改变(但是这里的改变只会对 lambda 表达式内部的代码有影响, 对外部不起作用)，如下所示：

```cpp
int x = 10;

auto add_x = [x](int a) mutable { x * = 2; return a + x;};
cout << add_x(10) << endk; //输出30
return 0;
```



而对于引用方式捕捉的变量，无论是否标记为`mutable`，都可以对变量进行修改，并且修改的值会影响到外部, 至于会不会在匿名类中创建数据成员，需要看不同编译器的具体实现。

`lambda`表达式只能作为右值，也就是说，它是不能被赋值的

```cpp
auto a = [] { cout << "A" << endl; };
auto b = [] { cout << "B" << endl; };

a = b;  // 非法，lambda表达式变量只能做右值
auto c = a; // 合法，生成一个副本
```



造成以上原因是因为禁用了赋值运算符：

```cpp
ClosureType &operator=(const ClosureType &) = delete;
```



但是没有禁用复制构造函数，所以仍然可以用是一个`lambda`表达式去初始化另一个（通过产生副本）。

关于`lambda`的捕捉块，主要有以下用法：

- []：默认不捕捉变量
- [=]：默认以值捕捉所有变量（最好不要用）
- [&]：默认以引用捕捉所有变量（最好不要用）
- [x]：仅以值捕捉变量x，其他变量不捕捉
- [&x]：仅以引用捕捉x，其他变量不捕捉
- [=, &x]：默认以值捕捉所有变量，但是x是例外，通过引用捕捉
- [&, x]：默认以引用捕捉所有变量，但是x是例外，通过值捕捉
- [this]：通过引用捕捉当前对象（其实是复制指针）
- [* this]：通过传值方式捕捉当前对象

通过以上的说明，可以看到`lambda表达式`可以作为返回值，赋值给对应类型的函数指针，但是使用函数指针貌似并不是那么方便，于是STL在头文件`<functional>`中定义了一个多态的函数对象封装`std::function`，其功能类似于函数指针。它可以绑定到任何类函数对象，只要参数与返回类型相同。如下面的返回一个bool且接收两个int的函数包装器：

```cpp
std::function<bool(int, int)> wrapper = [](int x, int y) { return x < y; };
```



`lambda`表达式还有一个很重要的应用是其可以作为函数的参数，如下所示：

```cpp
int value = 3;
vector<int> v{1, 2, 3, 4, 5, 6, 7};

int count == std::count_if(v.begin, v.end(), [value](int x) {return x > value;});
```



下面给出`lambda`表达式的完整语法：

```cpp
// 完整语法
[ capture - list ] ( params ) mutable(optional) constexpr(optional)(c++17) exception attribute -> ret { body }

// 可选的简化语法
[ capture - list ] ( params ) -> ret { body }
[ capture - list ] ( params ) { body }
[ capture - list ] { body }
```



- capture-list：捕捉列表，这个不用多说，前面已经讲过，记住它不能省略；
- params：参数列表，可以省略（但是后面必须紧跟函数体）；
- mutable：可选，将lambda表达式标记为mutable后，函数体就可以修改传值方式捕获的变量；
- constexpr：可选，C++17，可以指定lambda表达式是一个常量函数；
- exception：可选，指定lambda表达式可以抛出的异常；
- attribute：可选，指定lambda表达式的特性；
- ret：可选，返回值类型；
- body：函数执行体。

# lambda新特性（C++14）

在`C++14`中，`lambda`又得到了增强，一个是泛型`lambda`表达式，一个是`lambda`可以捕捉表达式。

## lambda捕捉表达式

前面讲过，`lambda`表达式可以按传值或者引用捕捉在其作用域范围内的变量。而有时候，我们希望捕捉不在其作用域范围内的变量，而且最重要的是我们希望捕捉右值。所以`C++14`中引入了表达式捕捉，其允许用任何类型的表达式初始化捕捉的变量，如下：

```cpp
// 利用表达式捕获，可以更灵活地处理作用域内的变量
int x = 4;
auto y = [&r = x, x = x + 1] { r += 2; return x * x; }();
// 此时 x 更新为6，y 为25

// 直接用字面值初始化变量
auto z = [str = "string"] { return str; }();
// 此时z是const char* 类型，存储字符串 string
```



可以看到捕捉表达式扩大了lambda表达式的捕捉能力，有时候你可以用std::move初始化变量。这对不能复制只能移动的对象很重要，比如 `std::unique_ptr`，因为其不支持复制操作，你无法以值方式捕捉到它。但是利用lambda捕捉表达式，可以通过移动来捕捉它：

```cpp
auto myPi = std::make_unique<double>(3.1415);
auto circle_area = [pi = std::move(myPi)](double r) { return *pi * r * r; };
cout << circle_area(1.0) << endl; // 3.1415
```

## 泛型lambda表达式

从C++14开始，lambda表达式支持泛型：其参数可以使用自动推断类型的功能，而不需要显示地声明具体类型。这就如同函数模板一样，参数要使用类型自动推断功能，只需要将其类型指定为auto，类型推断规则与函数模板一样。这里给出一个简单例子：

```cpp
auto add = [](auto x, auto y) { return x + y; };

int x = add(2, 3);   // 5
double y = add(2.5, 3.5);  // 6.0
```

# 函数对象

函数对象是一个广泛的概念，因为所有具有函数行为的对象都可以称为函数对象。这是一个高级抽象，我们不关心对象到底是什么，只要其具有函数行为即可。函数行为是指可以使用`()`调用并传递参数，如下所示：

```cpp
function(arg1, arg2, ...); //函数调用
```



由此，`lambda`表达式也是一个函数对象。该函数对象实际上是一个匿名类的实例，且这个类实现了函数调用运算符`()`。

泛型提供了高级抽象，不论是`lambda`表达式、函数对象、还是函数指针，都可以传入到STL算法中（如for_each）。