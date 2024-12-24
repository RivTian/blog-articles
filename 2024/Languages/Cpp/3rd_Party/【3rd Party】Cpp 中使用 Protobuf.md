#3rd_Party 

> 前置条件：
>
> * [【Protoc】VS2019 (VS平台) 使用 CMake 编译安装、使用 Protobuf 库 ](https://www.cnblogs.com/RioTian/p/17651778.html)
> * [【ToolChains】CLion（VS2019） + CMake + Vcpkg 的使用 ](https://www.cnblogs.com/RioTian/p/17679342.html)
>
> 参考博客：
>
> * [Protocol Buffers C++ 入门教程](https://blog.csdn.net/K346K346/article/details/51754431)
> * [高效的数据压缩编码方式 Protobuf](https://halfrost.com/protobuf_encode/#toc-0)

## ProtoBuf 的定义和描述

Protocol Buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。

> 怎么感觉到处都在对比 XML 的速度慢啊...

你可以定义数据的结构，然后使用特殊生成的源代码轻松的在各种数据流中使用各种语言进行编写和读取结构数据。你甚至可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。

## **使用 ProtoBuf教程：**

对 ProtoBuf 的基本概念有了一定了解之后，我们来看看具体该如何使用 ProtoBuf。

**第一步，创建 .proto 文件**，定义数据结构，如下所示：

```protobuf
//person.proto
package test;

message Person {
	required string name = 1;
	required int32 id = 2;
	required string email=3;
}
```

ProtoBuf 数据类型

| proto文件消息类型 | C++ 类型 | 说明                                                         |
| ----------------- | -------- | ------------------------------------------------------------ |
| double            | double   | 双精度浮点型                                                 |
| float             | float    | 单精度浮点型                                                 |
| int32             | int32    | 使用可变长编码方式，负数时不够高效，应该使用sint32           |
| int64             | int64    | 同上                                                         |
| uint32            | uint32   | 使用可变长编码方式                                           |
| uint64            | uint64   | 同上                                                         |
| sint32            | int32    | 使用可变长编码方式，有符号的整型值，负数编码时比通常的int32高效 |
| sint64            | sint64   | 同上                                                         |
| fixed32           | uint32   | 总是4个字节，如果数值总是比2^28大的话，这个类型会比uint32高效 |
| fixed64           | uint64   | 总是8个字节，如果数值总是比2^56大的话，这个类型会比uint64高效 |
| sfixed32          | int32    | 总是4个字节                                                  |
| sfixed64          | int64    | 总是8个字节                                                  |
| bool              | bool     |                                                              |
| string            | string   | 一个字符串必须是utf-8编码或者7-bit的ascii编码的文本          |
| bytes             | string   | 可能包含任意顺序的字节数据                                   |

我们在上例中定义了一个名为 Person的消息，语法很简单，message 关键字后跟上消息名称。之后我们在其中定义了 `message` 具有的字段，形式为：

```protobuf
message xxx {
  // 字段规则：required -> 字段只能也必须出现 1 次
  // 字段规则：optional -> 字段可出现 0 次或1次
  // 字段规则：repeated -> 字段可出现任意多次（包括 0）
  // 类型：int32、int64、sint32、sint64、string、32-bit ....
  // 字段编号：0 ~ 536870911（除去 19000 到 19999 之间的数字）
  // 请注意，范围 1 到 15 中的字段编号需要一个字节进行编码，包括字段编号和字段类型。
  // 范围 16 至 2047 中的字段编号需要两个字节。
  // 所以你应该保留数字 1 到 15 作为非常频繁出现的消息元素。
  // 请记住为将来可能添加的频繁出现的字段留出一些空间。
  字段规则 类型 名称 = 字段编号;
}
```

**第二步，protoc 编译 .proto 文件生成读写接口：**

在 `.proto` 文件中定义了数据结构，这些数据结构是面向开发者和业务程序的，并不面向存储和传输。当需要把这些数据进行存储或传输时，就需要将这些结构数据进行序列化、反序列化以及读写。**ProtoBuf** 提供相应的接口代码，可以通过 **protoc** 这个编译器来生成相应的接口代码，命令如下：

```bash
// $SRC_DIR: .proto 所在的源目录
// --cpp_out: 生成 c++ 代码
// $DST_DIR: 生成代码的目标目录
// xxx.proto: 要针对哪个 proto 文件生成接口代码
 
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/xxx.proto
 
protoc ./person.proto --cpp_out=./
```

生成的.h，.cpp文件为person.pb.h，person.pb.cpp，且.h的定义与proto文件的内容相关联：

```cpp
namespace test { // 对应 package test;
 
class Person : public ::google::protobuf::Message { //对应 message Person 且继承自::google::protobuf::Message
public:
  
  inline void set_name(const ::std::string& value); //对应message的字段内容
  inline void set_email(const ::std::string& value);
  inline void set_id(::google::protobuf::int32 value);
  ...
}
```

**第三步，编写C++业务代码：**

```cpp
// Person_Test.cpp
#include <iostream>
#include <fstream>
#include <google/protobuf/io/zero_copy_stream_impl.h>
#include <google/protobuf/text_format.h>
#include <fmt/core.h>
#include "person.pb.h"

using namespace test;

int main() {
    // 构造 Person 并填充信息
    Person p;
    p.set_name("Koshkaaa");
    p.set_id(100);
    p.set_email("kannajiahe@gmail.com");

    // 将 pb 二进制信息保存到字符串, 序列化
    std::string str;
    p.SerializeToString(&str);
    fmt::print("str: [ {} ]\n", str);

    // 将 pb 文件信息写入文件
    std::ofstream  fw;
    fw.open("../Person.txt", std::ios::out | std::ios::binary);
    auto *out = new google::protobuf::io::OstreamOutputStream(&fw);
    google::protobuf::TextFormat::Print(p, out);

    delete out;
    fw.close();

    // 将 pb 文本信息保存到字符串
    std::string str1;
    google::protobuf::TextFormat::PrintToString(p, &str1);
    fmt::print("str1: [ {} ]\n", str1);

    // 反序列化
    Person p1;
    p1.ParseFromString(str);
    fmt::print("Person1: name {0}, email {1}, id {2}", p1.name(), p1.email(), p1.id());

    return 0;
}
```

Cpp 业务代码对应的 CMakeList.txt ：

```cmake
# 使用 VcPkg
# ./vcpkg install protobuf:[special-version] fmt:[special-version]
cmake_minimum_required(VERSION 3.24)
project(protobuf_tutorial)

set(CMAKE_CXX_STANDARD 17)

find_package(protobuf CONFIG REQUIRED)
find_package(fmt CONFIG REQUIRED)

add_executable(protobuf_tutorial main.cpp
        person.pb.cc)

target_link_libraries(
        protobuf_tutorial
        PRIVATE
        protobuf::libprotoc protobuf::libprotobuf protobuf::libprotobuf-lite
        fmt::fmt
)
```

另一种可供的参考

```cmake
# 未使用 VcPkg
cmake_minimum_required(VERSION 3.24)
project(protobuf_tutorial)
set(CMAKE_CXX_FLAGS "-std=c++11  ${CMAKE_CXX_FLAGS}")

find_package(Protobuf REQUIRED)
include_directories(
  include
  ${PROTOBUF_INCLUDE_DIRS}
)
add_library(addressbook_protobuf person.pb.cc)
add_executable(test_person test_person.cpp)
target_link_libraries(
    test_person 
    addressbook_protobuf
    ${PROTOBUF_LIBRARIES}
)
```

---

**一个更复杂一些的例子**

```protobuf
package tutorial;

message Student {
	required uint64 id = 1;
	required string name =2;
	optional string email = 3;
	
	enum PhoneType {
		MOBILE 	= 0;
		HOME 	= 1;
	}
	
	message PhoneNumber { 
		required string number 	= 1;
	    optional PhoneType type = 2 [default = HOME];
	}
	repeated PhoneNumber phone = 4;
}
```

```cpp
// Student_Test.cpp
#include <iostream>
#include <fstream>
#include <google/protobuf/io/zero_copy_stream_impl.h>
#include <google/protobuf/text_format.h>
#include <fmt/core.h>
#include "student.pb.h"
// #include "person.pb.h"

using namespace tutorial;

int main() {
    // 作用 如果检测到版本不匹配时,程序将终止。
    GOOGLE_PROTOBUF_VERIFY_VERSION;
    // 给消息类 Student 对象 stu 赋值
    Student stu;
    stu.set_id(5720191825);
    *stu.mutable_name() = "Koshkaaa";
    stu.set_email("kannajiahe@gmail.com");

    // 添加一个号码对象
    Student::PhoneNumber *phoneNumber = stu.add_phone();
    phoneNumber->set_number("0564-4762652");
    phoneNumber->set_type(Student::HOME);

    // 再添加一个号码对象
    Student::PhoneNumber *phoneNumber1 = stu.add_phone();
    phoneNumber1->set_type(Student::MOBILE);
    phoneNumber1->set_number("15813354925");

    // 序列化到字符串中
    std::string serializeStr;
    stu.SerializeToString(&serializeStr);
    fmt::print("Serialization Result: {}\n", serializeStr);

    // ========== 下面开始 反序列化 ===========
    // 解析序列化的过程便称为 反序列化
    Student deSerializeStudent;
    if (!deSerializeStudent.ParseFromString(serializeStr)) {
        fmt::print("Error to parse student.\n");
        return -1;
    }

    fmt::print("deSerializeStudent debugString: {}\n", deSerializeStudent.DebugString());
    fmt::print("Student ID: {0}\nName: {1}\n", deSerializeStudent.id(),deSerializeStudent.name(), deSerializeStudent.email());
    if (deSerializeStudent.has_email())
    {
        fmt::print("EMail: {}\n", deSerializeStudent.email());
    }
    for (const auto& phone: deSerializeStudent.phone()) {
        switch ( phone.type() )
        {
            case Student_PhoneType_MOBILE:
            {
                fmt::print("Mobile Phone #{}\n", phone.number());
            }
                break;
            case Student_PhoneType_HOME:
            {
                fmt::print("Home Phone #{}\n", phone.number());
            }
                break;
        }
    }

    // 对 protobuf 全局资源的清理工作
    google::protobuf::ShutdownProtobufLibrary();
    return 0;
}
```

## 扩展 Protocol Buffers

无论或早或晚，在你发布出使用 Protocol Buffers 的代码后，你必定会想“改进“ Protocol Buffers 的定义，即我们自定义消息的 proto 文件。

如果你想让新的 proto 文件向后兼容（backward-compatible），并且旧的 proto 文件能够向前兼容（forward-compatible），你一定希望如此，那么你在新的 proto 文件中就要遵守如下规则：

1. 对已存在的任何字段，你都不能更改其标识（tag）号。

2. 你绝对不能添加或删除任何 required 的字段。

3. 你可以添加新的 optional 或 repeated 的字段，但是你必须使用新的标识（tag）号

   （例如，在这个 proto 文件中从未使用过的标识号——甚至于已经被删除过的字段使用过的标识号也不行）。

>  **（有一些例外情况，但是它们很少使用。）**

如果你遵守这些规则，老的代码将能很好地解析新的消息（message），并忽略掉任何新的字段。对老代码来说，已经被删除的 optional 字段将被赋予默认值，已被删除的 repeated 字段将是空的。新的代码也能够透明地读取旧的消息。

但是，请牢记心中：新的 optional 字段将不会出现在旧的消息中，所以你要么需要显式地检查它们是否由 has_ 前缀的函数置（set）了值，要么在你的 proto 文件中，在标识（tag）号的后面`[default = value]`提供一个合理的默认值。

如果没有为一个 optional 项指定默认值，那么就会使用与特定类型相关的默认值：对 string 来说，默认值是空字符串。对 boolean 来说，默认值是 false。对数值类型来说，默认值是0。

还要注意：如果你添加了一个新的 repeated 字段，你的新代码将无法告诉你它是否被留空了（被新代码），或者是否从未被置（set）值（被旧代码），这是因为它没有 has_ 标志。

## 优化小技巧（Optimization Tips）

Protocol Buffers 的 C++ 库已经做了极度优化。但是，正确的使用方法仍然会提高很多性能。下面是一些小技巧，用来提升 Protocol Buffers 库的最后一丝速度能力：

1. 如果有可能，重复利用消息（message）对象。即使被清除掉，消息（message）对象也会尽量保存所有被分配来重用的内存。

   这样的话，如果你正在处理很多类型相同的消息以及一系列相似的结构，有一个好办法就是重复使用同一个消息（message）对象，从而使内存分配的压力减小一些。然而，随着时间的流逝，对象占用的内存也有可能变得越来越大，尤其是当你的消息尺寸（译者注：各消息内容不同，有些消息内容多一些，有些消息内容少一些）不同的时候，或者你偶尔创建了一个比平常大很多的消息（message）的时候。

   你应该自己通过调用 SpaceUsed 函数监测消息（message）对象的大小，并在它太大的时候删除它。

2. 在多线程中分配大量小对象的内存的时候，你的操作系统的内存分配器可能优化得不够好。

   在这种情况下，你可以尝试用一下 Google’s tcmalloc。

## 高级用法（Advanced Usage）

Protocol Buffers 的作用绝不仅仅是简单的数据存取以及序列化。请阅读[C++ API reference](https://developers.google.com/protocol-buffers/docs/reference/cpp/)全文来看看你还能用它来做什么。

protocol 消息类所提供的一个关键特性就是反射。你不需要编写针对一个特殊的消息（message）类型的代码，就可以遍历一个消息的字段，并操纵它们的值，就像 XML 和 JSON 一样。“反射”的一个更高级的用法可能就是可以找出两个相同类型的消息之间的区别，或者开发某种“协议消息的正则表达式”，利用正则表达式，你可以对某种消息内容进行匹配。只要你发挥你的想像力，就有可能将 Protocol Buffers 应用到一个更广泛的、你可能一开始就期望解决的问题范围上。

> “反射”是由 Message::Reflection interface 提供的。