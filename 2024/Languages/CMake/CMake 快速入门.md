
https://www.coonote.com/linux-note/cmake-quick-start.html

https://www.hahack.com/codes/cmake/ **推荐**

# CMake 简介

## 什么是 CMake？

CMake 是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装（编译过程）。他能够输出各种各样的makefile 或者 project 文件，CMake 的配置文件取名为 CMakeLists.txt。也就是在 CMakeLists.txt 这个文件中写 cmake 代码。 一句话：cmake 就是将多个 cpp、hpp 文件组合构建为一个大工程的语言。

![CMake](https://static.coonote.com/2022/08/4374003620986273752.png)

## 优缺点

-   **优点：**
    -   开源，使用类BSD许可发布。
    -   跨平台，并可以生成 native 编译配置文件，在 Linux/Unix 平台，生成 makefile；在苹果平台可以生成 Xcode；在Windows 平台，可以生成 MSVC 的工程文件。
    -   能够管理大型项目。
    -   简化编译构建过程和编译过程。cmake 的工具链非常简单：cmake + make。
    -   高效率，因为 cmake 在工具链中没有 libtool。
    -   可扩展，可以为 cmake 编写特定功能的模块，扩展 cmake 功能。
-   **缺点：**
    -   cmake 只是看起来比较简单，而使用并不简单。
    -   cmake 编写的过程实际上是编程的过程，每个项目使用一个 CMakeLists.txt（每个目录一个），使用的是 cmake 语法。
    -   cmake 跟已有体系配合不是特别的理想，比如 pkgconfig。

## 编译流程

在 linux 下使用 CMake 生成 Makefile 并编译的流程如下：

1.  编写 CMake 配置文件 CMakeLists.txt 。
2.  在 CMakeLists.txt 文件所在目录创建一个 build 文件夹，然后进入目录。（这一步可以省略，但是生成的中间文件不易清理）
3.  执行命令 `cmake PATH` 或者 `ccmake PATH` 生成 Makefile（`ccmake` 和 `cmake` 的区别在于前者提供了一个交互式的界面）。其中， `PATH` 是 CMakeLists.txt 所在的目录。
4.  使用 `make` 命令进行编译，使用 `make install` 进行安装。

# CMake 实战

## 环境搭建

### 安装 CMake

这里以 macOS 为例：

```bash
brew install cmake gcc
```

### 安装示例项目

本文用到的所有示例都来自于 GitHub 上的 cmake-examples 和 cmake-demo，下面将结合实例来进行讲解。

```bash
git clone https://github.com/ttroy50/cmake-examples.git 
git clone https://github.com/wzpan/cmake-demo.git
```

## 编译单个源文件

> 本节对应的源代码路径如下：/cmake-examples/01-basic/A-hello-cmake/

首先查看一下本示例的目录结构：

```
.
├── CMakeLists.txt
├── main.cpp
└── README.adoc
```

在本目录下有三个文件，分别是源文件 main.cpp，cmake 构建规则 CMakeLists.txt 以及说明文件 README.adoc，下面来看看他们的具体内容。

```
#include <iostream>

int main(int argc, char *argv[])
{
  
   std::cout << "Hello CMake!" << std::endl;
   return 0;
}
```

源文件是一个简单的 Hello World。

```
cmake_minimum_required(VERSION 3.5)		# 设CMake最小版本号

project(hello_cmake)					# 设置工程名

add_executable(hello_cmake main.cpp)	# 生成可执行文件
```

CMakeLists 中主要包含了三个命令：

-   `cmake_minimum_required(VERSION 3.5)`：指定运行此配置文件所需的 CMake 的最低版本。
-   `project (hello_cmake)`：设置项目的名称，同时会自动生成 PROJECT_NAME 变量，使用 `${PROJECT_NAME}` 即可访问到 hello_cmake。
-   `add_executable(hello_cmake main.cpp)`：第一个参数是可执行文件名，第二个参数是要编译的源文件列表。z这里将名为 main.cpp 的源文件编译成一个名称为 hello_cmake 的可执行文件。

接着我们可以开始构建项目，构建的方法有以下两种：

-   **内部构建**：直接在源文件目录构建项目，会导致临时文件和源代码放在一起，不好清理。
-   **外部构建**：创建一个可以位于文件系统上任何位置的构建文件夹。 所有临时构建和目标文件都位于此目录中，以保持源代码树的整洁。

这里以外部构建为例，此时我们需要新建一个构建文件夹 build，并在该目录下运行 cmake 命令进行构建：

```
#创建并进入build目录
mkdir build && cd build

#构建当前目录
cmake ..

#使用cmake生成的makefile编译得到可执行文件
make
```

此时在当前目录下，就会生成可执行文件 hello_cmake。将其运行查看是否成功编译：

```
./hello_cmake
Hello CMake!
```

## 编译多个源文件

### 单个目录下的多个源文件

> 本节对应的源代码路径如下：/cmake-demo/Demo2

首先查看一下本示例的目录结构：

```
.
├── CMakeLists.txt
├── main.cc
├── MathFunctions.cc
└── MathFunctions.h
```

与上个示例不同，本示例在单个目录下有着多个源文件，此时 CMakeLists 如下：

```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo2)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(${
  PROJECT_NAME} ${
  DIR_SRCS})
```

在本示例中，为了避免一个个将所有源文件输入，使用了 `aux_source_directory` 命令。

-   `aux_source_directory` ：第一个参数是目录的路径，第二个参数是变量名。当我们使用这个命令时，就会将指定目录下的所有源文件保存到指定的变量名中。

如果不想使用这种方法，而是向一条条枚举每个变量，可以使用 `set` 来手动将源文件保存到变量名中：

```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo2)

set(DIR_SRCS
    MathFunctions.cc
    main.cc
)

# 指定生成目标
add_executable(${
  PROJECT_NAME} ${
  DIR_SRCS})
```

### 多个目录下的多个源文件

> 本节对应的源代码路径如下：/cmake-demo/Demo3

首先查看一下本示例的目录结构：

```
.
├── CMakeLists.txt
├── main.cc
└── math
    ├── CMakeLists.txt
    ├── MathFunctions.cc
    └── MathFunctions.h
```

与上个示例不同，本示例在多个目录下有着多个源文件。在这种情况下，我们需要在每个目录中都编写一个 CMakeLists.txt。这里为了方便，我们可以将 math 里的文件编译为一个静态库再有 main 函数调用。

首先看看 math 目录下的 CMakeLists.txt，这里主要做的事是将当前目录下的文件编译为一个静态库：

```
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 指定生成 MathFunctions 链接库
add_library (MathFunctions ${
  DIR_LIB_SRCS})
```

-   `add_library`：用于从某些源文件创建一个库，默认生成在构建文件夹。第一个参数为库名（不需要 lib 前缀，会自动添加），第二个参数用于指定 SHARED（动态库），STATIC（静态库）（如果不写，则通过全局的`BUILD_SHARED_LIBS` 的 `FALSE` 或 `TRUE` 来指定）。第三个参数即为源文件列表。

接着看看[根目录](https://www.coonote.com/linux-note/linux-root-directory.html "Linux根目录")的 CMakeLists.txt：

```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo3)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 添加 math 子目录
add_subdirectory(math)

# 指定生成目标
add_executable(Demo ${
  DIR_SRCS})

# 添加链接库
target_link_libraries(Demo MathFunctions)
```

-   `add_subdirectory`：用于表示该项目包含一个子目录，此时会去处理子目录下的 CMakeLists.txt 与源文件。
-   `target_link_libraries`：该命令用于指明可执行文件 Demo 需要链接 MathFunctions 库。第一个参数为可执行文件名，第二个参数为访问权限（PUBLIC、PRIVATE、INTERFACE，默认为 PUBLIC），第三个参数为库名（这两个参数可以为多个）。

## 导入外部库

### 本地导入（find_package）

> 本节对应的源代码路径如下：/cmake-examples/01-basic/H-third-party-library

首先查看一下本示例的目录结构：

```
.
├── CMakeLists.txt
├── main.cpp
└── README.adoc
```

这里主要演示如何导入一个本地的第三方库（这里以 boost 为例），接着看看 MakeLists.txt：

```
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (third_party_include)
# find a boost install with the libraries filesystem and system
#使用库文件系统和系统查找boost install
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
#这是第三方库，而不是自己生成的静态动态库
# check if boost was found
if(Boost_FOUND)
    message ("boost found")
else()
    message (FATAL_ERROR "Cannot find Boost")
endif()

# Add an executable
add_executable(third_party_include main.cpp)

# link against the boost libraries
target_link_libraries( third_party_include
    PRIVATE
        Boost::filesystem
)
```

这里使用 `find_package` 命令来在本地搜索对应的第三方库，Boost 代表需要查询的库名称；1.46.1 代表需要库的最低版本；REQUIRED 表示该库是必须的，如果找不到会报错；COMPONENTS 用于检测该库的对应组件是否存在，如果不存在则认为找到的库不满足条件。

### 外部导入（FetchContent）

FetchContent 是 3.11.0 版本开始提供的功能，只需要一个 URL 或者 Git 仓库即可引入一个库，这里以 GoogleTest 库为例：

```
cmake_minimum_required(VERSION 3.14)
project(my_project)

# GoogleTest requires at least C++11
set(CMAKE_CXX_STANDARD 11)

include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/609281088cfefc76f9d0ce82e1ff6c30cc3591e5.zip
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)
```

使用方法：

1.  `include(FetchContent)` ：表示引入 FetchContent。
2.  `FetchContent_Declare(第三方库)` ：获取第三方库，可以是一个 URL 或者一个 Git 仓库。
3.  `FetchContent_MakeAvailable(第三方库)` ：将这个第三方库引入项目。
4.  `target_link_libraries(主项目 PRIVATE 子模块::子模块)` ：链接这个第三方库。

## 测试与安装

CMake 也可以指定安装规则，以及添加测试。这两个功能分别可以通过在产生 Makefile 后使用 `make install` 和 `make test` 来执行。

> 本节对应的源代码路径如下：/cmake-demo/Demo8

首先查看一下本示例的目录结构：

```
.
├── CMakeLists.txt
├── config.h.in
├── License.txt
├── main.cc
└── math
    ├── CMakeLists.txt
    ├── MathFunctions.cc
    └── MathFunctions.h
```

### 自定义安装规则

首先查看 math 目录下的 CMakeLists.txt：

```
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 指定生成 MathFunctions 链接库
add_library (MathFunctions ${
  DIR_LIB_SRCS})

# 指定 MathFunctions 库的安装路径
install (TARGETS MathFunctions DESTINATION lib)
install (FILES MathFunctions.h DESTINATION include)
```

这里使用 `install` 命令表明了将静态库 MathFunctions 安装到 /usr/local/lib 目录下，将头文件 MathFunctions.h 安装到 /usr/local/include 目录下。

接着查看根目录的 `install` 内容：

```
# 指定安装路径
install (TARGETS Demo DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h"
         DESTINATION include)
```

这里将可执行程序 Demo 安装到了 /usr/local/lib 目录下，再将 config.h 安装到 /usr/local/lib 目录下。

> /usr/local/ 是默认安装的根目录，可以通过修改 `CMAKE_INSTALL_PREFIX` 变量的值来指定这些文件应该拷贝到哪个根目录

### 测试

CMake 提供了一个称为 CTest 的测试工具。我们要做的只是在项目根目录的 CMakeLists 文件中调用一系列的 `add_test` 命令。

```
# 启用测试
enable_testing()

# 测试程序是否成功运行
add_test (test_run Demo 5 2)

# 测试帮助信息是否可以正常提示
add_test (test_usage Demo)
set_tests_properties (test_usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")

# 测试 5 的平方
# add_test (test_5_2 Demo 5 2)

# set_tests_properties (test_5_2
#  PROPERTIES PASS_REGULAR_EXPRESSION "is 25")

# 测试 10 的 5 次方
# add_test (test_10_5 Demo 10 5)

# set_tests_properties (test_10_5
#  PROPERTIES PASS_REGULAR_EXPRESSION "is 100000")

# 测试 2 的 10 次方
# add_test (test_2_10 Demo 2 10)

# set_tests_properties (test_2_10
#  PROPERTIES PASS_REGULAR_EXPRESSION "is 1024")

# 定义一个宏，用来简化测试工作
macro (do_test arg1 arg2 result)
  add_test (test_${
  arg1}_${
  arg2} Demo ${
  arg1} ${
  arg2})
  set_tests_properties (test_${
  arg1}_${
  arg2}
    PROPERTIES PASS_REGULAR_EXPRESSION ${
  result})
endmacro (do_test)

# 利用 do_test 宏，测试一系列数据
do_test (5 2 "is 25")
do_test (10 5 "is 100000")
do_test (2 10 "is 1024")
```

-   `enable_testing`：用于启动测试。
-   `add_test`：用于添加测试，第一个参数为测试名，第二个参数为可执行程序，剩下的为可执行程序的参数。
-   `set_tests_properties`：测试的提示信息。
-   `macro`：宏，用于编写一个重复性操作来简化测试用例的编写。
-   `do_test`：编写的测试宏。

### 生成安装包

如果想要生成安装包，则需要使用 CPack，它是由 CMake 提供的一个工具，专门用于打包。此时需要在 CMakeLists.txt 中添加以下内容：

```
# 构建一个 CPack 安装包
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE
  "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${Demo_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Demo_VERSION_MINOR}")
include (CPack)
```

-   `include (InstallRequiredSystemLibraries)`：导入 InstallRequiredSystemLibraries 模块。
-   设置一些 CPack 相关变量。
-   `include (CPack)`：导入 CPack 模块。

接着执行 `cmake` 和 `make` 构建工程，此时再执行 `cpack` 命令即可生成安装包：

```
#生成二进制安装包
cpack -C CPackConfig.cmake

#生成源码安装包
cpack -C CPackSourceConfig.cmake
```

当命令执行成功后，就会在当前目录下生成 \*.sh、\*.tar.gz、\*.tar.Z 这三个格式的安装包。