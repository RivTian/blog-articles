---
title: "Sunday算法"
date: 2020-05-03T14:46:33+08:00
draft: false
tags:
  - 字符串
topics:
  - 
---

## 背景

我们第一次接触字符串匹配，想到的肯定是直接用2个循环来遍历，这样代码虽然简单，但时间复杂度却是$Ω(m*n)$，也就是达到了字符串匹配效率的下限。于是后来人经过研究，构造出了著名的KMP算法（Knuth-Morris-Pratt算法），让我们的时间复杂度降低到了$O(m+n)$，但现代文字处理器中，却很少使用KMP算法来做字符串匹配，因为还是太慢了。现在主流的算法是BM算法（Boyer-Moore算法），成功让平均时间复杂度降低到了$O(m/n)$，而Sunday算法，则是对BM算法的进一步小幅优化。

KMP算法很多人看了一遍遍以后，对`next[n]`数组的理解还是有点困难（包括笔者），写代码的时候总是容易变成这种情况（/捂脸.jpg）：



> （切到网页）：马冬梅
>
> （切到编译器）：马什么梅
>
> （切到网页）：马冬梅
>
> （切到编译器）：马冬什么
>
> （切到网页）：马冬梅
>
> （切到编译器）：什么冬梅



而Sunday算法，理解起来则是非常容易，同时极低的时间复杂度，让Sunday算法成为了我目前最常使用的字符串匹配算法

Sunday 算法是 Daniel M.Sunday 于 1990 年提出的字符串模式匹配。其效率在匹配随机的字符串时比其他匹配算法还要更快。Sunday 算法的实现可比 KMP，BM 的实现容易太多。

平均性能的时间复杂度为$O(n)$；
最差情况的时间复杂度为$O(n * m)$。

## 算法过程

Sunday算法和BM算法稍有不同的是，Sunday算法是从前往后匹配，在匹配失败时关注的是主串中参加匹配的最末位字符的下一位字符。

- 如果该字符没有在模式串中出现则直接跳过，即移动位数 = 模式串长度 + 1；
- 否则，其移动位数 = 模式串长度 - 该字符最右出现的位置(以0开始) = 模式串中该字符最右出现的位置到尾部的距离 + 1。



现在举个例子讲解Sunday算法



假定主串为 "HERE IS A SIMPLE EXAMPLE"，模式串为 "EXAMPLE"。

（1）

![](https://resource.ethsonliu.com/image/20190815_01.png)

从头部开始比较，发现不匹配。则 Sunday 算法要求如下：找到主串中位于模式串后面的第一个字符，即红色箭头所指的 "空格"，再在模式串中从后往前找 "空格"，没有找到，则直接把模式串移到 "空格" 的后面。

（2）

![](https://resource.ethsonliu.com/image/20190815_02.png)

依旧从头部开始比较，发现不匹配。找到主串中位于模式串后面的第一个字符 L，模式串中存在 L，则移动模式串使两个 L 对齐。


（3）

![](https://resource.ethsonliu.com/image/20190815_03.png)

找到匹配。

## 完整代码

```c++
#include <iostream>
#include <string>

#define MAX_CHAR 256
#define MAX_LENGTH 1000

using namespace std;

void GetNext(string & p, int & m, int next[])
{
	for (int i = 0; i < MAX_CHAR; i++)
		next[i] = -1;
	for (int i = 0; i < m; i++)
		next[p[i]] = i;
}

void Sunday(string & s, int & n, string & p, int & m)
{
	int next[MAX_CHAR];
	GetNext(p, m, next);

	int j;  // s 的下标
	int k;  // p 的下标
	int i = 0;
	bool is_find = false;
	while (i <= n - m)
	{
		j = i;
		k = 0;
		while (j < n && k < m && s[j] == p[k])
			j++, k++;

		if (k == m)
		{
			cout << "在主串下标 " << i << " 处找到匹配\n";
			is_find = true;
		}

		if (i + m < n)
			i += (m - next[s[i + m]]);
		else
			break;
	}

	if (!is_find)
		cout << "未找到匹配\n";
}

int main()
{
	string s, p;
	int n, m;

	while (cin >> s >> p)
	{
		n = s.size();
		m = p.size();
		Sunday(s, n, p, m);
		cout << endl;
	}

	return 0;
}
```

数据测试如下：

```cpp
here#is#a#example
example
在主串下标 10 处找到匹配

aaa
a
在主串下标 0 处找到匹配
在主串下标 1 处找到匹配
在主串下标 2 处找到匹配

aaa
b
未找到匹配
```

[附小吴师兄的动画讲解链接](https://blog.csdn.net/kexuanxiu1163/article/details/98557240)



## Sunday算法的缺点

看上去简单高效非常美好的Sunday算法，也有一些缺点。因为Sunday算法的核心依赖于move数组，而move数组的值则取决于模式串，那么就可能存在模式串构造出很差的move数组。例如下面一个例子

```cpp
主串：baaaabaaaabaaaabaaaa

模式串：aaaaa
```

这个模式串使得move[a]的值为1，即每次匹配失败时，只让模式串向后移动一位再进行匹配。这样就让Sunday算法的时间复杂度飙升到了`O(m*n)`，也就是字符串匹配的最坏情况,在这种情况下效率就明显低于KMP等算法了 例如：[HDU1686](http://acm.hdu.edu.cn/showproblem.php?pid=1686)

## 总结

当然，也不能因为存在最坏的情况就直接否定Sunday算法，大多数情况下，Sunday依然是一个简单高效的算法，值得我们熟练学习掌握。