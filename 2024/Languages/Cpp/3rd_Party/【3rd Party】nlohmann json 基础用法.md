
#3rd_Party 

> 参考链接：[Here](https://www.coonote.com/cplusplus-note/nlohmann-json.html)

## 什么是nlohman json ?

nlohman json [GitHub - nlohmann/json: JSON for Modern C++](https://github.com/nlohmann/json) 是一个为现代C++（C++11）设计的JSON解析库，主要特点是

1. 易于集成，仅需一个头文件，无需安装依赖
2. 易于使用，可以和STL无缝对接，使用体验近似python中的json

## Minimal Example

CMakeLists.txt 编写教程：[Here](https://blog.csdn.net/qq_38410730/article/details/102477162)

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(nlohmannJson)

set(CMAKE_CXX_STANDARD 17)

include_directories("include")
add_executable(${PROJECT_NAME} src/main.cpp)
```

```cpp
// main.cpp
#include <iostream>
#include "nlohmann/json.hpp"

using namespace std;
using namespace nlohmann;

int main() {
    auto config_json = json::parse(R"({"Happy": true, "pi": 3.1415})");
    cout << config_json << endl;

    return 0;
}
```

## Advanced Sample

### 示范用法

`nlohman::json` 库操作的基本对象是 ` json Object`，全部操作围绕此展开

### 引入代码

```cpp
#include <fstream>
#include <nlohmann/json.hpp>

using json = nlohmann::json;
```

### 读取 JSON 文件

```cpp
#include <fstream>
#include <nlohmann/json.hpp>
using json = nlohmann::json;

// ...

// std::ifstream f("example.json");
std::ifstream f("../config/example.json")
json data = json::parse(f);
```

### 从JSON文本创建JSON对象

假设您要在文件中创建此文本 JSON 值作为对象：

```json
{
  "pi": 3.141,
  "happy": true
}
```

有多种选择：

```cpp
// 用原始字符串和json::parse进行初始化
json ex1 = json::parse(R"(
        {
            "pi": 3.1415,
            "happy": true
        }
)");

// 用原始字符串和literals进行初始化
using namespace nlohmann::literals;
json ex2 = R"(
  {
    "pi": 3.141,
    "happy": true
  }
)"_json;

// 使用初始化列表
json ex3 = {
  {"happy", true},
  {"pi", 3.141},
};
```

### JSON 作为第一类数据类型

下面是一些示例，可让您了解如何使用该类。

假设您要创建 JSON 对象

```json
{
  "pi": 3.141,
  "happy": true,
  "name": "Koshkaaa",
  "nothing": null,
  "answer": {
    "everything": 42
  },
  "list": [1, 0, 2],
  "object": {
    "currency": "USD",
    "value": 42.99
  }
}
```

使用此库，您可以编写：

```cpp
// 创建一个空 json 对象 (null)
json j;

// 添加一个 double 类型成员
j["pi"] = 3.141;

// 添加一个 bool 类型的成员
j["happy"] = true;

// 添加一个字符串类型的成员
j["name"] = "Koshkaaa";

// 添加一个空成员
j["nothing"] = nullptr;

// 添加一个对象中的对象成员 {"answer":{"everything":42}}
j["answer"]["everything"] = 42;

// 添加一个数组对象, 使用 vector
j["list"] = {1,0,2};

// 添加一个对象，使用初始化列表 {"object":{"currency": "USD","value":42.99}}
j["object"] = { {"currency", "USD"}, {"value", 42.99} };

// 也可以一次性写完
json j2 = {
        {"pi", 3.141},
        {"happy", true},
        {"name", "Koshkaaa"},
        {"nothing", nullptr},
        {"answer", {
                       {"everything", 42}
               }},
        {"list", {1, 0, 2}},
        {"object", {
                       {"currency", "USD"},
                     {"value", 42.99}
               }}
};
```

请注意，在所有这些情况下，您永远不需要“告诉”编译器要使用哪种 JSON 值类型。如果你想明确或表达一些边缘情况，函数json::array（）和json::object（）会有所帮助：

```cpp
// 初始化一个空数组对象
json empty_array_explicit = json::array();

// 初始化一个对象
json empty_object_implicit = json({});
json empty_object_explicit = json::object();

// 初始化数组键值对 [["currency", "USD"], ["value", 42.99]]
json array_not_object = json::array({ {"currency", "USD"}, {"value", 42.99} });
```

### 序列化/反序列化

#### json对象和string互转JSON对象和string互转

您可以通过追加到字符串文本来创建 JSON 值（反序列化）：`_json`

```cpp
// 从字符串创建json对象
json j = "{ \"happy\": true, \"pi\": 3.141 }"_json;
// 使用原始字符串创建json对象
auto j2 = R"(
  {
    "happy": true,
    "pi": 3.141
  }
)"_json;
```

请注意，如果不附加后缀，则传递的字符串文字不会被解析，而只是用作 JSON 字符串 价值。也就是说，只会存储字符串而不是解析实际对象。`_jsonjson j = "{ \"happy\": true, \"pi\": 3.141 }""{ "happy": true, "pi": 3.141 }"`

上面的例子也可以用json::parse()显式表示：

```cpp
auto j3 = json::parse(R"({"happy": true, "pi": 3.141})");
```

获取 JSON 字符串（序列化）：

```cpp
std::string s = j.dump();    // {"happy":true,"pi":3.141}

// 序列化美化json形式
// 传入数字指定缩进空格数
std::cout << j.dump(4) << std::endl;
// {
//     "happy": true,
//     "pi": 3.141
// }
```

### 流输入输出（例如文件读写、字符串流）

您还可以使用流来序列化和反序列化：

```cpp
// 从标准输入中反序列化
json j;
std::cin >> j;

// 序列化到标准输出
std::cout << j;

// 美化输出
std::cout << std::setw(4) << j << std::endl;
```

文件读写json：std::istream，std::ostream

```cpp
// 读取json文件
std::ifstream i("file.json");
json j;
i >> j;

//json对象美化输出到文件
std::ofstream o("pretty.json");
o << std::setw(4) << j << std::endl;
```

从迭代器范围读取json

可以从迭代器范围解析 JSON;也就是说，从迭代器可访问的任何容器中，迭代器是 1、2 或 4 个字节的整数类型，将分别解释为 UTF-8、UTF-16 和 UTF-32。例如，`std::vector<std::uint8_t>`,`std::list<std::uint16_t>`

```cpp
std::vector<std::uint8_t> v = {'t', 'r', 'u', 'e'};
json j = json::parse(v.begin(), v.end());
```

```cpp
std::vector<std::uint8_t> v = {'t', 'r', 'u', 'e'};
json j = json::parse(v);
```

### 自定义数据源

由于 parse 函数接受任意迭代器范围，因此您可以通过实现概念来提供自己的数据源。`LegacyInputIterator`

```cpp
struct MyContainer {
  void advance();
  const char& get_current();
};

struct MyIterator {
    using difference_type = std::ptrdiff_t;
    using value_type = char;
    using pointer = const char*;
    using reference = const char&;
    using iterator_category = std::input_iterator_tag;

    MyIterator& operator++() {
        MyContainer.advance();
        return *this;
    }

    bool operator!=(const MyIterator& rhs) const {
        return rhs.target != target;
    }

    reference operator*() const {
        return target.get_current();
    }

    MyContainer* target = nullptr;
};

MyIterator begin(MyContainer& tgt) {
    return MyIterator{&tgt};
}

MyIterator end(const MyContainer&) {
    return {};
}

void foo() {
    MyContainer c;
    json j = json::parse(c);
}
```

## 类似 STL 的访问

我们将 JSON 类设计为类似于 STL 容器的行为。事实上，它满足可逆容器的要求。

```cpp
// 用push_back创建一个数组对象
json j;
j.push_back("foo");
j.push_back(1);
j.push_back(true);

// 使用 emplace_back
j.emplace_back(1.78);

// 迭代访问
for (json::iterator it = j.begin(); it != j.end(); ++it) {
  std::cout << *it << '\n';
}

// 范围迭代
for (auto& element : j) {
  std::cout << element << '\n';
}

// getter/setter接口
const auto tmp = j[0].get<std::string>();
j[1] = 42;
bool foo = j.at(2);

// 比较
j == R"(["foo", 1, true, 1.78])"_json;  // true

// 其他
j.size();     // 4 个对象
j.empty();    // 是否为空：false
j.type();     // 获取类型：json::value_t::array
j.clear();    // 清空

// 检查类型
j.is_null();
j.is_boolean();
j.is_number();
j.is_object();
j.is_array();
j.is_string();

// 创建一个对象
json o;
o["foo"] = 23;
o["bar"] = false;
o["baz"] = 3.141;

// 使用 emplace
o.emplace("weather", "sunny");

// 迭代访问
for (json::iterator it = o.begin(); it != o.end(); ++it) {
  std::cout << it.key() << " : " << it.value() << "\n";
}

// 循环访问
for (auto& el : o.items()) {
  std::cout << el.key() << " : " << el.value() << "\n";
}

// 匿名对象迭代器(C++17)
for (auto& [key, value] : o.items()) {
  std::cout << key << " : " << value << "\n";
}

// 查找key
if (o.contains("foo")) {
  // 找到foo的键值
}

// 通过迭代器查找
if (o.find("foo") != o.end()) {
  // 找到foo的键值
}

// 用count()统计是否有键值
int foo_present = o.count("foo"); // 1
int fob_present = o.count("fob"); // 0

// 删除foo对象
o.erase("foo");
```

## 从 STL 容器转换

从STL容器转换到json，std::list转json，std::vector转json

```cpp
std::vector<int> c_vector {1, 2, 3, 4};
json j_vec(c_vector);
// [1, 2, 3, 4]

std::deque<double> c_deque {1.2, 2.3, 3.4, 5.6};
json j_deque(c_deque);
// [1.2, 2.3, 3.4, 5.6]

std::list<bool> c_list {true, true, false, true};
json j_list(c_list);
// [true, true, false, true]

std::forward_list<int64_t> c_flist {12345678909876, 23456789098765, 34567890987654, 45678909876543};
json j_flist(c_flist);
// [12345678909876, 23456789098765, 34567890987654, 45678909876543]

std::array<unsigned long, 4> c_array {{1, 2, 3, 4}};
json j_array(c_array);
// [1, 2, 3, 4]

std::set<std::string> c_set {"one", "two", "three", "four", "one"};
json j_set(c_set); // only one entry for "one" is used
// ["four", "one", "three", "two"]

std::unordered_set<std::string> c_uset {"one", "two", "three", "four", "one"};
json j_uset(c_uset); // only one entry for "one" is used
// maybe ["two", "three", "four", "one"]

std::multiset<std::string> c_mset {"one", "two", "one", "four"};
json j_mset(c_mset); // both entries for "one" are used
// maybe ["one", "two", "one", "four"]

std::unordered_multiset<std::string> c_umset {"one", "two", "one", "four"};
json j_umset(c_umset); // both entries for "one" are used
// maybe ["one", "two", "one", "four"]
```

`std::map` 转 `json` 、`std::unordered` 转 `json`

```cpp
std::map<std::string, int> c_map { {"one", 1}, {"two", 2}, {"three", 3} };
json j_map(c_map);
// {"one": 1, "three": 3, "two": 2 }

std::unordered_map<const char*, double> c_umap { {"one", 1.2}, {"two", 2.3}, {"three", 3.4} };
json j_umap(c_umap);
// {"one": 1.2, "two": 2.3, "three": 3.4}

std::multimap<std::string, bool> c_mmap { {"one", true}, {"two", true}, {"three", false}, {"three", true} };
json j_mmap(c_mmap); // only one entry for key "three" is used
// maybe {"one": true, "two": true, "three": true}

std::unordered_multimap<std::string, bool> c_ummap { {"one", true}, {"two", true}, {"three", false}, {"three", true} };
json j_ummap(c_ummap); // only one entry for key "three" is used
// maybe {"one": true, "two": true, "three": true}
```
