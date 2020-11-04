---
title: "Miller-Rabin 素数检验算法"
date: 2020-11-04T19:59:58+08:00
draft: false
tags:
  - 数论
gitinfo: true
---

## 算法简介

**Miller-Rabin算法**，这是一个很高效的判断质数的方法，可以在用$O(logn)$  的复杂度快速判断一个数是否是质数。它运用了费马小定理和二次探测定理这两个筛质数效率极高的方法。

## 费马小定理判质数

$a^{p - 1}\ ≡\ 1\ mod\ p$

这个定理在 $p$ 为质数的时候是成立的，所以我们可以如果要判断 $p$ 是否是质数，可以 $rand$ 几个 $a$ 值然后照着这个式子来算，如果算出来不是 $1$ 那说明 $p$ 一定不是质数。

但在我们的自然数中，如果照着这个式子算出来的答案为1，也是有可能不是质数的。更有一类合数，它用费马小定理不管 rand 什么数都判不掉。这类合数称为 Carmichael数（[卡迈克尔数](https://zh.wikipedia.org/zh-cn/%E5%8D%A1%E9%82%81%E5%85%8B%E7%88%BE%E6%95%B8)），其中一个例子就是561（哇，居然这么小）。

## 二次探测定理

因为Carmichael数的存在，使得我们难以高效判断质数，所以我们还需要加入第二种判断方法使这种伪算法更优秀！而二次探测无疑就是为我们量身定制的算法，因为它要建立在同余式右边为1的基础上（而我们的费马小定理不正好满足了要求吗？）

若 $b^2≡1\ mod\ p$ 且 $p$ 为质数 $=>$ 则 $p$ 一定可以被 $b−1$ 和 $b+1$ 其中一个整除

这是二次探测定理，原理很简单，我们将上面的同余式左右都减1，根据平方差公式可以得出 $(b−1)(b+1)≡\ 0\ mod\ p$  这其实就代表着等式左边是模数的倍数，但若模数p是质数，则 $(b−1)$ 和 $(b+1)$ 必定存在一个是 $p$ 的倍数，所以要么 $b−1=p\ (b=1)$ 或者 $b+1=p\ (b=p−1)$ 如果不满足则 $p$ 一定不是质数！然后我们还可以发现若 $b=1$ 我们又可以进行新一轮二次探测！

根据这个道理，我们可以进行二次探测：因为 $a^{p−1}≡1\mod\ p$ 如果 $p−1$ 为偶数的话就可以化成： $a^{(\frac{p−1}2)^2}≡1\ mod\ p$ 这样就变成了二次探测的基本式。

```cpp
typedef long long ll;
typedef unsigned long long ull;
typedef long double lb;
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

inline bool mr(ll x, ll p) {
    if (ksm(x, p - 1, p) != 1) return 0;  //费马小定理
    ll y = p - 1, z;
    while (!(y & 1)) {  //一定要是能化成平方的形式
        y >>= 1;
        z = ksm(x, y, p);                    //计算
        if (z != 1 && z != p - 1) return 0;  //不是质数
        if (z == p - 1) return 1;  //一定要为1，才能继续二次探测
    }
    return 1;
}

inline bool prime(ll x) {
    if (x < 2) return 0;
    if (x == 2 || x == 3 || x == 5 || x == 7 || x == 43) return 1;
    return mr(2, x) && mr(3, x) && mr(5, x) && mr(7, x) && mr(43, x);
}
```

这样子加上二次探测之后，明显就能高效很多，基本上卡不了，大概要每 $10^{10}$ 个数才会出现一个判不掉的，这个概率可以说十分微小，可以忽略！

## 参考

1. https://zhuanlan.zhihu.com/p/220203643