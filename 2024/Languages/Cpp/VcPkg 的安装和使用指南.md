> 参考博客：
>
> * https://blog.51cto.com/u_15075510/4201238
> * http://t.csdn.cn/pADDU
> * https://zhuanlan.zhihu.com/p/454233496
> * https://blog.csdn.net/weixin_43803955/article/details/123544106
> * https://blog.songjiahao.com/archives/872#toc-head-19 重要

## Vcpkg

### 概述

Vcpkg 是微软社区开发的一个跨平台的 C++ 包管理工具。它旨在解决 C++ 开发过程中依赖管理的痛点，它支持多个操作系统，包括：Windows、Linux和 macOS，使开发者能够更加便捷地安装、管理和更新 C++ 项目所需的第三方库。

### Vcpkg 优势

相对于编译开源库的传统方法，使用 Vcpkg 的优点如下：

- 跨平台支持：Vcpkg 支持 Windows、Linux 和 macOS 等多个主流操作系统，使得开发者能够在不同的环境下进行一致的依赖管理。
- 自动解决依赖：Vcpkg 能够自动处理库之间的依赖关系，简化了依赖管理的复杂性，并且能够提供一致的库版本。

- 简单易用：Vcpkg 的命令行接口使得安装、更新和卸载库都变得简单快捷，节省了开发者的时间。
- 集成 Visual Studio：不需要设置库文件、头文件的所在目录，自动集成。



### Vcpkg 下载和安装

这一步、跟随官网文档中即可，https://vcpkg.io/en/getting-started

注意，Windows 下推荐使用 PowerShell 7 或者最新版本进行终端操作。 安装包下载地址：[Downloads](https://github.com/PowerShell/PowerShell/releases/download/v7.3.6/PowerShell-7.3.6-win-x64.msi)

```bash
git clone https://github.com/Microsoft/vcpkg.git
.\vcpkg\bootstrap-vcpkg.bat
```

```bash
# macOS
brew install pkg-config # 似乎是 vcpkg 安装包的前置条件？
git clone https://github.com/Microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh

```

```bash
# Git 太慢了
# https://www.v2ex.com/t/730171
# https://www.hi-cat.cn/10111
git config --global http.proxy http://127.0.0.1:7890  
git config --global https.proxy https://127.0.0.1:7890
```

不建议修改 Git 代理，使用 Clash Pro 的强化代理
### 添加环境变量

将 vcpkg.exe 的路径添加到环境变量，即可在任意位置执行 vcpkg 命令：

```bash
# 系统环境变量中添加
VCPKG_ROOT
D:\Soft\Language\vcpkg
VCPKG_DEFAULT_TRIPLET
x64-windows
```

### 更新 Vcpkg

vcpkg 包管理器在 GitHub 上定期更新。 若要将 vcpkg 的克隆更新到最新版本，执行 git pull 命令即可。

```bash
git pull
```

![](https://img-blog.csdnimg.cn/img_convert/7d99d6e5b8d1f62e900eb3a98aadce31.png)

再次执行引导脚本：

```bash
./bootstrap-vcpkg.bat
```

![](https://img-blog.csdnimg.cn/img_convert/de094ee4d6c6cbf7a4183bfa2608df1e.png)

## Vcpkg 使用

### 安装一个开源库

这里以安装 spdlog 库为例进行演示。安装第三方开源库的命令为 `vcpkg install pkgname`。

```bash
# 默认安装
# 如果不指定安装的架构，vcpkg 默认把开源库编译成 x86 的 Windows 版本的库：
vcpkg install spdlog
```

执行结果如下：

![](https://img-blog.csdnimg.cn/img_convert/fa4a58e46e95e8c9126b71b6128ace44.png)

编译完成后可以在目录中看到库的位置：

![](https://img-blog.csdnimg.cn/img_convert/24f62a871559eda11bec6e5e9616e2ae.png)

### 指定位数安装

如果要安装编译某一个架构的开源库，我们只需要在需要安装的包后面指定相应的 triplet 即可。例如要编译 64 位 Windows 版本的 spdlog，执行如下命令：

```bash
vcpkg install spdlog:x64-windows
# 以下是 Linux 端的 Sample
vcpkg install spdlog:x64-linux

# 同样，若要编译安装静态库，只需加上 -static 即可：
vcpkg install spdlog:x64-windows-static


```

### 删除一个开源库

要删除一个已安装的开源库的命令为：vcpkg remove pkgname:

```bash
vcpkg remove spdlog:x64-windows
```

![](https://img-blog.csdnimg.cn/img_convert/7a026afe75e06ddf74d5eeb69fff6539.png)

执行库删除命令后，其源码包和解压缩的源码并没有删除，若再次安装直接进行编译步骤：

![](https://img-blog.csdnimg.cn/img_convert/95ebdbbaeb84181f36894312f252482e.png)

### 更新一个开源库

**列出需要更新的库**

可以使用 update 或者 upgrade 命令列出需要更新的库：

![](https://img-blog.csdnimg.cn/img_convert/10c52576db16f4eca8d60fdcc2559bcd.png)

```bash
vcpkg update
# 也可以使用 upgrade 命令：
vcpkg upgrade
```

### 更新过时的库

不带参数使用 upgrade 命令时将一次升级所有过时的库。但默认情况下 upgrade 命令只列出要升级的库，但不升级它们。

```bash
# 要有效地升级它们，应该使用 -no-dry-run 选项：
vcpkg upgrade --no-dry-run 

```

![](https://img-blog.csdnimg.cn/img_convert/141bc04d9a5993f7c1e29a4a4e67579c.png)

### 更新指定的库

upgrade 可以指定一个或多个库名称作为参数：

```bash
vcpkg upgrade spdlog:x64-windows --no-dry-run 

```

![](https://img-blog.csdnimg.cn/img_convert/785f58851aa4e763295e8b132efd32bd.png)

### 开源库查询

**查询一个开源库**

使用 search 命令可以查询 vcpkg 中是否包括指定的库：

```bash
vcpkg search spdlog 
```

![](https://img-blog.csdnimg.cn/img_convert/8520b3431af2524439029f689232dc92.png)

### 查询已安装的开源库

使用 list 命令可以查询当前已经安装的开源库：

```bash
vcpkg list
```

![](https://img-blog.csdnimg.cn/img_convert/787611b2afe5df87a95252b0bb8d86d1.png)

### 导出一个开源库

通常在项目中使用第三方开源库时会把其拷贝到项目文件夹中使用，使用 export 命令可以导出指定的开源库：

```bash
vcpkg export spdlog:x64-windows --zip

```

![](https://img-blog.csdnimg.cn/img_convert/3eacd982fc8fa3525a47dd1eb36f9fad.png)

导出的压缩包位于根目录中：

![](https://img-blog.csdnimg.cn/img_convert/030de31c55c003c00f507105190c84f1.png)

如下图所示，导出的压缩包中包含了引用库需要的 include、lib 等目录：

![](https://img-blog.csdnimg.cn/img_convert/f5190c08c2cbf6e804bd91057a698e81.png)

或者直接从根目录的 packages 中拷贝对应的库文件夹也可以。

## Vckpg 集成

通常情况下要使用第三方库我们需要设置 include、lib 等目录。

Vcpkg 提供了一套机制，可以全自动的适配目录，而开发者不需要关心已安装的库的目录在哪里。

### Vckpg 集成到 Visual Studio

**全局集成**

全局集成即在任意的 Visual Studio 项目中可直接使用已安装的第三方库，使用 integrate install 命令可集成到全局：

```bash
vcpkg integrate install
```

![](https://img-blog.csdnimg.cn/img_convert/112588fbc1a82957bcdf9dc1f826ea44.png)

如下示例：集成后可在任意项目中直接使用 spdlog 库：

![](https://img-blog.csdnimg.cn/img_convert/9ab1718720b8bf96b4ce2c1fdb9554f7.png)

### 移除全局集成

若要移除全局集成，执行integrate remove 命令即可：

```bash
vcpkg integrate remove
```

![](https://img-blog.csdnimg.cn/img_convert/95840739801bf227336b331a5d27f943.png)

移除后项目引用的第三库即已不可用：

![](https://img-blog.csdnimg.cn/img_convert/5bedd25686c2e6da442084ac9bb3124f.png)

### 集成到项目

Visual Studio 也支持把 Vcpkg 集成到指定的某个项目中，是利用 Visual Studio 中的 nuget 插件来实现。

1. 生成配置文件

   执行命令 integrate project 生成 nuget 配置文件：

   ```bash
   vcpkg integrate project
   # 生成下方命令
   Install-Package "vcpkg.D.Soft.Language.vcpkg" -Source "D:\Soft\Language\vcpkg"
   ```

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_230905051229_pkg%20project.png)

2. 打开程序包管理器控制台

   通过菜单 “工具-NuGet 包管理器-程序包管理器控制台” 打开控制台界面：

![](https://img-blog.csdnimg.cn/img_convert/6e2ad95b825df105660c37b86d59dd68.png)

复制在上一步界面中提示的命令，粘贴到控制台界面中，然后执行：

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_230905051410_pkg%20project2.png)

现在 demo 项目中已经可以直接使用 Vcpkg 中安装的第三方库，如下图所示：

![](https://img-blog.csdnimg.cn/img_convert/1206dcd0dfa90de91db00c23a3998e71.png)

### 集成到 CMake

在 Vcpkg 中安装每个库的时候都会有提示怎么在 CMake 中引用，以 spdlog 库安装为例：

![](https://img-blog.csdnimg.cn/img_convert/4890e4f5460464464d45c2c5b46a504b.png)

**CMakeList.text 示例**

使用 CLion 新建一个项目，在 CMakeList.text 中添加以下内容：

```cmake
cmake_minimum_required(VERSION 3.25)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_TOOLCHAIN_FILE "D:/Soft/Language/vcpkg/scripts/buildsystems/vcpkg.cmake")

if(DEFINED ENV{VCPKG_DEFAULT_TRIPLET} AND NOT DEFINED VCPKG_TARGET_TRIPLET)
    set(VCPKG_TARGET_TRIPLET "$ENV{VCPKG_DEFAULT_TRIPLET}" CACHE STRING "")
endif()

project(spdlogTest)

add_executable(spdlogTest main.cpp)

find_package(spdlog CONFIG REQUIRED)
target_link_libraries(spdlogTest PRIVATE spdlog::spdlog_header_only)
```

### 库引用示例

此时在项目中即可使用 spdlog 库了，如下图所示：

![](https://img-blog.csdnimg.cn/img_convert/0adec95cbaac1c0d5359e3fe0022f52d.png)

## 遇到的问题

### python3 build error

相关 issue：[Click Here](https://github.com/microsoft/vcpkg/issues/33591#event-10308801070)

发生原因：unix 环境下，安装 vcpkg::python3 依赖于 `automake` -> `autoconf` -> `autoconf-archive` and `pkgconfig` 

> 开发版本常常是通过autogen.sh使用程序源代码生成的，构建过程包括验证程序功能和生成配置脚本。autogen.sh脚本依赖于autoreconf来调用autoconf，automake，aclocal和其它相关工具。
> 
> 丢失的aclocal是automake包的一部分，故需求先安装依赖

所以在确保这些全局包（HomeBrew or Linux Package Manager）中存在即可

macOS Command

```bash
brew install automake autoconf autoconf-archive pkg-config
j vcpkg
./vcpkg install boost:[speceial-version]
```
