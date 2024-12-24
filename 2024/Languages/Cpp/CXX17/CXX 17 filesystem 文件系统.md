
#CXX17

本文将针对常用的场景，对 `std::filesystem` 的使用逐一进行验证：

1. 判断文件夹是否存在
2. 创建单层目录
3. 逐级创建多层目录
4. 创建多级目录
5. 当前文件路径
6. 创建文件"`from.dat`"
7. 获取相对于base的绝对路径
8. 文件拷贝
9. 移动文件或重命名
10. 创建文件 “`example.dat`”
11. 获取文件大小
12. 获取文件最后修改时间
13. 删除文件
14. 递归删除目录下所有文件
15. 在临时文件夹下创建文件夹并删除

## 一、Cpp 17 的支持

> [cppreference - filesystem](https://en.cppreference.com/w/cpp/filesystem)

```cmake
# Sample CMakeLists.txt
cmake_minimum_required(VERSION 3.21)

# Define a CMake project

project(
        simple_executable_fileSystem
        VERSION 1.0
        DESCRIPTION "A simple C++ project to demonstrate basic CMake usage"
        LANGUAGES CXX)

# include_directories(include)

# find_package(fmt CONFIG REQUIRED)
find_package(fmt QUIET)
if (NOT fmt_FOUND)
    message(WARNING "fmt not found, skipping vcpkg example")
endif ()

add_executable(simple_executable_with_vcpkg)

target_include_directories(simple_executable_with_vcpkg
        PUBLIC include
)

target_compile_features(
        simple_executable_with_vcpkg
        PRIVATE cxx_std_17 # 设定 CXX 17 标准
)

target_sources(
        simple_executable_with_vcpkg
        PRIVATE src/main.cpp
)

target_link_libraries(
        simple_executable_with_vcpkg
        PRIVATE fmt::fmt-header-only
)
```

## 二、头文件和命名空间

```cpp
#include<filesystem>
using namespace std::filesystem;
```



## 三、常用类

 path 类：说白了该类只是对字符串（路径）进行一些处理，这也是文件系统的基石。

 directory_entry 类：功如其名，文件入口，这个类才真正接触文件。 

 directory_iterator 类：获取文件系统目录中文件的迭代器容器，其元素为 directory_entry对象（可用于遍历目录）

 file_status 类：用于获取和修改文件（或目录）的属性（需要了解C++11的强枚举类型（即枚举类））



## 四、使用用法

1. 需要有一个path对象为基础，如果需要修改路径，可以调用其成员函数进行修改（注意其实只是处理字符串）。
2. 需要获取文件信息需要通过path构造directory_entry，但需要path一定存在才能调用构造，所以需要实现调用exists(path .)函数确保目录存在才能构造directory_entry（注意文件入口中的exists无法判断）。
3. 若需遍历，则可以使用 directory_iterator，进行遍历



## Samples

```cpp
// Sample 1
#include <iostream>
#include <filesystem>

using namespace std;
using namespace std::filesystem;

int main() {
    path str("C:\\Windows");
    if (!exists(str))        //必须先检测目录是否存在才能使用文件入口.
        return 1;
    directory_entry entry(str);        //文件入口
    if (entry.status().type() == file_type::directory)    //这里用了C++11的强枚举类型
        cout << "该路径是一个目录" << endl;
    directory_iterator list(str);            //文件入口容器
    for ( auto & it : list )
        cout << it.path().filename() << endl;    //通过文件入口（it）获取path对象，再得到path对象的文件名，将之输出
    system("pause");
    return 0;
}
```

```cpp
// Sample 2
#include <fmt/core.h>
#include <filesystem>
#include <fstream>
#include <string>
#include <cassert>

namespace fs = std::filesystem;

int main() {
    // 1> 判断文件夹是否存在
    std::string dirName{ "log" };
    fs::path url(dirName);
    if (!fs::exists(url))
    {
        // fmt::print("{} is not exist\n",  std::quoted(dirName)); // CXX14 引入std::quoted用于给字符串添加双引号
        fmt::print("\"{}\" is not exist\n", dirName);
    }
    else
    {
        fmt::print("\"{}\" is exist\n", dirName);
    }

    // <2> 创建单层目录 in build directories
    bool okey = fs::create_directories(dirName);
    fmt::print("create_directories({}), result = {}\n", dirName, okey);

    // <3> 逐级创建多层目录
    std::error_code err;
    std::string subDir = dirName + "/subdir";
    okey = fs::create_directories(subDir, err);
    fmt::print("create_directories({}), result = {}\n", subDir, okey);
    fmt::print("err.value() = {}, err.message() = {}\n", err.value(), err.message());

    // <4> 创建多级目录
    dirName = "a/b//c/d/";
    okey = fs::create_directories(dirName, err);
    fmt::print("create_directories({}), result = {}\n", dirName, okey);
    fmt::print("err.value() = {}, err.message() = {}\n", err.value(), err.message());

    // <5> 当前文件路径
    fs::path currentPath = fs::current_path(); // D:\Coding\CLion\Cpp17-Complete-Guide\build\simple_executable
    fmt::print("currentPath = {}\n", currentPath.string());
    fmt::print("root_directory = {}\n", currentPath.root_directory().string());
    fmt::print("relative_path = {}\n",
               currentPath.relative_path().string()); // Coding\CLion\Cpp17-Complete-Guide\build\simple_executable
    fmt::print("root_name = {}\n", currentPath.root_name().string());
    fmt::print("root_path = {}\n", currentPath.root_path().string());

    // <6> 创建文件"from.dat"
    fs::path oldPath(fs::current_path() / "from.dat");
    std::fstream file(oldPath, std::ios::out | std::ios::trunc);
    if (!file)
    {
        fmt::print("Create file ({}) failed!!!\n", oldPath.string());
    }
    file.close();

    // <7> 获取相对于base的绝对路径
    fs::path absPath = fs::absolute(oldPath/*, fs::current_path()*/);
    fmt::print("absPath = {}\n", absPath.string());

    // <8> 文件拷贝
    fs::create_directories(fs::current_path() / "to");
    fs::path toPath(fs::current_path() / "to/from0.dat");
    fs::copy(oldPath, toPath);

    // <9> 移动文件或重命名
    fs::path newPath(fs::current_path() / "to/to.dat");
    fs::rename(oldPath, newPath);

    // <10> 创建文件 "example.dat"
    fs::path _path = fs::current_path() / "example.dat";
    fmt::print("example.data path: {}\n", _path.string());
    std::ofstream(_path).put('a'); // create file of size 1
    std::ofstream(_path).close();

    // 文件类型判定
    assert(fs::file_type::regular == fs::status(_path).type());

    // <11> 获取文件大小
    auto size = fs::file_size(_path);
    fmt::print("file_size = {}\n", size);

    // <12> 获取文件最后修改时间
    auto time = fs::last_write_time(_path);
    fmt::print("last_write_time = {}\n", time.time_since_epoch().count());

    // <13> 删除文件
    okey = fs::remove(_path);
    fmt::print("remove ({})\n", _path.string());

    // <14> 递归删除目录下所有文件,返回被成功删除的文件个数
    uintmax_t count = fs::remove_all(dirName);//dirName="a/b//c/d/",会把d目录也删掉
    fmt::print("remove_all ({}), {}\n", dirName, count);

    // <15> 在临时文件夹下创建文件夹并删除
    fs::path tmp = fs::temp_directory_path();//"C:\Users\Kandy\AppData\Local\Temp\"
    fmt::print("temp_directory_path = {}\n", tmp.string());
    fs::create_directories(tmp / "_abcdef/example");
    std::uintmax_t n = fs::remove_all(tmp / "_abcdef");
    fmt::print("Deleted {} files or directories\n", n);

    return 0;
}
```



## 五、常用库函数

<font color="#165612"><b>void</b></font> copy(<font color="#00BFFF">const path& from, const path& to</font>) ：<font color="#ff0000"><b>目录复制</b></font>

<font color="#165612"><b>path</b></font> absolute(<font color="#00BFFF">const path& pval, const path& base = current_path()</font>) ：<font color="#ff0000"><b>获取相对于base的绝对路径</b></font>

<font color="#165612"><b>bool</b></font> create_directory(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>当目录不存在时创建目录</b></font>

<font color="#165612"><b>bool</b></font> create_directories(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>形如/a/b/c这样的，如果都不存在，创建目录结构</b></font>

<font color="#165612"><b>bool</b></font> exists(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>用于判断path是否存在</b></font>

<font color="#165612"><b>uintmax_t</b></font> file_size(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>返回目录的大小</b></font>

<font color="#165612"><b>file_time_type</b></font> last_write_time(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>返回目录最后修改日期的file_time_type对象</b></font>

<font color="#165612"><b>bool</b></font> remove(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>删除目录</b></font>

<font color="#165612"><b>uintmax_t</b></font> remove_all(<font color="#00BFFF">const path& pval</font>) ：<font color="#ff0000"><b>递归删除目录下所有文件，返回被成功删除的文件个数</b></font>

<font color="#165612"><b>void</b></font> rename(<font color="#00BFFF">const path& from, const path& to</font>) ：<font color="#ff0000"><b>移动文件或者重命名</b></font>