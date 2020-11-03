---
title: "Windows vs Code 配置 C++ 开发环境"
date: 2020-10-14T16:02:55+08:00
draft: false
tags:
  - VScode
topics:
  - 
---

### 准备

- Windows 【这个相信大家都有 笑: )】
- [VS Code](https://code.visualstudio.com/)
- [MinGW-w64](http://mingw-w64.org/)
- [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

### 安装 MinGw-w64

具体说明细节和安装体验可以在[《⑨也懂系列：MinGW-w64安装教程》](http://rsreland.net/archives/1760)这里查看。这里只记录一下安装设置和变量设置。

如果刚 Next 就出现 `Cannot download reposytory.txt[0]` 错误，可以尝试关闭杀毒软件（包括 Windows Defender），并以管理员方式运行。

#### 安装设置说明

- **Version**：GCC 版本，若无特殊需求优选最高
- **Architecture**：系统架构，64 位系统选 `x86_64`，32 位选 `i686`
- **Threads**：操作系统 API，开发 Windows 程序选 `win32`，开发 Linux、Unix、Mac OS 等其他 posix 协议的操作系统程序选 `posix`
- **Exception**：异常处理模型
  - **64 位系统**
  - `seh`：性能优化好，优选
  - `sjlj`：稳定性好
  - **32 位系统**
  - `dwarf`：性能优化好
  - `sjlj`：稳定性好
- **Build revision**：修订版本，保持默认即可

#### 环境变量添加

打开 `此电脑-属性-高级系统设置-高级-环境变量`，在 `系统变量-Path` 编辑并新建添加你 MinGw-w64 安装目录下的 bin 文件夹路径，例如 `C:\Program Files\mingw-w64\x86_64-8.1.0-win32-seh-rt_v6-rev0\mingw64\bin`。保存即可。

![环境变量添加](https://gitee.com//riotian/blogimage/raw/master/img/20201013230211.png)

可以在 `cmd` 或 `powershell` 中运行 `gcc --version` 判断是否安装正确。（安装正确会输出版本号）

**重启 VS Code**。

### VS Code

这里需要先注意的是，这里的所有配置都是基于一个工作区（文件夹）的（因为编辑器编译调试需要配置文件），当然你也可以直接打开文件夹。但若要直接编译 `.c&.cpp` 文件，可以直接执行 `~gcc {.c} `, `~g++ {.cpp}` 进行编译。

先新建并打开你的工作（代码运行）区。

这里还需要注意的是，添加好环境变量后一定要**重启 VS Code**。必须退出后重新启动，否则编辑器无法识别编译指令。

#### C/C++ 拓展

![C/C++拓展](https://gitee.com//riotian/blogimage/raw/master/img/20201013230223.png)

C/C++ 拓展供了针对 C/C++ 的 IntelliSense 与调试等功能，搜索「C/C++」即可直接安装。

`Ctrl+Shift+P` 打开命令面板，运行 `C/Cpp: Edit configurations`，插件会在当前目录下创建 `.vscode/c_cpp_properties.json` 配置文件，只针对本次的项目。

若你系统已安装 Visual Studio，它会自动将编译器等设置链接过去，但我们这不需要它。可以参考以下配置：

```json
{
    "configurations": [
        {
            "name": "MinGW",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "compilerPath": "{your_mingw-w64_bin_gcc.exe_path}",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "gcc-x64"
        }
    ],
    "version": 4
}
```

上面的 `{your_mingw-w64_bin_gcc.exe_path}` 替换成你需要的编译器路径。例如：

- gcc：`C:/Program Files/mingw-w64/x86_64-8.1.0-win32-seh-rt_v6-rev0/mingw64/bin/gcc.exe`
- g++：`C:/Program Files/mingw-w64/x86_64-8.1.0-win32-seh-rt_v6-rev0/mingw64/bin/g++.exe`
- （`/`, `\\` 都是可行的）

关于库文件设置，C/C++ 扩展已经可以通过 `compilerPath` 自动推断头文件以及库文件所在的目录，所以无需手动配置。

#### 编译与调试

配置好上述文件后，可以先写一段 C/C++ 代码，测试编译并调试一下。（**以下所有展示的配置只适用于单文件的编译并调试**）

```c
#include <stdio.h>
int main() {
    printf("hello world");
    return 0;
}
```

##### 编译设置：tasks.json

编译之前，需要先创建 `.vscode/tasks.json`（无需手动创建）。这个配置文件提供命令行构建配置，让编译器调用然后终端执行。

`Ctrl+Shift+P` 打开命令面板，运行 `Tasks: Configure Task`，选择 `使用模板创建 tasks.json 文件`，选择 `Others 运行任意外部命令的示例`。至此，编辑器会生成 `.vscode/tasks.json` 文件。编辑文件，例如：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            // 任务标签名
            "label": "compile",
            // 任务类型
            "type": "shell",
            // 编译器选择
            "command": "gcc",
            // 编译预执行的命令集
            "args": [
                "-g",
                "\"${file}\"",
                "-o",
                "\"${fileDirname}\\${fileBasenameNoExtension}\""
            ],
            // 任务输出配置
            "presentation": {
                "reveal": "always",
                "panel": "shared",
                "focus": false,
                "echo": true
            },
            // 任务所属组名
            "group": {
                "kind": "build",
                "isDefault": true
            },
            // 编译问题输出匹配配置
            "problemMatcher": {
                "owner": "cpp",
                "fileLocation": "absolute",
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        }
    ]
}
```

简单地注释了一下每一个配置的目的，具体可以在 [Tasks in Visual Studio Code](https://code.visualstudio.com/docs/editor/tasks#vscode) 查看。

以上是 gcc 编译器配置，若要编译 C++ ，将 `command` 改为 `g++` 即可。

至此，编辑器的编译设置已经完成了。可以在 `Ctrl+Shift+P` 中输入 `Tasks: Run Build Task` 进行编译，但只会生成 exe 文件，并不会启动调试。

##### 调试设置：launch.json

光编译在有些时候肯定是不够的，那就需要调试功能了。就是所谓的 Debug 。

`Ctrl+Shift+P` 打开命令面板，输入 `Debug: Open launch.json`，选择 `C++ (GDB/LLDB)`，编辑器会创建调试配置文件 `.vscode/launch.json`。但需要稍加修改，例如（基于 MinGW-w64 的 GDB）：

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            // 调试名
            "name": "debug",
            // 调试器类型
            "type": "cppdbg",
            // 请求类型
            "request": "launch",
            // 调试的可执行文件（tasks.json 中配置的编译输出的文件）
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe",
            // 调试参数
            "args": [],
            "stopAtEntry": false,
            // 当前工作区
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "{your_mingw-w64_bin_gdb.exe_path}",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            // 调试前启动的任务
            // 要与 tasks.json 中配置的 label 一致
            // 总是要先编译再调试的嘛
            "preLaunchTask": "compile",
        }
    ]
}
```

上面的 `{your_mingw-w64_bin_gdb.exe_path}` 替换成你 MinGW-w64-GDB 路径。例如：

- `C:/Program Files/mingw-w64/x86_64-8.1.0-win32-seh-rt_v6-rev0/mingw64/bin/gdb.exe`

简单注释了一下，更多可以在 [Debugging in Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging#_launch-configurations) 查看。

##### 测试代码

至此，基本配置已全部完成。可以按 `F5` 开始调试代码。或者也可以在 VS Code 的调试界面点击左上角按钮进行调试。

![调试代码](https://gitee.com//riotian/blogimage/raw/master/img/20201013230253.png)

#### 另外

##### Code Runner 拓展

![Code Runner简介](https://gitee.com//riotian/blogimage/raw/master/img/20201013230256.png)

另外，毕竟都用 VS Code 这类轻量编辑器写代码了嘛，那就推荐一款超好用的代码调试工具：[Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)。

它可以让我们直接在 VS Code 运行各种语言代码，缺点就是，它很轻量，不支持复杂工程编译。 搜索「Code Runner」即可直接安装。接下来，我们需要设置一下 Code Runner 。

进入 VS Code 设置，右上角坐起第二个按钮（打开设置），在右栏中输入相关设置的 json 即可直接进行设置替换。

```json
// 在终端中运行编译命令，否则我们无法与程序通过标准输入交互
"code-runner.runInTerminal": true,
// 如果你全局设置中的默认终端是 WSL 之类的，那么可以在工作区设置中改回 PowerShell
"terminal.integrated.shell.windows": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
// 运行代码之前清除之前的输出
"code-runner.clearPreviousOutput": true,
// 开启这个后在运行编译命令之前会自动 cd 至文件所在目录
"code-runner.fileDirectoryAsCwd": true,
// 因为上面那个选项会自动 cd，所以我删除了默认编译命令中的 cd 语句
// 这里只保留了 C 和 C++ 的编译命令，有需要其他语言的请自行添加
"code-runner.executorMap": {
    "c": "gcc $fileName -o $fileNameWithoutExt && .\\$fileNameWithoutExt",
    "cpp": "g++ $fileName -o $fileNameWithoutExt && .\\$fileNameWithoutExt",
},
// 运行代码后切换焦点至终端，方便直接输入测试数据
"code-runner.preserveFocus": false,
// 在运行代码之前保存文件
"code-runner.saveFileBeforeRun": true,
```

如果没有特殊要求，编辑保存即可。

然后我们就可以直接在代码界面按下 `Ctrl+Alt+N` 即可进行编译调试输出，或者也可以直接 右键-Run Code。

![运行调试代码](https://gitee.com//riotian/blogimage/raw/master/img/20201013231757.png)

### 快速编辑

- 在`文件`->`首选项`->`用户代码片段`->`cpp.json`中添加

```json
{
	"codeforce": {
		"prefix": "cf",
		"body": [
			"// Author : RioTian",
			"// Time : ${CURRENT_YEAR_SHORT}/${CURRENT_MONTH}/${CURRENT_DATE}",
			"#include <bits/stdc++.h>",
			"using namespace std;",
			"typedef long long ll;",
			"int main() {",
			"    // freopen(\"in.txt\",\"r\",stdin);",
			"    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);",
			"    $0",
			"}"
		],
		"description": "cf"
	},
	"filein": {
		"prefix": "in",
		"body": [
			"freopen(\"in.txt\", \"r\", stdin);"
		],
		"description": "in"
	},
}
```

> VS Code 代码格式化工具clang-format：[click here](./VS Code 代码格式化工具clang-format.md)

### 后日谈

为什么要配置 Windows VS Code 的 C/C++ 开发环境呢，总的来说还是想要轻量。目前为止的 C 程序完全用不着 VS 这么庞大的 IDE 写（只有在认知实习任务中被要求写地铁项目），尽管刷刷 ACM 题VS很爽。但其实只需要轻量的 VS Code （颜值高😂） 就够了（而且比赛中是不被允许使用VS的）。

### 参考

配置文件代码来自 https://blessing.studio/。

- [使用 VS Code 搭建适用于 ACM 练习的 C/C++ 开发环境](https://blessing.studio/vscode-c-cpp-configuration-for-acm-oj/)
- [使用 Visual Studio Code 搭建 C/C++ 开发和调试环境](https://segmentfault.com/a/1190000014800106#articleHeader0)
- [《⑨也懂系列：MinGW-w64安装教程》](http://rsreland.net/archives/1760)
- [#410 Installer cannot download repository.txt](https://sourceforge.net/p/mingw-w64/bugs/410/)
- [#558 the file has been downloaded incorrectly!](https://sourceforge.net/p/mingw-w64/bugs/558/)
- [Tasks in Visual Studio Code](https://code.visualstudio.com/docs/editor/tasks#vscode)
- [Debugging in Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging#_launch-configurations)
- [ ACM VSCode 配置(备份)](https://www.wqrlink.tech/2019/09/11/acm-vscode%E9%85%8D%E7%BD%AE/)
- [使用VScode打Codeforces的提高效率小技巧](https://zhuanlan.zhihu.com/p/37269217)