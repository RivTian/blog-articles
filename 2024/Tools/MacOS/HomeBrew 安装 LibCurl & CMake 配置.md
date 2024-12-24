
LibCurl 在[官网中](https://formulae.brew.sh/formula/curl)明确指出支持 HomeBrew 进行安装。
那么在 macOS 端的安装就不会想 Win 下需要根据版本进行编译了，方便许多

```bash
brew install curl

# 安装完成后会提示 curl 在 macOS 库文件和依赖文件的安装路径
# M2 Pro

```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230729132136.png)


## CMake 配置

CMake 是一种跨平台的构建工具，可以帮助我们编译、配置和生成应用程序的 Makefile 或者 Visual Studio 项目文件。

如果要使用 libcurl 库进行编译，需要先安装 libcurl 库并配置环境变量。之后在 CMakeLists.txt 文件中添加如下内容：

```cmake
find_package(CURL REQUIRED)
include_directories(${CURL_INCLUDE_DIRS})
target_link_libraries(your_target ${CURL_LIBRARIES})
```

其中，your_target 是你的目标程序名称。

然后，使用 CMake 进行编译。在命令行中运行：

```bash
mkdir build && cd build
cmake .
make
```

这将在当前目录下的 build 生成可执行文件。

如果需要更具体的步骤，建议参考 libcurl 和 CMake 官方文档.

```cmake
# 个人 cmake 文件写法
cmake_minimum_required(VERSION 3.25)  
project(curlDemo)  
  
set(CMAKE_CXX_STANDARD 17)  
  
find_package(CURL REQUIRED)  
include_directories(${CURL_INCLUDE_DIRS})  
# HttpConntion 请参考博客中另一篇文章
# https://www.cnblogs.com/RioTian/p/17563205.html
add_executable(curlDemo main.cpp HttpConnetion.h)  
target_link_libraries(${PROJECT_NAME} ${CURL_LIBRARIES})  
# HomeBrew 在本机安装的位置
#target_link_libraries(/usr/local/opt/curl/lib)

```

## 参考

* https://cloud.tencent.com/developer/article/1979251