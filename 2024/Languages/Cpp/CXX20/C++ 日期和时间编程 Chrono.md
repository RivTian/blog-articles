#CXX20

> 本文学习自，[保罗的酒吧 - C++ 日期和时间编程](https://paul.pub/cpp-date-time/)

> 需要学习的文章：
>
> * [C++语言的单元测试与代码覆盖率](https://paul.pub/gtest-and-coverage/)
> * [C++ 并发编程（从C++11到C++17）](https://paul.pub/cpp-concurrency/)
> * [C++ 内存模型](https://paul.pub/cpp-memory-model/)
> * [C++与正则表达式](https://paul.pub/cpp-regex/)
> * [C++中的值类别](https://paul.pub/cpp-value-category/)
> * [如何做好接口设计](https://paul.pub/api-desgin/)

日期和时间是编程中非常常用的功能。本文是对C++11到C++17中相关编程接口的介绍。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-02-cpp-date-time/bg.jpeg)

# 介绍

C++中可以使用的日期时间API主要分为两类：

- C-style 日期时间库，位于`<ctime>`头文件中。这是原先`<time.h>`头文件的C++版本。
- `chrono`库：C++ 11中新增API，增加了时间点，时长和时钟等相关接口。

在C++11之前，C++编程只能使用C-style日期时间库。其精度只有秒级别，这对于有高精度要求的程序来说，是不够的。

但这个问题在C++11中得到了解决，C++11中不仅扩展了对于精度的要求，也为不同系统的时间要求提供了支持。

另一方面，对于只能使用C-style日期时间库的程序来说，C++17中也增加了timespec将精度提升到了纳秒级别。

# 代码示例

本文中所贴出的代码示例可以到我的Github上获取：[paulQuei/cpp-date-time](https://github.com/paulQuei/cpp-date-time)。

或者，你也可以直接通过下面这条命令获取所有源码：

```
git clone https://github.com/paulQuei/cpp-date-time.git
```

为了简化书写，本文中给出的代码都已经默认做了以下操作：

```
#include <chrono>
#include <ctime>
#include <iostream>

using namespace std;
```

# C-style 日期时间库

C-style 日期时间库中包含的函数和数据类型说明如下：

## 函数

| 函数                                                         | 说明                                 |
| :----------------------------------------------------------- | :----------------------------------- |
| `std::clock_t clock()`                                       | 返回自程序启动时起的处理器时钟时间   |
| `std::time_t time(std::time_t* arg)`                         | 返回自纪元起计的系统当前时间         |
| `double difftime(std::time_t time_end,`   `std::time_t time_beg)` | 计算时间之间的差                     |
| `int timespec_get(std::timespec* ts, int base)`∗∗            | 返回基于给定时间基底的日历时间       |
| `char* ctime(const std::time_t* time)`                       | 转换 time_t 对象为文本表示           |
| `char* asctime(const std::tm* time_ptr)`                     | 转换 tm 对象为文本表示               |
| `std::size_t strftime(char* str,`   `std::size_t count, const char* format,`   `const std::tm* time)` | 转换 tm 对象到自定义的文本表示       |
| `std::size_t wcsftime( wchar_t* str,`   `std::size_t count, const wchar_t* format,`   `const std::tm* time)` | 转换 tm 对象为定制的宽字符串文本表示 |
| `std::tm* gmtime(const std::time_t* time)`                   | 将time_t转换成UTC表示的时间          |
| `std::tm* localtime(const std::time_t *time)`                | 将time_t转换成本地时间               |
| `std::time_t mktime(std::tm* time)`                          | 将tm格式的时间转换成time_t表示的时间 |

## 数据类型

| 名称       | 说明                              |
| :--------- | :-------------------------------- |
| time_t     | 从纪元起的时间类型                |
| tm         | 日历时间类型                      |
| timespec∗∗ | 以秒和纳秒表示的时间              |
| clock_t    | 进程运行时间                      |
| size_t     | sizeof 运算符返回的无符号整数类型 |

## 结构梳理

这里有不少的函数和数据类型，刚开始接触的时候似乎不太容易记得住。

但实际上，如果我们把它们画成一张图就比较好理解了，如下所示：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-02-cpp-date-time/ctime.png)

在这幅图中，以数据类型为中心，带方向的实线箭头表示该函数能返回相应类型的结果。

- clock函数是相对独立的一个函数，它返回进程运行的时间，具体描述见下文。
- time_t描述了纪元时间，通过time函数可以获得它。但它只能精确到秒级别。
- timespec类型在time_t的基础上，增加了纳秒的精度，通过 `timespec_get` 获取。这是**C++17**上新增的
- tm是日历类型，因为它其中包含了年月日等信息。通过gmtime，localtime和mktime函数可以将time_t和tm类型互相转换。
- 考虑到时区的差异，因此存在gmtime和localtime两个函数。
- 无论是time_t还是tm结构，都可以将其以字符串格式输出。ctime和asctime输出的格式是固定的。如果需要自定义格式，需要使用strftime或者wcsftime函数。

## 进程运行时间

clock函数返回进程迄今为止所用的处理器时间。单独调度该函数一次所返回的值是没有意义的，只有两次不同值的差才有意义。

该值表示了进程从关联到程序执行的实现定义时期开始，所用的粗略处理器时间。而且这个值仅仅是处理器的时钟周期。如果希望将其转换为以秒为单位，还需要将它除以常量 `CLOCKS_PER_SEC` 。

下面是一段代码示例：

```cpp
clock_t time1 = clock();
double sum = 0;
for(int i = 0; i < 100000000; i++) {
  sum += sqrt(i);
}
clock_t time2 = clock();

double t = ((double)(time2 - time1)) / CLOCKS_PER_SEC ;
cout << "CLOCKS_PER_SEC: " << CLOCKS_PER_SEC << endl;
cout << "Process running time: " << t << "s" << endl;
```

其输出如下：

```bash
CLOCKS_PER_SEC: 1000000
Process running time: 0.80067s
```

你可能知道，现代的操作系统上进程都是分时占用处理器的，所以程序的处理器时间会小于真实世界流逝的时间。但这仅仅是对于单处理器而言的。在多处理器系统上，如果你的进程使用了多线程，那么其所用的处理器时间可能比真实世界流逝的时间值还要大。

## 关于纪元时间

纪元时间（Epoch time）又叫做Unix时间或者POSIX时间。它表示自1970 年 1 月 1 日 00:00 UTC 以来所经过的秒数（不考虑闰秒）。它在操作系统和文件格式中被广泛使用。

这个想法很简单：以一个时间为起点加上一个偏移量便可以表达任何一个其他的时间。

> 如果你好奇为什么选这个时间作为起点，可以点击这里：[Why is 1/1/1970 the “epoch time”?](https://stackoverflow.com/questions/1090869/why-is-1-1-1970-the-epoch-time)。

下面是一个代码示例：

```cpp
time_t epoch_time = time(nullptr);
cout << "Epoch time: " << epoch_time << endl;
```

其输出如下：

```bash
Epoch time: 1577433897
```

time函数接受一个指针，指向要存储时间的对象，通常可以传递一个空指针，然后通过返回值来接受结果。

虽然标准中没有给出定义，但time_t通常使用整形值来实现。

作为一个程序员，你可能马上会意识到整形的位数和溢出的问题。事实也刚好是这样，在一些历史实现上使用了32位有符号整数来实现time_t，其造成的结果就是：在[2038-01-19 03:14:07](https://en.wikipedia.org/wiki/Year_2038_problem)这个时间点，这个值会溢出。

不过不用担心太多，这个时间距现在还有将近20年，到那个时候，估计那些有问题的系统已经不会再继续运转或者已经被升级了。

## 计算时间差

在一些情况下，我们需要计算一个操作的时间长度。这自然的就需要计算两个时间点的差分。这时就可以使用difftime函数。

事实上，我们知道time_t以秒级别表示纪元时间，并且它又是以整形实现的，直接将两个time_t相减，可以得到相同的结果。

下面是一个代码示例：

```cpp
time_t time1 = time(nullptr);
double sum = 0;
for(int i = 0; i < 1000000000; i++) {
  sum += sqrt(i);
}
time_t time2 = time(nullptr);

double time_diff = difftime(time2, time1);
cout << "time1: " << time1 << endl;
cout << "time2: " << time2 << endl;
cout << "time_diff: " << time_diff << "s" << endl;
```

其输出如下，可以看到这正是time1和time2两个整数相减的结果：

```bash
time1: 1577434406
time2: 1577434414
time_diff: 8s
```

> 注意：time_t只精确到秒，它无法描述毫秒级别的时间，所以在有更高精度要求的情况下，需要使用下文提到的其他方法。

## 输出时间和日期

当然，我们还希望将时间以字符串的形式打印出来。这时就可以使用ctime函数。不过该函数打印的格式是固定的：`Www Mmm dd hh:mm:ss yyyy\n`。如果你希望自定义输出的格式，可以使用下文提到的其他方法。

下面是一个代码示例：

```cpp
time_t now = time(nullptr);
cout << "Now is: " << ctime(&now);
```

其输出如下：

```bash
Now is: Fri Dec 27 16:17:45 2019
```

## UTC时间与本地时间

对于一个具体的时刻来说，不同时区的具体时间是不一样的，例如：东京时间就比北京时间快了一个小时。

我们既有可能需要知道无差别的标准时间（UTC时间），例如：计算一个航班的时长。也有可能需要获取某个当地的具体时间：gmtime用来将 std::time_t 的纪元时间转换为UTC时间，而localtime则将纪元时间转换为本地时区所代表的日历时间。

日历时间使用tm结构描述。其结构如下：

```cpp
struct tm {
  int tm_sec;
  int tm_min
  int tm_hour;
  int tm_mday;
  int tm_mon;
  int tm_year;
  int tm_wday;
  int tm_yday;
  int tm_isdst;
};
```

需要注意的是：

- tm_mon 并非我们人类理解的月份，而是[0, 11]的范围。因此在某些时候你可能要对其+1.
- tm_year 是自1900起之年，因此你可能也需要对其 +1900。

`asctime` 可以直接将tm结构转换成人类理解的字符串格式，不过其格式是固定的：`Www Mmm dd hh:mm:ss yyyy\n`。对于有特定格式要求，可以使用下文提到的其他方法。

下面是一个代码示例：

```cpp
time_t now = time(nullptr);

tm* gm_time = gmtime(&now);
tm* local_time = localtime(&now);

cout << "gmtime: " << asctime(gm_time);
cout << "local_time: " << asctime(local_time);
```

其输出如下：

```bash
gmtime: Fri Dec 27 08:36:14 2019
local_time: Fri Dec 27 16:36:14 2019
```

由于北京时间的时区是GMT+8，所以我的本地时间比UTC时间快8个小时。

## 自定义时间格式

无论是ctime函数还是asctime函数，其输出的格式都是固定的。在有格式要求的情况下，它们并不能完成需求。

当然，你可以直接读取tm结构体中的字段来进行输出，例如：

```cpp
time_t now = time(nullptr);
tm* t = localtime(&now);

cout << "Now is: " << t->tm_year + 1900 << "/" << t->tm_mon + 1<< "/" << t->tm_mday << " ";
cout << t->tm_hour << ":" << t->tm_min << ":" << t->tm_sec << endl;
```

> 请注意这段代码中，需要对tm_year和tm_mon进行转换才是我们日常理解的日期。

很显然，这样写太啰嗦了。

更好的方法是：使用strftime或者wcsftime函数来指定格式输出。关于这两个函数的格式，可以看这个链接：[std::strftime format](https://zh.cppreference.com/w/cpp/chrono/c/strftime)。

想要输出上面代码同样的格式，只要这样就可以完成任务了：

```cpp
char buffer[32];
strftime(buffer, 32, "%Y/%m/%d %H:%M:%S", t);
cout << "Now is: " << buffer << endl;
```

它们会输出同样的结果：

```bash
Now is: 2019/12/27 16:40:39
```

> 除了`<ctime>`之外，为了方便时间的输入输出，从C++11开始，在`<iomanip>`头文件中还增加了[put_time](https://zh.cppreference.com/w/cpp/io/manip/put_time)与[get_time](https://zh.cppreference.com/w/cpp/io/manip/get_time)两个函数。

## 纳秒精度的timespec

前面我们已经说了，纪元时间的精度只有秒级别。这在很多时候是不够用的。

为了解决这个问题，C++17上增加了timespec类型提供了纳秒级别的精度。

以下是四个时间单位的换算和英文表示：
$$
1秒(Second)=10^3毫秒(millisecond)=10^6微妙(microsecond)=10^9纳秒(nanosecond)
$$
timespec类型结构如下：

```cpp
struct timespec {
  std::time_t tv_sec;
  long tv_nsec;
};
```

可以看到，这里是在time_t之外，增加了一个tv_nsec来表达纳秒的数量。为了获取这个值，C++17新增了timespec_get函数。

以下是一个代码示例：

```cpp
timespec ts;
timespec_get(&ts, TIME_UTC);
char buff[100];
strftime(buff, sizeof buff, "%D %T", std::gmtime(&ts.tv_sec));
printf("Current time: %s.%09ld UTC\n", buff, ts.tv_nsec);
```

> timespec_get 的第二个参数是一个整形表达的base。目前标准只定义了 `TIME_UTC`。

其输出如下：

```bash
Current time: 12/27/19 09:03:29.497456000 UTC
```

说完了C-style日期时间库，让我们再来看看C++11新增的**chrono**库。

# chrono 库

> “chrono”是英文chronology的缩写，其含义是“年表；年代学”。

## 时钟

为了满足不同类型的需求，C++11 **chrono**库中包含了三种类型的时钟，它们的说明如下：

| 名称                  | 说明                         |
| :-------------------- | :--------------------------- |
| system_clock          | 系统时钟                     |
| steady_clock          | 单调时钟，不会被调整         |
| high_resolution_clock | 拥有可用的最短嘀嗒周期的时钟 |

system_clock 的时间来源是系统时钟，而系统时间随时都可能被调整。所以如果你需要计算两个时间点的时间差，这不是一个好的选择。因为如果两次时间差中间系统时间被调整了，其结果就没有意义了。

steady_clock会保证单调性。它就好像物理时间只会向前移动，无法减少。它最适合用来度量间隔。

high_resolution_clock 表示实现提供的拥有最小计次周期的时钟。它可以是 system_clock 或 steady_clock 的别名，也可能是第三个独立时钟。

这三个时钟类有一些共同的成员，如下所示：

| 名称       | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| $now()$    | 静态成员函数，返回当前时间，类型为clock::time_point          |
| time_point | 成员类型，当前时钟的时间点类型，见下文“时间点”               |
| duration   | 成员类型，时钟的时长类型，见下文“时长”                       |
| rep        | 成员类型，时钟的tick类型，等同于clock::duration::rep         |
| period     | 成员类型，时钟的单位，等同于clock::duration::period          |
| is_steady  | 静态成员类型：是否是稳定时钟，对于steady_clock来说该值一定是true |

每种时钟类都有一个 $now()$ 静态函数来获取当前时间，返回的类型是由该时钟类下的time_point描述。这是一个std::chrono::time_point模板类的具体实例，例如：std::chrono::time_point\<std::chrono::system_clock>或者std::chrono::time_point\<std::chrono::steady_clock>。是的，这个类型太长了，不过在C++11中，你可以用auto关键字来简写。

例如，下面是不使用和使用auto关键字的写法：

```cpp
std::chrono::time_point<std::chrono::steady_clock> now = std::chrono::steady_clock::now();

auto now2 = std::chrono::steady_clock::now();
```

### 与C-style转换

system_clock与另外两个clock不一样的地方在于，它还提供了两个静态函数用来与std::time_t来回转换：

| 名称        | 说明                              |
| :---------- | :-------------------------------- |
| to_time_t   | 转换系统时钟时间点为 std::time_t  |
| from_time_t | 转换 std::time_t 到系统时钟时间点 |

由此，我们可以通过下面这幅图来描述几种时间类型的转换：

![img](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-02-cpp-date-time/conversion.png)

下面是一个代码示例：

```cpp
auto now = chrono::system_clock::now();
time_t time = chrono::system_clock::to_time_t(now);
cout << "Now is: " << ctime(&time) << endl;
```

## 时长

### ratio

人类对于精度的要求是没有止境的。虽然目前的精度已经从秒到毫秒，微妙甚至纳秒。但很难说今后还会不会有更高精度的要求。

前面我们已经看到，为了将精度从秒变成纳秒，C++17增加了timespec_get函数和timespec类型。如果以后还有更高的精度要求，C++标准还需要添加更多的函数和类型吗？老实说，这不是一个好的设计。

为了解决这个问题，C++11中添加了一个新的头文件和类型，那就是：ratio。

std::ratio描述了编译时的有理数，这是一个模板类。有了这个类型之后，我们就可以表示任意精度的值了。

例如：相对于秒来说，毫秒是11,00011,000，微妙是11,000,00011,000,000，纳秒是11,000,000,00011,000,000,000。通过ratio可以这样表达：

```cpp
std::ratio<1, 1000>       milliseconds;
std::ratio<1, 1000000>    microseconds;
std::ratio<1, 1000000000> nanoseconds;
```

其实上，`<ratio>`头文件中包含了很多以10为基数的各种分数，具体可以见这里：[编译时有理数算术](https://zh.cppreference.com/w/cpp/numeric/ratio)。

我们只需要通过：std::milli，std::micro，std::nano使用就好了。

当然，ratio能表达的数值不仅仅是以10为基底的。对于任意的分数都可以表达，例如: 5757，591023591023等等。

对于一个具体的ratio来说，可以通过`den`获取分母的值，`num`获取分子的值。

不仅仅如此，`<ratio>`头文件还包含了：ratio_add，ratio_subtract，ratio_multiply，ratio_divide来完成分数的加减乘除四则运算。

例如，想要计算57+59102357+591023，可以这样写：

```cpp
ratio_add<ratio<5, 7>, ratio<59, 1023>> result;
double value = ((double) result.num) / result.den;
cout << result.num << "/" << result.den << " = " << value << endl;
```

> 请注意：无论是分子还是分母，都是一个整形。在C++中，整形的除法结果仍然是整形，多余部分会被丢弃，因此想要获取double类型的结果，需要先将其转换成double。

这段代码输出的结果是：

```bash
5528/7161 = 0.771959
```

### 时长类型

类模板 std::chrono::duration 表示时间间隔。有了ratio之后，表达时长就很方便了，下面是chrono库中提供的很常用的几个时长单位：

| 类型                      | 定义                                                      |
| :------------------------ | :-------------------------------------------------------- |
| std::chrono::nanoseconds  | duration</*至少 64 位的有符号整数类型*/, std::nano>       |
| std::chrono::microseconds | duration</*至少 55 位的有符号整数类型*/, std::micro>      |
| std::chrono::milliseconds | duration</*至少 45 位的有符号整数类型*/, std::milli>      |
| std::chrono::seconds      | duration</*至少 35 位的有符号整数类型*/>                  |
| std::chrono::minutes      | duration</*至少 29 位的有符号整数类型*/, std::ratio<60»   |
| std::chrono::hours        | duration</*至少 23 位的有符号整数类型*/, std::ratio<3600» |

duration类的count()成员函数返回具体数值。

### 时长运算

时长之间最常用的运算自然是相加或者相减，这个通过“+”，“-”就可以完成。

除此之外，chrono库中还提供了下面几个常用的函数：

| 函数           | 说明                                         |
| :------------- | :------------------------------------------- |
| duration_cast  | 进行时长的转换                               |
| floor（C++17） | 以向下取整的方式，将一个时长转换为另一个时长 |
| ceil（C++17）  | 以向上取整的方式，将一个时长转换为另一个时长 |
| round（C++17） | 转换时长到另一个时长，就近取整，偶数优先     |
| abs（C++17）   | 获取时长的绝对值                             |

例如：想要知道2个小时零5分钟一共是多少秒，可以这样写：

```cpp
chrono::hours two_hours(2);
chrono::minutes five_minutes(5);

auto duration = two_hours + five_minutes;
auto seconds = chrono::duration_cast<chrono::seconds>(duration);
cout << "02:05 is " << seconds.count() << " seconds" << endl;
```

我们可以得到：

```bash
02:05 is 7500 seconds
```

从C++14开始，你甚至可以用字面值来描述常见的时长。这包括：

- `h`表示小时
- `min`表示分钟
- `s`表示秒
- `ms`表示毫秒
- `us`表示微妙
- `ns`表示纳秒

这些字面值位于`std::chrono_literals`命名空间下。于是，可以这样表达2个小时以及5分钟：

```cpp
using namespace std::chrono_literals;
auto two_hours = 2h;
auto five_minutes = 5min;
```

## 时间点

时间点中包含了时钟和时长两个信息，类模板  `std::chrono::time_point` 表示时间中的一个点。

时钟的 $now()$ 函数返回的值就是一个时间点。`time_point` 中的 `time_since_epoch()` 返回从其时钟起点开始的时长。

### 时间点运算

时间点有加法操作和减法操作，与我们的常识相一致：

```
时间点 + 时长 = 时间点
时间点 - 时间点 = 时长
```

例如：可以通过两个时间点相减计算一个时间间隔，下面是一个代码示例：

```cpp
auto start = chrono::steady_clock::now();
double sum = 0;
for(int i = 0; i < 100000000; i++) {
    sum += sqrt(i);
}
auto end = chrono::steady_clock::now();

auto time_diff = end - start;
auto duration = chrono::duration_cast<chrono::milliseconds>(time_diff);
cout << "Operation cost : " << duration.count() << "ms" << endl;
```

上面这个代码很好的说明了：有了duration和duration_cast，我们可以以任意的精度来描述结果的值。

除了相加和相减，两个时间点还有比较操作：判断一个时间点在另外一个时间点之前还是之后。对于这些操作，通过运算符： `==`，`!=`，`<`，`<=`，`>`，`>=`来完成。

# 展望 C++ 20

通过上面的讲解我们可以看到，自C++11以来，日期时间库功能获得了极大的提升。并且，在C++14和C++17上也有一些小改进。

但如果你使用过Java类库，就会觉得差距还是比较大。Java中的相关API早就有对于日历，时区等方面更好的支持。

对于这一点， 在C++下一个标准C++20中，将有相应的支持。这些改进包括：

- 更多类型的时钟，例如：utc_clock，tai_lock，gps_clock等。
- 时钟以及时间点的转换
- Calendar支持
- 时区支持

关于这一点，你可以看这个链接：[Date and time utilities](https://en.cppreference.com/w/cpp/chrono)。

如果想获取更多关于C++20的信息，可以直接看这里：[N4842：《Working Draft, Standard for Programming Language C++》](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4842.pdf)。

# 参考资料与推荐读物

- [cppreference: Date and time utilities](https://en.cppreference.com/w/cpp/chrono)
- [cppreference: Standard library header ](https://en.cppreference.com/w/cpp/header/ctime)
- [The C++ Standard Library: A Tutorial and Reference, 2nd Edition](https://www.amazon.com/Standard-Library-Tutorial-Reference-2nd/dp/0321623215/)