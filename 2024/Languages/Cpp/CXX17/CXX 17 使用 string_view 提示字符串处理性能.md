
C++标准库提供了一个非常优秀的字符串处理类`std::string`，我们可以通过该类完成各种字符串操作。但是`std::string`有一个缺点，它的很多操作都是针对字符串实体，存在不必要的内存拷贝的代码，导致字符串的处理性能不尽如人意。

针对这种情况C++17标准引入了`std::string_view`这个类，该类不会直接作用在字符串实体上，而是记录字符串处理的位置，这样就可以保证用最小的代价对字符串进行处理。

在几个月前写过 std::string_view 的一些简洁介绍，在其中有提及：[Here](https://www.cnblogs.com/RioTian/p/17695062.html)

> 做函数形参的时候，使用`std::string_view`基本一定优于老式的`const std::string&`这种写法。

为了验证这个结论，下面的代码实现了一个断词器，然后针对 64MB 的数据做断词处理并且分别记录使用`std::string`和`std::string_view`作为基础类型时断词器运行的时间：

```cpp
#include <iostream>
#include <chrono>
#include <string_view>

template<class T>
struct tokenizer {
    using string_type = T;
    using value_type = typename T::value_type;

    tokenizer(const string_type & str,
              std::enable_if_t<std::disjunction_v<
                      std::is_same<string_type, std::basic_string<value_type>>,
                      std::is_same<string_type, std::basic_string_view<value_type>>>> * = nullptr)
            : data_(str), begin_(0), end_(0) {}

    string_type operator()(const value_type sep) {
        for ( ; end_ < data_.size(); ++end_ )
        {
            if (data_[end_] == sep)
            {
                auto res = data_.substr(begin_, end_ - begin_);
                begin_ = ++end_;
                return res;
            }
        }
        if (end_ <= data_.size())
        {
            return data_.substr(begin_, end_);
        }

        return "";
    }

    bool more() const { return end_ < data_.size(); }

private:
    const string_type data_;
    size_t begin_, end_;
};

auto make_string_data(size_t count, char sep) {
    std::string data;
    for ( size_t i = 0; i < count; ++i )
    {
        data.push_back('a' + i % 26);
        if (i + 1 != count)
            data.push_back(sep);
    }
    return data;
}

int main() {
    auto data = make_string_data(1024 * 1024 * 32, ' ');

    {
        tokenizer<std::string> tk(data);
        auto start = std::chrono::high_resolution_clock::now();
        while ( tk.more())
        {
            tk(' ');
        }
        auto end = std::chrono::high_resolution_clock::now();
        std::chrono::duration<double> diff = end - start;
        std::cout << "elapsed time = " << diff.count() << std::endl;
    }

    {
        tokenizer<std::string_view> tk(data);
        auto start = std::chrono::high_resolution_clock::now();
        while ( tk.more())
        {
            tk(' ');
        }
        auto end = std::chrono::high_resolution_clock::now();
        std::chrono::duration<double> diff = end - start;
        std::cout << "elapsed time = " << diff.count() << std::endl;
    }

    return 0;
}
```

在上面的代码中`tokenizer`是一个断词器的类模板，接受`std::string`、`std::wstring`等`std::basic_string`模板实例化的类型，同时也能接受`std::string_view`、`std::wstring_view`等`std::basic_string_view`模板实例化的类型。这里采用了SFINAE的方法来约束`tokenizer`的模板实参必须为以上类型。如果编译环境是C++20标准，可以采用概念来约束模板实参类型。

这份代码`tokenizer<std::string>`运行结果是0.45秒，如果将`tokenizer<std::string>`替换为`tokenizer<std::string_view>`运行时间缩短为0.08秒，性能提升是非常明显的 。