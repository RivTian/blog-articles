本文主要记录了C/C++预处理指令，常见的预处理指令如下：

#### #空指令，无任何效果

#### #include包含一个源代码文件

#### #define定义宏

#### #undef取消已定义的宏

#### #if如果给定条件为真，则编译下面代码

#### #ifdef如果宏已经定义，则编译下面代码

#### #ifndef如果宏没有定义，则编译下面代码

#### #elif如果前面的#if给定条件不为真，当前条件为真，则编译下面代码

#### #endif结束一个#if……#else条件编译块

#### #error停止编译并显示错误信息

## 什么是预处理指令?

预处理指令是以#号开头的代码行。#号必须是该行除了任何空白字符外的第一个字符。#后是指令关键字，在关键字和#号之间允许存在任意个数的空白字符。整行语句构成了一条预处理指令，该指令将在编译器进行编译之前对源代码做某些转换。

以前没有在意的学者注意了,预处理指令是在编译器进行编译之前进行的操作.预处理过程扫描源代码，对其进行初步的转换，产生新的源代码提供给编译器。可见预处理过程先于编译器对源代码进行处理。在很多编程语言中，并没有任何内在的机制来完成如下一些功能：在编译时包含其他源文件、定义宏、根据条件决定编译时是否包含某些代码(防止重复包含某些文件)。要完成这些工作，就需要使用预处理程序。尽管在目前绝大多数编译器都包含了预处理程序，但通常认为它们是独立于编译器的。预处理过程读入源代码，检查包含预处理指令的语句和宏定义，并对源代码进行响应的转换。预处理过程还会删除程序中的注释和多余的空白字符。

## #include包含一个源代码文件

这个预处理指令，我想是见得最多的一个，简单说一下，第一种方法是用尖括号把头文件括起来。这种格式告诉预处理程序在编译器自带的或外部库的头文件中搜索被包含的头文件。第二种方法是用双引号把头文件括起来。这种格式告诉预处理程序在当前被编译的应用程序的源代码文件中搜索被包含的头文件，如果找不到，再搜索编译器自带的头文件。采用两种不同包含格式的理由在于，编译器是安装在公共子目录下的，而被编译的应用程序是在它们自己的私有子目录下的。一个应用程序既包含编译器提供的公共头文件，也包含自定义的私有头文件。采用两种不同的包含格式使得编译器能够在很多头文件中区别出一组公共的头文件。

## #define定义宏

有关#define这个宏定义，在C语言中使用的很多，因为#define存在一些不足，C++强调使用const来定义常量。宏定义了一个代表特定内容的标识符。预处理过程会把源代码中出现的宏标识符替换成宏定义时的值。记住仅仅是进行标识符的替换。

下面列举一些#define的使用：

- 用#define实现求最大值和最小值的宏

  ```cpp
  #include <stdio.h>
  #define MAX(x,y) (((x)>(y))?(x):(y))
  #define MIN(x,y) (((x)<(y))?(x):(y))
  int main(void)
  {
  #ifdef MAX    //判断这个宏是否被定义
      printf("3 and 5 the max is:%d\n",MAX(3,5));
  #endif
  #ifdef MIN
      printf("3 and 5 the min is:%d\n",MIN(3,5));
  #endif
      return 0;
  }
  
  /*
   * (1)三元运算符要比if,else效率高
   * （2）宏的使用一定要细心，需要把参数小心的用括号括起来，
   * 因为宏只是简单的文本替换，不注意，容易引起歧义错误。
  */
  ```

- 宏定义的错误使用

```cpp
#include <stdio.h>
#define SQR(x) (x*x)
int main(void)
{
    int b=3;
#ifdef SQR//只需要宏名就可以了，不需要参数，有参数的话会警告
    printf("a = %d\n",SQR(b+2));
#endif
    return 0;
}

/*
 *首先说明，这个宏的定义是错误的。并没有实现程序中的B+2的平方
 * 预处理的时候，替换成如下的结果：b+2*b+2
 * 正确的宏定义应该是：#define SQR(x) ((x)*(x))
 * 所以，尽量使用小括号，将参数括起来。
*/
```

- 宏参数的连接

```cpp
#include <stdio.h>
#define STR(s) #s
#define CONS(a,b) (int)(a##e##b)
int main(void)
{
#ifdef STR
    printf(STR(VCK));
#endif
#ifdef CONS
    printf("\n%d\n",CONS(2,3));
#endif
    return 0;
}

/* （绝大多数是使用不到这些的，使用到的话，查看手册就可以了）
 * 第一个宏，用#把参数转化为一个字符串
 * 第二个宏，用##把2个宏参数粘合在一起，及aeb,2e3也就是2000
*/
```

- 用宏得到一个字的高位或低位的字节

```cpp
#include <stdio.h>
#define WORD_LO(xxx) ((byte)((word)(xxx) & 255))
#define WORD_HI(xxx) ((byte)((word)(xxx) >> 8))
int main(void)
{
    return 0;
}

/*
 * 一个字2个字节，获得低字节（低8位），与255（0000,0000,1111,1111）按位相与
 * 获得高字节（高8位），右移8位即可。
*/
```

- 用宏定义得到一个数组所含元素的个数

```cpp
#include <stdio.h>
#define ARR_SIZE(a) (sizeof((a))/sizeof((a[0])))
int main(void)
{
    int array[100];
#ifdef ARR_SIZE
    printf("array has %d items.\n",ARR_SIZE(array));
#endif
    return 0;
}
/*
 *总的大小除以每个类型的大小
 */
```

关于#define宏的使用，应该特别小心，尤其是含有参数计算的时候如小2示例，最保险的做法将参数用括号括起来。

## #ifdef,#ifndef,#endif…的使用

以上这些预编译指令，都是条件编译指令，也就是说，将决定那些代码被编译，而哪些不被编译。

- 示例1:

  ```cpp
  #include <stdio.h>
  #include <stdlib.h>
  #define DEBUG
  int main(void)
  {
      int i = 0;
      char c;
      while(1)
      {
          i++;
          c = getchar();
          if('\n' != c)
          {
              getchar();
          }
          if('q' == c || 'Q' == c)
          {
  #ifdef DEBUG//判断DEBUG是否被定义了
              printf("We get:%c,about to exit.\n",c);
  #endif
              break;
          }
          else
          {
              printf("i = %d",i);
  #ifdef DEBUG
              printf(",we get:%c",c);
  #endif
              printf("\n");
          }
      }
      printf("Hello World!\n");
      return 0;
  }
  
  /*#endif用于终止#if预处理指令。*/
  ```

- ifdef 和 #ifndef

```cpp
#include <stdio.h>
#define DEBUG
main()
{
#ifdef DEBUG
    printf("yes ");
#endif
#ifndef DEBUG
    printf("no ");
#endif
}
//#ifdefined等价于#ifdef;
//#if!defined等价于#ifndef
```

- 其他一些指令

```cpp
#error指令将使编译器显示一条错误信息，然后停止编译。

#line指令可以改变编译器用来指出警告和错误信息的文件号和行号。

#pragma指令没有正式的定义。编译器可以自定义其用途。典型的用法是禁止或允许某些烦人的警告信息。
```

## 小结：

预处理就是在进行编译的第一遍词法扫描和语法分析之前所作的工作。说白了，就是对源文件进行编译前，先对预处理部分进行处理，然后对处理后的代码进行编译。这样做的好处是，经过处理后的代码，将会变的很精短。