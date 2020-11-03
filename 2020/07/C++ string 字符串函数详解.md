---
title: "C++ String 字符串函数详解"
date: 2020-07-23T14:56:57+08:00
draft: false
tags:
  - C++
  - 字符串
  - 函数
topics:
  - 
---

## 运算符重载

1. \+ 和 +=：连接字符串
2. =：字符串赋值
3. \>、>=、< 和 <=：字符串比较（例如a < b, aa < ab）
4. ==、!=：比较字符串
5. <<、>>：输出、输入字符串

注意：使用重载的运算符 + 时，必须保证前两个操作数至少有一个为 string 类型。例如，下面的写法是不合法的：

```cpp
#include <iostream>
#include <string>
int main()
{
    string str = "cat";
    cout << "apple" + "boy" + str; // illegal!
    return 0;
}
```

## 查找

```cpp
string str;
cin >> str;

str.find("ab");//返回字符串 ab 在 str 的位置
str.find("ab", 2);//在 str[2]~str[n-1] 范围内查找并返回字符串 ab 在 str 的位置
str.rfind("ab", 2);//在 str[0]~str[2] 范围内查找并返回字符串 ab 在 str 的位置

//first 系列函数
str.find_first_of("apple");//返回 apple 中任何一个字符首次在 str 中出现的位置
str.find_first_of("apple", 2);//返回 apple 中任何一个字符首次在 str[2]~str[n-1] 范围中出现的位置
str.find_first_not_of("apple");//返回除 apple 以外的任何一个字符在 str 中首次出现的位置
str.find_first_not_of("apple", 2);//返回除 apple 以外的任何一个字符在 str[2]~str[n-1] 范围中首次出现的位置

//last 系列函数
str.find_last_of("apple");//返回 apple 中任何一个字符最后一次在 str 中出现的位置
str.find_last_of("apple", 2);//返回 apple 中任何一个字符最后一次在 str[0]~str[2] 范围中出现的位置
str.find_last_not_of("apple");//返回除 apple 以外的任何一个字符在 str 中最后一次出现的位置
str.find_last_not_of("apple", 2);//返回除 apple 以外的任何一个字符在 str[0]~str[2] 范围中最后一次出现的位置

//以上函数如果没有找到，均返回string::npos
cout << string::npos;
```

## 子串

```cpp
str.substr(3); //返回 [3] 及以后的子串
str.substr(2, 4); //返回 str[2]~str[2+(4-1)] 子串(即从[2]开始4个字符组成的字符串)
```

## 替换

```cpp
str.replace(2, 4, "sz");//返回把 [2]~[2+(4-1)] 的内容替换为 "sz" 后的新字符串
str.replace(2, 4, "abcd", 3);//返回把 [2]~[2+(4-1)] 的内容替换为 "abcd" 的前3个字符后的新字符串
```

## 插入

```cpp
str.insert(2, "sz");//从 [2] 位置开始添加字符串 "sz"，并返回形成的新字符串
str.insert(2, "abcd", 3);//从 [2] 位置开始添加字符串 "abcd" 的前 3 个字符，并返回形成的新字符串
str.insert(2, "abcd", 1, 3);//从 [2] 位置开始添加字符串 "abcd" 的前 [2]~[2+(3-1)] 个字符，并返回形成的新字符串
```

## 追加

除了用重载的 `+` 操作符，还可以使用函数来完成。

```cpp
str.push_back('a');//在 str 末尾添加字符'a'
str.append("abc");//在 str 末尾添加字符串"abc"
```

## 删除

```cpp
str.erase(3);//删除 [3] 及以后的字符，并返回新字符串
str.erase(3, 5);//删除从 [3] 开始的 5 个字符，并返回新字符串
```

## 交换

```cpp
str1.swap(str2);//把 str1 与 str2 交换
```

## 其他

```cpp
str.size();//返回字符串长度
str.length();//返回字符串长度
str.empty();//检查 str 是否为空，为空返回 1，否则返回 0
str[n];//存取 str 第 n + 1 个字符
str.at(n);//存取 str 第 n + 1 个字符（如果溢出会抛出异常）
```

## 实例

### 查找给定字符串并把相应子串替换为另一给定字符串

string 并没有提供这样的函数，所以我们自己来实现。由于给定字符串可能出现多次，所以需要用到 `find()` 成员函数的第二个参数，每次查找之后，从找到位置往后继续搜索。直接看代码（这个函数返回替换的次数，如果返回值是 0 说明没有替换）：

```cpp
int str_replace(string &str, const string &src, const string &dest)
{
    int counter = 0;
    string::size_type pos = 0;
    while ((pos = str.find(src, pos)) != string::npos) {
        str.replace(pos, src.size(), dest);
        ++counter;
        pos += dest.size();
    }
    return counter;
}
```

### 从给定字符串中删除一给定字串

方法和上面相似，内部使用 `erase()` 完成。代码：

```cpp
int str_erase(string &str, const string src)
{
    int counter = 0;
    string::size_type pos = 0;
    while ((pos = str.find(src, pos)) != string::npos) {
        str.erase(pos, src.size());
        ++counter;
    }
    return counter;
}
```

### 给定一字符串和一字符集，从字符串剔除字符集中的任意字符

```cpp
int str_wash(string &str, const string src)
{
    int counter = 0;
    string::size_type pos = 0;
    while ((pos = str.find_first_of(src, pos)) != string::npos) {
        str.erase(pos, 1);
        ++counter;
    }
    return counter;
}
```