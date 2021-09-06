## 引言

众所周知，打算法竞赛最频繁使用的语言是 C++。然而，对于那些不卡复杂度的题目，可以考虑使用 Python 编写（因为 Python 真的好写）。

本文将简单地介绍一些 Python 使用技巧和语法糖。

## 一些技巧

## 框架

初学 Python 时，由于 Python 著名的交互性，大家可能会直接逐行编写 Python 语句。然而，这样便于代码逻辑组织和函数的编写。如同 C++ 中的 `int main() { ... }` 一样，Python 中也可以构建类似的程序框架。

```python
def foo():
    print("Hello")

def main():
    foo()

if __name__ == '__main__':
    main()
```

最后一句的意义为：

- 当执行本文件时，正常运行 `main()` 函数；
- 当从外部调用文件时，可以导入本文件中编写的函数，而不执行 `main()` 函数。

这样，保证了编写的代码既可以执行，又可以外部调用。

## 全局变量

Python 中的全局变量需要在函数中用 `global` 命令声明后才能进行修改，例子如下。

```python
x = 0

def foo():
    x += 10

def bar():
    global x
    x += 10

def main():
    foo()
    bar()
```

调用 `foo()` 会抛出 `UnboundLocalError: local variable 'x' referenced before assignment`。

> 具体原因参看 [Python中的global关键字，你了解吗](https://zhuanlan.zhihu.com/p/111284408)

## 重定向输入输出

如同 C++ 中的 `freopen()` 函数，Python 中也有将标准 I/O 重定向到文件的命令。

```python
import sys
sys.stdin = open(r'hack.txt', 'r')
sys.stdout = open(r'hack.out', 'w')
```

这样就可以把样例放到 `hack.txt` 中，输出到 `hack.out` 中了。

## \__debug__

Python 中有一个内置常量 `__debug__`，其默认值为 `True`。当编译命令中开启 `-O` 时，其值为 `False`。

我用这个技巧来替代 C++ 中的 `#if(n)def ... #else ... #endif` 语句（不同于 C/C++，Python 没有预编译过程，故也没有完全对应的命令）。

默认 OJ 是不会开启 `-O` 命令的（不过还是在提交前查看 OJ 对应的语言编译命令说明为好），所以我们可以定义一个常量

```python
LOCAL = not __debug__ # True if compile option '-O'
```

这样我们在本地运行开启 `-O` 命令之后，就可以使用

```python
if LOCAL:
    print("Hello")
```

来指定本地编写时执行但 OJ 不执行的命令。

> 例如将上一节的重定向可以加上这句，就不用担心由于忘记注释掉重定向而收获一发 RE 了

## 断言

Python 的断言关键字与 C++ 相同，都是 `assert`；在抛出 `AssertionError` 时可以指定报错信息。例子如下：

```python
a, b = 1, 2
assert a == b, f'{a} != {b}'
```

终端会抛出 `AssertionError: 1 != 2`

## 随机数生成器与对拍

### 随机数生成

例如生成 A+B problem 的随机数据。生成的数据直接输出到 `hack.txt` 中。

```python
import sys
from random import *

# seed(12345)
sys.stdout = open(r'hack.txt', 'w')

n = int(1e3)
print(n)
for i in range(n):
    a = randint(-1e9, 1e9)
    b = randint(-1e9, 1e9)
    print(a, b)
```

### 对拍

检验两个程序是否有相同输出，构造样例和 debug 时常用。

> 不一定是两个 `*.exe`，只要是有输入输出的东西都可以拍。

```python
from os import system

while True:
    system("python3 randgen.py")
    system("true.exe < ../hack.txt > true.txt")
    system("false.exe < ../hack.txt > false.txt")
    if system("fc true.txt false.txt"):
        system("pause")
```

更进阶的用法，考虑使用 [洛谷推出的对拍工具 CYaRon](https://github.com/luogu-dev/cyaron)，使用 pip 直接安装即可。

## 代码运行计时

使用标准库中的 time 模块

```python
import time

if __name__ == '__main__':
    T1 = time.time()
    main()
    T2 = time.time()
    print("Runtime: %.3f s." % (T2 - T1), file=sys.stderr)
```

`print()` 的 kwg 中指定 `file=sys.stderr` 使得运行时间在终端输出。这样当重定向输入输出时，该语句不受影响。

> 不过通常来讲，选择使用 Python 编写时，基本都是对运行时间要求不高的题目。

## 模板

贴上自用的 Python 模板

```python
#!/usr/bin/python3
import sys
import os
import time
from functools import reduce
from math import *

LOCAL = not __debug__  # True if compile option '-O'


def main():
    pass


if __name__ == "__main__":
    T1 = time.time()
    if LOCAL:
        sys.stdin = open(r"hack.txt", "r")
        sys.stdout = open(r"hack.out", "w")
    t = int(input())  # 1
    for i in range(t):
        print(f"Case #{i+1}:", end=' ')
        main()
    T2 = time.time()
    print("Runtime: %.3f s." % (T2 - T1), file=sys.stderr)
```

我的赛前模板[仓库](https://github.com/RivTian/Python-Content)

```bash
git clone git@github.com:RivTian/Python-Content.git
```

**参考**

1. Python 3 官方文档 - 中文 [https://docs.python.org/zh-cn/3/](https://docs.python.org/zh-cn/3/)
2. Python 中的 global 关键字，你了解吗？[https://zhuanlan.zhihu.com/p/111284408](https://zhuanlan.zhihu.com/p/111284408)

