
最近要做 Qt 小组作业，需要建立能够多人协作的项目。

这篇文章记录一下项目和环境配置的过程。

## 目标

-   能够通过 Github 协作和共享代码。
-   支持在 Qt Creator 和 CLion 中编辑和运行。
-   尽可能最小化配置步骤，尤其是 Qt Creator 的配置步骤。

## 环境

小组成员的开发环境都是 Windows 系统。需要安装 Git for Windows。

Qt 选用从官网下载的开源版 6.2.4 LTS，并捆绑安装 Qt Creator、MinGW 11.2、Ninja、CMake。

CLion 可选，版本为最新版（2021.3.4）。

## 创建项目

项目从 Qt Creator 中创建。如果从 CLion 中创建，会导致 Qt Creator 无法读取项目配置。

创建一个 Qt Widgets Application，选择构建系统为 CMake，套件为 Qt 6.2.4 MinGW，版本控制系统为 Git。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234622.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234637.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234649.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234700.png)

## 项目配置

项目创建后，编辑 `.gitignore` 文件，向末尾写入以下内容：

```bash
# cmake generated files
CMakeLists.txt.user
cmake-build-*
```

`CMakeLists.txt.user` 为 Qt Creator 生成的机器特定配置，不需要包含。

`cmake-build-*` 是 CLion 的 CMake 默认输出目录， 同样不需要包含。

完成后，在 Qt Creator 中测试运行，之后写入并提交到 git 中。

```bash
git add .
git commit -m init
# 推送到 github
git remote add origin git@github.com:XXX/XXX.git
git push --set-upstream origin master
```

---

如果不准备使用 CLion，只用 Qt Creator，那么配置到此完成。如果需要使用 CLion，请继续下面的内容。

## 配置 CLion

### 配置工具链

在 CLion 设置 > 构建、执行、部署 > 工具链中，配置 Qt 工具链。

新建一个工具链配置，工具集选择 Qt 安装的 MinGW 11.2（`Qt目录/Tools/mingw1120_64`）。CMake 和调试器可以使用 CLion 内置的版本，无需设置。

应用设置，CMake 会检测配置合法性，成功后继续。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234733.png)

### 配置项目

在 CLion 中打开刚才创建的项目，会弹出“打开项目向导”。

推荐勾选“在编辑 CMakeLists.txt 或其他 CMake 配置文件时重新加载 CMake 项目”。

工具链选择上一节创建的 Qt 工具链。生成器和 CMake 选项保持默认。

在“环境”中，添加查找 Qt 套件所需要的环境变量。点击文本框右端的配置按钮，在弹出的窗口中增加名为 `CMAKE_PREFIX_PATH` 的环境变量，值为 Qt 套件的位置（`Qt目录/版本/mingw_64`） 。

如果在这里因为没有提前配置工具链，错过了“打开项目向导”，**之后也可以从设置 > 构建、执行、部署 > CMake 中更改这些配置。**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234757.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234819.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234837.png)

> 解释：Qt Creator 创建的 CMakeLists.txt 中用 find_package 寻找 Qt 的位置，CMake 在处理 find_package 时，会在 CMAKE_PREFIX_PATH 环境变量配置的目录中查找配置文件。  
> 需要注意的是，如果安装过 Anaconda，有可能 CMake 在查找 Qt 路径时找到的是 Anaconda 中的 Qt。遇到这种情况，需要把系统环境变量中 PATH 变量里的 Anaconda 删除，只保留 `D:\Anaconda` 和 `D:\Anaconda\condabin` （以 Anaconda 安装在 D 盘根目录为例）。

> macOS 下不用像 Win 那么复杂，Clion 启动 QT 项目选择默认的工具链即可

### 配置运行

此时如没有问题，已经可以编译 Qt Creator 创建的项目了。但在 CLion 中运行程序，会直接退出，原因是找不到 Qt 的 DLL 文件。

接下来还需要配置程序运行时的环境，让程序能找到 Qt 的 DLL 文件。

在 CLion 界面的右上角，下拉选框进入编辑配置。编辑环境变量，添加 `PATH` 环境变量为 Qt 的 DLL 文件的路径（`Qt目录/版本/mingw_64/bin`）。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234855.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234906.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234924.png)

> 解释：Windows 系统查找 DLL 时，会在包含 PATH 环境变量的一系列目录中查找。此处指定 PATH 环境变量，让 Windows 能找到 Qt 的 DLL 路径。

### 配置外部工具

为了能在 CLion 中直接打开 Qt Designer，图形化编辑界面，需要进行一些配置。

在设置 > 工具 > 外部工具中，点击加号新建配置。

程序路径填写 Qt Designer 的路径（`Qt目录/版本/mingw_64/bin/designer.exe`），实参填写 `$FileName$` or `$FilePath$`，工作目录填写 `$ProjectFileDir$`。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508235011.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/202305082350745.png)

保存配置。然后在项目文件列表中，右键点击 `.ui` 文件，就能在 `External Tools` 中找到 Qt Designer。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508235037.png)

### 使用 QDebug

Windows 系统上，Qt 程序运行时不会显示命令行窗口，即使在命令行里打开也是这样。

因此，如果直接在 CLion 里“运行” Qt 程序，是看不到 `QDebug` 的输出的。

网上有一些办法，比如说在 `CMakeLists.txt` 里把 Win32 关掉，但是 Qt Creator 生成的 `CMakeLists.txt` 里并没有这一个选项。

实际上，正确的操作藏在 Qt 文档的一句话里：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508235059.png)

所以，只要在 CLion 里选择“调试”程序，就能在下面的调试控制台里看到 `QDebug` 的输出了。

---

以上是关于开发环境的配置。下面的部分是关于引入第三方库的配置过程。

项目需要引入 Box2D，这个库在 vcpkg 上有一份托管，因此选择用 vcpkg 安装管理。

## 引入第三方库（vcpkg）

### 初次配置

首先，将 vcpkg 作为一个 git submodule 引入：

```text
git submodule add git@github.com:microsoft/vcpkg.git vcpkg
```

然后下载 vcpkg 的程序：

```text
./vcpkg/bootstrap-vcpkg
```

这一步需要从 Github 下载文件，请自行解决网络问题。对于使用 Clash for Windows 的同学，推荐打开 TUN 模式，防止命令行中某些程序不走系统代理设置。

下载完成后，配置 vcpkg 的环境变量选项，在 cmd 中：

```powershell
set VCPKG_ROOT=
set VCPKG_DEFAULT_TRIPLET=x64-mingw-dynamic
set VCPKG_DEFAULT_HOST_TRIPLET=x64-mingw-dynamic
```

或者在 Powershell 中

```powershell
del env:VCPKG_ROOT
$env:VCPKG_DEFAULT_TRIPLET="x64-mingw-dynamic"
$env:VCPKG_DEFAULT_HOST_TRIPLET="x64-mingw-dynamic"
```

第一条是删除 vcpkg 的全局目录配置，防止与全局安装的 vcpkg 冲突，保证下载的包都在当前目录下。

第二条和第三条是指定使用 MinGW 编译器，因为之前 Qt 配置的是 MinGW，而 vcpkg 默认使用的是 Visual Studio。

同时，还需要把 Qt 的 MinGW 添加到 PATH 环境变量中：

```powershell
# cmd
set PATH=%PATH%;D:\Qt\Tools\mingw1120_64\bin
# Powershell
$env:PATH+=";D:\Qt\Tools\mingw1120_64\bin"
```

如果之前 PATH 变量中有其他版本的 MinGW 的路径，建议删除以避免冲突。

配置完环境变量之后，就可以用 vcpkg 下载 box2d了。

```text
./vcpkg/vcpkg install box2d
```

这一步同样需要注意网络问题。下载完成后会自动编译，如果找不到编译器，请检查上一步中 PATH 的配置。

下载完成之后，在 `./vcpkg/packages` 里可以找到编译完成的文件。目录结构类似于这样：

```text
packages
├─box2d_x64-mingw-dynamic
│  ├─debug
│  │  └─lib
│  ├─include
│  │  └─box2d
│  ├─lib
│  └─share
│      └─box2d
├─detect_compiler_x64-mingw-dynamic
└─detect_compiler_x86-windows
```

这里的 include 目录就是包含头文件的目录。从 box2d 给出的目录结构看，头文件在名为 `box2d` 的二级目录里，因此应该通过 `#include <box2d/box2d.h>` 的方式引入。

下一步，在 CMake 中配置 vcpkg。在 `CMakeLists.txt` 的开头（具体来说是`cmake_minimum_required`和`find_package` 之间）添加这两行：

```cmake
set(CMAKE_TOOLCHAIN_FILE ./vcpkg/scripts/buildsystems/vcpkg.cmake CACHE STRING "Vcpkg toolchain file")

set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} ./vcpkg/packages)
```

然后就可以使用 `find_package` 引入 box2d，在 Qt 的 `find_package` 之后添加：

```cmake
find_package(box2d CONFIG REQUIRED)
```

然后修改 `target_link_libraries` 为：

```cmake
target_link_libraries(Test PRIVATE Qt${QT_VERSION_MAJOR}::Widgets box2d::box2d)
```

编译一下测试，配置完成，将更改提交到 git 仓库中。

### 二次配置

当项目不是初次创建，而是从远程仓库克隆下来时，也需要一定的配置，但是没有那么麻烦。

这些配置中有一些是通用的，可以把这些写成一个脚本，这样在克隆下来后，只需要执行一次即可：

```powershell
::setup_vcpkg.bat
 
@echo off
chcp 65001

git submodule update --init --recursive
if not exist .\vcpkg\vcpkg.exe (
    call .\vcpkg\bootstrap-vcpkg.bat
)

set VCPKG_ROOT=
set VCPKG_DEFAULT_TRIPLET=x64-mingw-dynamic
set VCPKG_DEFAULT_HOST_TRIPLET=x64-mingw-dynamic

.\vcpkg\vcpkg install box2d 
```

执行之前，需要把 Qt 的 MinGW 添加到 PATH 环境变量中：

```powershell
# cmd
set PATH=%PATH%;D:\Qt\Tools\mingw1120_64\bin
# Powershell
$env:PATH+=";D:\Qt\Tools\mingw1120_64\bin"
```


* 参考：https://zhuanlan.zhihu.com/p/511577111