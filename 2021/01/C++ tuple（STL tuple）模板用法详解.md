`tuple` 是**C++ 11**新引进的 `build-in structure`，但其实在其他语言中tuple的使用已经行之有年(e.g. Javascript和Python中都有tuple)。C++ 11中tuple的引进是为了降低不同programming languages之间的隔阂，比方说有些programmer于本是写Python的，他在Python的世界中用tuple已经很习惯了，但是当他今天转行来写C++时，如果原本在python里熟悉且常用的东西已经不存在了，这就会增加他转行写C++的难度与门槛。而C#在.NET 4.0后也开始support tuple。

另外，tuple 这个术语也适用于很多其他的场景，例如数据库，这里一个 tuple 就是由一些类型的不同数据项组成的，这和 tuple 的概念相似。tuple 对象有很多用途。当需要将多个对象当作一个对象传给函数时，tuple 类型是很有用的。

在介绍C++ 11的tuple之前，我们先来看其他语言的tuple：

### Tuple in Javascript and Python

$$
=====\ Tuple\ in\ Javascript =====
$$

```javascript
function Tuple_In_JavaScript(val){
    var tuple = [24,"Kanna",[13.1,"Rio"]];
    alert(tuple[0]);
    alert(tuple[1]);
    alert(tuple[2][0]);
    alert(tuple[2][1]);
}
```

$$
=====\ Tuple\ in\ Python =====
$$

```python
tuple = (241,"Kanna",13.1);
print(tuple[0])
print(tuple[1])
print(tuple[2])
```

简单的来说，tuple就是**一组有序的值**(an ordered set of values)。在关系型数据库(relational database)中，某个table的一个row也可以称为tuple；而在programming language中，tuple亦可以用来储存从database table中捞出来的row。除此之外，tuple也可以让function用来return 2个以上的return value。

### C++ 11中tuple的使用

代码：

```C++
// #include <bits/stdc++.h>
#include <iostream>
#include <string>
#include <tuple>
using namespace std;

void UsePair() {
    cout << "====UsePair====\n";
    pair<int, string> p1 = {1, "Kanna"};
    pair<float, char> p2 = make_pair(13.1f, 'a');

    cout << p1.first << " " << p1.second << endl;
    cout << p2.first << " " << p2.second << endl;

    cout << "\n";
}

void UseTuple() {
    cout << "====UseTuple====\n";
    tuple<int, double, string> t1(1, 13.1, "RioTian");
    cout << get<0>(t1) << endl;
    cout << get<1>(t1) << endl;
    cout << get<2>(t1) << endl;

    // 使用 tuple 头文件中辅助函数 make_tuple() 生成tuple 对象
    tuple<int, float, char> t2 = make_tuple(2, 15.6f, 'a');
    cout << get<0>(t2) << endl;
    cout << get<1>(t2) << endl;
    cout << get<2>(t2) << endl;
}

int main() {
    UsePair();
    UseTuple();

    return 0;
}
```

对C++的programming来说对pair应该不陌生，tuple可视为pair的一般化。比较麻烦的是用tuple时必须使用std::get来取得tuple内的值。

Initialization of a tuple：

```cpp
// #include <bits/stdc++.h>
#include <iostream>
#include <tuple>
using namespace std;

void UseTuple() {
    // error, until C++17
    tuple<int, float, char> t3 = {2, 35.2f, 'a'};
    cout << get<0>(t2) << endl;
    cout << get<1>(t2) << endl;
    cout << get<2>(t2) << endl;
}

tuple<int, int> ReturnTuple() {
    // error, until C++17
    return {5, 6};

    // in C++ 11, we can only use
    // return std::make_tuple(5, 6);
}

int main() {
    UseTuple();
    ReturnTuple();
    return 0;
}
```

在C++ 11中，我们已经可以透过std::vector v = {1, 2, 3, 4};轻易地来初始化vector。但是tuple必须要到C++ 17后才会support这个初始化的方式。所以在执行上面范例时会产生compile error。

### tuple 基本操作

$$
=====\ tuple\ 基本操作 =====
$$

#### **tuple的创建和初始化**

```cpp
 std::tuple<T1, T2, TN> t1;            //创建一个空的tuple对象（使用默认构造），它对应的元素分别是T1和T2...Tn类型，采用值初始化。
std::tuple<T1, T2, TN> t2(v1, v2, ... TN);    //创建一个tuple对象，它的两个元素分别是T1和T2 ...Tn类型; 要获取元素的值需要通过tuple的成员get<Ith>(obj)进行获取(Ith是指获取在tuple中的第几个元素，请看后面具体实例)。
std::tuple<T1&> t3(ref&); // tuple的元素类型可以是一个引用
std::make_tuple(v1, v2); // 像pair一样也可以通过make_tuple进行创建一个tuple对象
```

 tuple的元素类型为引用：

```cpp
std::string name;
std::tuple<string &, int> tpRef(name, 30);
// 对tpRef第一个元素赋值，同时name也被赋值 - 引用
std::get<0>(tpRef) = "Sven";
 
// name输出也是Sven
std::cout << "name: " << name << '\n';
```

**有关tuple元素的操作**

1. 等价结构体

   开篇讲过在某些时候tuple可以等同于结构体一样使用，这样既方便又快捷。如：

```cpp
struct person {
    char *m_name;
    char *m_addr;
    int  *m_ages;
};

//可以用tuple来表示这样的一个结构类型，作用是一样的。
std::tuple<const char *, const char *, int>
```

2. 如何获取tuple元素个数

  当有一个tuple对象但不知道有多少元素可以通过如下查询：

```cpp
// tuple_size
#include <iostream>     // std::cout
#include <tuple>        // std::tuple, std::tuple_size
 
int main ()
{
  std::tuple<int, char, double> mytuple (10, 'a', 3.14);
 
  std::cout << "mytuple has ";
  std::cout << std::tuple_size<decltype(mytuple)>::value;
  std::cout << " elements." << '\n';
 
  return 0;
}
 
//输出结果：
mytuple has 3 elements
```

 3.如何获取元素的值

获取tuple对象元素的值可以通过get<Ith>(obj)方法进行获取；

Ith - 是想获取的元素在tuple对象中的位置。

obj - 是想获取tuple的对象

```cpp
// tuple_size
#include <iostream>     // std::cout
#include <tuple>        // std::tuple, std::tuple_size
 
int main ()
{
  std::tuple<int, char, double> mytuple (10, 'a', 3.14);
 
  std::cout << "mytuple has ";
  std::cout << std::tuple_size<decltype(mytuple)>::value;
  std::cout << " elements." << '\n';
 
  //获取元素
  std::cout << "the elements is: ";
  std::cout << std::get<0>(mytuple) << " ";
  std::cout << std::get<1>(mytuple) << " ";
  std::cout << std::get<2>(mytuple) << " ";
 
  std::cout << '\n';
 
  return 0;
}
 
//输出结果：
mytuple has 3 elements.
the elements is: 10 a 3.14 
```

  tuple不支持迭代，只能通过元素索引(或tie解包)进行获取元素的值。但是给定的索引必须是在编译器就已经给定，不能在运行期进行动态传递，否则将发生编译错误：

```cpp
for(int i=0; i<3; i++)
    std::cout << std::get<i>(mytuple) << " "; //将引发编译错误
```

4.获取元素的类型

 要想得到元素类型可以通过tuple_element方法获取，如有以下元组对象：

```cpp
std::tuple<std::string, int> tp("Sven", 20);
 
// 得到第二个元素类型
 
std::tuple_element<1, decltype(tp)>::type ages;  // ages就为int类型
 
ages = std::get<1>(tp);
 
std::cout << "ages: " << ages << '\n';
 
//输出结果： 
ages: 20
```

 5.利用tie进行解包元素的值

**tie 函数的详解**：[Click here](https://www.cnblogs.com/RioTian/p/14076214.html)

 如同pair一样也是可以通过tie进行解包tuple的各个元素的值。如下tuple对象有4个元素，通过tie解包将会把这4个元素的值分别赋值给tie提供的4个变量中。

```cpp
#include <iostream>
#include <tuple>
#include <utility>
 
int main(int argc, char **argv) {
    std::tuple<std::string, int, std::string, int> tp;
    tp = std::make_tuple("Sven", 25, "Shanghai", 21);
 
    // 定义接收变量
    std::string name;
    std::string addr;
    int ages;
    int areaCode;
 
    std::tie(name, ages, addr, areaCode) = tp;
    std::cout << "Output: " << '\n';
    std::cout << "name: " << name <<", ";
    std::cout << "addr: " << addr << ", ";
    std::cout << "ages: " << ages << ", ";
    std::cout << "areaCode: " << areaCode << '\n';
 
    return 0;
}
 
//输出结果：
Output: 
name: Sven, addr: Shanghai, ages: 25, areaCode: 21
```

但有时候tuple包含的多个元素时只需要其中的一个或两个元素，如此可以通过std::ignore进行变量占位，这样将会忽略提取对应的元素。可以修改上述例程：

```cpp
#include <iostream>
#include <tuple>
#include <utility>
 
int main(int argc, char **argv) {
    std::tuple<std::string, int, std::string, int> tp;
    tp = std::make_tuple("Sven", 25, "Shanghai", 21);
 
    // 定义接收变量
    std::string name;
    std::string addr;
    int ages;
    int areaCode = 110;
 
    std::tie(name, ages, std::ignore, std::ignore) = tp;
    std::cout << "Output: " << '\n';
    std::cout << "name: " << name <<", ";
    std::cout << "addr: " << addr << ", ";
    std::cout << "ages: " << ages << ", ";
    std::cout << "areaCode: " << areaCode << '\n';
 
    return 0;
}
 
//输出结果：
Output: 
name: Sven, addr: , ages: 25, areaCode: 110
```

6. tuple元素的引用

   前面已经列举了将引用作为tuple的元素类型。下面通过引用搭配make_tuple()可以提取tuple的元素值，将某些变量值设给它们，并通过改变这些变量来改变tuple元素的值：

```cpp
#include <iostream>
#include <tuple>
#include <functional>
 
int main(int argc, char **agrv) {
 
    std::tuple<std::string, int, float> tp1("Sven Cheng", 77, 66.1);
 
    std::string name;
    int weight;
    float f;
 
    auto tp2 = std::make_tuple(std::ref(name), std::ref(weight), std::ref(f)) = tp1;
 
    std::cout << "Before change: " << '\n';
    std::cout << "name: " << name << ", ";
    std::cout << "weight: " << weight << ", ";
    std::cout << "f: " << f << '\n';
 
    name = "Sven";
    weight = 80;
    f = 3.14;
 
	std::cout << "After change: " << '\n';
	std::cout << "element 1st: " << std::get<0>(tp2) << ", ";
	std::cout << "element 2nd: " << std::get<1>(tp2) << ", ";
	std::cout << "element 3rd: " << std::get<2>(tp2) << '\n';
 
    return 0;
}
 
//输出结果：
Before change: 
name: Sven Cheng, weight: 77, f: 66.1
After change: 
element 1st: Sven, element 2nd: 80, element 3rd: 3.14
 
 
```



### 参考

* http://c.biancheng.net/view/517.html
* https://blog.csdn.net/sevenjoin/article/details/88420885
* [C++函数：std::tie 详解](https://www.cnblogs.com/RioTian/p/14076214.html)
* [std::tuple](https://www.cnblogs.com/RioTian/p/14307223.html#stdtuple)