---
title: "神奇的字符串匹配：扩展KMP算法"
date: 2020-10-05T15:56:37+08:00
draft: false
tags:
  - 字符串
  - KMP
topics:
  - 
---

## 引言

一个算是冷门的算法（在竞赛上），不过其算法思想值得深究。

## 前置知识

1. **kmp**的算法**思想**，具体可以参考 → [Click here](https://www.cnblogs.com/RioTian/p/12686870.html)
2. **trie树（字典树）**。

## 正文

**问题定义：**给定两个字符串 S 和 T（长度分别为 n 和 m），下标从 0 开始，定义 `extend[i]` 等于 `S[i]...S[n-1]` 与 T 的最长相同前缀的长度，求出所有的 `extend[i]`。举个例子，看下表：

|     i     |  0   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |
| :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|     S     |  a   |  a   |  a   |  a   |  a   |  b   |  b   |  b   |
|     T     |  a   |  a   |  a   |  a   |  a   |  c   |      |      |
| extend[i] |  5   |  4   |  3   |  2   |  1   |  0   |  0   |  0   |

为什么说这是 KMP 算法的扩展呢？显然，如果在 S 的若干个位置 i 有 `extend[i]` 等于 m，则可知在 S 中找到了匹配串 T，并且匹配的首位置是 i，这就是标准的KMP问题。但是，扩展 KMP 算法可以找到 S 中所有 T 的匹配。接下来具体介绍下这个算法。

### 算法流程

（1）

![](https://gitee.com//riotian/blogimage/raw/master/img/20201005135842.png)

如上图，假设当前遍历到 S 串位置 i，即 `extend[0]...extend[i - 1]`这 i 个位置的值已经计算得到。设置两个变量，a 和 p。p 代表以 a 为起始位置的字符匹配成功的最右边界，也就是 "p = 最后一个匹配成功位置 + 1"。相较于字符串 T 得出，**S[a...p) 等于 T[0...p-a)**。

再定义一个辅助数组 `int next[]`，其中 `next[i]` 含义为：`T[i]...T[m - 1]` 与 T 的最长相同前缀长度，m 为串 T 的长度。举个例子：

|    i    |  0   |  1   |  2   |  3   |  4   |  5   |
| :-----: | :--: | :--: | :--: | :--: | :--: | :--: |
|    T    |  a   |  a   |  a   |  a   |  a   |  c   |
| next[i] |  6   |  4   |  3   |  2   |  1   |  0   |

（2）

![](https://gitee.com//riotian/blogimage/raw/master/img/20201005135840.png)

`S[i]` 对应 `T[i - a]`，如果 `i + next[i - a] < p`，如上图，三个椭圆长度相同，根据 next 数组的定义，此时 `extend[i] = next[i - a]`。

（3）

![](https://gitee.com//riotian/blogimage/raw/master/img/20201005135850.png)

如果 `i + next[i - a] == p` 呢？如上图，三个椭圆都是完全相同的，`S[p] != T[p - a]` 且 `T[p - i] != T[p - a]`，但 `S[p]` 有可能等于 `T[p - i]`，所以我们可以直接从 `S[p]` 与 `T[p - i]` 开始往后匹配，加快了速度。

（4）

![](https://gitee.com//riotian/blogimage/raw/master/img/20201005135854.png)

如果 `i + next[i - a] > p` 呢？那说明 `S[i...p)` 与 `T[i-a...p-a)` 相同，注意到 `S[p] != T[p - a]` 且 `T[p - i]  == T[p - a]`，也就是说 `S[p] != T[p - i]`，所以就没有继续往下判断的必要了，我们可以直接将 `extend[i]` 赋值为 `p - i`。

（5）最后，就是求解 next 数组。我们再来看下 `next[i]` 与 `extend[i]` 的定义：

- **next[i]**： `T[i]...T[m - 1]` 与 T 的最长相同前缀长度；
- **extend[i]**： `S[i]...S[n - 1]` 与 T 的最长相同前缀长度。

恍然大悟，求解 `next[i]` 的过程不就是 T 自己和自己的一个匹配过程嘛，下面直接看代码。

### 代码

```c++
#include<iostream>
#include<algorithm>

using namespace std;

//求解 T 中 next[]，注释参考 GetExtend()
void GetNext(string& T, int& m, int next[]) {
    int a = 0, p = 0;
    next[0] = m;
    
    for (int i = 1; i < m; ++i)
        if (i >= p || i + next[i - a] >= p) {
            if (i >= p)
                p = i;

            while (p < m && T[p] == T[p - i])
                p++;

            next[i] = p - i;
            a = i;
        }
        else
            next[i] = next[i - a];
}

// 求解 extend[]
void GetExtend(string& S, int& n, string& T, int& m, int extend[], int next[]) {
    int a = 0, p = 0;
    GetNext(T, m, next);

    for (int i = 0; i < n; ++i) {
        // i >= p 的作用：举个典型例子，S 和 T 无一字符相同
        if (i >= p || i + next[i - a] >= p) {
            if (i >= p)
                p = i;

            while (p < n && p - i < m && S[p] == T[p - i])
                p++;

            extend[i] = p - i;
            a = i;
        }
        else
            extend[i] = next[i - a];
    }
}

int main() {
    int next[100], extend[100];
    string S, T;
    int n, m;

    while (cin >> S >> T) {
        int n = S.length();
        int m = T.length();
        GetExtend(S, n, T, m, extend, next);

        // 打印 next
        cout << "next:   ";
        for (int i = 0; i < m; ++i)
            cout << next[i] << " ";

        // 打印 extend
        cout << "\nextend: ";
        for (int i = 0; i < n; i++)
            cout << extend[i] << " ";

        cout << endl << endl;
    }
    return 0;
}
```

数据测试如下：

```plaintext
aaaaabbb
aaaaac
next:   6 4 3 2 1 0
extend: 5 4 3 2 1 0 0 0

abc
def
next:   3 0 0
extend: 0 0 0
```

## 参考

OI Wiki：https://oi-wiki.org/string/z-func/

拓展kmp算法总结：https://blog.csdn.net/dyx404514/article/details/41831947