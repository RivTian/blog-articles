> 写算法题的时候突然发现自己忘记基本的C++：cout格式化输出了，赶紧拉出以前的C++学习笔记重新看一看。
>
> 部分内容来自教程：C语言中文网（一个很棒的网站）

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183424.gif"/></div>

有时希望按照一定的格式进行输出，如按十六进制输出整数，输出浮点数时保留小数点后面两位，输出整数时按 6 个数字的宽度输出，宽度不足时左边补 0，等等。C语言中的 printf() 函数使用以`%`开头的格式控制符，例如 %X、%.2f、%6d 等；C++ 中的 cout 对象则使用流操作算子（你也可以叫做格式控制符）或者成员函数进行控制。

## 使用流操作算子

C++ 中常用的输出流操纵算子如表 1 所示，它们都是在头文件 iomanip 中定义的；要使用这些流操纵算子，必须包含该头文件。

> 注意：“流操纵算子”一栏中的星号`*`不是算子的一部分，星号表示在没有使用任何算子的情况下，就等效于使用了该算子。例如，在默认情况下，整数是用十进制形式输出的，等效于使用了 dec 算子。

| 流操纵算子          | 作  用                                                       |        |
| ------------------- | ------------------------------------------------------------ | ------ |
| *dec                | 以十进制形式输出整数                                         | 常用   |
| hex                 | 以十六进制形式输出整数                                       | \|     |
| oct                 | 以八进制形式输出整数                                         | \|     |
| fixed               | 以普通小数形式输出浮点数                                     | \|     |
| scientific          | 以科学计数法形式输出浮点数                                   | \|     |
| left                | 左对齐，即在宽度不足时将填充字符添加到右边                   | \|     |
| *right              | 右对齐，即在宽度不足时将填充字符添加到左边                   | \|     |
| setbase(b)          | 设置输出整数时的进制，b=8、10 或 16                          | \|     |
| setw(w)             | 指定输出宽度为 w 个字符，或输人字符串时读入 w 个字符         | \|     |
| setfill(c)          | 在指定输出宽度的情况下，输出的宽度不足时用字符 c 填充（默认情况是用空格填充） | \|     |
| setprecision(n)     | 设置输出浮点数的精度为 n。  在使用非 fixed 且非 scientific 方式输出的情况下，n 即为有效数字最多的位数，如果有效数字位数超过 n，则小数部分四舍五人，或自动变为科学计 数法输出并保留一共 n 位有效数字。  在使用 fixed 方式和 scientific 方式输出的情况下，n 是小数点后面应保留的位数。 | \|     |
| setiosflags(flag)   | 将某个输出格式标志置为 1                                     | \|     |
| resetiosflags(flag) | 将某个输出格式标志置为 0                                     | \|     |
| boolapha            | 把 true 和 false 输出为字符串                                | 不常用 |
| *noboolalpha        | 把 true 和 false 输出为 0、1                                 | -      |
| showbase            | 输出表示数值的进制的前缀                                     | -      |
| *noshowbase         | 不输出表示数值的进制.的前缀                                  | -      |
| showpoint           | 总是输出小数点                                               | -      |
| *noshowpoint        | 只有当小数部分存在时才显示小数点                             | -      |
| showpos             | 在非负数值中显示 +                                           | -      |
| *noshowpos          | 在非负数值中不显示 +                                         | -      |
| *skipws             | 输入时跳过空白字符                                           | -      |
| noskipws            | 输入时不跳过空白字符                                         | -      |
| uppercase           | 十六进制数中使用 A~E。若输出前缀，则前缀输出 0X，科学计数法中输出 E | -      |
| *nouppercase        | 十六进制数中使用 a~e。若输出前缀，则前缀输出 0x，科学计数法中输出 e。 | -      |
| internal            | 数值的符号（正负号）在指定宽度内左对齐，数值右对 齐，中间由填充字符填充。 | -      |

> '|'：代表常用，'-'：代表不常用

#### 流操作算子的使用方法

使用这些算子的方法是将算子用 << 和 cout 连用。例如：

```cpp
cout << hex << 12 << "," << 24;
```

这条语句的作用是指定以十六进制形式输出后面两个数，因此输出结果是：

```
c, 18
```

#### setiosflags() 算子

setiosflags() 算子实际上是一个库函数，它以一些标志作为参数，这些标志可以是在 iostream 头文件中定义的以下几种取值，它们的含义和同名算子一样。

| 标 志           | 作 用                                                        |
| --------------- | ------------------------------------------------------------ |
| ios::left       | 输出数据在本域宽范围内向左对齐                               |
| ios::right      | 输出数据在本域宽范围内向右对齐                               |
| ios::internal   | 数值的符号位在域宽内左对齐，数值右对齐，中间由填充字符填充   |
| ios::dec        | 设置整数的基数为 10                                          |
| ios::oct        | 设置整数的基数为 8                                           |
| ios::hex        | 设置整数的基数为 16                                          |
| ios::showbase   | 强制输出整数的基数（八进制数以 0 开头，十六进制数以 0x 打头） |
| ios::showpoint  | 强制输出浮点数的小点和尾数 0                                 |
| ios::uppercase  | 在以科学记数法格式 E 和以十六进制输出字母时以大写表示        |
| ios::showpos    | 对正数显示“+”号                                              |
| ios::scientific | 浮点数以科学记数法格式输出                                   |
| ios::fixed      | 浮点数以定点格式（小数形式）输出                             |
| ios::unitbuf    | 每次输出之后刷新所有的流                                     |
| ios::stdio      | 每次输出之后清除 stdout, stderr                              |


这些标志实际上都是仅有某比特位为 1，而其他比特位都为 0 的整数。

多个标志可以用`|`运算符连接，表示同时设置。例如：

```cpp
cout << setiosflags(ios::scientific|ios::showpos) << 12.34;
```

输出结果是：

```
+1.234000e+001
```



如果两个相互矛盾的标志同时被设置，如先设置 `setiosflags(ios::fixed)`，然后又设置 `setiosflags(ios::scientific)`，那么结果可能就是两个标志都不起作用。因此，在设置了某标志，又要设置其他与之矛盾的标志时，就应该用 `resetiosflags` 清除原先的标志。例如下面三条语句：

```cpp
cout << setiosflags(ios::fixed) << 12.34 << endl;
cout << resetiosflags(ios::fixed) << setiosflags(ios::scientific | ios::showpos) << 12.34 << endl;
cout << resetiosflags(ios::showpos) << 12.34 << endl;  //清除要输出正号的标志
```

输出结果是：
12.340000
+1.234000e+001
1.234000e+001

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183415.png"/></div>

#### 综合示例

关于流操纵算子的使用，来看下面的程序。

```cpp
#include <iomanip>
#include <iostream>
using namespace std;
int main() {
    int n = 141;
    // 1) 分别以十六进制、十进制、八进制先后输出 n
    cout << "1)" << hex << n << " " << dec << n << " " << oct << n << endl;
    double x = 1234567.89, y = 12.34567;
    // 2)保留5位有效数字
    cout << "2)" << setprecision(5) << x << " " << y << " " << endl;
    // 3)保留小数点后面5位
    cout << "3)" << fixed << setprecision(5) << x << " " << y << endl;
    // 4)科学计数法输出，且保留小数点后面5位
    cout << "4)" << scientific << setprecision(5) << x << " " << y << endl;
    // 5)非负数显示正号，输出宽度为12字符，宽度不足则用 * 填补
    cout << "5)" << showpos << fixed << setw(12) << setfill('*') << 12.1
         << endl;
    // 6)非负数不显示正号，输出宽度为12字符，宽度不足则右边用填充字符填充
    cout << "6)" << noshowpos << setw(12) << left << 12.1 << endl;
    // 7)输出宽度为 12 字符，宽度不足则左边用填充字符填充
    cout << "7)" << setw(12) << right << 12.1 << endl;
    // 8)宽度不足时，负号和数值分列左右，中间用填充字符填充
    cout << "8)" << setw(12) << internal << -12.1 << endl;
    cout << "9)" << 12.1 << endl;
    return 0;
}
```

程序的输出结果是：

```
1)8d 141 215
2)1.2346e+06 12.346
3)1234567.89000 12.34567
4)1.23457e+06 1.23457e+01
5)***+12.10000
6)12.10000****
7)****12.10000
8)-***12.10000
9)12.10000
```



需要注意的是，`setw()` 算子所起的作用是一次性的，即只影响下一次输出。每次需要指定输出宽度时都要使用 `setw()`。因此可以看到，第 9) 行的输出因为没有使用 `setw()`，输出的宽度就不再是前面指定的 12 个字符。

在读入字符串时，`setw()` 还能影响 `cin` 的行为。例如下面的程序：

```
#include <iomanip>
#include <iostream>
using namespace std;
int main() {
    string s1, s2;
    cin >> setw(4) >> s1 >> setw(3) >> s2;
    cout << s1 << "," << s2 << endl;
    return 0;
}
```

输入：
1234567890↙

程序的输出结果是：
1234,567

说明`setw(4)`使得读入 s1 时，只读入 4 个字符，其后的`setw(3)`使得读入 s2 时只读入 3 个字符。

setw() 用于 cin 时，同样只影响下一次的输入。

思考题：setw() 究竟是如何实现的，以至于能和 cout 连用来指定输出宽度？自行查看编译器所带的 `iomanip` 头文件，然后写一个功能和 `setw()` 完全相同的 `mysetw()`。

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183415.png"/></div>

## 调用cout的成员函数

ostream 类有一些成员函数，通过 cout 调用它们也能用于控制输出的格式，其作用和流操纵算子相同，如表 3 所示。

| 成员函数     | 作用相同的流操纵算子 | 说明                                                         |
| ------------ | -------------------- | ------------------------------------------------------------ |
| precision(n) | setprecision(n)      | 设置输出浮点数的精度为 n。                                   |
| width(w)     | setw(w)              | 指定输出宽度为 w 个字符。                                    |
| fill(c)      | setfill (c)          | 在指定输出宽度的情况下，输出的宽度不足时用字符 c 填充（默认情况是用空格填充）。 |
| setf(flag)   | setiosflags(flag)    | 将某个输出格式标志置为 1。                                   |
| unsetf(flag) | resetiosflags(flag)  | 将某个输出格式标志置为 0。                                   |

 

setf 和 unsetf 函数用到的`flag`，与` setiosflags` 和 `resetiosflags` 用到的完全相同。

这些成员函数的用法十分简单。例如下面的三行程序：

```cpp
cout.setf(ios::scientific);cout.precision(8);cout << 12.23 << endl;
```

输出结果是：
1.22300000e+001

