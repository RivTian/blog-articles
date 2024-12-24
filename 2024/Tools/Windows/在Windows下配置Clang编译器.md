## Preferences

* [Linux & macOS 平台LLVM 相关工具链下载](https://buaa-se-compiling.github.io/miniSysY-tutorial/pre/llvm_download.html)
* [2019年，在Windows下配置Clang编译器](https://marvinsblog.net/post/2019-01-08-clang-on-windows/)
* [Visual Studio 2022 中使用 Clang](https://lvv.me/posts/2022/05/15_llvm_and_visual_studio/)
* [clion使用clang编译](https://www.cnblogs.com/INnoVationv2/p/14718371.html)
* [Clion 2020.3：如何设置Clang编译器](https://zhuanlan.zhihu.com/p/334520673)

这篇文章主要介绍如何在Windows使用Clang编译器来编译C/C++程序（在命令行下，`clang`是C编译器，编译C++需要使用`clang++`）。

## 简介

基于LLVM强大的模块性和优化能力，作为C/C++编译器的Clang后发优势惊人。

- Firefox在所有平台上都是用Clang编译了：[Firefox is now built with clang LTO on all* platforms](https://glandium.org/blog/?p=3888)
- Chrome在Windows上也使用Clang来编译：[Clang is now used to build Chrome for Windows](http://blog.llvm.org/2018/03/clang-is-now-used-to-build-chrome-for.html)

## MacOS  安装 LLVM

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/931697095765_.pic.jpg)

```bash
brew install llvm
echo 'export PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
lli --version # 查看llvm版本，23/10/12 安装的 17.0.2
```
## Windows 安装 Clang 预编译好的二进制包

在 [LLVM Download Page](https://github.com/llvm/llvm-project/releases) 页面可以下载预编译好的 Clang Windows 包（目前最新的包是17.01）。这个包里面内容还挺全的，包含[Clang Tools](https://clang.llvm.org/docs/ClangTools.html)和[Extra Clang Tools](http://clang.llvm.org/extra/)。

### 手动编译

如果你不想使用预先编译好的包，比如你想对编译选项做一些调整，可以手动编译Clang。可以参考下面文档：

- https://llvm.org/docs/CMake.html
- https://clang.llvm.org/get_started.html
- [Getting Started with the LLVM System using Microsoft Visual Studio¶](https://llvm.org/docs/GettingStartedVS.html)

有几个关键步骤在这里阐述一下：

- 到

  [LLVM Download Page](https://releases.llvm.org/download.html)

  下载源代码：

  - cfe-7.0.1.src.tar.xz
  - clang-tools-extra-7.0.1.src.tar.xz
  - llvm-7.0.1.src.tar.xz

- 把上面的源代码解压后放在正确的目录下：

  - llvm-7.0.1.src.tar.xz 解压并重命名成llvm
  - cfe-7.0.1.src.tar.xz解压llvm/tools目录下，并重命名成clang
  - clang-tools-extra-7.0.1.src.tar.xz解压到llvm/tools/clang/tools目录下，并重命名成extra

- 安装Visual Studio 2017 (Community版即可)
- 安装CMake和GnuWin32Utils
- 在llvm目录下创建一个build目录，并进入
- 打开命令行提示符（确保CMake和GnuWin32Utils都在PATH中），执行`cmake -G "Visual Studio 15 2017" -A x64 -Thost=x64 ..`
- 用Visual Studio打开LLVM.sln，设置目标项目为ALL_BUILD，配置类型为Release，然后开始构建。睡个午觉之后，差不多就编译好了，生成的文件在build/Release目录下

## C++标准库

Clang的C++标准库[libc++]（https://libcxx.llvm.org/)看起来不支持Winodws，所以用clang++编译下面这个C++程序的时候：

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello!" << endl;
    return 0;
}
```

会显示错误：

```bash
clang++.exe: warning: unable to find a Visual Studio installation; try running C
lang from a developer command prompt [-Wmsvc-not-found]
a.cpp:1:10: fatal error: 'iostream' file not found
#include <iostream>
         ^~~~~~~~~~
1 error generated.
```

解决这个问题有两个办法，一个是使用Visual Studio提供的C++库，另一个是使用MinGW提供的GCC的C++标准库(libstdc++)。

### 使用Visual Studio的C++库

这是Clang的默认选项。

执行`clang -v`可以看到：

```bash
clang version 17.0.1
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: D:\Soft\Language\LLVM\bin
```

默认的Target是x86_64-pc-windows-msvc，也就是使用isual Studio的C++标准库。

如果你安装了全套的Visual Studio (建议 VS2017之后的版本)，那么从开始菜单的Visual Studio目录下打开Visual Studio的命令行，在这个命令行里面使用clang编译C++，clang会自己找到相应的C++库。

如果你没有安装全套的Visual Studio也不要紧，其实所需要的只是**Visual Studio Build Tools**，可以在[VisualStudio Download](https://visualstudio.microsoft.com/downloads/)下面。 关于如何使用Build Tools，可以参照这篇文章 [How to compile C++ code with VS Code and Clang](https://www.40tude.fr/blog/compile-cpp-code-with-vscode-clang/) or [Windows Tools | How To Install VS Microsoft C++ Build Tools on Windows](https://www.cnblogs.com/RioTian/p/17730245.html)。

> 使用Visual Studio或者其相关的工具需要受到微软的License约束，目前Visual Studio的Community版本可以用于个人或者非商业项目。如果公司项目中使用Visual Studio Community版则有点违反授权的意味。
>
> Update:
>
> 根据https://devblogs.microsoft.com/cppblog/updates-to-visual-studio-build-tools-license-for-c-and-cpp-open-source-projects/，https://visualstudio.microsoft.com/visual-cpp-build-tools/的授权限制放宽了。

### 使用MinGW提供libstdc++

是的，Clang可以从GCC那借用C++标准库，也就是libstdc++。在Windows上[MinGW](https://mingw-w64.org/)项目提供了一个Windows版的GCC，包含libstd++可供Clang使用。

在[MinGW的下载页面](https://mingw-w64.org/doku.php/download)可以看到很多下载选项，适合Windows的有Cygwin、MingW-W64-builds和Msys2。

没什么别的需求的话，可以选用[MingW-W64-builds](https://mingw-w64.org/doku.php/download/mingw-builds)。

> Update: 
>
> 可以从下面地址获取较新的基于MinGW的GCC：
>
> - http://gnutoolchains.com/mingw64/
> - https://nuwen.net/mingw.html

> MingW-W64-builds提供一个安装器，来帮你选择合适的编译版本。最主要的是要选好i686版本，还是x86_64版本。本文以x86_64为例。

要让Clang使用MinGW，需要为clang指定命令行选项`-target x86_64-pc-windows-gnu`，但是我们执行`clang++ -target x86_64-pc-windows-gnu a.cpp`发现`a.cpp:1:10: fatal error: 'iostream' file not found`的错误依然存在。

这主要是因为MinGW默认安装在`C:\Program Files\mingw-w64`下面，Clang找不到MinGW。使用额外的`-v`选项，我们可以发现：

```bash
clang -cc1 version 7.0.1 based upon LLVM 7.0.1 default target x86_64-pc-win32
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include\c++"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include\c++\x86_64-w64-mingw32"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include\c++\backward"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include\c++\"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include\c++\\x86_64-w64-mingw32"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include\c++\\backward"
ignoring nonexistent directory "C:\Program Files\LLVM\include\c++\"
ignoring nonexistent directory "C:\Program Files\LLVM\include\c++\\x86_64-w64-mingw32"
ignoring nonexistent directory "C:\Program Files\LLVM\include\c++\\backward"
ignoring nonexistent directory "include\c++"
ignoring nonexistent directory "include\c++\x86_64-w64-mingw32"
ignoring nonexistent directory "include\c++\backward"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32/sys-root/mingw/include"
ignoring nonexistent directory "C:\Program Files\LLVM\x86_64-w64-mingw32\include
"
#include "..." search starts here:
#include <...> search starts here:
 C:\Program Files\LLVM\lib\clang\7.0.1\include
 C:\Program Files\LLVM\include
End of search list.
a.cpp:1:10: fatal error: 'iostream' file not found
#include <iostream>
         ^~~~~~~~~~
1 error generated.
```

Clang默认在自己的安装目录`C:\Program Files\LLVM`下查找MinGW。要解决这个问题，一个办法是把MinGW安装或者链接到Clang需要的目录。

此外还有一个办法，就是把MingGW的`g++`命令添加到PATH环境变量中去。以我的MinGW安装为例，在命令行中执行

```bash
set path=C:\Program Files\mingw-w64\x86_64-8.1.0-win32-seh-rt_v6-rev0\mingw64\bin;%path%
```

然后Clang就可以通过`g++`顺藤摸瓜，找到相应的头文件和库。

## 其他

- [How to build the latest clang-tidy](https://stackoverflow.com/questions/47255526/how-to-build-the-latest-clang-tidy)
- [解决Clang在Windows下无法使用的问题](https://blog.csdn.net/qq_31359295/article/details/72973666)
- [在VS Code中使用Clang作为你的C++编译器](https://www.jianshu.com/p/afe0ffa7839d)
- [Homebrew LLVM receipe](https://github.com/Homebrew/homebrew-core/blob/382d3defb5bc48ce2dccd17261be70c4ada9a124/Formula/llvm.rb#L181)
- [How to use clang with mingw-w64 headers on windows](https://stackoverflow.com/questions/39871656/how-to-use-clang-with-mingw-w64-headers-on-windows)
- [LLVM MinGW](https://github.com/mstorsjo/llvm-mingw)
- [Clang C++ Cross Compiler - Generating Windows Executable from Mac OS X](https://stackoverflow.com/questions/23248989/clang-c-cross-compiler-generating-windows-executable-from-mac-os-x)
- [Clang+llvm windows运行环境配置](http://www.voidcn.com/article/p-ayxzkflb-hc.html)
- [Installing Clang 3.5 for Windows](https://yongweiwu.wordpress.com/2014/12/24/installing-clang-3-5-for-windows/)
- [Home / Guide to predefined macros in C++ compilers (gcc, clang, msvc etc.) edit](https://blog.kowalczyk.info/article/j/guide-to-predefined-macros-in-c-compilers-gcc-clang-msvc-etc..html)
- [How to compile C++ for Windows with clang in Visual Studio 2015](https://stackoverflow.com/questions/31351372/how-to-compile-c-for-windows-with-clang-in-visual-studio-2015)
- [HackerNews: mingw-w64 llvm maintainer/developer here.](https://news.ycombinator.com/item?id=16525513)
- [Setting up Clang on Windows](https://github.com/boostorg/hana/wiki/Setting-up-Clang-on-Windows)
- [General notes on compiler support](https://github.com/boostorg/hana/wiki/General-notes-on-compiler-support)
- [How To Cross-Compile Clang/LLVM using Clang/LLVM](https://llvm.org/docs/HowToCrossCompileLLVM.html)
- [build llvm again](https://zhuanlan.zhihu.com/p/41395825)
- https://github.com/martell/mingw-w64-clang
- https://github.com/mstorsjo/llvm-mingw
- https://github.com/tpoechtrager/wclang
- http://blog.johannesmp.com/2015/09/01/installing-clang-on-windows-pt1/
- https://stackoverflow.com/questions/32239122/what-do-you-need-to-install-to-use-clang-on-windows-to-build-c14-for-64-bit
- https://stackoverflow.com/questions/14023696/how-to-set-up-clang-to-use-mingw-libstdc
- https://www.reddit.com/r/cpp/comments/3pe6j9/why_should_i_use_clang_over_g/