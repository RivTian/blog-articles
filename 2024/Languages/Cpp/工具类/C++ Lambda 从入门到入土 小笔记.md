
> Lambda表达式是C++11标准中引入的，允许在代码中定义匿名函数。本文的每一个章节都会有大量的代码案例帮助理解。  
   本文部分代码参考了 [微软官方文档 | Lambda expressions in C++ | Microsoft Learn](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/en-us/cpp/cpp/lambda-expressions-in-cpp%3Fview%3Dmsvc-170%26source%3Ddocs)。

## 基础篇

### 1. Lambda基本语法

Lambda基本长这样：

```cpp
[捕获子句](参数列表) -> 返回类型 {
    // 函数体
}
[ capture_clause ] ( parameters ) -> return_type {
    // function_body
}
```

- 捕获子句（**capture_clause**）决定了外部作用域中的哪些变量将被这个lambda捕获以及如何捕获（通过值、引用或不捕获）。下一章我们详细讨论捕获子句。  
    
- 参数列表（**parameters**）和函数体（**function_body**）跟普通函数一样，没啥区别。  
    
- 返回类型（**return_type**）稍微有些不同。如果函数体包含多条语句，且需要返回值，则必须显式指定返回类型，除非所有 `return` 语句都返回相同类型，那么返回类型也可以被自动推断。  
    

### 2. 如何使用Lambda表达式

语法例子：

```cpp
// 一个不捕获任何外部变量、不接受参数、没有返回值的lambda
auto greet = [] { std::cout << "Hello, World!" << std::endl; };

// 一个通过引用捕获外部变量、接受一个int参数、返回int类型的lambda
int x = 42;
auto add_to_x = [&x](int y) -> int { return x + y; };

// 一个通过值捕获所有外部变量、接受两个参数、返回类型被自动推断的lambda
int a = 1, b = 2;
auto sum = [=](int x, int y) { return a + b + x + y; };

// 使用初始化捕获创建新变量的lambda（C++14特性）
auto multiply = [product = a * b](int scalar) { return product * scalar; };
```

实际例子：

1. 作为排序准则

```cpp
// 作为排序准则
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v{4, 1, 3, 5, 2};
    std::sort(v.begin(), v.end(), [](int a, int b) {
        return a < b; // 升序排列
    });

    for (int i : v) {
        std::cout << i << ' ';
    }
    // 输出：1 2 3 4 5
}
```

1. 用于forEach操作

```cpp
#include <vector>
#include <iostream>
#include <algorithm>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};
    std::for_each(v.begin(), v.end(), [](int i) {
        std::cout << i * i << ' '; // 打印每个数字的平方
    });
    // 输出：1 4 9 16 25
}
```

1. 用于累积函数

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};
    int sum = std::accumulate(v.begin(), v.end(), 0, [](int a, int b) {
        return a + b; // 求和
    });

    std::cout << sum << std::endl; // 输出：15
}
```

1. 用于线程构造函数

```cpp
#include <thread>
#include <iostream>

int main() {
    int x = 10;
    std::thread t([x]() {
        std::cout << "Value in thread: " << x << std::endl;
    });
    t.join(); // 输出：Value in thread: 10
    // 注意：线程中使用的x是在创建线程时按值捕获的
}
```

### 3. 详细讨论捕获列表（Capture List）

捕获列表是可选的。指定lambda表达式内部可以访问的外部变量。被引用的外部变量可以在lambda表达式内部被修改，但是按值捕获的外部变量是不可修改的，也就是说，有与号 (`&`) 前缀的变量通过引用进行访问，没有该前缀的变量通过值进行访问。

1. **不捕获任何外部变量**:

`cpp []{ /*...*/ }`

这个lambda不捕获任何外部作用域中的变量。

1. **默认捕获所有外部变量（通过引用）**:

`cpp [&]{ /*...*/ }`

这个lambda捕获所有外部作用域中的变量，并通过引用捕获它们。如果捕获的变量在lambda被调用时已经销毁或超出作用域，则会产生未定义行为。

1. **默认捕获所有外部变量（通过值）**:

`cpp [=]{ /*...*/ }`

这个lambda通过值捕获所有外部作用域中的变量，这意味着它使用的是变量的副本。

1. **显式捕获特定变量（通过值）**:

`cpp [x]{ /*...*/ }`

这个lambda通过值捕获外部变量`x`。

1. **显式捕获特定变量（通过引用）**:

`cpp [&x]{ /*...*/ }`

这个lambda通过引用捕获外部变量`x`。

1. **混合捕获（通过值和引用）**:

`cpp [x, &y]{ /*...*/ }`

这个lambda通过值捕获变量`x`，通过引用捕获变量`y`。

1. **默认通过值捕获，但某些变量通过引用**:

`cpp [=, &x, &y]{ /*...*/ }`

这个lambda默认通过值捕获所有外部变量，但通过引用捕获变量`x`和`y`。

1. **默认通过引用捕获，但某些变量通过值**:

`cpp [&, x, y]{ /*...*/ }`

这个lambda默认通过引用捕获所有外部变量，但通过值捕获变量`x`和`y`。

1. **捕获this指针**:

`cpp [this]{ /*...*/ }`

这允许lambda表达式捕获类成员函数的`this`指针，从而可以访问类的成员变量和函数。

1. **以初始化表达式捕获 （C++14起） - 泛型lambda捕获**:  
    `cpp [x = 42]{ /*...*/ }`  
    创建了一个在lambda内部的匿名变量 `x` ，可以在lambda的函数体中使用这个变量。这个东西还是比较有用的，比如可以直接用移动语义转移 `std::unique_ptr` ，在下面「引用」中详细讨论。  
    
2. **捕获星号this（C++17起）**:  
    `cpp [*this]{ /*...*/ }`  
    这个lambda通过值捕获当前对象（其所在的类的实例）。这样做可以避免在lambda生命周期内this指针变为悬挂指针的风险。在C++17之前，可以通过引用获取this，但是这有一个潜在的内存风险，也就是如果this的生命周期结束了，就会造成内存泄漏。用了星号this，就相当于深度拷贝了一份当前的对象。  
    

> `std::unique_ptr` 是一个独占所有权的智能指针，其设计初衷就是确保同一时间内只有一个实体可以拥有对对象的所有权。因此，`std::unique_ptr` 不能被复制，只能被移动。  
> 如果你想通过值来捕获，那编译器会被报错捏。  
> 如果通过引用捕获，编译器不会报错。但是会有潜在的问题，我想到了三种：  

1. `std::unique_ptr` 的生命比lambda先结束了。这种情况下，lambda内部访问这个已经销毁的 `std::unique_ptr` 会导致程序崩溃。  
    
2. `std::unique_ptr`在捕获后被移动了，那么在lambda的那个引用就是空的，进而导致程序崩溃。  
    
3. 多线程环境中，上面这两个问题会更加频繁地出现。  
    

为了避免这些问题，可以考虑通过值捕获，即显式使用 `std::move` 来转移所有权。多线程环境中就加锁。

代码案例：

1. 使用Lambda作为回调函数 - 该例子也涉及到 `function()`

```cpp
#include <iostream>
#include <functional>

// 假设有一个函数，它在某个操作完成后调用回调函数
void performOperationAsync(std::function<void(int)> callback) {
    // 异步操作...
    int result = 42; // 假设这是异步操作的结果
    callback(result); // 调用回调函数
}

int main() {
    int capture = 100;
    performOperationAsync([capture](int result) {
        std::cout << "Async operation result: " << result
                  << " with captured value: " << capture << std::endl;
    });
}
```

2. 与智能指针一起使用 - 该例子也涉及到 mutable 关键字

```text
#include <iostream>
#include <memory>

void processResource(std::unique_ptr<int> ptr) {
    // 做一些处理
    std::cout << "Processing resource with value " << *ptr << std::endl;
}

int main() {
    auto ptr = std::make_unique<int>(10);

    // 使用Lambda延迟资源处理
    auto deferredProcess = [p = std::move(ptr)]() {
        processResource(std::move(p));
    };

    // 做一些其他操作...
    // ...

    deferredProcess(); // 最终处理资源
}
```

3. 在多线程中同步数据访问

```cpp
int main() {
    std::vector<int> data;
    std::mutex data_mutex;
    std::vector<std::thread> threadsPool;

    // Lambda用于添加数据到vector，确保线程安全
    auto addData = [&](int value) {
        std::lock_guard<std::mutex> lock(data_mutex);
        data.push_back(value);
        std::cout << "Added " << value << " to the data structure." << std::endl;
    };

    threadsPool.reserve(10);
    for (int i = 0; i < 10; ++i) {
        threadsPool.emplace_back(addData, i);
    }

    // 等待所有线程完成
    for (auto& thread : threadsPool) {
        thread.join();
    }
}
```

4. Lambda在范围查询中的应用

```text
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int lower_bound = 3;
    int upper_bound = 7;

    // 使用Lambda找到特定范围内的所有数
    auto range_begin = std::find_if(v.begin(), v.end(), [lower_bound](int x) { return x >= lower_bound; });
    auto range_end = std::find_if(range_begin, v.end(), [upper_bound](int x) { return x > upper_bound; });

    std::cout << "Range: ";
    std::for_each(range_begin, range_end, [](int x) { std::cout << x << ' '; });
    std::cout << std::endl;
}
```

5. 延迟执行

```text
#include <chrono>

// 模拟一个可能需要耗时的操作
void expensiveOperation(int data) {
    // 模拟耗时操作
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Processed data: " << data << std::endl;
}

int main() {
    std::vector<std::function<void()>> deferredOperations;

    deferredOperations.reserve(10);
    // 假设这是一个需要执行耗时操作的循环，但我们不想立即执行它们
    for (int i = 0; i < 10; ++i) {
        // 捕获i并延迟执行
        deferredOperations.emplace_back([i] {
            expensiveOperation(i);
        });
    }

    std::cout << "All operations have been scheduled, doing other work now." << std::endl;

    // 假设现在是一个较好的时间点去执行这些耗时的操作
    for (auto& operation : deferredOperations) {
        // 在一个新线程上执行Lambda表达式以避免阻塞主线程
        std::thread(operation).detach();
    }

    // 给线程一些时间来处理操作
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Main thread finished." << std::endl;
}
/* 注意：在实际的多线程程序中，通常需要考虑线程同步和资源管理
，例如使用 std::async 而不是 std::thread().detach()，
以及使用适当的同步机制如互斥锁和条件变量来保证线程安全。
在这个简化的例子中，为了保持清晰和集中在延迟操作上，
这些细节被省略了。*/

// 下面演示这个例子更合理的版本

#include <iostream>
#include <vector>
#include <future>
#include <chrono>
// 模拟一个可能需要耗时的操作
int expensiveOperation(int data) {
    // 模拟耗时操作
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return data * data; // 返回一些处理结果
}
int main() {
    std::vector<std::future<int>> deferredResults;
    // 启动多个异步任务
    deferredResults.reserve(10);
    for (int i = 0; i < 10; ++i) {
        deferredResults.emplace_back(
            std::async(std::launch::async, expensiveOperation, i)
        );
    }
    std::cout << "All operations have been scheduled, doing other work now." << std::endl;
    // 获取异步任务的结果
    for (auto& future : deferredResults) {
        // get() 会阻塞直到异步操作完成并返回结果
        std::cout << "Processed data: " << future.get() << std::endl;
    }
    std::cout << "Main thread finished." << std::endl;
}
/* 备注：std::async 为我们管理了这一切。
我们也不需要使用互斥锁或其他同步机制，因为每个异步操作都在它自己的线程上运行，
不会互相干扰，并且返回的 future 对象为我们处理了所有必要的同步。
std::async 与 std::launch::async 参数一起使用，
    这会保证每个任务都在不同的线程上异步运行。
如果你没有指定 std::launch::async，C++ 运行时可以决定同步（延迟）执行任务，
这并不是我们希望看到的。
    future.get() 调用将阻塞主线程，直到相应的任务完成，并返回结果。
这使得我们可以安全地获取结果，而不会发生竞争条件或者需要使用互斥锁。*/
```

### 4. `mutable`关键字

首先回顾一下什么是 `mutable` 关键字。除了在lambda表达式中使用，一般我们还会在类成员声明中使用。

当在一个**类成员变量**前使用 `mutable` 关键字时，你可以在该类的 `const` 成员函数中修改这个成员变量。这通常用于那些不影响对象外部状态的成员，例如缓存、调试信息或者可以延迟计算的数据。

```cpp
class MyClass {
public:
    mutable int cache; // 可以在const成员函数中修改
    int data;

    MyClass() : data(0), cache(0) {}

    void setData(int d) const {
        // data = d; // 编译错误：不能在const函数中修改非mutable成员
        cache = d;
    }
};
```

**在lambda表达式中**，`mutable` 关键字允许你修改Lambda内捕获的变量的副本。默认情况下，Lambda表达式中的 `()` 是 `const` 的，一般来说你不能修改通过值捕获的变量。除非使用 `mutable` 。

这里的**关键点**是`mutable`允许修改的是闭包自己的成员变量的**副本**，而不是外部作用域的原始变量。这意味着闭包对外部作用域的 **“封闭性”仍然得以保持**，因为它并没有改变外部作用域的状态，只是改变了自己内部的状态。

不合法的例子：

```cpp
int x = 0;
auto f = [x]() {  x++; // 错误：不能修改捕获的变量  };
f();
```

应该这样：

```cpp
int x = 0;
auto f = [x]() mutable { x++; std::cout << x << std::endl; };
f(); // 正确：输出1
```

实际例子：

1. 捕获变量修改

```text
#include <iostream>
#include <vector>

int main() {
    int count = 0;

    // 创建一个可变lambda表达式，每次调用都递增count
    auto increment = [count]() mutable {
        count++;
        std::cout << count << std::endl;
    };

    increment(); // 输出 1
    increment(); // 输出 2
    increment(); // 输出 3

    // 外部的count仍然是0，因为它是通过值捕获的
    std::cout << "External count: " << count << std::endl; // 输出 External count: 0
}
```

2. 生成唯一的ID

```text
#include <iostream>

int main() {
    int lastId = 0;

    auto generateId = [lastId]() mutable -> int {
        return ++lastId; // 递增并返回新的ID
    };

    std::cout << "New ID: " << generateId() << std::endl; // 输出 New ID: 1
    std::cout << "New ID: " << generateId() << std::endl; // 输出 New ID: 2
    std::cout << "New ID: " << generateId() << std::endl; // 输出 New ID: 3
}
```

3. 状态保持

```text
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 初始状态
    int accumulator = 0;

    // 创建一个可变lambda表达式来累加值
    auto sum = [accumulator](int value) mutable {
        accumulator += value;
        return accumulator; // 返回当前累加值
    };

    std::vector<int> runningTotals(numbers.size());

    // 对每个元素应用sum，生成运行总和
    std::transform(numbers.begin(), numbers.end(), runningTotals.begin(), sum);

    // 输出运行总和
    for (int total : runningTotals) {
        std::cout << total << " "; // 输出 1 3 6 10 15
    }
    std::cout << std::endl;
}
```

### 5. Lambda返回值推导

在C++11中引入了Lambda表达式时，Lambda的返回类型通常需要明确指定。

从C++14开始，对Lambda返回值的推导进行了改进，引入了自动类型推导。

C++14中Lambda返回值的推导遵循以下规则：

1. 如果Lambda的函数体中包含了 `return` 关键字，且所有 `return` 语句后面的表达式的类型都相同，那么Lambda的返回类型被推导为该类型。
2. 如果Lambda的函数体是一个单一的返回语句，或者可以视为一个单一的返回语句（比如一个构造函数或者花括号初始化器），则返回类型被推导为该返回语句表达式的类型。
3. 如果Lambda不返回任何值（即函数体中没有 `return` 语句），或者函数体只包含不返回值的 `return` 语句（即 `return;`），则推导的返回类型为 `void`。
4. **C++11的返回值推导例子**

在C++11中，如果Lambda体包含多个返回语句，必须显式指定返回类型。

```cpp
auto f = [](int x) -> double { // 显式指定返回类型
    if (x > 0)
        return x * 2.5;
    else
        return x / 2.0;
};
```

- **C++14自动推导**

在C++14中，上述Lambda表达式的返回类型可以被自动推导。

```cpp
auto f = [](int x) { // 返回类型自动推导为double
    if (x > 0)
        return x * 2.5; // double
    else
        return x / 2.0; // double
};
```

- **错误示范**

如果返回语句的类型不匹配，不能进行自动推导，这会导致编译错误。

```cpp
auto g = [](int x) { // 编译错误，因为返回类型不一致
    if (x > 0)
        return x * 2.5; // double
    else
        return x;       // int
};
```

但是在C++17之后，如果返回的类型非常不同，以至于无法直接或通过转换统一为一个共同的类型，可以使用 `std::variant` 或 `std::any` ，这样可以包含多种不同的类型：

```cpp
#include <variant>

auto g = [](int x) -> std::variant<int, double> {
    if (x > 0)
        return x * 2.5; // 返回double类型
    else
        return x;       // 返回int类型
};
```

Lambda表达式返回一个 `std::variant<int, double>` 类型，也就是返回一个 `int` 或 `double` 类型的叠加态，后续调用者然后可以检查这个变量，并相应地处理。这部分的内容不做过多讨论。

### 6. 嵌套Lambda

也可以叫做套娃lambda，在一个lambda内再写一个lambda，是一种高级的函数式编程技巧。

简单举一个例子：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 外层Lambda用于遍历集合
    std::for_each(numbers.begin(), numbers.end(), [](int x) {
        // 嵌套Lambda用于计算平方
        auto square = [](int y) { return y * y; };

        // 调用嵌套Lambda并打印结果
        std::cout << square(x) << ' ';
    });

    std::cout << std::endl;
    return 0;
}
```

但是我们需要注意很多问题：

1. 不要写得太复杂了，可读性需要重点考虑。  
    
2. 注意捕获列表的变量的生命周期，下面的例子也会详细讨论。  
    
3. 捕获列表应该尽可能的简单，避免错误。  
    
4. 编译器对嵌套Lambda的优化可能不如顶层函数或类成员函数。  
    

嵌套Lambda如果捕获外层Lambda的局部变量，需要注意变量的生命周期。如果嵌套Lambda的执行延续到外层Lambda的生命周期之外，那么捕获的局部变量将不再有效，就会报错了。

```cpp
#include <iostream>
#include <functional>

std::function<int()> createLambda() {
    int localValue = 10; // 外层Lambda的局部变量

    // 返回一个捕获localValue的Lambda
    return [localValue]() mutable {
        return ++localValue; // 试图修改捕获的变量（由于是值捕获，这是合法的）
    };
}

int main() {
    auto myLambda = createLambda(); // myLambda现在持有一个捕获了已经销毁的局部变量的副本

    std::cout << myLambda() << std::endl; // 这将输出11，但是依赖于已经销毁的localValue的副本
    std::cout << myLambda() << std::endl; // 再次调用将输出12，继续依赖于那个副本

    return 0;
}
```

解释一下，由于Lambda是以值捕获的方式捕获 `localValue` 的，所以它持有 `localValue` 的一个副本，该副本的生命周期与返回的Lambda对象相同。

当我们在 `main` 函数中调用 `myLambda()` 时，它操作的是 `localValue` 副本的状态，而非原始的 `localValue` （已经在 `createLambda` 函数执行完毕后销毁）。这里虽然没有引发未定义行为，但是如果我们使用引用捕获，情况就不一样了：

```cpp
std::function<int()> createLambda() {
    int localValue = 10; // 外层Lambda的局部变量

    // 返回一个捕获localValue引用的Lambda
    return [&localValue]() mutable {
        return ++localValue; // 试图修改捕获的变量
    };
}
// 此时使用createLambda返回的Lambda将会导致未定义行为
```

### 7. Lambda、`std:function`与委托

Lambda表达式、`std::function`和委托是C++中用于实现函数调用和回调机制的三个不同的概念。接下来我们分别讲解三者。

- **Lambda**

C++11引入的一种定义匿名函数对象的语法。Lambda被用于创建一个可调用的实体，即Lambda闭包，通常用于传递给算法或用作回调函数。Lambda表达式可以捕获作用域内的变量，可以按值捕获（拷贝），也可以按引用捕获。Lambda表达式是定义在函数内部的，它们的类型是唯一的，并且不可显式指定。

```cpp
auto lambda = [](int a, int b) { return a + b; };
auto result = lambda(2, 3); // 调用Lambda表达式
```

- `std::function`

`std::function` 是C++11引入的类型擦除包装器，它可以**存储**、**调用**和**复制**任何可调用实体，如函数指针、成员函数指针、Lambda表达式和函数对象。代价就是开销较大。

```cpp
std::function<int(int, int)> func = lambda;
auto result = func(2, 3); // 使用std::function对象调用Lambda表达式
```

- **委托**

委托在C++中不是一个正式的术语。委托通常是一种将函数调用委托给其他对象的机制。在C#中，委托是一种类型安全的函数指针。在C++中，委托的实现一般有几种方式：函数指针、成员函数指针、 `std::function` 和函数对象。下面是一个委托构造函数的例子。

```cpp
class MyClass {
public:
    MyClass(int value) : MyClass(value, "default") { // 委托给另一个构造函数
        std::cout << "Constructor with single parameter called." << std::endl;
    }

    MyClass(int value, std::string text) {
        std::cout << "Constructor with two parameters called: " << value << ", " << text << std::endl;
    }
};

int main() {
    MyClass obj(30); // 这将调用两个构造函数
}
```

---

- **三者对比**

**Lambda表达式**是轻量级的，并且非常适合用于定义简单的局部回调和作为算法的参数。

`std::function` 更重量级，但灵活性更高。例如，如果你有一个需要存储不同类型的回调函数的场景，`std::function`是理想的选择，因为它可以存储任意类型的可调用实体。体现其灵活性的例子。

```cpp
#include <iostream>
#include <functional>
#include <vector>

// 一个接收int并返回void的函数
void printNumber(int number) {
    std::cout << "Number: " << number << std::endl;
}

// 一个Lambda表达式
auto printSum = [](int a, int b) {
    std::cout << "Sum: " << (a + b) << std::endl;
};

// 一个函数对象
class PrintMessage {
public:
    void operator()(const std::string &message) const {
        std::cout << "Message: " << message << std::endl;
    }
};

int main() {
    // 创建一个std::function的向量，可以存储任何类型的可调用对象
    std::vector<std::function<void()>> callbacks;

    // 添加一个普通函数的回调
    int number_to_print = 42;
    callbacks.push_back([=]{ printNumber(number_to_print); });

    // 添加一个Lambda表达式的回调
    int a = 10, b = 20;
    callbacks.push_back([=]{ printSum(a, b); });

    // 添加一个函数对象的回调
    std::string message = "Hello World";
    PrintMessage printMessage;
    callbacks.push_back([=]{ printMessage(message); });

    // 执行所有的回调
    for (auto& callback : callbacks) {
        callback();
    }

    return 0;
}
```

**委托**通常与事件处理相关，在C++中没有内置的事件处理机制，因此`std::function`和Lambda表达式经常用来实现委托模式。具体来讲就是，你定义一个回调接口，用户可以向这个接口注册自己的函数或Lambda表达式，以便在事件发生时调用。一般步骤如下（顺便举个例子）：

1. **定义可以被调用的类型**：你需要确定你的回调函数或Lambda表达式需要接受什么参数，返回什么类型的结果。

```text
using Callback = std::function<void()>; // 没有参数和返回值的回调
```

2. **创建一个类来管理回调**：这个类会持有所有回调函数，并允许用户添加或者移除回调。

```text
class Button {
private:
    std::vector<Callback> onClickCallbacks; // 存储回调的容器

public:
    void addClickListener(const Callback& callback) {
        onClickCallbacks.push_back(callback);
    }

    void click() {
        for (auto& callback : onClickCallbacks) {
            callback(); // 执行每一个回调
        }
    }
};
```

3. **提供一个方法来添加回调**：这个方法允许用户将他们自己的函数或Lambda表达式**注册为回调**。

```text
Button button;
button.addClickListener([]() {
    std::cout << "Button was clicked!" << std::endl;
});
```

4. **提供一个方法来执行回调**：当需要的时候，这个方法会**调用所有已经注册的回调函数**。

```text
button.click(); // 用户点击按钮，触发所有的回调
```

是不是非常简单呢，接下来再来一个例子，加深一下理解。

```cpp
#include <functional>
#include <iostream>
#include <vector>

class Delegate {
public:
    using Callback = std::function<void(int)>;  // 定义回调类型，这里的回调接收一个int参数

    // 注册回调函数
    void registerCallback(const Callback& callback) {
        callbacks.push_back(callback);
    }

    // 触发所有回调函数
    void notify(int value) {
        for (const auto& callback : callbacks) {
            callback(value);  // 执行回调
        }
    }

private:
    std::vector<Callback> callbacks;  // 存储回调的容器
};

int main() {
    Delegate del;

    // 用户注册自己的函数
    del.registerCallback([](int n) {
        std::cout << "Lambda 1: " << n << std::endl;
    });

    // 另一个Lambda表达式
    del.registerCallback([](int n) {
        std::cout << "Lambda 2: " << n * n << std::endl;
    });

    // 触发回调
    del.notify(10);  // 这将调用所有注册的Lambda表达式

    return 0;
}
```

### 8. Lambda在异步和并发编程中

全因为Lambda有个捕获和存储状态的功能，导致我们在编写现代C++并发编程的时候非常有用。

- **Lambda和线程**

直接在 `std::thread` 构造函数中使用 Lambda 表达式来定义线程应该执行的代码。

```text
#include <thread>
#include <iostream>

int main() {
    int value = 42;

    // 创建一个新线程，使用 Lambda 表达式作为线程函数
    std::thread worker([value]() {
        std::cout << "Value in thread: " << value << std::endl;
    });

    // 主线程继续执行...

    // 等待工作线程完成
    worker.join();

    return 0;
}
```

- **Lambda 与 `std::async`**

`std::async` 是一个轻松创建异步的东西，计算完后返回一个 `std::future` 对象，可以调用 `get` 但若未执行完会阻塞。关于这个async还有很多有趣的内容，这里就不赘述了。

```text
#include <future>
#include <iostream>

int main() {
    // 启动一个异步任务
    auto future = std::async([]() {
        // 执行一些操作...
        return "Result from async task";
    });

    // 在此期间，主线程可以执行其他任务...

    // 获取异步操作的结果
    std::string result = future.get();
    std::cout << result << std::endl;

    return 0;
}
```

- **Lambda和 `std::funtion`**

这两个也是经常结合在一起使用的，让我们来这个存储可调用的回调的例子吧。

```cpp
#include <functional>
#include <vector>
#include <iostream>
#include <thread>

// 一个存储 std::function 对象的任务队列
std::vector<std::function<void()>> tasks;

// 添加任务的函数
void addTask(const std::function<void()>& task) {
    tasks.push_back(task);
}

int main() {
    // 添加一个 Lambda 表达式作为任务
    addTask([]() {
        std::cout << "Task 1 executed" << std::endl;
    });

    // 启动一个新线程来处理任务
    std::thread worker([]() {
        for (auto& task : tasks) {
            task(); // 执行任务
        }
    });

    // 主线程继续执行...
    worker.join();

    return 0;
}
```

### 9. 泛型Lambda（C++14）

使用 auto 关键字在参数列表中进行类型推导。

**泛型基本语法：**

```cpp
auto lambda = [](auto x, auto y) {
    return x + y;
};
```

使用例子：

```cpp
#include <numeric>

int main() {
    std::vector<int> vi = {1, 2, 3, 4};
    std::vector<double> vd = {1.1, 2.2, 3.3, 4.4, 5.5};

    // 使用泛型 Lambda 打印 int 类型的元素
    std::for_each(vi.begin(), vi.end(), [](auto n) {
        std::cout << n << ' ';
    });
    std::cout << '\n';

    // 使用泛型 Lambda 打印 double 类型的元素
    std::for_each(vd.begin(), vd.end(), [](auto n) {
        std::cout << n << ' ';
    });
    std::cout << '\n';

    // 使用泛型 Lambda 计算 int 类型的向量的和
    auto sum_vi = std::accumulate(vi.begin(), vi.end(), 0, [](auto total, auto n) {
        return total + n;
    });
    std::cout << "Sum of vi: " << sum_vi << '\n';

    // 使用泛型 Lambda 计算 double 类型的向量的和
    auto sum_vd = std::accumulate(vd.begin(), vd.end(), 0.0, [](auto total, auto n) {
        return total + n;
    });
    std::cout << "Sum of vd: " << sum_vd << '\n';

    return 0;
}
```

也可以做一个打印任何类型容器的lambda。

```cpp
#include <list>

int main() {
    std::vector<int> vec{1, 2, 3, 4};
    std::list<double> lst{1.1, 2.2, 3.3, 4.4};

    auto print = [](const auto& container) {
        for (const auto& val : container) {
            std::cout << val << ' ';
        }
        std::cout << '\n';
    };

    print(vec); // 打印 vector<int>
    print(lst); // 打印 list<double>

    return 0;
}
```

### 10. Lambda的作用域

首先，Lambda可以捕获其定义的作用域内的局部变量，捕获之后，即使原作用域结束，这些变量的副本或引用（取决于捕获方式）仍然可以继续使用。

需要特别注意的点是，引用捕获一个变量，如果这个变量原先所在的作用域已经销毁，那么这就会导致未定义行为。

Lambda也可以捕获全局变量，但是此时就不是通过捕获列表实现的了，因为全局变量不论在哪都可以被访问。

如果有一个 Lambda 嵌套在另一个 Lambda 内部，内部 Lambda 可以捕获外部 Lambda 的捕获列表中的变量。

当 Lambda 捕获了值，即使原本的值没了、Lambda也走了（返回去别的地方了），所有值捕获的变量也将被复制到 Lambda 对象中。这些变量的生命周期将自动延续，直到 Lambda 对象本身被销毁。下面举一个例子：

```cpp
#include <iostream>
#include <functional>

std::function<void()> createLambda() {
    int localValue = 100;  // 局部变量
    return [=]() mutable {  // 以值捕获的方式复制localValue
        std::cout << localValue++ << '\n';
    };
}

int main() {
    auto myLambda = createLambda();  // Lambda复制了localValue
    myLambda();  // 即使createLambda的作用域已经结束，复制的localValue仍然存在于myLambda中
    myLambda();  // 可以安全地继续访问和修改该副本
}
```

当 Lambda 捕获了引用，就是另外一个 Story 了。聪明的读者应该也能猜到，如果原始变量的作用域结束了，Lambda 依赖的是一个悬空引用，这将导致未定义的行为。

### 11. 实践 - 函数计算库

啰里八嗦这么多，现在需要动手实践一下了。无论做啥，目前我们需要掌握的知识点都是那几个：

- 捕获  
    
- 高阶函数  
    
- 可调用对象  
    
- lambda存储  
    
- 可变Lambdas（`mutable`）  
    
- 泛型Lambda  
    

**我们本节的目标是创建一个数学库。支持向量运算、矩阵运算以及提供一个函数解析器，它可以接受字符串形式的数学表达式并返回一个可计算的 Lambda，我们马上开始吧。**

这个项目从简单的数学函数计算开始，逐步扩展到复杂的数学表达式解析和计算。项目编写步骤：

- **基本向量和矩阵运算**  
    
- **函数解析器**  
    
- **更高级的数学函数**  
    
- **复合函数**  
    
- **高阶数学操作**  
    
- **更多拓展...**  
    

### 基本向量和矩阵运算

首先定义向量和矩阵的数据结构，实现基本的算术运算（加减）。

为了简化项目，专注与Lambda的使用，我没有使用模版，因此所有的数据用 `std::vector<double>` 实现。

在下面代码中，我已经实现了一个最基本的向量框架。请读者自行完善框架，包括向量的减法、点乘等操作。

```cpp
// Vector.h
#include <vector>
#include <ostream>

class Vector {
private:
    std::vector<double> elements;
public:
    // 构造函数 - explicit防止隐式转换
    Vector() = default;
    explicit Vector(const std::vector<double> &elems);

    Vector operator+const Vector& rhs) const;

    // 获取向量大小
    [[nodiscard]] size_t size() const { return elements.size(); }

    // 访问元素，返回对象的引用 double&。如果Vector对象是常量，就使用下面的版本
    double& operator[](size_t index) { return elements[index]; }
    const double& operator[](size_t index) const { return elements[index]; }

    // 迭代器支持
    auto begin() { return elements.begin(); }
    auto end() { return elements.end(); }
    auto begin() const { return elements.cbegin(); }
    auto end() const { return elements.cend(); }

    // 让重载的流输出运算符成为友元函数，以便它可以访问私有成员
    friend std::ostream& operator<<(std::ostream& os, const Vector& v);
};
/// Vector.cpp
#include "Vector.h"
Vector::Vector(const std::vector<double>& elems) : elements(elems){}

Vector Vector::operator+(const Vector &rhs) const {
    // 首先确保两个向量一致
    if( this->size() != rhs.size() )
        throw std::length_error("向量大小不一致！");
    Vector result;
    result.elements.reserve(this->size()); // 提前分配内存
    // 使用迭代器遍历向量各个元素
    std::transform(this->begin(), this->end(), rhs.begin(), std::back_inserter(result.elements),
                   [](double_t a,double_t b){ return a+b; });
    return result;
}

std::ostream& operator<<(std::ostream& os, const Vector& v) {
    os << '[';
    for (size_t i = 0; i < v.elements.size(); ++i) {
        os << v.elements[i];
        if (i < v.elements.size() - 1) {
            os << ", ";
        }
    }
    os << ']';
    return os;
}
```

> 可以在声明运算操作中使用 `[[nodiscard]]` 标签，提醒编译器注意检查返回值是否得到使用，然后使用该库的用户就可以在编辑器中得到提醒，例如下面。

### 函数解析器

设计一个函数解析器，它可以将字符串形式的数学表达式转换为 Lambda 表达式。

创建一个能够解析字符串形式的数学表达式并转换为 Lambda 表达式的函数解析器涉及到解析理论，为了简化例子，我们目前只解析最基本的 + 和 - 。然后将函数解析器打包进一个 `ExpressionParser` 的工具类里面。

首先我们先创建一个识别出 + 号和 - 号的解析器：

```cpp
// ExpressionParser.h
#include <functional>
#include <string>

using ExprFunction = std::function<double(double, double)>;

class ExpressionParser {
public:
    static ExprFunction parse_simple_expr(const std::string& expr);
};
// ExpressionParser.cpp
#include "ExpressionParser.h"

ExprFunction ExpressionParser::parse_simple_expr
        (const std::string &expr)
{
    if (expr.find('+') != std::string::npos) {
        return [](double x, double y) { return x + y; };
    }
    else if (expr.find('-') != std::string::npos) {
        return [](double x, double y) { return x - y; };
    }
    // 更多操作...
    return nullptr;
}
```

这一段与Lambda关系不大，可跳过。然后我们可以在这个基础上，改进函数解析器以识别数字。将字符串分割成令牌（数字和操作符），然后根据操作符执行操作。对于更加复杂的表达式，就需要使用比如RPN等算法或者现有的解析库，这里就不弄这么复杂了。

```cpp
// ExpressionParser.h
...
#include <sstream>
...
static double parse_and_compute(const std::string& expr);
...
// ExpressionParser.cpp
...
double ExpressionParser::parse_and_compute(const std::string& expr) {
    std::istringstream iss(expr);
    std::vector<std::string> tokens;
    std::string token;
    while (iss >> token) {
        tokens.push_back(token);
    }

    if (tokens.size() != 3) {
        throw std::runtime_error("Invalid expression format.");
    }

    double num1 = std::stod(tokens[0]);
    const std::string& op = tokens[1];
    double num2 = std::stod(tokens[2]);

    if (op == "+") {
        return num1 + num2;
    } else if (op == "-") {
        return num1 - num2;
    } else {
        throw std::runtime_error("Unsupported operator.");
    }
}
```

测试：

```cpp
// main.cpp
#include "ExpressionParser.h"
...
std::string expr = "10 - 25";
std::cout << expr << " = " << ExpressionParser::parse_and_compute(expr) << std::endl;
```

> 感兴趣的读者也可以尝试解析多个运算符的算法，使用操作符优先级解析算法（如Shunting Yard算法）来转换中缀表达式为逆波兰表示法（RPN）。下面 ~~展示~~ 胡扯 一下数据结构的知识，与Lambda关系不大。

```text
#include <iostream>
#include <stack>
#include <vector>
#include <sstream>
#include <map>
#include <cctype>

// 确定是否为操作符
bool is_operator(const std::string& token) {
    return token == "+" || token == "-" || token == "*" || token == "/";
}

// 确定操作符优先级
int precedence(const std::string& token) {
    if (token == "+" || token == "-") return 1;
    if (token == "*" || token == "/") return 2;
    return 0;
}

// 将中缀表达式转换为逆波兰表示法
std::vector<std::string> infix_to_rpn(const std::vector<std::string>& tokens) {
    std::vector<std::string> output;
    std::stack<std::string> operators;

    for (const auto& token : tokens) {
        if (is_operator(token)) {
            while (!operators.empty() && precedence(operators.top()) >= precedence(token)) {
                output.push_back(operators.top());
                operators.pop();
            }
            operators.push(token);
        } else if (token == "(") {
            operators.push(token);
        } else if (token == ")") {
            while (!operators.empty() && operators.top() != "(") {
                output.push_back(operators.top());
                operators.pop();
            }
            if (!operators.empty()) operators.pop();
        } else {
            output.push_back(token);
        }
    }

    while (!operators.empty()) {
        output.push_back(operators.top());
        operators.pop();
    }

    return output;
}

// 计算逆波兰表示法
double compute_rpn(const std::vector<std::string>& tokens) {
    std::stack<double> operands;

    for (const auto& token : tokens) {
        if (is_operator(token)) {
            double rhs = operands.top(); operands.pop();
            double lhs = operands.top(); operands.pop();
            if (token == "+") operands.push(lhs + rhs);
            else if (token == "-") operands.push(lhs - rhs);
            else if (token == "*") operands.push(lhs * rhs);
            else operands.push(lhs / rhs);
        } else {
            operands.push(std::stod(token));
        }
    }

    return operands.top();
}

// 主函数
int main() {
    std::string input = "3 + 4 * 2 / ( 1 - 5 )";
    std::istringstream iss(input);
    std::vector<std::string> tokens;
    std::string token;
    while (iss >> token) {
        tokens.push_back(token);
    }

    auto rpn = infix_to_rpn(tokens);
    for (const auto& t : rpn) {
        std::cout << t << " ";
    }
    std::cout << std::endl;

    double result = compute_rpn(rpn);
    std::cout << "Result: " << result << std::endl;

    return 0;
}
```

### 更高级的数学函数

**假设**我们的解析器已经能够识别出了更高级的数学操作，如三角函数、对数、指数等，我们就需要为对应的操作提供一个Lambda表达式。

首先我们修改两种不同签名的 `std::function` 的别名。

```cpp
// ExpressionParser.cpp
using UnaryFunction = std::function<double(double)>;
using BinaryFunction = std::function<double(double, double)>;

...

// ExpressionParser.cpp
UnaryFunction ExpressionParser::parse_complex_expr
        (const std::string& expr)
{
    using _t = std::unordered_map<std::string, UnaryFunction>;
    static const _t functions = {
            {"sin", [](double x) -> double { return std::sin(x); }},
            {"cos", [](double x) -> double { return std::cos(x); }},
            {"log", [](double x) -> double { return std::log(x); }},
            // ... 添加更多函数
    };
    auto it = functions.find(expr);
    if (it != functions.end()) {
        return it->second;
    } else {
        // 处理错误或返回一个默认的函数
        return [](double) -> double { return 0.0; }; // 示例错误处理
    }
}
```

### 复合函数

实现复合数学函数的功能，可以通过组合多个 Lambda 表达式来实现。下面是一个小例子：

```cpp
#include <iostream>
#include <cmath>
#include <functional>

int main() {
    // 定义第一个函数 f(x) = sin(x)
    auto f = [](double x) {
        return std::sin(x);
    };

    // 定义第二个函数 g(x) = cos(x)
    auto g = [](double x) {
        return std::cos(x);
    };

    // 创建复合函数 h(x) = g(f(x)) = cos(sin(x))
    auto h = [f, g](double x) {
        return g(f(x));
    };

    // 使用复合函数
    double value = M_PI / 4;  // PI/4
    std::cout << "h(pi/4) = cos(sin(pi/4)) = " << h(value) << std::endl;

    return 0;
}
```

如果想要一个更复杂的复合函数，比如说 $\text{cos}(\text{sin}(\text{exp}(x))$ ，可以这样做：

```cpp
auto exp_func = [](double x) {
    return std::exp(x);
};

// 创建复合函数 h(x) = cos(sin(exp(x)))
auto h_complex = [f, g, exp_func](double x) {
    return g(f(exp_func(x)));
};

std::cout << "h_complex(1) = cos(sin(exp(1))) = " << h_complex(1) << std::endl;
```

使用 Lambda 表达式进行函数组合的优点之一是它们允许你轻松地创建高阶函数，也就是层层套娃的复合函数。

```cpp
auto compose = [](auto f, auto g) {
    return [f, g](double x) {
        return g(f(x));
    };
};

auto h_composed = compose(f, g);
std::cout << "h_composed(pi/4) = " << h_composed(M_PI / 4) << std::endl;
```

上面这个例子就是高阶函数的核心思想。

### 高阶数学操作

实现微分和积分计算器，这些操作可以使用 Lambda 表达式来近似数学函数的导数和积分。

这里的微分使用数值微分的前向差分法来近似倒数 $f'(x)$ 。

积分采用梯形法则的数值积分方法。

```cpp
// 微分
auto derivative = [](auto func, double h = 1e-5) {
    return [func, h](double x) {
        return (func(x + h) - func(x)) / h;
    };
};
// 例如，对 sin(x) 的微分
auto sin_derivative = derivative([](double x) { return std::sin(x); });
std::cout << "sin'(pi/4) ≈ " << sin_derivative(M_PI / 4) << std::endl;


// 积分 - 积分下限 a，积分上限 b 和分割数量 n 
auto trapezoidal_integral = [](auto func, double a, double b, int n = 1000) {
    double h = (b - a) / n;
    double sum = 0.5 * (func(a) + func(b));
    for (int i = 1; i < n; i++) {
        sum += func(a + i * h);
    }
    return sum * h;
};
// 例如，对 sin(x) 在 0 到 pi/2 上的积分
auto integral_sin = trapezoidal_integral([](double x) { return std::sin(x); }, 0, M_PI / 2);
std::cout << "∫sin(x)dx from 0 to pi/2 ≈ " << integral_sin << std::endl;
```

**数值微分 - 前向差分法**

对函数 $$f(x)$$ 在点 $$x$$ 处的导数的数值近似可以通过前向差分公式给出 :

这里的 $$h$$ 代表 $$x$$ 值的一个微小增加。当 $$h$$ 趋向于 0 时，这个比率会趋向于导数的真实值。在代码中我们设置了一个比较小的数值 $$10^{-5}$$ 。

**数值积分 - 梯形法则**

定积分 $$\int_a^b f(x) d x$$ 的数值近似可以使用梯形法则来计算 :

其中 $$n$$ 是区间 $$[ a, b ]$$ 被分成的小区间的数量， $$h$$ 是每个小区间的宽度，计算方法为:

---

## 中级篇

### 1. Lambda的底层实现

从表面上看，lambda表达式似乎只是语法糖，但实际上，编译器会对每个lambda表达式做一些底层转换。

首先，每个lambda表达式的类型都是独一无二的，编译器会为每个lambda生成一个唯一的类类型，这通常被称为**闭包类型**。

> 闭包(closure)这个概念来源于数学中的闭包，它指的是一种结构，这种结构内部的操作是封闭的，不依赖于结构外部的元素。也就是说，任何对集合内元素应用这个操作的结果仍然会在这个集合内。在编程中，这个词被用来描述一个函数与其上下文环境的组合。  
> 一个闭包允许你访问一个外部函数作用域中的变量，即使这个外部函数已经执行结束。函数“封闭”了或“捕获”了其创建时的环境状态。  
> lambda表达式默认情况下生成的闭包类的`operator()`是`const`的，这个情况下开发者不能修改闭包内部的任何数据，即保证了它们不会修改捕获的值，这与闭包的数学和函数式起源相符。

编译器为每个lambda表达式生成一个**闭包类**。这个类**重载**了`operator()`，使得闭包对象可以像函数一样被调用。这个重载的操作符包含了lambda表达式的代码。

lambda表达式可以**捕获**外部变量，这通过闭包类的成员变量实现。捕获可以是值捕获或引用捕获，分别对应于闭包类中值的复制和引用的存储。

闭包类有一个构造函数，该构造函数用于初始化捕获的外部变量。如果是值捕获，这些值会被复制到闭包对象中。如果是引用捕获，外部变量的引用会被存储。

当调用lambda表达式时，实际上是调用闭包对象的`operator()`。

假设lambda表达式如下：

```cpp
[capture](parameters) -> return_type { body }
```

一段编译器可能会生成的伪代码：

```cpp
// 闭包类的伪代码可能如下所示：
class UniqueClosureName {
private:
    // 捕获的变量
    capture_type captured_variable;

public:
    // 构造函数，用于初始化捕获的变量
    UniqueClosureName(capture_type captured) : captured_variable(captured) {}

    // 重载的函数调用操作符
    return_type operator()(parameter_type parameters) const {
        // lambda表达式的主体
        body
    }
};

// 使用闭包类的实例
UniqueClosureName closure_instance(captured_value);
auto result = closure_instance(parameters); // 这相当于调用lambda表达式
```

### 2. Lambda的类型和`decltype`与条件编译`constexpr`（C++17）

我们知道，每个lambda表达式都有其独特的类型，这是由编译器自动生成的。即使两个lambda表达式看起来完全相同，它们的类型也是不同的。这些类型无法直接在代码中表示，我们是借助模板和类型推导机制来操作和推断它们。

获取一个lambda表达式的类型可以使用`decltype`关键字。下面例子中，`decltype(lambda)`得到的是lambda表达式的确切类型。这样就可以声明另一个同类型的变量`another_lambda`，并将原始lambda赋值给它。这种特性一般在模版编程中发挥重要作用。

看下面厨师做菜的例子。你目前不知道食材 `ingredient` 的类型，但是可以用 `decltype` 得到食材的类型。这个的关键点就是，可以明确得到返回值的类型，并且为lambda标记返回类型。

```cpp
template <typename T>
auto cookDish(T ingredient) -> decltype(ingredient.prepare()) {
    return ingredient.prepare();
}
```

进一步的，`decltype` 在 C++ 中的一个重要用途是在**编译时**根据不同的类型选择不同的代码路径，也就是**条件编译**。

```cpp
#include <type_traits>

template <typename T>
void process(T value) {
    if constexpr (std::is_same<decltype(value), int>::value) {
        std::cout << "处理整数: " << value << std::endl;
    } else if constexpr (std::is_same<decltype(value), double>::value) {
        std::cout << "处理浮点数: " << value << std::endl;
    } else {
        std::cout << "处理其他类型: " << value << std::endl;
    }
}
```

下面例子是关于lambda的。

```cpp
#include <iostream>
#include <type_traits>

// 一个泛型函数，根据传入的 lambda 类型执行不同的操作
template <typename T>
void executeLambda(T lambda) {
    if constexpr (std::is_same<decltype(lambda), void(*)()>::value) {
        std::cout << "Lambda is a void function with no parameters." << std::endl;
        lambda();
    } else if constexpr (std::is_same<decltype(lambda), void(*)(int)>::value) {
        std::cout << "Lambda is a void function taking an int." << std::endl;
        lambda(10);
    } else {
        std::cout << "Lambda is of an unknown type." << std::endl;
    }
}

int main() {
    // Lambda with no parameters
    auto lambda1 = []() { std::cout << "Hello from lambda1!" << std::endl; };

    // Lambda with one int parameter
    auto lambda2 = [](int x) { std::cout << "Hello from lambda2, x = " << x << std::endl; };

    executeLambda(lambda1);
    executeLambda(lambda2);

    return 0;
}
```

### 3. Lambda在新标准中的进化

**C++11**

- **引入Lambda表达式**: C++11标准首次引入了Lambda表达式，可以便捷地定义匿名函数对象。基本形式是 `[capture](parameters) -> return_type { body }`。
- **捕获列表**: 支持通过值（`=`）或引用（`&`）捕获外部变量。

**C++14**

- **泛型Lambda**: 允许在参数列表中使用`auto`关键字，使Lambda可以像模板函数一样工作。
- **捕获初始化**: 允许在捕获列表中使用初始化表达式，创建Lambda专有的数据成员。

**C++17**

- **默认构造和赋值**: Lambda表达式产生的闭包类型在某些条件下可以是默认构造的和可赋值的。
- **捕获*this指针**: 通过`*this`捕获，可以值拷贝当前对象到Lambda中，避免悬挂指针问题。
- **constexpr Lambda**: `constexpr` Lambda可以用于在编译时进行计算。在模板元编程、编译时数据生成等场景特别有用。

**C++20**

- **模板Lambda**: Lambda表达式可以有模板参数列表，类似于模板函数。
- **更灵活的捕获列表**: 允许使用`[=, this]`和`[&, this]`形式的捕获列表。
- **隐式移动捕获**: 在适当的情况下，自动采用移动捕获（C++14中仅支持拷贝和引用捕获）。

### 4. 状态保持的Lambda

下面例子，值、引用捕获变量 `x` 就是让Lambda保持状态的关键。还可以捕获并保持自己的状态。

```cpp
#include <iostream>

int main() {
    int x0 = 10, x1 = 20, count = 0;
    auto addX = [x0, &x1, count](int y) mutable {
        count++;
        return x0 + x1 + y + count;
    };

    std::cout << addX(5) << std::endl;  // 输出 36
    std::cout << addX(5) << std::endl;  // 输出 37
    std::cout << addX(5) << std::endl;  // 输出 38
}
```

### 5. 优化与Lambda

Lambda为什么好？

- 內联优化：Lambda一般比较短小，內联优化减少函数调用开销。  
    
- 避免非必要的对象创建：引用捕获、移动语义可以减少大型对象转移的开销、复制。  
    
- 延迟计算：在真正需要结果时才执行计算。  
    

### 6. 与其他编程范式的结合

### 函数式编程

```cpp
class StringBuilder {
private:
    std::string str;

public:
    StringBuilder& append(const std::string& text) {
        str += text;
        return *this;
    }

    const std::string& toString() const {
        return str;
    }
};

// 使用
StringBuilder builder;
builder.append("Hello, ").append("world! ");
std::cout << builder.toString() << std::endl;  // 输出 "Hello, world! "
```

### 流水线调用

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    auto pipeline = vec 
                    | std::views::transform([](int x) { return x * 2; })
                    | std::views::filter([](int x) { return x > 5; });

    for (int n : pipeline) std::cout << n << " "; // 输出满足条件的元素
}
```

### 7. Lambda与异常处理

```cpp
auto divide = [](double numerator, double denominator) {
    if (denominator == 0) {
        throw std::runtime_error("Division by zero.");
    }
    return numerator / denominator;
};

try {
    auto result = divide(10.0, 0.0);
} catch (const std::runtime_error& e) {
    std::cerr << "Caught exception: " << e.what() << std::endl;
}
```

虽然Lambda表达式本身不能包含`try-catch`块（在C++20之前），但可以在Lambda表达式的外部进行异常捕获。即：

```cpp
auto riskyTask = []() {
    // 假设这里有可能抛出异常的代码
};

try {
    riskyTask();
} catch (...) {
    // 处理异常
}
```

从C++20开始，Lambda表达式支持异常规范。

> 在 C++17 之前，可以在函数声明中使用动态异常规范，例如 `throw(Type)`，来指定函数可能抛出的异常类型。但是，这种做法在 C++17 中被废弃，并在 C++20 中完全移除。取而代之的是 noexcept 关键字，它用来指示一个函数是否会抛出异常。

```cpp
auto lambdaNoExcept = []() noexcept {
    // 这里保证不会抛出任何异常
};
```

---

## 进阶篇

### 1. Lambda与`noexcept` （C++11）

`noexcept` 可用于指明Lambda表达式是否保证不抛出异常。

```cpp
auto lambda = []() noexcept {
    // 这里的代码保证不抛出异常
};
```

当编译器知道一个函数不会抛出异常时，它可以生成更优化的代码。

也可以显式的抛出异常，提高代码可读性。但是和不写是一样的。

```cpp
auto lambdaWithException = []() noexcept(false) {
    // 这里的代码可能会抛出异常
};
```

### 2. Lambda中的模板参数（C++20）

在C++20中，Lambda表达式得到了一个重要的增强，即支持模板参数，太酷啦。

```cpp
auto lambda = []<typename T>(T param) {
    // 使用模板参数T的代码
};
auto print = []<typename T>(const T& value) {
    std::cout << value << std::endl;
};

print(10);        // 打印一个整数
print("Hello");   // 打印一个字符串
```

### 3. Lambda的反射

不知道，晚点再写。

### 4. 跨平台和ABI的问题

不知道，晚点再写。