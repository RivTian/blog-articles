![](https://upload-images.jianshu.io/upload_images/1489066-2daebb1d1c51bfea.gif?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

[转到github](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%23examples)

nlohmann::json在项目中的实际使用详见C++静态代码检测工具：[cppdetector](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fzhlongfj%2Fcppdetector)

## 设计目标

已经有无数的[JSON](https://links.jianshu.com/go?to=http%3A%2F%2Fjson.org%2F)库，每个库都有其存在的理由。我们有这些设计目标：

- **直观的语法**。在像Python这样的语言中，JSON感觉就像是一级数据类型。我们使用现代C++的操作魔法来实现类似普通代码般的感觉。看看[下面的例子](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%23examples)，你就会明白我的意思。
    
- **整合**。我们的整个代码只有一个头文件[`json.hpp`](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fblob%2Fdevelop%2Fsingle_include%2Fnlohmann%2Fjson.hpp)。没有库，没有子项目，没有依赖项，没有复杂的构建系统，仅仅使用标准C ++ 11编写。总而言之，不需要调整任何编译器标志或项目设置。
    
- **压力测试**。我们的类经过严格的[单元测试](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Ftree%2Fdevelop%2Ftest%2Fsrc)，涵盖了[100％](https://links.jianshu.com/go?to=https%3A%2F%2Fcoveralls.io%2Fr%2Fnlohmann%2Fjson)的代码，包括所有特殊行为。此外，我们使用[Valgrind](https://links.jianshu.com/go?to=http%3A%2F%2Fvalgrind.org%2F)和[Clang Sanitizers](https://links.jianshu.com/go?to=https%3A%2F%2Fclang.llvm.org%2Fdocs%2Findex.html)做了检测，没有内存泄漏。还使用[谷歌OSS-Fuzz](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoogle%2Foss-fuzz%2Ftree%2Fmaster%2Fprojects%2Fjson)对所有解析器进行24/7模糊测试，到目前为止有效地执行了数十亿次测试。为了保持高质量，该项目遵循[核心基础设施倡议（CII）的最佳实践](https://links.jianshu.com/go?to=https%3A%2F%2Fbestpractices.coreinfrastructure.org%2Fprojects%2F289)。
    

对我们来说并不那么重要的其他方面：

- **内存效率**。每个JSON对象都有一个指针（联合的最大大小）和一个枚举元素（1个字节）的开销。默认泛化使用以下C ++数据类型：`std::string`用于字符串，`int64_t`，`uint64_t`或`double`用于数字，`std::map`用于对象，`std::vector`用于数组和`bool`用于布尔值。但是，您可以根据需要特化`basic_json`通用类。
    
- **速度**。肯定有[更快的JSON库](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fmiloyip%2Fnativejson-benchmark%23parsing-time)。但是，如果您的目标是通过添加单个头文件添加JSON支持来加速开发，那么这个库就是您的选择。如果您知道如何使用`std::vector`或`std::map`，你就已经准备好了。
    

有关详细信息，请参阅[贡献指南](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fblob%2Fmaster%2F.github%2FCONTRIBUTING.md%23please-dont)。

- **整合**  
    [`json.hpp`](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fblob%2Fdevelop%2Fsingle_include%2Fnlohmann%2Fjson.hpp)唯一所需的文件，在目录`single_include/nlohmann`或[这里](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Freleases)。你需要添加：

```source-c
#include <nlohmann/json.hpp>

// for convenience
using json = nlohmann::json;
```

到您要处理JSON的文件时，设置必要的开关以启用C ++ 11（例如，`-std=c++11`对于GCC和Clang）。

您可以进一步使用文件[`include/nlohmann/json_fwd.hpp`](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fblob%2Fdevelop%2Finclude%2Fnlohmann%2Fjson_fwd.hpp)进行前向声明。安装json_fwd.hpp（作为cmake安装步骤的一部分），可以通过设置`-DJSON_MultipleHeaders=ON`来实现。

### 包管理器

🍺如果您使用的是OS X和[Homebrew](https://links.jianshu.com/go?to=http%3A%2F%2Fbrew.sh%2F)，只需键入`brew tap nlohmann/json`并`brew install nlohmann_json`设置即可。如果您想要最新版本而不是最新版本，请使用`brew install nlohmann_json --HEAD`。

如果您正在使用[Meson Build System](https://links.jianshu.com/go?to=http%3A%2F%2Fmesonbuild.com%2F)，那么您可以将此存储库包装为子项目。

如果您使用[Conan](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.conan.io%2F)来管理您的依赖项，只需添加`jsonformoderncpp/x.y.z@vthiery/stable`您`conanfile.py`的要求，`x.y.z`您要使用的发行版本在哪里。如果您遇到包装问题，请[在此处提出](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fvthiery%2Fconan-jsonformoderncpp%2Fissues)问题。

如果您使用[Spack](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.spack.io%2F)来管理依赖项，则可以使用该`nlohmann_json`程序包。有关包装的任何问题，请参阅[spack项目](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fspack%2Fspack)。

如果你在项目中使用[猎人](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fruslo%2Fhunter%2F)来获取外部依赖关系，那么你可以使用[nlohmann_json包](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.hunter.sh%2Fen%2Flatest%2Fpackages%2Fpkg%2Fnlohmann_json.html)。有关包装的任何问题，请参阅猎人项目。

如果您使用的是[Buckaroo](https://links.jianshu.com/go?to=https%3A%2F%2Fbuckaroo.pm%2F)，则可以安装此库的模块`buckaroo install nlohmann/json`。请[在这里](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FLoopPerfect%2Fbuckaroo-recipes%2Fissues%2Fnew%3Ftitle%3Dnlohmann%2Fnlohmann%2Fjson)提出问题。

如果您在项目中使用[vcpkg](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMicrosoft%2Fvcpkg%2F)来获取外部依赖项，那么您可以使用[nlohmann-json包](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMicrosoft%2Fvcpkg%2Ftree%2Fmaster%2Fports%2Fnlohmann-json)。有关包装的任何问题，请参阅vcpkg项目。

如果您使用的是[cget](https://links.jianshu.com/go?to=http%3A%2F%2Fcget.readthedocs.io%2Fen%2Flatest%2F)，则可以安装最新的开发版本`cget install nlohmann/json`。可以安装特定版本`cget install nlohmann/json@v3.1.0`。此外，可以通过添加`-DJSON_MultipleHeaders=ON`标志（即，`cget install nlohmann/json -DJSON_MultipleHeaders=ON`）来安装多标头版本。

如果您使用的是[CocoaPods](https://links.jianshu.com/go?to=https%3A%2F%2Fcocoapods.org%2F)，则可以通过将pod添加`"nlohmann_json", '~>3.1.2'`到podfile 来使用该库（请参阅[示例](https://links.jianshu.com/go?to=https%3A%2F%2Fbitbucket.org%2Fbenman%2Fnlohmann_json-cocoapod%2Fsrc%2Fmaster%2F)）。请[在这里](https://links.jianshu.com/go?to=https%3A%2F%2Fbitbucket.org%2Fbenman%2Fnlohmann_json-cocoapod%2Fissues%3Fstatus%3Dnew%26status%3Dopen)提出问题。

## 例子

除了下面的示例，您可能需要查看每个函数（包含单独代码示例）的[文档](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2F)（例如，查看[`emplace()`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_a5338e282d1d02bed389d852dd670d98d.html%23a5338e282d1d02bed389d852dd670d98d)）。所有[示例文件](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Ftree%2Fdevelop%2Fdoc%2Fexamples)都可以自己编译和执行（例如，文件[emplace.cpp](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fblob%2Fdevelop%2Fdoc%2Fexamples%2Femplace.cpp)）。

### JSON作为一级的数据类型

以下是一些示例，可以让您了解如何使用该类。

假设您要创建如下的JSON对象：

```source-c
{
  "pi": 3.141,
  "happy": true,
  "name": "Niels",
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

你可以这样写：

```source-c
// create an empty structure (null)
json j;

// add a number that is stored as double (note the implicit conversion of j to an object)
j["pi"] = 3.141;

// add a Boolean that is stored as bool
j["happy"] = true;

// add a string that is stored as std::string
j["name"] = "Niels";

// add another null object by passing nullptr
j["nothing"] = nullptr;

// add an object inside the object
j["answer"]["everything"] = 42;

// add an array that is stored as std::vector (using an initializer list)
j["list"] = { 1, 0, 2 };

// add another object (using an initializer list of pairs)
j["object"] = { {"currency", "USD"}, {"value", 42.99} };

// instead, you could also write (which looks very similar to the JSON above)
json j2 = {
  {"pi", 3.141},
  {"happy", true},
  {"name", "Niels"},
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

请注意，在所有这些情况下，您永远不需要“告诉”编译器您要使用哪种JSON值类型。如果你想显性指定类型或表达一些特定的意图，函数：[`json::array`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_aa80485befaffcadaa39965494e0b4d2e.html%23aa80485befaffcadaa39965494e0b4d2e)和[`json::object`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_aa13f7c0615867542ce80337cbcf13ada.html%23aa13f7c0615867542ce80337cbcf13ada)可满足您的需求：

```php
// a way to express the empty array []
json empty_array_explicit = json::array();

// ways to express the empty object {}
json empty_object_implicit = json({});
json empty_object_explicit = json::object();

// a way to express an _array_ of key/value pairs [["currency", "USD"], ["value", 42.99]]
json array_not_object = json::array({ {"currency", "USD"}, {"value", 42.99} });
```

### 序列化/反序列化

#### To/from strings

您可以通过附加`_json`到字符串来创建JSON值（反序列化）：

```php
// create object from string literal
json j = "{ \"happy\": true, \"pi\": 3.141 }"_json;

// or even nicer with a raw string literal
auto j2 = R"(
  {
    "happy": true,
    "pi": 3.141
  }
)"_json;
```

请注意，如果不附加`_json`后缀，则不会解析传递的字符串文字，而只是用作JSON字符串值。也就是说，`json j = "{ \"happy\": true, \"pi\": 3.141 }"`只是存储字符串`"{ "happy": true, "pi": 3.141 }"`而不是解析实际对象。

以上示例也可以使用[`json::parse()`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_aa9676414f2e36383c4b181fe856aa3c0.html%23aa9676414f2e36383c4b181fe856aa3c0)明确表达：

```cpp
// parse explicitly
auto j3 = json::parse("{ \"happy\": true, \"pi\": 3.141 }");
```

您还可以获取JSON值的字符串表示形式（序列化）：

```cpp
// explicit conversion to string
std::string s = j.dump();    // {\"happy\":true,\"pi\":3.141}

// serialization with pretty printing
// pass in the amount of spaces to indent
std::cout << j.dump(4) << std::endl;
// {
//     "happy": true,
//     "pi": 3.141
// }
```

注意序列化和赋值之间的区别：

```cpp
// store a string in a JSON value
json j_string = "this is a string";

// retrieve the string value (implicit JSON to std::string conversion)
std::string cpp_string = j_string;
// retrieve the string value (explicit JSON to std::string conversion)
auto cpp_string2 = j_string.get<std::string>();

// retrieve the serialized value (explicit JSON serialization)
std::string serialized_string = j_string.dump();

// output of original string
std::cout << cpp_string << " == " << cpp_string2 << " == " << j_string.get<std::string>() << '\n';
// output of serialized value
std::cout << j_string << " == " << serialized_string << std::endl;
```

[`.dump()`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_a5adea76fedba9898d404fef8598aa663.html%23a5adea76fedba9898d404fef8598aa663)始终返回序列化值，而[`.get<std::string>()`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_a16f9445f7629f634221a42b967cdcd43.html%23a16f9445f7629f634221a42b967cdcd43)返回最初存储的字符串值。

请注意，该库仅支持UTF-8。当您在库中存储具有不同编码的字符串时，调用[`dump()`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_a5adea76fedba9898d404fef8598aa663.html%23a5adea76fedba9898d404fef8598aa663)可能会抛出异常。

#### To/from streams (e.g. files, string streams)

您还可以使用流来序列化和反序列化：

```cpp
// deserialize from standard input
json j;
std::cin >> j;

// serialize to standard output
std::cout << j;

// the setw manipulator was overloaded to set the indentation for pretty printing
std::cout << std::setw(4) << j << std::endl;
```

这些运算符适用于`std::istream`或`std::ostream`的任何子类。这是文件的类似示例：

```cpp
// read a JSON file
std::ifstream i("file.json");
json j;
i >> j;

// write prettified JSON to another file
std::ofstream o("pretty.json");
o << std::setw(4) << j << std::endl;
```

请注意，为`failbit`设置异常位不适用此用例。由于使用了`noexcept`说明符，它将导致程序终止。

#### 从迭代器范围读取

您还可以从迭代器范围解析JSON; 也就是说，其存储内容为连续的字节序列，可从迭代器访问的任何容器，例如std::vector<std::uint8_t>：

```cpp
std::vector<std::uint8_t> v = {'t', 'r', 'u', 'e'};
json j = json::parse(v.begin(), v.end());
```

您也可以去掉范围[begin, end)操作符：

```cpp
std::vector<std::uint8_t> v = {'t', 'r', 'u', 'e'};
json j = json::parse(v);
```

#### SAX interface(待补充)

### STL-like access

我们定义的JSON类的行为与STL容器一样。事实上，它遵循[**ReversibleContainer**](https://links.jianshu.com/go?to=https%3A%2F%2Fen.cppreference.com%2Fw%2Fcpp%2Fnamed_req%2FReversibleContainer)规范。

```cpp
// create an array using push_back
json j;
j.push_back("foo");
j.push_back(1);
j.push_back(true);

// also use emplace_back
j.emplace_back(1.78);

// iterate the array
for (json::iterator it = j.begin(); it != j.end(); ++it) {
  std::cout << *it << '\n';
}

// range-based for
for (auto& element : j) {
  std::cout << element << '\n';
}

// getter/setter
const std::string tmp = j[0];
j[1] = 42;
bool foo = j.at(2);

// comparison
j == "[\"foo\", 1, true]"_json;  // true

// other stuff
j.size();     // 3 entries
j.empty();    // false
j.type();     // json::value_t::array
j.clear();    // the array is empty again

// convenience type checkers
j.is_null();
j.is_boolean();
j.is_number();
j.is_object();
j.is_array();
j.is_string();

// create an object
json o;
o["foo"] = 23;
o["bar"] = false;
o["baz"] = 3.141;

// also use emplace
o.emplace("weather", "sunny");

// special iterator member functions for objects
for (json::iterator it = o.begin(); it != o.end(); ++it) {
  std::cout << it.key() << " : " << it.value() << "\n";
}

// find an entry
if (o.find("foo") != o.end()) {
  // there is an entry with key "foo"
}

// or simpler using count()
int foo_present = o.count("foo"); // 1
int fob_present = o.count("fob"); // 0

// delete an entry
o.erase("foo");
```

### 从STL容器转换

任何序列容器（std::array，std::vector，std::deque，std::forward_list，std::list），其值可以被用于构建JSON值（例如，整数，浮点数，布尔值，字符串类型，或者在本节中描述的STL容器）都可被用于创建JSON数组。这同样适用于类似的关联容器（std::set，std::multiset，std::unordered_set，std::unordered_multiset），但是在这些情况下，数组中的元素的顺序取决于元素是如何在各个STL容器排序。

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

同样，任何键值对容器（std::map，std::multimap，std::unordered_map，std::unordered_multimap），其键可以构造一个std::string，并且其值可以被用于构建JSON值（见上文示例）可用于创建一个JSON对象。请注意，在多映射的情况下，JSON对象中只使用一个键，值取决于STL容器的内部顺序。

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

### JSON Pointer和JSON Patch

该库支持**JSON Pointer**（[RFC 6901](https://links.jianshu.com/go?to=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6901)）作为处理结构化值的替代方法。最重要的是，**JSON Patch**（[RFC 6902](https://links.jianshu.com/go?to=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6902)）允许描述两个JSON值之间的差异 - 有效地允许Unix中已知的patch和diff操作。

```bash
// a JSON value
json j_original = R"({
  "baz": ["one", "two", "three"],
  "foo": "bar"
})"_json;

// access members with a JSON pointer (RFC 6901)
j_original["/baz/1"_json_pointer];
// "two"

// a JSON patch (RFC 6902)
json j_patch = R"([
  { "op": "replace", "path": "/baz", "value": "boo" },
  { "op": "add", "path": "/hello", "value": ["world"] },
  { "op": "remove", "path": "/foo"}
])"_json;

// apply the patch
json j_result = j_original.patch(j_patch);
// {
//    "baz": "boo",
//    "hello": ["world"]
// }

// calculate a JSON patch from two JSON values
json::diff(j_result, j_original);
// [
//   { "op":" replace", "path": "/baz", "value": ["one", "two", "three"] },
//   { "op": "remove","path": "/hello" },
//   { "op": "add", "path": "/foo", "value": "bar" }
// ]
```

### JSON合并Patch

该库支持**JSON Merge Patch**（[RFC 7386](https://links.jianshu.com/go?to=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7386)）作为补丁格式。它不是使用JSON指针（参见上文）来指定要操作的值，而是使用与所修改文档非常相似的语法来描述更改。

```swift
// a JSON value
json j_document = R"({
  "a": "b",
  "c": {
    "d": "e",
    "f": "g"
  }
})"_json;

// a patch
json j_patch = R"({
  "a":"z",
  "c": {
    "f": null
  }
})"_json;

// apply the patch
j_original.merge_patch(j_patch);
// {
//  "a": "z",
//  "c": {
//    "d": "e"
//  }
// }
```

隐式转换  
JSON对象的类型由要存储的表达式自动确定。同样，隐式转换存储的值。

```cpp
// strings
std::string s1 = "Hello, world!";
json js = s1;
std::string s2 = js;

// Booleans
bool b1 = true;
json jb = b1;
bool b2 = jb;

// numbers
int i = 42;
json jn = i;
double f = jn;

// etc.
```

您也可以显式请求值：

```cpp
std::string vs = js.get<std::string>();
bool vb = jb.get<bool>();
int vi = jn.get<int>();

// etc.
```

请注意`char`类型不能自动转换成JSON字符串，而是转换成整型数。要转换成字符串必须显式指定：

```cpp
char ch = 'A';                       // ASCII value 65
json j_default = ch;                 // stores integer number 65
json j_string = std::string(1, ch);  // stores string "A"
```

### 任意类型转换

任何类型都可以用JSON序列化，而不仅仅是STL容器和标量类型。通常，您将遵循这些准绳来做事：

```cpp
namespace ns {
    // a simple struct to model a person
    struct person {
        std::string name;
        std::string address;
        int age;
    };
}

ns::person p = {"Ned Flanders", "744 Evergreen Terrace", 60};

// convert to JSON: copy each value into the JSON object
json j;
j["name"] = p.name;
j["address"] = p.address;
j["age"] = p.age;

// ...

// convert from JSON: copy each value from the JSON object
ns::person p {
    j["name"].get<std::string>(),
    j["address"].get<std::string>(),
    j["age"].get<int>()
};
```

这个代码没有问题，但是有点啰嗦，我们有一个更好的办法：

```cpp
// create a person
ns::person p {"Ned Flanders", "744 Evergreen Terrace", 60};

// conversion: person -> json
json j = p;

std::cout << j << std::endl;
// {"address":"744 Evergreen Terrace","age":60,"name":"Ned Flanders"}

// conversion: json -> person
ns::person p2 = j;

// that's it
assert(p == p2);
```

#### 基本用法

要使其适用于您的某种类型，您只需提供两个功能：

```cpp
using nlohmann::json;

namespace ns {
    void to_json(json& j, const person& p) {
        j = json{{"name", p.name}, {"address", p.address}, {"age", p.age}};
    }

    void from_json(const json& j, person& p) {
        p.name = j.at("name").get<std::string>();
        p.address = j.at("address").get<std::string>();
        p.age = j.at("age").get<int>();
    }
} // namespace ns
```

就这么简单！`json`使用您的类型调用构造函数时，您的自定义方法`to_json`将被自动调用。同样，在调用`get<your_type>()`时，`from_json`将被自动调用。  
一些重要的点：

- 那些方法**必须**在你的类型的命名空间（可以是全局命名空间）中，否则库将无法找到它们（在这个例子中，`person`在命名空间中`ns`中定义）。
- 在您使用隐式转换的地方，那些方法必须是有效的（例如：正确的头文件必须被包含）查看[问题1108](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fissues%2F1108)，了解可能发生的错误。
- 使用`get<your_type>()`时，`your_type` **必须**是[DefaultConstructible](https://links.jianshu.com/go?to=https%3A%2F%2Fen.cppreference.com%2Fw%2Fcpp%2Fnamed_req%2FDefaultConstructible)。（后面会描述一种可以绕过这个要求的方法。）
- 在函数`from_json`中，使用函数[`at()`](https://links.jianshu.com/go?to=https%3A%2F%2Fnlohmann.github.io%2Fjson%2Fclassnlohmann_1_1basic__json_a93403e803947b86f4da2d1fb3345cf2c.html%23a93403e803947b86f4da2d1fb3345cf2c)来访问对象值而不是`operator[]`。如果某个键不存在，则`at`抛出一个可以处理的异常，然而`operator[]`显示未定义的行为。
- 如果您的类型包含多个`operator=`定义，则代码`your_variable = your_json;` [可能无法编译](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fissues%2F667)。您需要改写成`your_variable = your_json.get<decltype your_variable>();`。
- 您不需要为STL类型添加序列化或反序列化程序，例如`std::vector`：库已经实现了这些。
- 注意函数`from_json`/ `to_json`的定义顺序：如果一个类型`B`有类型的成员`A`，你**必须**在定义`to_json(B)`之前，定义`to_json(A)`。请查看[问题561](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fissues%2F561)以获取更多详细信息。

#### 怎么转换第三方库的类型？

这需要更高级的技术。首先，让我们来看看这种转换机制是如何工作的：  
该库使用**JSON Serializers**将类型转换成json。`nlohmann::json`的默认序列化程序是`nlohmann::adl_serializer`(ADL means [Argument-Dependent Lookup](https://links.jianshu.com/go?to=https%3A%2F%2Fen.cppreference.com%2Fw%2Fcpp%2Flanguage%2Fadl))。  
它是这样实现的（简化）：

```cpp
template <typename T>
struct adl_serializer {
    static void to_json(json& j, const T& value) {
        // calls the "to_json" method in T's namespace
    }

    static void from_json(const json& j, T& value) {
        // same thing, but with the "from_json" method
    }
};
```

当您控制类型的命名空间时，此序列化程序可以正常工作。但是，`boost::optional`或者`std::filesystem::path(C ++ 17)`呢？盗用`boost`命名空间是非常糟糕的，并且向stl中添加模板特化以外的东西是非法......  
为了解决这个问题，您需要向命名空间`nlohmann`中添加`adl_serializer`，示例如下：

```cpp
// partial specialization (full specialization works too)
namespace nlohmann {
   template <typename T>
   struct adl_serializer<boost::optional<T>> {
       static void to_json(json& j, const boost::optional<T>& opt) {
           if (opt == boost::none) {
               j = nullptr;
           } else {
             j = *opt; // this will call adl_serializer<T>::to_json which will
                       // find the free function to_json in T's namespace!
           }
       }

       static void from_json(const json& j, boost::optional<T>& opt) {
           if (j.is_null()) {
               opt = boost::none;
           } else {
               opt = j.get<T>(); // same as above, but with
                                 // adl_serializer<T>::from_json
           }
       }
   };
}
```

#### 如何在没有默认构造函数或不可拷贝的类型使用`get()`？

有个办法，如果您的类型是[MoveConstructible](https://links.jianshu.com/go?to=https%3A%2F%2Fen.cppreference.com%2Fw%2Fcpp%2Fnamed_req%2FMoveConstructible)的。您也将需要特化`adl_serializer`，重载`from_json`：

```cpp
struct move_only_type {
    move_only_type() = delete;
    move_only_type(int ii): i(ii) {}
    move_only_type(const move_only_type&) = delete;
    move_only_type(move_only_type&&) = default;

    int i;
};

namespace nlohmann {
    template <>
    struct adl_serializer<move_only_type> {
        // note: the return type is no longer 'void', and the method only takes
        // one argument
        static move_only_type from_json(const json& j) {
            return {j.get<int>()};
        }

        // Here's the catch! You must provide a to_json method! Otherwise you
        // will not be able to convert move_only_type to json, since you fully
        // specialized adl_serializer on that type
        static void to_json(json& j, move_only_type t) {
            j = t.i;
        }
    };
}
```

#### 怎么编写自己的序列化程序？（高级用途）

您可以看下在测试套件[`unit-udt.cpp`](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnlohmann%2Fjson%2Fblob%2Fdevelop%2Ftest%2Fsrc%2Funit-udt.cpp)查看一些示例。  
如果您编写自己的序列化程序，则需要执行以下操作：

- 对`basic_json`，使用与 `nlohmann::json`中不同的别名（`basic_json`的最后一个模板参数是`JSONSerializer`）
- 在您所有的`to_json`/`from_json`函数中，使用您对`basic_json`起的别名（或模板参数）。
- 当您需要ADL时，使用`nlohmann::to_json`和`nlohmann::from_json`

下面是一个示例，只接受大小<=32的类型并且使用ADL。

```cpp
// You should use void as a second template argument
// if you don't need compile-time checks on T
template<typename T, typename SFINAE = typename std::enable_if<sizeof(T) <= 32>::type>
struct less_than_32_serializer {
    template <typename BasicJsonType>
    static void to_json(BasicJsonType& j, T value) {
        // we want to use ADL, and call the correct to_json overload
        using nlohmann::to_json; // this method is called by adl_serializer,
                                 // this is where the magic happens
        to_json(j, value);
    }

    template <typename BasicJsonType>
    static void from_json(const BasicJsonType& j, T& value) {
        // same thing here
        using nlohmann::from_json;
        from_json(j, value);
    }
};
```

重新实现序列化程序时，要**非常**小心，如果不注意，可能堆栈溢出：

```source-c
template <typename T, void>
struct bad_serializer
{
    template <typename BasicJsonType>
    static void to_json(BasicJsonType& j, const T& value) {
      // this calls BasicJsonType::json_serializer<T>::to_json(j, value);
      // if BasicJsonType::json_serializer == bad_serializer ... oops!
      j = value;
    }

    template <typename BasicJsonType>
    static void to_json(const BasicJsonType& j, T& value) {
      // this calls BasicJsonType::json_serializer<T>::from_json(j, value);
      // if BasicJsonType::json_serializer == bad_serializer ... oops!
      value = j.template get<T>(); // oops!
    }
};
```

### 二进制格式(CBOR, MessagePack, and UBJSON)

虽然JSON是一种普遍存在的数据格式，但它并不是一种非常紧凑的格式，适用于数据交换，例如通过网络。因为，该库支持[CBOR](https://links.jianshu.com/go?to=http%3A%2F%2Fcbor.io%2F) (简明二进制对象表示)，[MessagePack](https://links.jianshu.com/go?to=http%3A%2F%2Fmsgpack.org%2F)和[UBJSON](https://links.jianshu.com/go?to=http%3A%2F%2Fubjson.org%2F) (通用二进制规范)以有效地将JSON值编码为字节向量和解码此类向量。

```source-c
// create a JSON value
json j = R"({"compact": true, "schema": 0})"_json;

// serialize to CBOR
std::vector<std::uint8_t> v_cbor = json::to_cbor(j);

// 0xA2, 0x67, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0xF5, 0x66, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x00

// roundtrip
json j_from_cbor = json::from_cbor(v_cbor);

// serialize to MessagePack
std::vector<std::uint8_t> v_msgpack = json::to_msgpack(j);

// 0x82, 0xA7, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0xC3, 0xA6, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x00

// roundtrip
json j_from_msgpack = json::from_msgpack(v_msgpack);

// serialize to UBJSON
std::vector<std::uint8_t> v_ubjson = json::to_ubjson(j);

// 0x7B, 0x69, 0x07, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0x54, 0x69, 0x06, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x69, 0x00, 0x7D

// roundtrip
json j_from_ubjson = json::from_ubjson(v_ubjson);
```