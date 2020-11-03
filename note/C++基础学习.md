# C++基础学习

---

## 命名空间
当使用两个已经封装好的产品，而他们包含一个或多个名称相同的功能(函数、方法等)，为了让编译器知道使用的是哪个产品中的功能，通常需要指定命名空间。

### 命名空间的用法，以`std`为例。

第一种：
```cpp
std::cout << "Hello";
```
第二种：
```cpp
using namespace std
...
std::cout << "Hello";
```
第三种：
```cpp
using std::cout;
...
cout << "cout";
```
需要注意的点
- 当`using namspace std`在函数中时，则只有该函数中可以使用`std`中的功能

## 类型强制转换
传统的C：
```cpp
char a = 'a';
cout << (int)a << "\n";
```
C++中新的强制转换：
```cpp
char a = 'a';
cout << static_cast<int>(a) << "\n";
```

## 变量的初始化
### C:
```cpp
int a = 20;
```
### C++:
#### 第一种
```cpp
int a(20);
```
#### 第二种
**普通变量**
```cpp
int a{20};
int a = {20};
```
**类成员**
类和结构体的初始化类似，但是注意结构体成员默认public，类则是默认private
```cpp
class S {
public:
    int a;
    int b;
};

int main() {
//    S s = {1, 2};
    S s{1, 2};       // 也可以
    cout << s.a << " " << s.b;
    return 0;
}
```
**数组**
```cpp
vector<int> a {1, 2, 3, 4};       // vector的
// int a[4] {1, 2, 3, 4};            // 数组的
for (int it : a) cout << it << "\n";
```
```cpp
int b[4] {};          // 所有元素都初始化为零
vector<int> b(4);     // 同理
```
## cout格式化输出
**用的时候再查阅文档**
```cpp
printf();
```
```cpp
cout.setf();
cout.precision();
```
## 共用体union
共用体union是一种数据格式，能存储不同的数据类型，但是同时只能存储一种类型。也就是说，结构体可以存储int、long、double，共用体只能存储int或long或double。
```cpp
union one4all {
    int int_val;
    long long_val;
    double double_val;
}

int main() {
    one4all pail;
    pail.int_val = 15;
    cout << pail.int_val << "\n";
    pail.double_val = 15;
    cout << pail.double_val << "\n";
    return 0;
}
```
- 由于共用体每次只能存储一个值，因此它必须有足够大的空间来存储最大的成员，所以，共用体的长度为其最大成员的长度。
- 共用体的用途之一是，当数据项使用两种或更多格式（但不会同时使用）时，可节省空间。

## 枚举量enum
### 枚举量初始化
```cpp
enum bits{one = 1, two = 2, four = 4, eight = 8};
enum bigstep{first = -6, second = 100, third};  
```
```cpp
enum {zero, null = 0, one, numero_uno = 1};  // 枚举量zero和null都是0，one和numero_uno都是1
```
- 指定的值必须是整数
- `first`默认为零，后面未被初始化的枚举量的值将比其前面的枚举量大1。因此，`third`的值为101
- 最后，可以创建多个相同的枚举量

### 枚举的取值范围
每个枚举都有取值范围（range），通过强制类型转换，可以将取值范围中的任何整数值赋给枚举变量，即使这个值不是枚举值。例如
```cpp
enum bits{one = 1, two = 2, four = 4, eight = 8};
bits myflag;
myflag = bits(6);
```
如上的`myflag = bits(6);`是合法的，虽然6不是枚举值，但它位于枚举定义的取值范围内。
- 上限：大于枚举量最大值的，第一个2的幂数减去1，即是上限，如前面`bigstep`中上限为127
- 下限：如果枚举量最小值不小于零，最下限为零；如果是负数，则如上限一样计算，比如`bigstep`中下限为-7

## 指针
- `&`是`地址运算`符，获取当前变量的地址
- `*`是`间接值运算符`或者说`解除引用运算符`
### 指针初始化
一定要在对指针应用解除引用运算符(*)之前，将指针初始化为一个确定的、适当的地址。这是关于使用指针的金科玉律！
在C++中创建指针时，计算机将分配用来存储地址的内存，但不会分配用来存储指针所指向的数据的内存！即为数据分配空间是一个独立的步骤！
**正确的指针初始化：**
```cpp
int a = 20;
int *pt = &a;
cout << pt << "->" << a;
```
**危险的指针初始化！！！**
```cpp
int *pt;
*pt = 23333;
```
这样的话指针`pt`指向哪个值，代表的是哪个地址，我们并不知道，然后通过`*pt = 23333;`的赋值操作，会改变`pt`所代表地址上的值，很有可能发生危险，这样的指针叫**野指针！**不过现在的IDE都会对这种操作进行危险提示。

## 数组指针
### 一维数组
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

int main() {
    int a[4] = {1, 2, 3, 5};
    int *pt = a;
    cout << pt[1] << "\n";
    cout << *pt << "\n";
    cout << *(++pt) << "\n";
    return 0;
}
```
### 二维数组
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

void show(int a[][1]) {
    cout << a[0][0] << "\n";
}

void show(int (*pt)[1], double size) {
    cout << pt[0][0] << "\n";
}

int main() {
    int a[2][1] = {{1}, {2}};
    int (*pt)[1] = a;
    cout << pt[0][0] << "\n";
    show(a);
    show(pt);
    show(a, 1);
    return 0;
}
```
可以发现如下特点

- 一维或者二维数组名都是地址，都可以让指针指向它
- 传参或者访问数组时，指针和数组名作用相同，用法相同
- 数组名作为地址是不可变的，而指针时可以再被赋值，指向新的数组
- `*数组名`同样能访问值，即重载运算符`*`能够得到存储在该地址的值


### new和delete分配和释放内存
即`typeName *pt = new typeName`
如下操作分配的是一块新的内存，指针表示着这块新的内存。。。当然也就就不属于野指针。
```cpp
int *pt = new int;
*pt = 20;
cout << pt << "-->" << *pt;
```
`new`和`delete`应该配对使用
正确操作：
```cpp
int *pt = new int;
*pt = 20;
cout << pt << "-->" << *pt;
delete pt;
```
示范错误：
```cpp
int *pt = new int;
*pt = 20;
cout << pt << "-->" << *pt;
delete pt;     
delete pt;     // 不能释放已经释放的内存快，会产生不确定的结果
int a = 20;
int *pi = &a;
delete pi;     // 不能释放声明变量所获得的内存
```
- `new`和`delete`必须配对使用，即delete只能释放new分配的内存
- 释放内存后，原指向该内存的指针可以重新指向另一个分配的内存块（使用`new`）；
- `delete`不能用来释放已经释放的内存块，因为重复`delete 指针`的操作，会释放不确定区域的内存
- `delete`不能释放声明变量所获得的内存
- 对空指针`delete`是安全的

### 使用new来创建动态数组
注意`new`和`delete`也要配对使用
```cpp
int *pt = new int[10];
delete [] pt;
```
- 不要使用`delete`释放不是`new`分配的内存
- 不要使用`delete`释放同一个内存块两次
- 如果使用`new []`为数组分配内存，则应使用`delete`来释放

### 简单文件输入和输出
对于文件输入，C++使用类似于`cout`的东西。

- 必须包含头文件`fstream`
- 头文件`fstream`定义了一个用于处理输出的`ofstream`类
- 需要声明一个或多个`ofstream`对象
- 必须指明命名空间`std`
- 需要将`ofstream`对象与文件关联起来。为此，方法之一是使用`open()`方法
- 使用完文件以后，应该用`close()`方法关闭
- 可结合使用`ofstream`对象和运算符`<<`来输出各种类型的数据
#### **写出到文件**
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

int main() {
    char name[50] = "bbkgl";
    int age = 22;
    double weight = 69.5;
    char info[100] = "Give me a girl.";

    // 输出在屏幕上
    cout << "Name: " << name << endl;
    cout << "Age: " << age << endl;
    cout << "Weight: " << weight << endl;
    cout << "Info: " << info << endl;

    // 输出文件
    ofstream file;
    file.open("fish.txt", ios::binary);
    file.clear();
    file << "Name: " << name << endl;
    file << "Age: " << age << endl;
    file << "Weight: " << weight << endl;
    file << "Info: " << info << endl;
    return 0;
}
```
#### **文件读入到内存**
对于文件输入，C++使用类似于`cin`的东西。

- 必须包含头文件`fstream`
- 头文件`fstream`定义了一个用于处理输出的`ifstream`类
- 需要声明一个或多个`ifstream`对象
- 必须指明命名空间`std`
- 需要将`ofstream`对象与文件关联起来。为此，方法之一是使用`open()`方法
- 使用完文件以后，应该用`close()`方法关闭
- 可结合使用`ifstream`对象和运算符`>>`来读取各种类型的数据
- 可结合使用`ifstream`对象和运算符`get()`方法读取一个字符
- 可结合使用`ifstream`对象和`getline()`方法读取一行文字
- 可以结合使用`ifstream`对象和`eof()、fail()`等方法来判断输入是否成功
- `ifstream`对象本身被用作测试条件时，如果最后一个读取操作成功，它将被转换为布尔值`true`,否则转换为`false`
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

int main() {
    string line, word;
    ifstream file;
    file.open("fish.txt");
    while (file.good()) {
        getline(file, line);
        cout << line << "\n";
    }
    file.close();
    return 0;
}
```

## 函数指针
### 1.获取函数的地址
获取函数的地址很简单：只要使用函数名即可。也就是说，如果`think()`是一个函数，则`think`就是函数的地址。
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

void think() {
    cout << "让我想想" << "\n";
}

void thinkThink() {
    cout << "让我再想想" << "\n";
}

int main() {
    cout << think << "\n";
    cout << thinkThink << "\n";
    return 0;
}
```

### 2.声明函数指针
声明指向某种数据类型的指针时，必须指定指针指向的类型。同样，声明指向函数的指针时，也必须指定指针指向的函数类型。
**举例**：
如果函数声明如下：
```cpp
double pam( double, double);
```
那么函数指针应该如下：
```cpp
double (*pf)(double, double);
```
即直接用(*pointerName)替换掉函数名

### 使用指针来调用函数
举例：
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

double pam( double, double);
void showFunc(double (*pf)(double, double));

int main() {
    double (*pf)(double, double) = pam;
    cout << pf << endl;
    showFunc(pam);
    showFunc(pf);
    cout << pf(1, 2) << endl;
    return 0;
}

double pam(double x, double y) {
    return x * y;
}

void showFunc(double (*pf)(double, double)) {
    cout << pf << "\n";
}

打印结果：
1
1
1
2
```

可以发现如下特点：

- 作为形参传入时，函数指针和函数名有同样的功能和效果
- 打印函数指针和函数名，是一样的值（地址）
- 函数名和函数指针都能用来调用函数
- 如果要让函数作为参数传入其他函数，则需要声明形参为函数指针类型

函数指针和函数名的不同之处：

- 函数指针可以再次被赋值，即可以指向其他的函数，但函数名作为直接的地址不可以
- 函数名和数组名都相当于常量，而函数指针和数组指针相当于变量

## 函数重载
函数重载的关键是函数的参数列表————也称函数特征标。如果两个函数的参数类型和数目相同，同时排列顺序也相同，则它们的特征标相同，而与变量名无关。

- 如果多个函数特征标不相同而函数名相同，则这多个函数构成重载
- C++编译器在检查函数特征标时，将把类型引用和类型本身视为同一个特征标
- 匹配函数的时候，C++不区分`const`和`非const`变量，视为同一个特征标
- 返回值类型不同不构成重载
- 符号重载也属于函数重载

## 函数模板
函数模板是通用的函数描述，也就是说，它们使用泛型来定义函数，泛型即指可以用具体的类型（int、double）来进行替换的一种通用类型。
```cpp
template <typename Anytype>
void Swap(AnyType &a, AnyType &b) {
    AnyType temp;
    temp = a;
    a = b;
    b = temp;
}
```
简单例子：
```cpp
#include <cstdio>
#include <iostream>

using namespace std;

template <typename T>
T mul(T a, T b) {
    return a * b;
}

int main() {
    cout << mul(1, 2) << "\n";
    cout << mul(1.0, 2.0) << "\n";
    return 0;
}
```

输出：
```
1 2
2 1
123 456
456 123
```

## 函数的返回值

- 函数中声明为引用的参数可以作为引用返回值
- 当函数返回引用类型时，返回的是对象本身
- 千万不能**返回局部对象（变量）的引用！千万不能返回指向局部对象（变量）的指针**
- 返回局部对象的引用或者指针，局部对象被销毁后，引用和指针会指向**未知的地址!**

## 显示具体化模板函数以及重载
**用到的时候再查阅**

## class和struct的区别

- class对象默认是private，但是struct默认是public
- 其他的区别我暂时也不知道

## 构造函数和析构函数
### 构造函数

 - 构造函数没有返回值，没有生命类型
 - 默认构造函数能让对象在创建时就能给所有成员提供隐式初始值
 - 隐式调用构造函数时，不需要加圆括号---`Stock stock`即可
 - 构造函数原型可以设置默认参数，函数实现时就不要设置默认参数了
 - 构造函数原型设置默认参数时，从右到左设置，任何一个参数设置了默认参数，则其右边的参数必须设置
 - 应避免重载没有参数和带缺省参数的构造函数。即不能既有无参构造函数，又有全是默认参数的构造函数。

### 析构函数
用构造函数创建对象后，程序负责跟踪对象，直到其过期为止。对象过期时，程序将自动调用一个特殊的成员函数，该函数的名称就是——析构函数。析构函数完成清理工作，实际上很有用。如果构造函数使用new来分配内存，则析构函数将使用delete来释放这些内存。如果没有`new`分配的内存，则不需要析构函数中利用`delete`来释放，这个情况下，只需要让编译器生成一个什么都不做的隐式析构函数即可。
**析构函数的原型和函数定义**
```cpp
~Stock()    // 原型


Stock::~Stock() {       // 析构函数定义
    std::cout << "Bye, " << company << "!\n";
}
```

**析构函数的特点**

- 默认存在一个隐式的析构函数
- 析构函数的作用是完成清理工作
- 如果构造函数用`new`分配内存，则需要析构函数用`delete`来清理内存
- 以类`Stock`为例，析构函数的原型必须是`~Stock()`

## 引用作为形参，并在方法中传回引用（声明函数时，不同位置const的作用）

```cpp
const Stock &topval(const Stock &s) const;
```
该函数隐式地访问一个对象，而显式地访问另一个对象，并返回其中一个对象的引用。

- 括号前的`const`让方法返回一个不会被修改的引用。
- 括号中的`const`表明，该函数不会修改被显式地访问的对象。
- 括号后的`const`表明，该函数不会修改被隐式地访问的对象。

## 类的声明
通常，将类声明分为两部分，两部分通常保存在不同的文件中。类声明（包括由函数原型表示的方法）应该放到头文件中，定义成员函数的源代码放在方法文件中。这样便将接口描述和实现细节分开了，理论上说只要知道公有接口就可以使用类。

## 运算符重载
运算符重载也是C++多态的一种形式，其将重载的概念扩展到运算符上，允许赋予C++多种含义和功能。实际上，很多C++（也包括C语言）运算符已经被重载。例如将`*`运算符用于地址，将得到存储在这个地址中的值；但将它用于两个数字时，得到的将是它们的乘积。
举个例子：
```cpp
#include <cstdio>
#include <iostream>

using namespace std;

class Time {
private:
    int hours;
    int minutes;
public:
    Time ();
    Time(int h, int m = 0) ;
    void AddMin(int m) ;
    void AddHr(int h) ;
    void Reset(int h = 0, int m = 0) ;
    Time Sum(const Time &t) const;
    void Show() const;
    Time operator+(const Time &t) const;
};

Time::Time() {
    hours = minutes = 0;
}

Time::Time(int h, int m) {
    hours = h;
    minutes = m;
}

void Time::AddMin(int m) {
    minutes += m;
    hours += minutes / 60;
    minutes %= 60;
}

void Time::AddHr(int h) {
    hours += h;
}

void Time::Reset(int h, int m) {
    hours = h;
    minutes = m;
}

Time Time::Sum(const Time &t) const {
    Time sum;
    sum.minutes = minutes + t.minutes;
    sum.hours = hours + t.hours + sum.minutes / 60;
    sum.minutes %= 60;
    return sum;
}

void Time::Show() const {
    std::cout << hours << " hours, " << minutes << " minutes" << std::endl;
}

Time Time::operator+(const Time &t) const {
    Time sum;
    sum.minutes = minutes + t.minutes;
    sum.hours = hours + t.hours + sum.minutes / 60;
    sum.minutes %= 60;
    return sum;
}

int main() {
    Time t1{8, 40}, t2{2, 50};
    Time t3 = t1 + t2;
    Time t4 = t1.Sum(t2);
    t3.Show();
    t4.Show();
    return 0;
}
```

### 运算符重载限制
上述的运算符`+`重载实质上可以用成员函数表示，即：
```cpp
Time t3 = t1 + t2;
Time t3 = t1.operator+(t2);
```
可以看到运算符重载实际上就是函数重载，而运算符左侧的对象就是调用该运算符的对象。

1. 重载后的运算符至少有一个操作数是用户定义的类型，这将防止用户为标准类型重载运算符。因此，不能将肩负运算符`-`重载为计算两个double值的和，而不是它们的差。
2. 使用运算符时不能违反运算符原来的句法规则（比如`+`，只能让两个类相加，而不能用来对一个类进行计算），同时不能修改运算符的优先级。因此，如果将加号运算符重载成两个类相加，则新的运算符与原来的加号具有相同的优先级。
3. 不能创建新的运算符。例如，不能定义`operator**()`函数来表示求幂。
4. 不能重载下面的运算符。
- `sizeof()`：sizeof运算符
- `.`：成员运算符
- `.*`：成员指针运算符
- `::`：作用于解析运算符
- `?:`：条件运算符
- `typeid`：一个RTTI（运行时类型检查）运算符
- `const_cast`：强制类型转换运算符
- `dynamic_cast`：强制类型转换运算符
- `reinterpret_cast`：强制类型转换运算符
- `static_cast`：强制类型转换运算符
5. 大多数运算符都可以通过成员和非成员函数进行重载，但下面的运算符只能通过成员函数进行重载。
- `=`：赋值运算符
- `()`：函数调用运算符
- `[]`：下标运算符
- `->`：通过指针访问类成员的运算符

## explicit关键字
当构造器中涉及到类型转换时，需要考虑是隐式/显示地类型转换。隐式转换即是可以由单个实参来调用的构造函数定义了一个从形参类型到该类型的隐式转换。编译器在试图编译某一条涉及类型转换的语句的时候，如果类型不一样，编译器会试图进行隐式转换，如果隐式转换后能够正常编译，编译器继续编译过程，否则报错。`explicit`关键字的作用是让构造器不能进行隐式转换。
```cpp
#include <cstdio>
#include <iostream>
#include <utility>
#include "mytime.h"

class Person {
public:
    std::string name;
    int age;

    Person() ;
    //explicit Person(string n = "no name") ;     // IDE提示只有一个参数的构造函数需要explicit强制显示声明
    explicit Person(std::string n) ;              // IDE提示只有一个参数的构造函数需要explicit强制显示声明

    // Person(std::string n, int a) ;             // IDE没有提示需要explicit强制显示声明
    //explicit Person(std::string n, int a = 0) ;          // IDE提示只有一个参数的构造函数需要explicit强制显示声明
    explicit Person(std::string n = "no name", int a = 0) ;// IDE提示只有一个参数的构造函数需要explicit强制显示声明
};

Person::Person(std::string n) {
    name = std::move(n);
    age = 0;
}

Person::Person() {
    name = "no name";
    age = 0;
}

Person::Person(std::string n, int a) {
    name = std::move(n);
    age = a;
}
```
根据上面的例子，可以得出如下结论

- 当构造函数`person()`只有一个参数时，会进行自动隐式转换，`Person p = "xiaoming"`，此时IDE会提示利用`explicit`来强制只能进行显示转换，即`Person p = Person("xiaoming")`，再用`Person p = "xiaoming"`创建对象会报错
- 当构造函数非默认参数个数>=1个时，无法进行隐式转换`Person p = "xiaoming"`，也就不需要`explicit`了。
- 上一条的另一种解释：当构造函数全是默认参数或者只有一个参数时默认参数时，可以进行隐式转换，这时候IDE会提示`只有一个参数的构造函数需要explicit强制显示声明`
- `explicit`关键字只能用于类内部的构造函数声明上，不能用于类外部的函数定义（实现）上
- 与`explicit`关键字相对应的关键字是`implicit`

## 友元函数（重载运算符）
有一类特殊的非成员函数可以访问类的私有成员，它们被称为友元函数。
创建友元函数的第一步是将其原型放在类声明中，并在原型声明前加上关键字`friend`:
```cpp
friend Time operator*(double m, const Time &t) ;
```
友元函数的定义：
```cpp
Time operator*(double m, const Time &t) {
    int min = static_cast<int> (m * t.minutes);
    int h = (static_cast<int> (m * t.hours)) + min / 60;
    h %= 24;
    min %= 60;
    return Time(h, min);
}
```
测试友元函数：
```cpp
int main() {
    Time a = Time(13, 40);
    Time b;
    double mul = 2.0;
    Time c = mul * a;
    c.Show();
}


打印结果：
3 hours, 20 minutes
```
可以得出如下结论
- 类的友元函数是非成员函数，其访问权限与成员函数相同
- 友元函数在声明时，需要用`friend`来修饰
- 友元函数在在定义（实现）时，不需要`Time::`限定符，因为不是成员函数
- 友元并不会有悖于OOP封装的思想，在类声明时，可以决定哪些函数是友元函数。类方法和友元是表达类接口的两种不同机制

## 常用友元，重载<<运算符
例子：
创建友元函数的第一步是将其原型放在类声明中，并在原型声明前加上关键字`friend`:
```cpp
friend void operator<<(std::ostream &os, const Time &t) ;
```
友元`<<`定义（实现）：
```cpp
void operator<<(std::ostream &os, const Time &t) {
    os << t.hours << " hours, " << t.minutes << " minutes" << std::endl;
}
```
测试友元：
```cpp
int main() {
    Time a = Time(13, 40);
    Time b;
    double mul = 2.0;
    Time c = mul * a;
    std::cout << c;
    c.Show();
}

打印：
3 hours, 20 minutes
3 hours, 20 minutes
```

根据上述例子，可以得出重载`<<`作为友元的特点：

- `<<`运算符已经被重载过多次了，原先是执行位运算的运算符
- `cout`是一个`ostream`的对象，同时我们在定义友元函数时，参数之一便是`ostream`对象`os`
- 平时使用`<<`运算符时，`cout`也是一个对象，同时他输出`<<`右侧的对象
- `ostream`对象必须在`<<`的左侧，而要输出的对象在运算符右侧

随之而来的问题出现了，重载后的`<<`运算符不能再像如下情况一样使用了：
```cpp
Time a = Time(13, 40);
Time b;
double mul = 2.0;
Time c = mul * a;
std::cout << c << 2333 << "Hey";
```
编译无法通过。由此我们需要了解关于`cout`运算符操作的一些知识，上述语句其实是这么执行的
```cpp
(((std::cout << c) << 2333) << "Hey");
```
正常的`cout << Obj`操作即在执行完`(std::cout << c)`后，会返回一个`ostream`对象，使得上述输入
变成如下情况：
```cpp
using std::cout;
(((cout << c) << 2333) << "Hey");
cout << c --------- return ostream---->cout
cout << "2333" ---- return ostream---->cout
cout << "Hey" ----- return ostream---->cout
```
所以，要使得执行成功，则每次`cout << `以后需要返回一个`ostream`对象，上述`<<`重载改成如下实现
函数声明
```cpp
friend std::ostream &operator<<(std::ostream &os, const Time &t) ;
```

函数实现
```cpp
std::ostream &operator<<(std::ostream &os, const Time &t) {
    os << t.hours << " hours, " << t.minutes << " minutes";
    return os;
}
```

测试
```cpp
int main() {
    using std::cout;
    using std::endl;
    Time a = Time(13, 40);
    Time b;
    double mul = 2.0;
    Time c = mul * a;
    cout << c << " 2333" << " Hey" << endl;
    c.Show();
}

打印：
3 hours, 20 minutes 2333 Hey
3 hours, 20 minutes
```

## 重载运算符作为成员函数/非成员函数
对于很多运算符来说，可以选择使用成员函数或非成员函数来实现运算符重载。一般来说，非成员函数应该是友元函数，这样它才能直接访问类的私有数据。
分如下一些情况：

- 对于成员函数版本来说，一个操作数通过`this`指针隐式传递另一个操作数作为函数参数显式地传递；
- 对于友元版本来说，两个操作数都作为参数来传递
- 非成员版本的重载运算符函数所需要的性参数目与运算符使用的操作数数目相同
- 而成员版本所需的参数数目少一个，因为其中的一个操作数是被隐式地传递的调用对象

## 矢量（Vector）类的实现
- 注意一下矢量和标量的区别
- 实现矢量的加法，乘法运算，矢量的反向
- 重载`*、+、-、<<`运算符，实现运算和输出
- 使用枚举类来确定矢量的表示方法（直角坐标、极坐标）

## C++中namespace和class的异同（`namspace::sth`区别`class::sth`）
`class`和`namespace`都是命名空间

- `class`是封闭的、一次性的，只能在一个文件里对其进行添加和定义；`namespace`是开放的，可以在不同文件中对同一个`namespace`添加东西，比如stl里面的东西都是在名字空间std里面，但是却定义在了多个文件里面。
- 对`class A`的成员`M`用 `A::M` 并不总是允许的，这种用法只在两种情况下有效：
1. 类的成员函数引用本类（含祖先类）的成员
2. 其它函数引用类的静态成员
**因为类中定义了一个（非静态）成员变量，并没有定义其实体，只是定义了一个名字。只有创建了类的实例（对象），成员变量的实体才存在。**
- 但 `namespace` 不存在这个问题，因为，`namespace` 中定义的变量，就是一个实体，在任何情况下（只要在可见域内），对`namespace A`，`A::M` 的写法都是允许的。


## 转换函数（从`class`到基本数据类型如`string->char []`）

- 要将类对象转换成其他类型，必须定义转换函数
- 转换函数必须是成员函数
- 转换函数原型：`operator typeName();`
- 隐式转换有时候是必要的，比如`String s = "23333";`，但最好不要依赖于隐式转换函数

## 复制构造函数
复制构造函数将一个对象复制到新创建的对象中。也就是说，它用于初始化过程中（包含按值传递），而不是常规的复制过程中。类的复制构造函数原型通常如下：
```cpp
Class_name(const Class_name &);
```
首先`StringBad`类的声明如下：
```cpp
//
// Created by heart on 2019/1/16.
//

#ifndef STRINGBAD_STRINGBAD_H
#define STRINGBAD_STRINGBAD_H
#include <iostream>

class StringBad {
private:
    char *str;
    int len;
    static int num_strings;    // 生成对象的多少

public:
    StringBad(const char *s);
    StringBad();
    ~StringBad();
    // friend function
    friend std::ostream &operator<<(std::ostream &os, const StringBad &st);
};


#endif //STRINGBAD_STRINGBAD_H

```
它接受一个指向类对象的常量引用作为参数。例如，`StringBad`类的复制构造函数的原型如下：
```cpp
StringBad(const StringBad &);
```
复制构造函数的调用通常发生在如下情形（以`StringBad`类为例）：
```cpp
#include <iostream>
#include "stringbad.h"

int main() {
    StringBad s = "23333";
    StringBad s1 = s;
    StringBad s2 = StringBad(s);
    StringBad s3(s);
    StringBad *s4 = new StringBad(s);
    return 0;
}

打印结果：
1: "23333" object created
"23333" object delete, 0 left
"23333" object delete, -1 left
"23333" object delete, -2 left
"23333" object delete, -3 left
```
根据如上结果，我们可以得到一些结论：

- 新建一个对象并将其初始化为同类对象时，复制构造函数都将被调用。
- 构造函数只是被调用了一次，但是析构函数调用了4次
- 用于计数的静态变量`num_strings`因为构造函数调用一次，析构函数调用4次最终值为-3
- 复制构造函数产生的对象，同样由析构函数进行清理
- 复制构造函数是默认的，没有重写的系统默认提供，默认提供的显然不会改变`num_strings`的值
- 为了避免这种问题的出现，我们需要重写复制构造函数

如果说构造函数长这样：
```cpp
StringBad::StringBad(const char *s) {
    len = std::strlen(s);
    str = new char[len + 1];
    std::strcpy(str, s);
    num_strings++;
    cout << num_strings << ": \"" << str << "\" object created\n";
}
```
那么复制构造函数就应该长这样：
```cpp
StringBad::StringBad(const StringBad &sb) {
    num_strings++;
    len = sb.len;
    str = new char[len + 1];
    std::strcpy(str, sb.str);
    cout << num_strings << ": \"" << str << "\" object created\n";
}
```
测试0：
```cpp
#include <iostream>
#include "stringbad.h"

int main() {
    StringBad s = "23333";
    StringBad s1 = s;
    StringBad s2 = StringBad(s);
    StringBad s3(s);
    StringBad *s4 = new StringBad(s);
    return 0;
}

打印结果：
1: "23333" object created
2: "23333" object created
3: "23333" object created
4: "23333" object created
5: "23333" object created
"23333" object delete, 4 left
"23333" object delete, 3 left
"23333" object delete, 2 left
"23333" object delete, 1 left
```
这里的新问题是为什么会有`1 left`存在？
其实根据前面学到的`new`和`delete`必须成对存在，显然这里`new new StringBad(s)`并没有`delete`。
`new`的对象是不会调用析构函数的，只有当`delete`的时候才会调用析构函数。所以测试如下，
测试1：
```cpp
#include <iostream>
#include "stringbad.h"

int main() {
    StringBad s = "23333";
    StringBad s1 = s;
    StringBad s2 = StringBad(s);
    StringBad s3(s);
    StringBad *s4 = new StringBad(s);
    delete s4;
    return 0;
}

打印结果：
1: "23333" object created
2: "23333" object created
3: "23333" object created
4: "23333" object created
5: "23333" object created
"23333" object delete, 4 left
"23333" object delete, 3 left
"23333" object delete, 2 left
"23333" object delete, 1 left
"23333" object delete, 0 left
```

## 派生类和基类的关系

- 派生类对象可以使用基类的方法，条件是方法不是`private`的、
- 基类对象可以在不进行显式转换的情况下被派生类对象赋值（多态）
- 基类指针可以在不进行显式类型转换的情况下指向派生类对象（多态）
- 基类引用可以在不进行显式类型转换的情况下指向派生类对象（多态）
- 不可以将派生类指针和引用指向基类对象
- 指向派生类对象的基类指针和引用只能调用基类的方法和属性

**派生类包含基类，少的调用多的没问题，多的调用少的就可能出问题了。**

## 虚函数的作用

- 被virtual关键字修饰的函数，就是虚函数。
- 实现多态性，多态性是将接口和实现进行分离。
- 当基类**指针**指向一个子类对象，通过这个指针调用子类和基类同名成员函数的时候，基类声明为虚函数就会调子类的这个函数，不声明就会调用基类的。
- 即当调用一个虚函数时，被执行的代码必须和调用函数的对象的**动态类型**相一致。

## 虚函数的注意事项

- 包含虚函数的类指针列表会增大
- 构造函数不能声明为虚函数
- 不在析构和构造函数中调用虚函数
- 友元不能声明为虚函数
- 如果派生类没有重新定义函数，将使用该函数的基类版本；
- 重新定义继承的函数，将不会构成重载，会隐藏同名的基类方法，即只能调用派生类的方法；

## 虚析构函数

- 析构函数的作用是在对象撤销之前做必要的“清理工作”。
- 定义虚析构函数后，当派生类的对象从内存中撤销的时候，会优先调用派生类的析构函数然后再调用基类的析构函数。（作用）

## 对象的静态类型和动态类型
- 静态类型：对象在声明时采用的类型，是在编译期确定的；
- 动态类型：目前所指对象的类型，是在运行期确定的；

## 静态联编（静态绑定）和动态联编（动态绑定）
- 静态绑定：绑定的是对象的静态类型，某特性（比如函数）依赖于对象的静态类型，发生在编译期；
- 动态绑定：绑定的是对象的静态类型，某特性（比如函数）依赖于对象的动态类型，发生在运行期；
- 静态联编说的是在编译期间就已经确定好了调用和被调用者的关系；
- 动态联编说的是在运行期间才确定好调用和被调用的关系；
- 动态联编必须包括两下方面：成员函数必须声明为`virtual`；基类中声明了为虚函数，则派生类中不用再声明；
- `virtual`函数是动态绑定的，而缺省参数却是静态绑定的；
- `Java`中的函数（方法）默认就是虚函数，默认就是动态绑定的

## `protected`和`private`的区别
- 派生类可以直接访问基类的`protected`成员
- 派生类不可以直接访问基类的`private`成员

## 纯虚函数和抽象基类
- 包含纯虚函数的类叫做抽象类
- 对于抽象类来说，`C++`是不允许它去实例化对象的。即抽象类无法实例化对象。
- 抽象类的子类也可以是抽象类
- 抽象类的子类必须将抽象类中的所有纯虚函数全部实现，这个子类才能实例化对象
纯虚函数的声明：
```cpp
virtual void print() = 0;
```

## 继承与动态分配
若基类使用动态内存分配，并重新定义赋值和复制构造函数，派生类需考虑如下情况：

- 若派生类没有使用动态内存分配，则派生类中不用定义显式析构函数，复制构造函数和赋值运算符；
- 如果派生类使用动态内存分配，则必须为派生类定义显式析构函数，复制构造函数和赋值运算符；
- 对于友元，基类的友元可以访问基类的`private`属性，派生类的友元可以访问派生类的`private`属性，派生类若要访问基类的`private`属性，需将友元参数中的派生类对象强制转换成基类对象。


## 接口继承与实现继承
**接口继承：**派生类只继承函数接口，也就是只继承函数声明。
**实现继承：**派生类同时继承函数的接口和实现。

## 私有继承private

- 私有继承是实现`has-a`关系的途径
- 基类的私有成员会被隐藏
- 基类的共有成员和保护成员都将称为派生类的私有成员（方法或者函数同理）
- 私有继承不继承接口，但是继承实现（即派生类可以在成员函数中使用基类方法，但是基类方法不会成为派生类对象公有接口的一部分）

## 使用包含还是私有继承
包含和私有继承都能实现`has-a`关系，大部分情况下使用包含较多。

- 私有继承需要重新定义虚函数
- 私有继承会使得关系更抽象
- 包含能包括多个同类的子对象，而继承只能使用一个这样的对象
- 私有继承提供的特性比包含多

*通常，应该使用包含来建立`has-a`关系；如果新类需要访问原有类的保护成员，或需要重新定义虚函数，则应使用私有继承。*

## 使用using重新定义访问权限
使用保护继承或者私有继承时，基类的公有成员将成为保护成员或者私有成员。假设要让基类的方法在派生类外面可用，可以重新定义一个实现同样功能的方法；但更好的是使用`using`关键字重新定义访问权限，即使用一个`using`声明来指出派生类可以使用特定的基类成员（就像命名空间那样）。

## 多重继承
需要注意一下虚基类继承。

## 数组模板示例和非类型参数
例如使用模板参数来提供数组的大小：
```cpp
template<class T, int n>
class Vector {
    ...
private:
    T a[n];
    ...
};

int main() {
    Vector<double, 12> v; 
    return 0;
}
```

## 模板中使用多个类型参数及默认参数值
```cpp
template<class T1, class T2, class T3 = int>
class Vector {
    //...
private:
    T1 a[1];
    T2 b[2];
    T3 c[3];
    //...
};

int main() {
    Vector<double, char> v;
    return 0;
}
```

## 成员模板
在类的子类前再使用模板
```cpp
template<class T>
class A{
public:
    template<class V>
    class B{
    public:
        V a;
    };
    B<T> b;
};

int main() {
    A<int> a;
    std::cout << a.b.a;
    return 0;
}
```

## 模板作为模板参数
有如下声明：
```cpp
template<template<typename T> class Thing>
class Crab{
    ...
};
```
模板参数是`template<typename T> class Thing`，其中`template<typename T> class`是类型，`Thing`是参数。
假设有如下声明：
```cpp
Crab<King> legs;
```
为了使得上述声明被接受，模板参数`King`必须是一个模板类，其声明与模板参数`Thing`的声明匹配：
```cpp
template<typename T>
class King{
    ...
};
```

## 模板类和友元
C++好啰嗦啊。。。

## 模板别名
C++真的太啰嗦了。

## 友元类及友元类方法
再说。。。

## 嵌套类
再说。。。

## 异常机制

- `(throw)try-catch`，`catch`内可以放类，标准异常类或者自己定义的类

## RTTI（运行阶段类型识别）
C++有三个支持RTTI的元素：

- 如果可能的话，`dynamic_cast`运算符将使用一个指向基类的指针来生成一个指向派生类的指针，否则，该运算符返回0---空指针；
- typeid运算符返回一个指出对象类型的值；
- type_info结构储存了有关特性类型的信息；

**RTTI仅适用于包含虚函数的类**

## 智能指针
三个智能指针模板（auto_ptr、unique_ptr和shared_ptr）都定义了类似指针的对象，可以将new获得（直接或间接）的地址赋给这种对象。当智能指针过期时，其析构函数将使用delete来释放内存。因此，如果将new的地址赋给这些对象，将无需记住稍后释放：在智能指针过期时，这些内存将自动被释放。

## 深拷贝和浅拷贝
- 浅拷贝：编译器提供的默认的拷贝构造函数和赋值运算符重载函数，仅仅是将对象中各个数据成员的值拷贝给另一个同一个类对象对应的数据成员。 
- 深拷贝：对于对象中动态成员，就不能仅仅简单地赋值，而应该重新动态分配空间。



## 虚函数表在什么时候创建？

- 虚函数表在编译时创建
- 运行时每个对象会有一个指向虚函数表的指针
- 同一个类的所有对象共享这一个虚函数表