---
title: "GDB调试基础使用方法"
date: 2020-11-05T12:50:32+08:00
draft: false
tags:
  - C++
  - GDB
gitinfo: true

---

> 尽管目前使用的VS code可以使用插件一键构建和运行程序，但GDB作为调试利器，还是值得花时间去学习的。
>

**概述**

 **GDB(GNU Debugger)** 是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。

 参考: [gdb调试利器](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html)

## 进入GDB环境调试

### 加载编译选项” –g”

以便在GDB调试环境中能够显示出具体的错误位置。

例：

```
g++ filename.cpp –g –o filename
```

在cmake编译中，可以通过可选项 `-DCMAKE_VERBOSE_MAKEFILE=1` ,具体显示编译过程，保证-g选项在编译过程中使用。

### 加载可执行文件到GDB中

在执行文件所在文件夹里，使用 `gdb` 命令进入到GDB调试环境中。之后使用 `file filename`
或者使用 `gdb filename` （filename可执行文件名）。

### 对执行文件进行输入参数的设置

使用命令 `set args parameter` (parameter为输入的参数)，加载运行所需的参数。通过 `show args` 查看加载参数的情况。

例：

```
set args /home/hello.png 2016 12
```

使用’show args’后:

```
/home/hello.png 2016 12
```

### GDB内使用make

外部文件有修改的情况下，直接在GDB环境中使用“make”进行编译。

### 文件执行

使用 `r` 即可对文件进行执行。

### 退出GDB调试环境

使用 `q` 退出调试环境。

## 断点设置

### 断点设置的方法

- 使用行号：`b linenumber` 例：“b 2017”
- 使用函数名 `b function` 例：“b hello”
- 使用地址 `b *address` 例：“b *0x404”

> 若在不同文件上打断点，在行号或函数名前加上文件名和冒号“filename:”。

例：

```
b hello.cpp:hello 
b hello.cpp:2017
```

### 条件断点

在1小点方法后面加入条件，断点在条件成立时起停止作用。

例:

```
b 12 if x > y  // 在x > y情况下，断点起作用。
```

### 断点条件更改

将指定断点号的条件进行修改:

```
condition breakpointnumber expression
```

例：

```
condition 12 if x = y
```

> `condition breakpointnumber` 停止使用条件，断点不受条件限制使用。

例：

```
condition 12
```

### 断点信息查看

- 查看所有的断点信息: `info b`
- 查看指定断点号断点信息: `info b number`

### 断点的使能

- 停止该号断点。但未删除: `dis breakpointnumber`
- 使能改号断点: `enable breakpointnumber`

## 调试的方式

### 打印变量 `p`

- 打印变量值：`p variable` 例: “p x”
- 打印变量地址：`p &variable` 例: “p &x”
- 打印指针内容：`p *point` 例: “p *pData”

### 单步调试 `s`

类似于 **step in**

- 单步执行，遇到函数，进入到函数内部执行。
  `s number` 进行多步执行

### 单步执行 `n`

类似 **step over** 执行，遇到函数，不进入函数内部，直接执行完函数。

 `n number` 进行多步执行

### 继续执行 `c`

在程序在执行中遇到断点后，使用 `c` 继续执行

### 执行完当前函数并打印出信息 `finish`

在函数内部使用 `finish` ，执行完当前整个函数打印返回信息

### 设置变量值调试

`set var variable = x` 将变量设置为x后进行调试

例：

```
set var tmp = 5
```

将变量 tmp 设置为5，var关键词确保不产生冲突

## 段错误的查找

### 段回溯 `bt`

- 在出现 segment fault 时，使用命令进行段错误的查看。
- 可以得到错误坐在的函数。像得到某人家庭地址。
- 也可使用 `where`

### 进入错误段

- 在查看到错误的位置之后，使用 `frame number` 进入到该段内部。
- 进入内部之后便可以进行局部变量的打印调试。

例：

```
frame 5
```

就进入到段5.

- 切换到上一层段: `up`
- 切换到下一层段: `down`

### 代码显示

- 显示当前代码: `l`
- 显示当前之前的代码: `l -`
- 显示该行周围代码: `l number`
- 显示该函数周围代码: `l function` （C++代码需要在函数名前加类名，重载函数需要在函数内加参数类型）
- 显示指定文件指定行周围代码: `l filename:linenumber`

例:

```
l hello.cpp:1   // 从hello.cpp第一行开始显示
```

- 显示指定文件指定函数周围代码: `l filename:function`

例：

```
l hello.cpp:Util::hello    // 显示hello.cpp文件下，Util类的函数hello
```

- 显示起始终止行内代码: `l fisrt,last`

例：

```
l 1,30  // 从第0行显示到第30行的内容
```

---

关于GDB调试的相关文章：https://www.cnblogs.com/acceptedzhs/p/13161213.html