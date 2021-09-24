## C#简介

C# 是一个现代的、通用的、面向对象的编程语言。

.Net 包括 .Net平台 和 .Net FrameWork 框架。平台是应用运行的基础，框架则提供支持应用在该平台上运行的能力。

C# 可以开发基于 .Net 平台的应用。

<u>*综上，可以明白在电脑上安装某些软件需要额外安装.Net的原因，与之相对应的是：Java既是语言又是平台。*</u>

**.Net 框架**

.Net框架可以开发 window应用程序、Web应用程序、Web服务。

适用C#、C++、Visual Basic、Jscript、COBOL 等等

核心组件：

公共语言运行库（Common Language Runtime - CLR）、.Net 框架类库（.Net Framework Class Library）、元数据（Metadata）和组件（Assemblies）、Windows 窗体（Windows Forms）

**C# 集成开发环境（IDE）**

Visual Studio，Visual Studio Code

> 关于VSC配置教程，详细见：[Here](https://blog.csdn.net/qq_29339467/article/details/108666479)

## 文件结构

- .sln 解决方案文件，点击即可打开整个工程
- .csproj 项目文件
- .cs 类文件

三者关系：解决方案文件包括项目文件，项目文件包括类文件（即代码文件）

## 程序结构

一个 C# 程序主要包括以下部分：

- 命名空间声明（Namespace declaration）
- 一个 class
- Class 方法
- Class 属性
- 一个 Main 方法
- 语句（Statements）& 表达式（Expressions）
- 注释

C# 文件的后缀为 **.cs**。

以下创建一个 **test.cs** 文件，文件包含了可以打印出 `Hello World! C#` 的简单代码：

```c#
using System;
namespace demo
{
    class Hello_World
    {
        static void Main(string[] args)
        {
             /* 第一个 C# 程序*/
            Console.WriteLine("Hello World! C#");
            Console.ReadKey();
        }
    }
}
```

让我们看一下上面程序的各个部分：

- 程序的第一行 **using System;** - **using** 关键字用于在程序中包含 **System** 命名空间。 一个程序一般有多个 **using** 语句。

- 下一行是 **namespace** 声明。一个 **namespace** 里包含了一系列的类。*HelloWorldApplication* 命名空间包含了类 *HelloWorld*。

- 下一行是 **class** 声明。类 *HelloWorld* 包含了程序使用的数据和方法声明。类一般包含多个方法。方法定义了类的行为。在这里，*HelloWorld* 类只有一个 **Main** 方法。

- 下一行定义了 **Main** 方法，是所有 C# 程序的 **入口点**。**Main** 方法说明当执行时 类将做什么动作。

- 下一行  `/*...*/`  将会被编译器忽略，且它会在程序中添加额外的 **注释**。

- Main 方法通过语句

  Console.WriteLine("Hello World"); 

  指定了它的行为。

  *WriteLine* 是一个定义在 *System* 命名空间中的 *Console* 类的一个方法。该语句会在屏幕上显示消息 "Hello World"。

- 最后一行 **Console.ReadKey();** 是针对 VS.NET 用户的。这使得程序会等待一个按键的动作，防止程序从 Visual Studio .NET 启动时屏幕会快速运行并关闭。

以下几点值得注意：

- C# 是大小写敏感的。
- 所有的语句和表达式必须以分号（;）结尾。
- 程序的执行从 **Main** 方法开始。(是 Main 而不是 main)
- **<u>与 Java 不同的是，文件名可以不同于类的名称。</u>**

**C#注释**

- 单行注释 //
- 多行注释 /**/
- 函数说明 ///

*其他注意事项：大小写敏感、Main开始、分号结尾、文件名可与类名不同*

## 基础语法

一个简单的例子如下，基础语言见注释。

```C#
// 引入命名空间
using System;

// 声明命名空间
namespace RectangleApplication
{
    // 声明类
    class Rectangle
    {
        // 成员变量
        double length;
        double width;
        
        // 成员函数
        public void Acceptdetails()
        {
            length = 4.5;    
            width = 3.5;
        }
        public double GetArea()
        {
            return length * width;
        }
        public void Display()
        {
            Console.WriteLine("Length: {0}", length);
            Console.WriteLine("Width: {0}", width);
            Console.WriteLine("Area: {0}", GetArea());
        }
    }
   
    class ExecuteRectangle
    {
        // 开始运行点
        static void Main(string[] args)
        {
            // 实例化一个类
            Rectangle r = new Rectangle();
            r.Acceptdetails();
            r.Display();
            Console.ReadLine();
        }
    }
}
```

**标识符**

标识符是用来识别类、变量、函数或任何其它用户定义的项目。

简单的说就是一个名字。

- 标识符必须以字母、下划线或 @ 开头，后面可以跟一系列的字母、数字（ 0 - 9 ）、下划线（ _ ）、`@`。
- 标识符中的第一个字符不能是数字。
- 标识符必须不包含任何嵌入的空格或符号，比如 ? - +! # % ^ & * ( ) [ ] { } . ; : " ' / \。
- 标识符不能是 C# 关键字。除非它们有一个 @ 前缀。 例如，@if 是有效的标识符，但 if 不是，因为 if 是关键字。
- 标识符必须区分大小写。大写字母和小写字母被认为是不同的字母。
- 不能与C#的类库名称相同。

**关键字**

以下关键字看一遍，有个印象即可。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825150031.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825150052.png)



## 数据类型

### 值类型

![值类型（Value types）表](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825150158.png)



共13个值类型，便于记忆如下：

*无符号的一般加前缀u，byte比较特殊（有符号加s）*

*byte short int long 分别对应8 16 32 64位整数*

*float double decimal 分别对应32 64 128位浮点数*

*bool 为布尔值*

*char 为16位字符*

**sizeof()语句**

表达式 sizeof(type) 产生以字节为单位存储对象或类型的存储尺寸，如sizeof(int) = 4

### 引用类型

**对象（Object）类型**

对象（Object）类型 是 C# 通用类型系统（Common Type System - CTS）中所有数据类型的终极基类。Object 是 System.Object 类的别名。

```C#
object obj;
```

**动态（Dynamic）类型**

可以存储任何类型的值在动态数据类型变量中。这些变量的类型检查是在运行时发生的。

```C#
dynamic 变量名 = 变量值;
dynamic d = 20;
```

*对象类型与动态类型的区别：对象类型变量的类型检查是在编译时发生的，而动态类型变量的类型检查是在运行时发生的。*

**字符串（String）类型**

字符串（String）类型 允许您给变量分配任何字符串值。

字符串（String）类型是 System.String 类的别名。它是从对象（Object）类型派生的。

字符串（String）类型的值可以通过两种形式进行分配：引号和 @引号。

```C#
String str = "runoob.com";

// @（称作"逐字字符串"）将转义字符（\）当作普通字符对待
string str = @"C:\Windows"; 等价 string str = "C:\\Windows";
```

补充：**String与string的区别**

string 是 C# 中的类，String 是 .net Framework 的类(在 C# IDE 中不会显示蓝色) C# string 映射为 .net Framework 的String 如果用 string, 编译器会把它编译成 String，所以如果直接用 String 就可以让编译器少做一点点工作。如果使用 C#，**建议使用 string**，比较符合规范

### 指针类型（Pointer types）

指针类型变量存储另一种类型的内存地址。C# 中的指针与 C 或 C++ 中的指针有相同的功能。

```C#
数据类型* 指针名
char* cptr;
int* iptr;
```

### 值类型与引用类型的区别

值类型存放在栈内存中，变量名对应的是栈地址，栈地址保存的是变量值。

引用类型存放在堆内存和栈内存中，变量名对应栈地址，栈地址中保存的是堆的地址，堆地址保存的是变量值。

**速度上**：值类型存取速度快，引用类型存取速度慢。

**来源上**：值类型继承自System.ValueType，引用类型继承自System.Object。

**保存的数据上**：值类型表示实际数据，引用类型表示指向存储在内存堆中的数据的指针或引用。

## 类型转换

类型转换就是将一种类型的数据转换为另一种类型。

可以由值类型转换为值类型，也可由值类型转换成引用类型。

**隐式类型转换**

C# 默认的以安全方式进行的转换, 不会导致数据丢失。

```C#
int a = 12;
object b = a;
```

**显式类型转换（强制转换）**

需要强制转换符，可能造成数据丢失。

```C#
double a = 12.823;
int b = (int) a;  // b=12
```

以下为类型转换方法，使用方式为 **变量.方法名()**

1. ToBoolean 如果可能的话，把类型转换为布尔型。
2. ToByte 把类型转换为字节类型。
3. ToChar 如果可能的话，把类型转换为单个 Unicode 字符类型。
4. ToDateTime 把类型（整数或字符串类型）转换为 日期-时间 结构。
5. ToDecimal 把浮点型或整数类型转换为十进制类型。
6. ToDouble 把类型转换为双精度浮点型。
7. ToInt16 把类型转换为 16 位整数类型。
8. ToInt32 把类型转换为 32 位整数类型。
9. ToInt64 把类型转换为 64 位整数类型。
10. ToSbyte 把类型转换为有符号字节类型。
11. ToSingle 把类型转换为小浮点数类型。
12. ToString 把类型转换为字符串类型。
13. ToType 把类型转换为指定类型。
14. ToUInt16 把类型转换为 16 位无符号整数类型。
15. ToUInt32 把类型转换为 32 位无符号整数类型。
16. ToUInt64 把类型转换为 64 位无符号整数类型。

**装箱与拆箱**

当一个值类型转换为对象类型时，则被称为 装箱；

当一个对象类型转换为值类型时，则被称为 拆箱。

装箱可以隐式转换，拆箱必须显示转换/用Convert函数

```C#
// 装箱
object obj;
obj = 100;

// 拆箱
object obj = 12;
int b = (int) a;
int c = Convert.ToInt32(a);
```

## 变量

变量是一个供程序操作的存储区的名字，其数据类型决定了变量的内存大小和布局。

数据类型分类：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825153827.png)



更复杂的还有 enum 、 class 、 object...

**变量定义与赋值实例**

```C#
// 数据类型 变量名,变量名,...
int i, j, k;

// 允许定义时初始化
int i = 100;
int d = 3, f = 5; 

// 接收来自用户的值
// 注意Console.ReadLine()得到的是字符串
int num;
num = Convert.ToInt32(Console.ReadLine());
```

**静态变量**

在 C# 中没有全局变量的概念，所有变量必须由该类的实例进行操作，这样做提升了安全性，但是在某些情况下却显得力不从心。

```C#
// 声明语法
static <data_type> <variable_name> = value;
```

## 常量

### 基本类型常量

常量可以是任何基本数据类型，比如整数常量、浮点常量、字符常量或者字符串常量，还有枚举常量。

**整数常量**

前缀表示进制：0x 或 0X 表示十六进制，0 表示八进制，没有前缀则表示十进制。

后缀表示有无符号：U 和 L 分别表示 unsigned 和 long。后缀可以是大写或者小写，多个后缀以任意顺序进行组合。

**浮点常量**

```C#
3.14159       /* 合法 */
314159E-5L    /* 合法 */
510E          /* 非法：不完全指数 */
210f          /* 非法：没有小数或指数 */
.e55          /* 非法：缺少整数或小数 */
```

**字符常量**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825154123.png)



**字符串常量**

括在双引号 "" 里，或者是括在 @"" 里

```C#
string a = "hello, world";                  // hello, world
string b = @"hello, world";               // hello, world
string c = "hello \t world";               // hello     world
string d = @"hello \t world";               // hello \t world
string e = "Joe said \"Hello\" to me";      // Joe said "Hello" to me
string f = @"Joe said ""Hello"" to me";   // Joe said "Hello" to me
string g = "\\\\server\\share\\file.txt";   // \\server\share\file.txt
string h = @"\\server\share\file.txt";      // \\server\share\file.txt
string i = "one\r\ntwo\r\nthree";
string j = @"one
two
three";
```

### 定义常量

**静态常量 const**

在编译时就确定了值，必须在声明时就进行初始化且之后不能进行更改，可在类和方法中定义。

```C#
const <data_type> <constant_name> = value;
const double a=3.14;
```

**动态常量 readonly**

在运行时确定值，只能在声明时或构造函数中初始化，只能在类中定义。

```C#
class Program
{
    readonly int a=1;  // 声明时初始化
    readonly int b;    // 构造函数中初始化
    Program()
    {
        b=2;
    }
    static void Main()
    {
    }
}

```

## 运算符

**算术运算符**

加+ 、 减- 、 乘* 、 除/ 、 取模% 、 自加++ 、 自减

取模：相除，取余数

自加/自减：

```C#
c = a++: 先将 a 赋值给 c，再对 a 进行自增运算
c = ++a: 先将 a 进行自增运算，再将 a 赋值给 c
```

**关系运算符**

相等== 、 不等!= 、 大于> 、 小于< 、 大于等于>= 、 小于等于<=

返回值为 布尔值

**逻辑运算符**

与&& 、 或|| 、 非！

**位运算符**

与% 、 或| 、非~ 、 异或^ 、 左移<< 、 右移>> (补零)

位运算需要先将十进制转换为二进制，然后按对应位进行运算，运算完毕再转换为十进制

```C#
A = 60   0011 1100
B = 13   0000 1101

A&B = 0000 1100
A|B = 0011 1101
A^B = 0011 0001
~A  = 1100 0011
A<<2 = 1111 0000
A>>2 = 0000 1111
```

**赋值运算符**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825154314.png)



**其他运算符**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825154332.png)



**优先级**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210825154350.png)

### 条件语句



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/1/16/16fad605c33a5da7~tplv-t2oaga2asx-watermark.awebp)

**三元表达式 ？ ：**



switch语法

```C#
switch(name){
    case 'A' :
       // ...
       break; 
    case 'B' :
       // ..
       break; 
    default : /* 可选的 */
       // ...
       break; 
}
复制代码
```

### 循环语句



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/1/16/16fad63f18e1c726~tplv-t2oaga2asx-watermark.awebp)





![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/1/16/16fad64308ff2d89~tplv-t2oaga2asx-watermark.awebp)



**foreach语法**

```
public void demo()
{
    int[] fibarray = new int[] { 0, 1, 1, 2, 3, 5, 8, 13 };
    foreach (int element in fibarray)
    {
        System.Console.WriteLine(element);
    }
}
```

## 封装

封装的定义："把一个或多个项目封闭在一个物理的或者逻辑的包中"

**访问修饰符**

- public：所有对象都可以访问；
- private：对象内部可以访问；
- protected：只有该类对象及其子类对象可以访问
- internal：同一个程序集的对象可以访问；
- protected internal：访问限于当前程序集或派生自包含类的类型。（并 的关系）

比如说：一个人A为父类，他的儿子B，妻子C，私生子D（注：D不在他家里），如果我们给A的事情增加修饰符：

public事件，地球人都知道，全公开
 protected事件，A，B，D知道（A和他的所有儿子知道，妻子C不知道）
 private事件，只有A知道（隐私？心事？）
 internal事件，A，B，C知道（A家里人都知道，私生子D不知道）
 protected internal事件，A，B，C，D都知道,其它人不知道

```C#
using System;

namespace RectangleApplication
{
    class Rectangle
    {
        //成员变量
        private double length;
        private double width;

        public void Acceptdetails()
        {
            Console.WriteLine("请输入长度：");
            length = Convert.ToDouble(Console.ReadLine());
            Console.WriteLine("请输入宽度：");
            width = Convert.ToDouble(Console.ReadLine());
        }
        public double GetArea()
        {
            return length * width;
        }
        public void Display()
        {
            Console.WriteLine("长度： {0}", length);
            Console.WriteLine("宽度： {0}", width);
            Console.WriteLine("面积： {0}", GetArea());
        }
    }//end class Rectangle    
    class ExecuteRectangle
    {
        static void Main(string[] args)
        {
            Rectangle r = new Rectangle();
            r.Acceptdetails();
            r.Display();
            Console.ReadLine();
        }
    }
}
```

在上面的实例中，成员变量 length 和 width 被声明为 private，所以它们不能被函数 Main() 访问。

成员函数 AcceptDetails() 和 Display() 可以访问这些变量。

由于成员函数 AcceptDetails() 和 Display() 被声明为 public，所以它们可以被 Main() 使用
Rectangle 类的实例 r 访问。

**默认修饰符**

如果没有指定访问修饰符，则使用类成员的默认访问修饰符，即为 private。

## 方法

### 定义与调用

**定义**

```C#
<Access Specifier> <Return Type> <Method Name>(Parameter List)
{
   Method Body
}
```

- Access Specifier：访问修饰符，这个决定了变量或方法对于另一个类的可见性。
- Return type：返回类型，一个方法可以返回一个值。返回类型是方法返回的值的数据类型。如果方法不返回任何值，则返回类型为 void。
- Method name：方法名称，是一个唯一的标识符，且是大小写敏感的。它不能与类中声明的其他标识符相同。
- Parameter list：参数列表，使用圆括号括起来，该参数是用来传递和接收方法的数据。参数列表是指方法的参数类型、顺序和数量。参数是可选的，也就是说，一个方法可能不包含参数。
- Method body：方法主体，包含了完成任务所需的指令集。

**调用**

先生成类的实例，通过 实例.方法名() 来调用方法。

同一个命名空间下，可以在同一个类下调用方法，也可以在一个类下调用另一个类的方法。

**递归调用**

```C#
using System;

namespace CalculatorApplication
{
    class NumberManipulator
    {
        public int factorial(int num)
        {
            /* 局部变量定义 */
            int result;

            if (num == 1)
            {
                return 1;
            }
            else
            {
                result = factorial(num - 1) * num;
                return result;
            }
        }
   
        static void Main(string[] args)
        {
            NumberManipulator n = new NumberManipulator();
            //调用 factorial 方法
            Console.WriteLine("6 的阶乘是： {0}", n.factorial(6));
            Console.WriteLine("7 的阶乘是： {0}", n.factorial(7));
            Console.WriteLine("8 的阶乘是： {0}", n.factorial(8));
            Console.ReadLine();

        }
    }
}
```

### 参数传递

**按值传递参数**

参数传递的**默认**方式，实际参数的值会复制给形参，实参和形参使用的是两个不同内存中的值。所以，当形参的值发生改变时，**不会影响实参**的值，从而保证了实参数据的安全。

这里的值传递，指的不是数据类型，即使是引用类型也可以按值传递。

```C#
using System;
namespace CalculatorApplication
{
   class NumberManipulator
   {
      public void swap(int x, int y)
      {
         int temp;
         
         temp = x; /* 保存 x 的值 */
         x = y;    /* 把 y 赋值给 x */
         y = temp; /* 把 temp 赋值给 y */
      }
     
      static void Main(string[] args)
      {
         NumberManipulator n = new NumberManipulator();
         /* 局部变量定义 */
         int a = 100;
         int b = 200;
         
         Console.WriteLine("在交换之前，a 的值： {0}", a);
         Console.WriteLine("在交换之前，b 的值： {0}", b);
         
         /* 调用函数来交换值 */
         n.swap(a, b);
         
         Console.WriteLine("在交换之后，a 的值： {0}", a);
         Console.WriteLine("在交换之后，b 的值： {0}", b);
         
         Console.ReadLine();
      }
   }
}
```

<br>

```c#
输出
在交换之前，a 的值：100
在交换之前，b 的值：200
在交换之后，a 的值：100
在交换之后，b 的值：200
```



**按引用传递参数**

使用 **ref 关键字** 声明引用参数，该种类型的参数传递变量地址给方法，实参与形参**会相互影响**。

```
using System;
namespace CalculatorApplication
{
   class NumberManipulator
   {
      public void swap(ref int x, ref int y)
      {
         int temp;

         temp = x; /* 保存 x 的值 */
         x = y;    /* 把 y 赋值给 x */
         y = temp; /* 把 temp 赋值给 y */
       }
   
      static void Main(string[] args)
      {
         NumberManipulator n = new NumberManipulator();
         /* 局部变量定义 */
         int a = 100;
         int b = 200;

         Console.WriteLine("在交换之前，a 的值： {0}", a);
         Console.WriteLine("在交换之前，b 的值： {0}", b);

         /* 调用函数来交换值 */
         n.swap(ref a, ref b);

         Console.WriteLine("在交换之后，a 的值： {0}", a);
         Console.WriteLine("在交换之后，b 的值： {0}", b);
 
         Console.ReadLine();

      }
   }
}

输出
在交换之前，a 的值：100
在交换之前，b 的值：200
在交换之后，a 的值：200
在交换之后，b 的值：100
```

**按输出传递参数**

使用 **out 关键字** 声明引用参数，输出参数会把方法输出的数据赋给自己。

提供给输出参数的变量不需要赋值。当需要从一个参数没有指定初始值的方法中返回值时，输出参数特别有用。

```C#
using System;

namespace CalculatorApplication
{
   class NumberManipulator
   {
      public void getValue(out int x )
      {
         int temp = 5;
         x = temp;
      }
   
      static void Main(string[] args)
      {
         NumberManipulator n = new NumberManipulator();
         /* 局部变量定义 */
         int a = 100;
         
         Console.WriteLine("在方法调用之前，a 的值： {0}", a);
         
         /* 调用函数来获取值 */
         n.getValue(out a);

         Console.WriteLine("在方法调用之后，a 的值： {0}", a);
         Console.ReadLine();

      }
   }
}

输出
在方法调用之前，a 的值： 100
在方法调用之后，a 的值： 5
```

**ref 和 out 的区别**

- 是否需要**初始化**。ref 型传递变量前，变量必须初始化，否则编译器会报错, 而 out 型则不需要初始化
- ref 型传递变量，数值可以传入方法中，而 out 型无法将数据传入方法中。换而言之，**ref 型有进有出，out 型只出不进**。
- out型数据在方法中**必须要赋值**，否则编译器会报错。即out 必须要出，ref不必须。
- 重载时的区别。重载方法时若两个方法的区别仅限于一个参数类型为ref 另一个方法中为out，编译器会报错。

**对于复杂引用类型参数传递的控制**

所谓复杂，是参数是数组或集合类型，或者参数包含这些类型数据，这种情况下上面的方法不能保证参数数据不被修改，因为即使对象为只读的，但是对象中的数组或集合字段（属性）还是可以修改的。

1、集合参数（包含集合字段的引用参数也一样）

.net 4.5以前版本可以使用不包含修改集合元素方法的接口来代替具体集合类型。例如使用IEnumerable接口代替List。4.5版本可以直接使用IReadOnlyCollection接口或实现该接口的集合类型。

2、数组参数

没有好的办法保护数组类型参数不被修改，所以尽量避免使用数组类型作为方法参数，除非用到可选参数时候。

## 可空类型

**可空类型？**

nullable 类型（可空类型），可空类型可以表示其基础值类型正常范围内的值，再加上一个 null 值。

`用法`：变量定义仍和以前一样，只不过在数据类型后加个？，如`int? num = 45;`，这样num既可以被赋值32位整型，也可以被赋值null。

`好处`：在处理数据库和其他包含可能未赋值的元素的数据类型时，将 null 赋值给数值类型或布尔型的功能特别有用。

```C#
int i; //默认值0
int? ii; //默认值null

using System;
namespace CalculatorApplication
{
   class NullablesAtShow
   {
      static void Main(string[] args)
      {
         int? num1 = null;
         int? num2 = 45;
         double? num3 = new double?();
         double? num4 = 3.14157;
         
         bool? boolval = new bool?();

         // 显示值
         
         Console.WriteLine("显示可空类型的值： {0}, {1}, {2}, {3}",
                            num1, num2, num3, num4);
         Console.WriteLine("一个可空的布尔值： {0}", boolval);
         Console.ReadLine();

      }
   }
}

输出
显示可空类型的值： , 45,  , 3.14157
一个可空的布尔值：
```

**Null 合并运算符？？**

Null 合并运算符用于定义可空类型和引用类型的默认值。

`??的作用`如果第一个操作数的值为 null，则运算符返回第二个操作数的值，否则返回第一个操作数的值。

```C#
using System;
namespace CalculatorApplication
{
   class NullablesAtShow
   {
         
      static void Main(string[] args)
      {
         
         double? num1 = null;
         double? num2 = 3.14157;
         double num3;
         num3 = num1 ?? 5.34;      // num1 如果为空值则返回 5.34
         Console.WriteLine("num3 的值： {0}", num3);
         num3 = num2 ?? 5.34;
         Console.WriteLine("num3 的值： {0}", num3);
         Console.ReadLine();

      }
   }
}

输出
num3 的值： 5.34
num3 的值： 3.14157
```

用三元表达式也能实现以上效果

## 数组（Array）

### 一维数组

**定义**

数组是一个存储**相同类型元素**的固定大小的顺序集合。所有的数组都是由**连续的内存**位置组成的。数组是一个**引用类型**，需要使用 **new 关键字**来创建数组的实例。

**声明**

```C#
// 数据类型 维度 数组名
datatype[] arrayName;
```

**初始化**

```C#
// 先声明，再依次初始化
double[] balance = new double[10];
balance[0] = 4500.0;

// 声明的同时初始化1
double[] balance = { 2340.0, 4523.69, 3421.0};

// 声明的同时初始化2（数组大小可省略）
int [] marks = new int[5]  { 99,  98, 92, 97, 95};

// 用其他数组赋值
int [] marks = new int[]  { 99,  98, 92, 97, 95};
int[] score = marks;
```

**默认值**

C# 编译器会根据数组类型隐式初始化每个数组元素为一个默认值。例如，int 数组的所有元素都会被初始化为 0。

**foreach遍历**

```C#
using System;

namespace ArrayApplication
{
   class MyArray
   {
      static void Main(string[] args)
      {
         int []  n = new int[10]; /* n 是一个带有 10 个整数的数组 */


         /* 初始化数组 n 中的元素 */        
         for ( int i = 0; i < 10; i++ )
         {
            n[i] = i + 100;
         }

         /* 输出每个数组元素的值 */
         foreach (int j in n )
         {
            int i = j-100;
            Console.WriteLine("Element[{0}] = {1}", i, j);
         }
         Console.ReadKey();
      }
   }
}

输出
Element[0] = 100
Element[1] = 101
Element[2] = 102
Element[3] = 103
Element[4] = 104
Element[5] = 105
Element[6] = 106
Element[7] = 107
Element[8] = 108
Element[9] = 109
```

### 多维数组

**声明**

```C#
// 二维
string [,] names;

// 三维
int [ , , ] m;
```

**初始化**

```C#
// 二维
int [,] a = new int [3,4] {
 {0, 1, 2, 3} ,   /*  初始化索引号为 0 的行 */
 {4, 5, 6, 7} ,   /*  初始化索引号为 1 的行 */
 {8, 9, 10, 11}   /*  初始化索引号为 2 的行 */
};
复制代码
// 三维
int[, ,] muarr = new int[2, 2, 3]
{
  {{1,2,3},{4,5,6}},
  {{7,8,9},{2,3,4}}  
};
```

**取值**

数组相当于一个矩阵



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/1/17/16fb28ac26a3cf05~tplv-t2oaga2asx-watermark.awebp)



```C#
// 行索引 列索引
int val = a[2,3];
```

**遍历多维数组**

```C#
int[, ,] muarr = new int[2, 2, 3]
{
  {{1,2,3},{4,5,6}},
  {{7,8,9},{2,3,4}}  
};

int rank = muarr.Rank;
Console.WriteLine("该多维数组的维数为:{0}",rank);
int rlength = muarr.GetLength(1);
Console.WriteLine("该多维数组的第二维有{0}个元素",rlength);
Console.WriteLine("开始遍历多维数组");
Console.WriteLine("----------------------------------");
int wei = 0;
for (int i = 0; i < muarr.GetLength(0);i++ )
{
    for (int js1 = 0; js1< muarr.GetLength(1); js1++)
    {
        for(int js2 = 0;js2<muarr.GetLength(2);js2++)
        {
             Console.WriteLine("最低维度{0}的值为{1}",wei,muarr[i,js1,js2]);
        }
        ++wei;
    }
}
```

### 交错数组

本质是一维数组，数组的数组

与多维数组的区别：交错数组每一行的长度是可以不一样的，而多维数组每行长度都一样

**声明**

```C#
int [][] scores;
```

**初始化**

```C#
int[][] scores = new int[2][]{new int[]{92,93,94},new int[]{85,66,87,88}};
```

**取值**

```C#
scores[0][2]
```

**实例**

当然了，这个例子比较理想，一般交错数组每行长度都不一样

```C#
using System;

namespace ArrayApplication
{
    class MyArray
    {
        static void Main(string[] args)
        {
            /* 一个由 5 个整型数组组成的交错数组 */
            int[][] a = new int[][]{new int[]{0,0},new int[]{1,2},
            new int[]{2,4},new int[]{ 3, 6 }, new int[]{ 4, 8 } };

            int i, j;

            /* 输出数组中每个元素的值 */
            for (i = 0; i < 5; i++)
            {
                for (j = 0; j < 2; j++)
                {
                    Console.WriteLine("a[{0}][{1}] = {2}", i, j, a[i][j]);
                }
            }
           Console.ReadKey();
        }
    }
}

输出
a[0][0] = 0
a[0][1] = 0
a[1][0] = 1
a[1][1] = 2
a[2][0] = 2
a[2][1] = 4
a[3][0] = 3
a[3][1] = 6
a[4][0] = 4
a[4][1] = 8
```

### 数组作为函数参数

一维数组

```C#
using System;

namespace ArrayApplication
{
   class MyArray
   {
      double getAverage(int[] arr, int size)
      {
         int i;
         double avg;
         int sum = 0;

         for (i = 0; i < size; ++i)
         {
            sum += arr[i];
         }

         avg = (double)sum / size;
         return avg;
      }
      static void Main(string[] args)
      {
         MyArray app = new MyArray();
         /* 一个带有 5 个元素的 int 数组 */
         int [] balance = new int[]{1000, 2, 3, 17, 50};
         double avg;

         /* 传递数组的指针作为参数 */
         avg = app.getAverage(balance, 5 ) ;

         /* 输出返回值 */
         Console.WriteLine( "平均值是： {0} ", avg );
         Console.ReadKey();
      }
   }
}
```

交错数组

```C#
using System;

namespace ArrayApplication
{
    class Program
    {
        static void Main(String[] args)
        {
            int[][] array = new int[3][];
            array[0] = new int[] { 0, 1 };
            array[1] = new int[] { 2, 3, 4 };
            array[2] = new int[] { 5, 6, 7, 8 };
            Program pro = new Program();
            int summ=pro.getArea(array);   //函数名作为参数
            Console.WriteLine("{0}", summ);
        }

        public int  getArea(int[][] array)
        {
            int sum=0;
            for (int j = 0; j < array.Length;j++ )
                foreach (int i in array[j])
                    sum += i;
            return sum;
        }
    }
}
```

### params 参数数组

参数数组通常用于传递未知数量的参数给函数。

```C#
public 返回类型 方法名称( params 类型名称[] 数组名称 )
复制代码
using System;

namespace ArrayApplication
{
   class ParamArray
   {
      public int AddElements(params int[] arr)
      {
         int sum = 0;
         foreach (int i in arr)
         {
            sum += i;
         }
         return sum;
      }
   }
     
   class TestClass
   {
      static void Main(string[] args)
      {
         ParamArray app = new ParamArray();
         int sum = app.AddElements(512, 720, 250, 567, 889);
         Console.WriteLine("总和是： {0}", sum);
         Console.ReadKey();
      }
   }
}
```

注意：

1. 带 params 关键字的参数类型必须是一维数组，不能使用在多维数组上；
2. 不允许和 ref、out 同时使用；
3. 带 params 关键字的参数必须是最后一个参数，并且在方法声明中只允许一个 params 关键字。
4. 不能仅使用 params 来使用重载方法。
5. 没有 params 关键字的方法的优先级高于带有params关键字的方法的优先级

### Array类

Array 类是 C# 中所有数组的基类，它是在 System 命名空间中定义。Array 类提供了各种用于数组的属性和方法。

**属性**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210908100016.png)



**方法**

使用方式：Array.方法名()

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210908100048.png)



## 字符串（String）

string 关键字是 System.String 类的别名，用来声明字符串变量。**引用类型**。

**创建String对象**

- 通过给 String 变量指定一个字符串
- 通过使用 String 类构造函数
- 通过使用字符串串联运算符（ + ）
- 通过检索属性或调用一个返回字符串的方法
- 通过格式化方法来转换一个值或对象为它的字符串表示形式

```C#
using System;

namespace StringApplication
{
    class Program
    {
        static void Main(string[] args)
        {
           // 字符串常量""
            string fname, lname;
            fname = "Rowan";
            lname = "Atkinson";
            
            // 字符串连接+
            string fullname = fname + lname;
            Console.WriteLine("Full Name: {0}", fullname);

            //通过使用 string 构造函数
            char[] letters = { 'H', 'e', 'l', 'l','o' };
            string greetings = new string(letters);
            Console.WriteLine("Greetings: {0}", greetings);

            //方法返回字符串
            string[] sarray = { "Hello", "From", "Tutorials", "Point" };
            string message = String.Join(" ", sarray);
            Console.WriteLine("Message: {0}", message);

            //用于转化值的格式化方法
            DateTime waiting = new DateTime(2012, 10, 10, 17, 58, 1);
            string chat = String.Format("Message sent at {0:t} on {0:D}",
            waiting);
            Console.WriteLine("Message: {0}", chat);
            Console.ReadKey() ;
        }
    }
}

输出
Full Name: RowanAtkinson
Greetings: Hello
Message: Hello From Tutorials Point
Message: Message sent at 17:58 on Wednesday, 10 October 2012
```

**String类的属性**



![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210908100129.png)



**String类的方法**

```c#
1.比较字符串
String.Compare(str1, str2) == 0

2.包含字符串
str.Contains("test")

3.截取子字符串
str.Substring(23)

4.连接字符串
String.Join("\n", starray) // starray是个字符串数组
```

## 结构体（Struct）

可以存储各种数据类型的相关数据，**struct** 关键字用于创建结构体，结构体是**值类型**数据结构。

**定义**

可以在内部定义属性和方法，但是不能赋初值

```C#
struct Books
{
    public string title;
    public string author;
    public string subject;
    public int book_id;
};
```

**声明**

```C#
// 方式1
Books Book1;

// 方式2：new
Books Book1 = new Books();
```

**初始化**

1. 声明结构体后，通过 `实例.属性名 = 属性值` 来初始化
2. 在结构体内部定义一个初始化函数，接收参数，为属性赋值，声明后的实例调用该函数完成初始化

**特点**

- 结构可带有方法、字段、索引、属性、运算符方法和事件。
- 结构可定义构造函数，但不能定义析构函数。但是，您不能为结构定义无参构造函数。无参构造函数(默认)是自动定义的，且不能被改变。
- 与类不同，结构**不能继承**其他的结构或类。
- 结构不能作为其他结构或类的基础结构。
- 结构可实现一个或多个接口。
- 结构成员不能指定为 abstract、virtual 或 protected。
- 当您使用 New 操作符创建一个结构对象时，会调用适当的构造函数来创建结构。与类不同，结构**可以不使用 New 操作符**即可被实例化。
- 如果不使用 New 操作符，只有在所有的字段都被初始化之后，字段才被赋值，对象才被使用。

**类和结构体的比较**

- 类是引用类型，结构是值类型。
- 类可以继承，结构不支持继承。
- 结构不能声明默认的构造函数。
- 结构体中声明的字段不能赋予初值，类可以。

当我们描述一个轻量级对象的时候，结构可提高效率，成本更低。

因为类的对象是存储在堆空间中，结构存储在栈中。堆空间大，但访问速度较慢，栈空间小，访问速度相对更快。

## 枚举（Enum）

枚举是一组命名整型常量。枚举类型是使用 enum 关键字声明的，**值类型**。

**定义**

```C#
enum <enum_name>
{
    enumeration list
};
```

**使用**

```C#
(int)enum_name.list
using System;

public class EnumTest
{
    enum Day { Sun, Mon, Tue, Wed, Thu, Fri, Sat };

    static void Main()
    {
        int x = (int)Day.Sun;
        int y = (int)Day.Fri;
        Console.WriteLine("Sun = {0}", x);
        Console.WriteLine("Fri = {0}", y);
    }
}
```

枚举列表中的每个符号代表一个整数值，一个比它前面的符号大的整数值。默认情况下，第一个枚举符号的值是 0。

但是，也可以自定义每个符号的值：

```C#
using System;

class Program
{
    enum Day { a = 8, b, c = 1, e, f, g };
    static void Main(string[] args)
    {
        int a = (int)Day.a;
        int e = (int)Day.e;
        Console.WriteLine(a);
        Console.WriteLine(e);
        Console.ReadKey();
    }
}

输出
8
2
```

## 类（Class）

对象是类的实例。

**类的定义**

```C#
访问修饰符 class 类名
{
    // 定义属性
    访问修饰符 数据类型 属性名 = 属性值;
    
    // 定义方法
    访问修饰符 返回类型 方法名(参数)
    { //... }
}
```

补充：

- 默认的访问修饰符，类为internal，成员为private
- 通过点（.）运算符访问类的成员

实例：

```C#
using System;
namespace BoxApplication
{
    class Box
    {
        public double length;   // 长度
        public double breadth;  // 宽度
        public double height;   // 高度
    }
    class Boxtester
    {
        static void Main(string[] args)
        {
            Box Box1 = new Box();        // 声明 Box1，类型为 Box
            Box Box2 = new Box();        // 声明 Box2，类型为 Box
            double volume = 0.0;         // 体积

            // Box1 详述
            Box1.height = 5.0;
            Box1.length = 6.0;
            Box1.breadth = 7.0;

            // Box2 详述
            Box2.height = 10.0;
            Box2.length = 12.0;
            Box2.breadth = 13.0;

            // Box1 的体积
            volume = Box1.height * Box1.length * Box1.breadth;
            Console.WriteLine("Box1 的体积： {0}", volume);

            // Box2 的体积
            volume = Box2.height * Box2.length * Box2.breadth;
            Console.WriteLine("Box2 的体积： {0}", volume);
            Console.ReadKey();
        }
    }
}
```

**封装**

实现封装：设置成员变量为私有属性，设置特定的成员方法为共有属性，通过这些共有的方法来对私有属性进行修改，从而实现封装。

**构造函数**

类的 构造函数 是类的一个特殊的成员函数，当**创建类的新对象时执行**。

构造函数的名称与类的名称完全相同，它没有任何返回类型。

```C#
using System;
namespace LineApplication
{
    class Line
    {
        private double length;   // 线条的长度
        public Line()
        {
            Console.WriteLine("对象已创建");
        }

        static void Main(string[] args)
        {
            Line line = new Line();
        }
    }
}
```

**参数化构造函数**

默认的构造函数没有任何参数。但是如果你需要一个带有参数的构造函数可以有参数，这种构造函数叫做参数化构造函数。

```C#
using System;
namespace LineApplication
{
    class Line
    {
        private double length;   // 线条的长度
        public Line(double len)  // 参数化构造函数
        {
            Console.WriteLine("对象已创建，length = {0}", len);
            length = len;
        }

        static void Main(string[] args)
        {
            Line line = new Line(10.0);
        }
    }
}
```

**隐式构造函数**

倘若在类的声明中没有显式地提供实例构造函数，在这种情况下编译器会提供一个隐式的默认构造函数，它具有以下特点：

①不带参数；

②方法体为空。

若声明了构造函数，则用声明的构造函数。

若声明的构造函数含有参数，而在实例化的时候未传入参数，则会报错。

**析构函数**

类的 析构函数 是类的一个特殊的成员函数，当类的对象超出范围时执行。

析构函数的名称是在类的名称前加上一个 **波浪形（~）** 作为前缀，它不返回值，也不带任何参数。

析构函数用于在**结束程序**（比如关闭文件、释放内存等）之前释放资源。析构函数不能继承或重载。

```C#
using System;
namespace LineApplication
{
    class Line
    {
        private double length;   // 线条的长度
        ~Line() //析构函数
        {
            Console.WriteLine("对象已删除");
        }

        static void Main(string[] args)
        {
            Line line = new Line();
        }
    }
}
```

**静态成员**

当我们声明一个类成员为静态时，意味着无论有多少个类的对象被创建，只会有一个该静态成员的副本。静态变量用于定义常量，因为它们的值可以通过直接调用类而**不需要创建类的实例来获取**，因为在创建对象之前，静态成员就已经存在了。

```C#
using System;

namespace ConsoleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            int num = AddClass.Add(2, 3);  //编译通过，无实例化直接调用方法
            Console.WriteLine(num);
        }
    }

    class AddClass
    {
        public static int Add(int x, int y)
        {
            return x + y;
        }
    }
}
```

**静态变量实例：**

```C#
using System;
namespace StaticVarApplication
{
    class StaticVar
    {
        public static int num;
        public void count()
        {
            num++;
        }
        public int getNum()
        {
            return num;
        }
    }
    class StaticTester
    {
        static void Main(string[] args)
        {
            StaticVar s1 = new StaticVar();
            StaticVar s2 = new StaticVar();
            s1.count();
            s1.count();
            s1.count();
            s2.count();
            s2.count();
            s2.count();
            Console.WriteLine("s1 的变量 num： {0}", s1.getNum());
            Console.WriteLine("s2 的变量 num： {0}", s2.getNum());
            Console.ReadKey();
        }
    }
}

输出
s1 的变量 num： 6
s2 的变量 num： 6
```

**静态函数实例：**

静态函数只能访问静态变量，静态函数在对象被创建之前就已经存在。

```C#
using System;
namespace StaticVarApplication
{
    class StaticVar
    {
        public static int num;
        public void count()
        {
            num++;
        }
        public static int getNum()
        {
            return num;
        }
    }
    class StaticTester
    {
        static void Main(string[] args)
        {
            StaticVar s = new StaticVar();
            s.count();
            s.count();
            s.count();
            Console.WriteLine("变量 num： {0}", StaticVar.getNum());
            Console.ReadKey();
        }
    }
}

输出
变量 num： 3
```

## 继承

已有的类被称为的**基类**，这个新的类被称为**派生类**。

```C#
class <派生类> : <基类>
{
 ...
}
```

- 继承的语法：`class 子类类名 : class 父类类名{ //子类类体 }`
- 继承的特点：子类拥有所有父类中所有的字段、属性和方法
- 一个类可以有多个子类，但是父类只能有一个（C#无多重继承）
- 一个类在继承另一个类的同时，还可以被其他类继承
- 在 C# 中，所有的类都直接或者间接的继承自 Object 类

**基类访问**（访问隐藏的基类成员）

基类访问表达式由关键字base后面跟一个点和成员的名称组成。例如：

```C#
Console.WriteLine("{0}",base.Field1);
```

**基类的初始化**

```C#
using System;
namespace RectangleApplication
{
    class Rectangle
    {
        // 成员变量
        protected double length;
        protected double width;
        public Rectangle(double l, double w)
        {
            length = l;
            width = w;
        }
        public double GetArea()
        {
            return length * width;
        }
        public void Display()
        {
            Console.WriteLine("长度： {0}", length);
            Console.WriteLine("宽度： {0}", width);
            Console.WriteLine("面积： {0}", GetArea());
        }
    }//end class Rectangle  
    class Tabletop : Rectangle
    {
        private double cost;
        public Tabletop(double l, double w) : base(l, w)
        { }
        public double GetCost()
        {
            double cost;
            cost = GetArea() * 70;
            return cost;
        }
        public void Display()
        {
            base.Display();
            Console.WriteLine("成本： {0}", GetCost());
        }
    }
    class ExecuteRectangle
    {
        static void Main(string[] args)
        {
            Tabletop t = new Tabletop(4.5, 7.5);
            t.Display();
            Console.ReadLine();
        }
    }
}

输出
长度： 4.5
宽度： 7.5
面积： 33.75
成本： 2362.5
```

**使用接口来实现多重继承**

```C#
using System;
namespace InheritanceApplication
{
    class Shape
    {
        public void setWidth(int w)
        {
            width = w;
        }
        public void setHeight(int h)
        {
            height = h;
        }
        protected int width;
        protected int height;
    }

    // 基类 PaintCost
    public interface PaintCost
    {
        int getCost(int area);

    }
    // 派生类
    class Rectangle : Shape, PaintCost
    {
        public int getArea()
        {
            return (width * height);
        }
        public int getCost(int area)
        {
            return area * 70;
        }
    }
    class RectangleTester
    {
        static void Main(string[] args)
        {
            Rectangle Rect = new Rectangle();
            int area;
            Rect.setWidth(5);
            Rect.setHeight(7);
            area = Rect.getArea();
            // 打印对象的面积
            Console.WriteLine("总面积： {0}", Rect.getArea());
            Console.WriteLine("油漆总成本： ${0}", Rect.getCost(area));
            Console.ReadKey();
        }
    }
}

输出
总面积： 35
油漆总成本： $2450
```

**对象可以用父类声明，却用子类实例化**

这个实例是子类的，但是因为你声明时是用父类声明的，所以你用正常的办法访问不到子类自己的成员，只能访问到从父类继承来的成员。

在子类中用 override 重写父类中用 virtual 申明的虚方法时，实例化父类调用该方法，执行时调用的是子类中重写的方法；

如果子类中用 new 覆盖父类中用 virtual 申明的虚方法时，实例化父类调用该方法，执行时调用的是父类中的虚方法；

## 多态性

多态是同一个行为具有多个不同表现形式或形态的能力，简单来说就是 **"一个接口，多个功能"**。

**静态多态性** 和 **动态多态性**

静态多态性中，函数的响应是在编译时发生的。在动态多态性中，函数的响应是在运行时发生的。

### 静态多态性

- 函数重载
- 运算符重载

**函数重载**

函数名相同，函数定义不同（如参数个数、参数类型不同），但是不可以只是返回类型不同。

**运算符重载**

可以重定义或重载 C# 中内置的运算符，通过关键字 operator 后跟运算符的符号来定义的

```C#
public static Box operator +(Box b, Box c)
{
    Box box = new Box();
    box.length = b.length + c.length;
    box.breadth = b.breadth + c.breadth;
    box.height = b.height + c.height;
    return box;
}
```

上面的函数为用户自定义的类 Box 实现了加法运算符（+）。

它把两个 Box 对象的属性相加，并返回相加后的 Box 对象。

### 动态多态性

动态多态性是通过 **抽象类** 和 **虚方法** 实现的。

**抽象类**

使用关键字 abstract 创建抽象类，用于提供接口的部分类的实现。当一个派生类继承自该抽象类时，实现即完成。抽象类包含抽象方法，抽象方法可被派生类实现。派生类具有更专业的功能。

实例：

```C#
using System;
namespace PolymorphismApplication
{
    abstract class Shape
    {
        abstract public int area();
    }
    class Rectangle : Shape
    {
        private int length;
        private int width;
        public Rectangle(int a = 0, int b = 0)
        {
            length = a;
            width = b;
        }
        public override int area()
        {
            Console.WriteLine("Rectangle 类的面积：");
            return (width * length);
        }
    }

    class RectangleTester
    {
        static void Main(string[] args)
        {
            Rectangle r = new Rectangle(10, 7);
            double a = r.area();
            Console.WriteLine("面积： {0}", a);
            Console.ReadKey();
        }
    }
}

输出
Rectangle 类的面积：
面积： 70
```

**虚方法**

虚方法是使用关键字 virtual 声明的，可以在不同的继承类中有不同的实现，对虚方法的调用是在运行时发生的。

**virtual 与 abstract**

1. virtual修饰的方法必须有实现（哪怕是仅仅添加一对大括号),而abstract修饰的方法一定不能实现。
2. virtual可以被子类重写，而abstract必须被子类重写。
3. 如果类成员被abstract修饰，则该类前必须添加abstract，因为只有抽象类才可以有抽象方法。
4. 无法创建abstract类的实例，只能被继承无法实例化。

**动态多态性**

```C#
using System;
namespace PolymorphismApplication
{
    class Shape
    {
        protected int width, height;
        public Shape(int a = 0, int b = 0)
        {
            width = a;
            height = b;
        }
        public virtual int area()
        {
            Console.WriteLine("父类的面积：");
            return 0;
        }
    }
    class Rectangle : Shape
    {
        public Rectangle(int a = 0, int b = 0) : base(a, b)
        {

        }
        public override int area()
        {
            Console.WriteLine("Rectangle 类的面积：");
            return (width * height);
        }
    }
    class Triangle : Shape
    {
        public Triangle(int a = 0, int b = 0) : base(a, b)
        {

        }
        public override int area()
        {
            Console.WriteLine("Triangle 类的面积：");
            return (width * height / 2);
        }
    }
    class Caller
    {
        public void CallArea(Shape sh)
        {
            int a;
            a = sh.area();
            Console.WriteLine("面积： {0}", a);
        }
    }
    class Tester
    {

        static void Main(string[] args)
        {
            Caller c = new Caller();
            Rectangle r = new Rectangle(10, 7);
            Triangle t = new Triangle(10, 5);
            c.CallArea(r);
            c.CallArea(t);
            Console.ReadKey();
        }
    }
}

输出
Rectangle 类的面积：
面积：70
Triangle 类的面积：
面积：25
```

## 接口

接口只包含了成员的声明，成员的定义是派生类的责任，接口提供了派生类应遵循的标准结构。

**声明**

接口使用 interface 关键字声明，它与类的声明类似。接口声明默认是 public 的。

```C#
using System;

interface IMyInterface
{
    // 接口成员
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
    }

    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }
}
```

通常接口命令以 I 字母开头

**接口继承**

```C#
using System;

interface IParentInterface
{
    void ParentInterfaceMethod();
}

interface IMyInterface : IParentInterface
{
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
        iImp.ParentInterfaceMethod();
    }

    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }

    public void ParentInterfaceMethod()
    {
        Console.WriteLine("ParentInterfaceMethod() called.");
    }
}
```

## 命名空间

命名空间的设计目的是提供一种让一组名称与其他名称分隔开的方式。

在一个命名空间中声明的类的名称与另一个命名空间中声明的相同的类的名称不冲突。

简单来说：相当于不同文件夹下的文件可以重名。

**命名空间的声明与使用**

```C#
using System;
namespace first_space
{
    class namespace_cl
    {
        public void func()
        {
            Console.WriteLine("Inside first_space");
        }
    }
}
namespace second_space
{
    class namespace_cl
    {
        public void func()
        {
            Console.WriteLine("Inside second_space");
        }
    }
}
class TestClass
{
    static void Main(string[] args)
    {
        first_space.namespace_cl fc = new first_space.namespace_cl();
        second_space.namespace_cl sc = new second_space.namespace_cl();
        fc.func();
        sc.func();
        Console.ReadKey();
    }
}

输出
Inside first_space
Inside second_space
```

**using 关键字**

1. 引入命名空间，这样在使用的时候就不用在前面加上命名空间名称

```C#
using System;
using Namespace1.SubNameSpace;
```

1. using static 指令：指定无需指定类型名称即可访问其静态成员的类型

```C#
using static System.Math;
var = PI; // 直接使用System.Math.PI
```

1. 起别名

```C#
using Project = PC.MyCompany.Project;
```

1. using语句：将实例与代码绑定

```C#
using (Font font3 = new Font("Arial", 10.0f),
            font4 = new Font("Arial", 10.0f))
{
    // Use font3 and font4.
}
```

**嵌套命名空间**

使用点（.）运算符访问嵌套的命名空间的成员

```C#
using System;
using SomeNameSpace;
using SomeNameSpace.Nested;

namespace SomeNameSpace
{
    public class MyClass
    {
        static void Main()
        {
            Console.WriteLine("In SomeNameSpace");
            Nested.NestedNameSpaceClass.SayHello();
        }
    }

    // 内嵌命名空间
    namespace Nested
    {
        public class NestedNameSpaceClass
        {
            public static void SayHello()
            {
                Console.WriteLine("In Nested");
            }
        }
    }
}
```

## 预处理器指令

预处理器指令指导编译器在实际编译开始之前对信息进行预处理。

所有的预处理器指令都是以 # 开始，不以分号（;）结束

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210908100545.png)



\#define 预处理器指令创建符号常量。使用符号作为传递给 #if 指令的表达式，表达式将返回 true。

```C#
#define PI
using System;
namespace PreprocessorDAppl
{
    class Program
    {
        static void Main(string[] args)
        {
#if (PI)
            Console.WriteLine("PI is defined");
#else
            Console.WriteLine("PI is not defined");
#endif
            Console.ReadKey();
        }
    }
}
```

