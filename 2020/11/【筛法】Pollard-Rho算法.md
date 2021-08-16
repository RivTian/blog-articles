---
title: "Pollard Rho算法"
date: 2020-11-04T20:44:05+08:00
draft: false
tags:
  - 数论
gitinfo: true
---

**Pollard-Rho** 是一个很神奇的算法，用于在 $O(n^{\frac{1}4}) $的期望时间复杂度内计算合数 n 的某个非平凡因子（除了1和它本身以外能整除它的数）。事书上给出的复杂度是 $O(\sqrt{p})$ ， p 是 n 的某个最小因子，满足 p 与 n/p 互质。虽然是随机的，但 Pollard **RhoPollard Rho** 算法在实际环境中运行的相当不错，不会被卡。

> 简单来说：**Pollard-Rho算法**是 **John Pollard**发明的一种能 <u>快速找到大整数的一个非1、非自身的因子</u> 的算法。

## 前缀知识

1. [快速乘](https://www.cnblogs.com/RioTian/p/13927813.html)
2. [Miller-Rabin 素数检验算法](https://www.cnblogs.com/RioTian/p/13927952.html)

## PollardRhoPollardRho 算法：

### 问题模型：

给一个数 $n$ ，你需要快速求出它的一个非平凡因子。

对于一个比较小的数（ $n≤10^9$ ），我们直接暴力枚举质因子就行但太大了我们就必须考虑一下随机算法了。

对于一个非常大的合数 $n≤10^{18}$ （如果可能是质数，我们可以用**Miller Rabin**判一下）我们要求 nn 的某一个非平凡因子，如果 $n$ 的质因子很多（就是约数很多）我们也可以暴力随机弄一弄，但如果是一些（像由两个差不多的而且很大的质数乘得的 $n$ ）它的非平凡因子就只有两个而且 $n$ 本身还很大，此时如果我们随机的话复杂度 $O(n)$ ，这个太难接受了，所以我们想办法优化一下。

### 1.我们求 gcd

如果我们直接随机求 $n$ 的某一个约数复杂度很高，我们可以考虑求一下 gcdgcd 。因为我们可以保证一个合数它绝对存在一个质因子小于 $√n$ ，所以在 $n$ 就存在至少 $√n$ 个数与 $n$ 有大于一的公约数。于是我们随机的可能性就提高到了 $O(\sqrt{n})$ 。

### 2.玄学随机法： $x^2+c \ \ _{x=rand(),c=rand()}$

其实网上大多都叫这种方法为生日悖论（悖论是不符合经验但符合事实的结论），有兴趣的可以去看看。但博主也不懂觉得这个方法跟我们的 **PollardRho** 的关联。在1中我们用gcd的方案，现在我们考虑有没有办法能够快速的找到这些与 $n$ 有公约数的数（或者说找到一种随机生成方法，使随机生成这类我们需要的数的概率变大一点）。

这个随机生成方法最终被 $Pollard$ 研发出来了：就是标题那个式子！ 我们定义 

$$(y=x\ x=x×x+c \ g=gcd(y−x,p))$$ 

为一次操作，我们发现 $g>1$ （就是找到我们需要的数）的几率出奇的高，根据国际上大佬的疯狂测试，在 $n$ 中找到一对 $y−x$ 使两者有公约数的概率接近 $n^{\frac14}$  ，不得不说太玄学又太优秀了！

### 判环

这种随机生成方法虽然优秀，但也有一些需要注意之处，比如有可能会生成一个环，并不断在这个环上生成以前生成过一次的数，所以我们必须写点东西来判环：

1. 我们可以让 $y$ 根据生成公式以比 $x$ 快两倍的速度向后生成，这样当$y$ 再次与 $x$ 相等时，$x$ 一定跑完了一个圈且没重复跑多少！
2. 我们可以用倍增的思想，让 $y$ 记住 $x$ 的位置，然后 $x$ 再跑当前跑过次数的一倍的次数。这样不断让 $y$ 记住 $x$ 的位置，$x$ 再往下跑，因为倍增所以当 $x$ 跑到 $y$ 时，已经跑完一个圈，并且也不会多跑太多（这个必须好好想一想，也可以看看代码如何实现的）（因为这种方法会比第一种用的更多，因为它在 $PollardRho$  二次优化时还可以起到第二个作用）（这里就给第二种的代码吧）

```cpp
inline ll rho(ll n) {
    ll x, y, c, g;
    rg i, j;
    x = y = rand();
    c = rand();    //初始化
    i = 0, j = 1;  //倍增初始化
    while (++i) {
        x = (ksc(x, x, n) + c) % n;  // x只跑一次
        if (x == y) break;           //跑完了一个环
        g = gcd(Abs(y - x), n);      //求gcd（注意绝对值）
        if (g > 1) return g;
        if (i == j) y = x, j <<= 1;  //倍增的实现
    }
}
```

## PollardRho 算法的二次优化：

我们发现其实 $PollardRho$ 算法其实复杂度不止 $O(n^{\frac14})$ ，它每一次操作都会求一次 $gcd$ ，所以复杂度会变成 $O(n^{\frac14}log)$ 我们发现这个 $log$  实在有些拖慢速度（虽然实际上也很快了）。于是一个很奇妙的优化诞生了！

**二次优化**：我们发现我们在每一次生成操作中，乘法之后所模的数就是我们的 $n$ ，而我们要求的就是 $n$ 的某一个约数！也就是说我们现在的模数并不是一个质数！而根据取模的性质：如果模数和被模的数都含有一个公约数，那么这次模运算的结果必然也会是这个公约数的倍数！！！所以如果我们将若干个 $(y−x)$ 乘到一起，因为中途模的是 $n$ ，所以如果我的若干个 $(y−x)$ 中有一个与 $n$ 有公约数，最后的结果定然也会含有这个公约数！所以我们完全可以多算几次 $(y−x)$ 的乘积在来求 $gcd$ （一般连续算127次再求一次 $gcd$ （这个应该有大佬测试过了）），这样我们的复杂度就能降一个 $log$ 级别，跑的飞快！

**对原算法的一些影响：**任何一个优化都是要考虑其对原算法的影响的。这个优化也会有一些影响：首先，因为我们会等大概127次后再去 $gcd$ ，然而我们有可能在生成时碰上一个环，我们有可能还没生成127次就跳出这个环了，这样就无法得出答案；其次，我们的 $PollardRho$ 算法实在太玄学优秀了，可能我们跑127次之后，所有 $(y−x)$ 的乘积就变成了 $n$ 的倍数（模 $n$ 意义下得到 $0$ ），所以我们不能完全就只呆板的等127次在取模。

**那怎么办呢？**（在上文就已经提到过了：倍增时我们的 $i$ 和 $j$ 会在二次优化时起到另一种作用。）没错！就是倍增！我们可以用倍增，分别在生成（1次，2次，4次，8次，16次，32次......）之后进行 $gcd$ ！这样就可以完全避免上述的两个影响，而且我们是倍增来gcd的这对我们的复杂度影响不大！！！！！然后我们看代码吧！

```cpp
inline ll rho(ll p) {  //求出p的非平凡因子
    ll x, y, z, c, g;
    rg i, j;                 //先摆出来（z用来存（y-x）的乘积）
    while (1) {              //保证一定求出一个因子来
        y = x = rand() % p;  //随机初始化
        z = 1;
        c = rand() % p;                  //初始化
        i = 0, j = 1;                    //倍增初始化
        while (++i) {                    //开始玄学生成
            x = (ksc(x, x, p) + c) % p;  //可能要用快速乘
            z = ksc(z, Abs(y - x), p);  //我们将每一次的（y-x）都累乘起来
            if (x == y || !z)
                break;  //如果跑完了环就再换一组试试（注意当z=0时，继续下去是没意义的）
            if (!(i % 127) ||
                i == j) {  //我们不仅在等127次之后gcd我们还会倍增的来gcd
                g = gcd(z, p);
                if (g > 1) return g;
                if (i == j)
                    y = x, j <<= 1;  //维护倍增正确性，并判环（一箭双雕）
            }
        }
    }
}
```

然后是两道道例题，都有点难度的。



## code1：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define rg register int
typedef long long ll;
typedef unsigned long long ull;
typedef long double lb;

int n;
ll ans, m;

inline ll qr() {  //快读
    char ch;
    while ((ch = getchar()) < '0' || ch > '9')
        ;
    ll res = ch ^ 48;
    while ((ch = getchar()) >= '0' && ch <= '9') res = res * 10 + (ch ^ 48);
    return res;
}

inline ll Abs(ll x) { return x < 0 ? -x : x; }  //取绝对值
inline ll gcd(ll x, ll y) {                     //非递归求gcd
    ll z;
    while (y) {
        z = x;
        x = y;
        y = z % y;
    }
    return x;
}

inline ll ksc(ull x, ull y, ll p) {  // O(1)快速乘（防爆long long）
    return (x * y - (ull)((lb)x / p * y) * p + p) % p;
}

inline ll ksm(ll x, ll y, ll p) {  //快速幂
    ll res = 1;
    while (y) {
        if (y & 1) res = ksc(res, x, p);
        x = ksc(x, x, p);
        y >>= 1;
    }
    return res;
}

inline bool mr(ll x, ll p) {              // mille rabin判质数
    if (ksm(x, p - 1, p) != 1) return 0;  //费马小定理
    ll y = p - 1, z;
    while (!(y & 1)) {  //二次探测
        y >>= 1;
        z = ksm(x, y, p);
        if (z != 1 && z != p - 1) return 0;
        if (z == p - 1) return 1;
    }
    return 1;
}

inline bool prime(ll x) {
    if (x < 2) return 0;  // mille rabin判质数
    if (x == 2 || x == 3 || x == 5 || x == 7 || x == 43) return 1;
    return mr(2, x) && mr(3, x) && mr(5, x) && mr(7, x) && mr(43, x);
}

inline ll rho(ll p) {  //求出p的非平凡因子
    ll x, y, z, c, g;
    rg i, j;                 //先摆出来（z用来存（y-x）的乘积）
    while (1) {              //保证一定求出一个因子来
        y = x = rand() % p;  //随机初始化
        z = 1;
        c = rand() % p;                  //初始化
        i = 0, j = 1;                    //倍增初始化
        while (++i) {                    //开始玄学生成
            x = (ksc(x, x, p) + c) % p;  //可能要用快速乘
            z = ksc(z, Abs(y - x), p);  //我们将每一次的（y-x）都累乘起来
            if (x == y || !z)
                break;  //如果跑完了环就再换一组试试（注意当z=0时，继续下去是没意义的）
            if (!(i % 127) ||
                i == j) {  //我们不仅在等127次之后gcd我们还会倍增的来gcd
                g = gcd(z, p);
                if (g > 1) return g;
                if (i == j)
                    y = x, j <<= 1;  //维护倍增正确性，并判环（一箭双雕）
            }
        }
    }
}

inline void prho(ll p) {   //不断的找他的质因子
    if (p <= ans) return;  //最优性剪纸
    if (prime(p)) {
        ans = p;
        return;
    }
    ll pi = rho(p);  //我们一次必定能求的出一个因子，所以不用while
    while (p % pi == 0) p /= pi;  //记得要除尽
    return prho(pi), prho(p);     //分开继续分解质因数
}

int main() {
    // freopen("in.txt","r",stdin);
    // ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = qr();
    srand(time(0));  //随机数生成必备！！！
    for (rg i = 1; i <= n; ++i) {
        ans = 1;
        prho((m = qr()));
        if (ans == m)
            puts("Prime");
        else
            printf("%lld\n", ans);
    }
    return 0;
}
```

## code2：

题目链接：[杭电3864 D_num](http://acm.hdu.edu.cn/showproblem.php?pid=3864)

根据这道题的答案性质，我们知道这个数在除了1以外还有三个约数，仔细想一下不难发现这类数最多只能有两个质因子，而这两个质因子分别就是它的两个约数，而第三个约数自然就是它自己了！（当然还有可能出现质数的三次方这类特殊的数，所以我们要特判一下）

所以我们的思路就是，先求出 $n$ 的一个小于它的约数，然后我们可以用这个约数算出 $n$ 的另一个约数，然后我们判一下它是不是质数的三次方这类特殊的数（是就可以直接输出答案了），不是的话我们判断这两个约数是否分别是不同的质因子，是就能输出了，不是就说明这不是 $D\_num$

```cpp
#include <bits/stdc++.h>
using namespace std;

#define ll long long
#define db double
#define rg register int
#define cut {puts("is not a D_num");continue;} //这个可以看一看

ll n;

inline ll qr() {
    char ch;
    while ((ch = getchar()) < '0' || ch > '9')
        ;
    ll res = ch ^ 48;
    while ((ch = getchar()) >= '0' && ch <= '9') res = res * 10 + (ch ^ 48);
    return res;
}

inline ll Abs(ll x) { return x < 0 ? -x : x; }
inline ll gcd(ll x, ll y) {
    ll z;
    while (y) {
        z = x, x = y, y = z % y;
    }
    return x;
}

inline ll ksm(ll x, ll y, ll p) {
    ll res = 1;
    while (y) {
        if (y & 1) res = (__int128)res * x % p;
        x = (__int128)x * x % p;
        y >>= 1;
    }
    return res;
}

inline bool mr(ll x, ll p) {
    if (ksm(x, p - 1, p) != 1) return 0;
    ll y = p - 1, z;
    while (!(y & 1)) {
        y >>= 1;
        z = ksm(x, y, p);
        if (z != 1 && z != p - 1) return 0;
        if (z == p - 1) return 1;
    }
    return 1;
}

inline bool prime(ll p) {
    if (p < 2) return 0;
    if (p == 2 || p == 3 || p == 5 || p == 7 || p == 43) return 1;
    return mr(2, p) && mr(3, p) && mr(5, p) && mr(7, p) && mr(43, p);
}

inline ll rho(ll p) {
    ll x, y, z, c, g;
    rg i, j;
    while (1) {
        y = x = rand() % p;
        z = 1, c = rand() % p;
        i = 0, j = 1;
        while (++i) {
            x = ((__int128)x * x + c) % p;
            z = (__int128)z * Abs(y - x) % p;
            if (x == y) break;
            if (i % 72 || i == j) {
                g = gcd(z, p);
                if (g > 1 && g != p) return g;
                if (i == j) y = x, j <<= 1;
            }
        }
    }
}

int main() {
    // freopen("in.txt","r",stdin);
    // ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    srand(time(0));
    ll p, q;
    while (scanf("%lld", &n) != EOF) {
        if (prime(n) || n <= 2) cut;
        p = rho(n);
        q = n / p;
        if ((__int128)p * p == q || (__int128)q * q == p)  //这题很卡细节的
        {
            printf("%lld %lld %lld\n", p, q, n);
            continue;
        }
        if (!prime(p) || !prime(q) || p == q) cut;  //注意两个相等也不行
        printf("%lld %lld %lld\n", min(p, q), max(p, q), n);
    }
    return 0;
}
```

## 参考 

https://blog.csdn.net/qq_39972971/article/details/82346390

https://ld246.com/article/1569220928268

https://zhuanlan.zhihu.com/p/267884783