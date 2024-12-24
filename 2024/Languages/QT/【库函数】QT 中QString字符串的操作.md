
#Qt 

QString是QT提供的字符串类，相应的也就提供了很多很方便对字符串的处理方法。这里把这些对字符串的操作做一个整理和总结。

### **1. 将一个字符串追加到另一个字符串的末尾**

```cpp
QString str1 = "hello ";
QString str2 = "world";
str1.append(str2);                   // str1 = "hello world"
str1.append(" !");                   // str1 = "hello world !"
// 对字符串直接 + 另一个字符串也可以实现，但是不对原本的字符串进行操作
QString str3 = str1 + str2 + " !";   // str3 = "hello world !"   str1 = "hello "
```

### 2. 从字符串中查找某个字符串

```cpp
QString x = "sticky question";
QString y = "sti";
x.indexOf(y);              // 返回0
// 函数indexOf()会返回要查找的字符串在字符串中第一次出现的位置
// 如果没有要查找的字符串，返回-1
```

### 3. 用某个字符填满字符串

```cpp
QString str = "Hello";
str.fill('x');             // str == "xxxxx"
str.fill('A', 2);          // str == "AA"
```

### 4. 判断字符串是否为空

```cpp
QString().isEmpty();            // 返回 true
QString("").isEmpty();          // 返回 true
QString(" ").isEmpty();         // 返回 false
QString("abc").isEmpty();       // 返回 false
```

### 5. 判断字符串是否存在

```cpp
QString().isNull();             // 返回 true
QString("").isNull();           // 返回 false
QString("abc").isNull();        // 返回 false
```

### 6. 从左向右截取字符串

```cpp
QString str = "Hello World !";
QString str1 = str.left(5);     // str1 = "Hello"
```

### 7. 从中间截取字符串

```cpp
QString str = "I love C++!";
QString str1 = str.mid(2, 4);       // str1 == "love"
QString str2 = str.mid(2);          // str2 == "love C++!"
```

### 8. 截取字符串中最右边几个字符

```cpp
QString str = "I love C++!";
QString str1 = str.right(4);        // str = I love C++! str1 = "C++!" 
```

###  9. 删除字符串中的最后几个字符

```cpp
QString str = "Hello World !";
str.chop(8);                        // str = "Hello" 
```

### 10. 删除字符串中间某个字符

```cpp
QString str = "Hello World!";
str.remove(5, 6);                   // str = "Hello!"
```

### 11. 指定位置插入字符串

```cpp
QString str = "Hello!";
str.insert(5, QString(" World"));   // str = "Hello World!"
```

### 12. 用几个字符替换字符串中的几个字符

```cpp
QString x = "Say yes!";
QString y = "no";
x.replace(4, 3, y);                 // x = "Say no!"
x.replace("yes", "no");             // x = "Say no!"
```

### 13.字符串补零到指定位

```cpp
QString str = "A6";
//如果要把str补全到8位，空位用0补全
QString str1 = QString("%1").arg(str, 8, QLatin1Char('0'));    //str1 = "000000A6"
//arg里第一个参数是要补全的字符串，第二个参数是要补全到的位数，第三个参数是用什么字符补全，可以不是0的其他字符
```

### 14.整型十进制转为十六进制字符串并补零到指定位

```cpp
int n = 66;
//如果要把n转换为十六进制并补全到8位
QString str = QString("%1").arg(n, 8, 16, QLatin1Char('0'));   // str = "00000042"
//arg里第一个参数是十进制整型的数，第二个参数是要补全到的位数，第三个参数是要转换的进制，可以是十六进制也可以是十进制
//第四个参数是用什么字符补全，可以不是0的其他字符
```

### 15. 以某个字符切割字符串

```cpp
QString csv = "forename,middlename,surname,phone";
QString path = "/usr/local/bin/myapp";
 
//section()会把字符串以第一个参数的符号切割分成数个字符串，后面两个参数是得到的字符串的开始和结束的位置
QString str;
str = csv.section(',', 2, 2);       // str = "surname"
str = path.section('/', 3, 3);      // str = "bin"
str = path.section('/', 3, 4);      // str = "bin/myapp"
str = path.section('/', 3, 3, QString::SectionSkipEmpty);      // str = "myapp"
```

### 16. 以某个字符切割字符串另一种方法

```cpp
QString str = "a,,b,c";
 
QStringList list1 = str.split(',');
// list1: [ "a", "", "b", "c" ]
 
QStringList list2 = str.split(',', QString::SkipEmptyParts);
// list2: [ "a", "b", "c" ]
```

###  17. 检查字符串是否以某个字符串开头或结尾

```cpp
QString str = "http://www.baidu.com";
str.startsWith("http:");            // 返回true
str.endsWith("cn");                 // 返回false
//这个方法比left()和right()要简单快速，通常用来检查路径或者网址
```

### 18. 比较两个字符串是否相等

```cpp
QString str1 = "xxx";
QString str2 = "XXX";
 
//localeAwareCompare()会比较两个参数的大小，如果str1小于str2,返回小于0的数，等于返回0，大于返回大于0的数
QString::localeAwareCompare(str1, str2)          // 返回 -1
 
//localeAwareCompare()的比较区分大小写，想要不区分大小写，可以使用toLower()或toUpper()使字符串全变成小写或大写
if(QString::localeAwareCompare(str1.toUpper(), str2) == 0)
{
    qDebug() << "str1和str2相等";                // 返回0，输出：str1和str2相等
}
```
