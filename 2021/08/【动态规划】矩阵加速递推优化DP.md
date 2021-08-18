> 之前写过一篇线性代数中[矩阵加速递推](https://www.cnblogs.com/RioTian/p/14772150.html)的文章，同时在动态规划的优化中矩阵也能起很大作用。
>
> 矩阵优化 $dp$​​ 通常用于线性递推式的 $dp$​​​ 优化，<u>**能以优异的时间复杂度实现大量的状态转移（本质）**</u>。
>
> 前排感谢 [辰星凌](https://www.cnblogs.com/Xing-Ling/)桑 的转载同意！！！

## 一、题型特征

1. 类似线性递推（**划重点**，包括有向图上的递推等等）  
2. 转移次数在 $10^9$ 左右（雾）
3. 决策点较少（常数）  

---

$$
QAQ
$$

---

## 二、前置芝士

### 【前言】

首先要清楚矩阵是个什么东西，在对 $dp$​ 进行优化时通常只会用到**矩阵乘法**和**矩阵加法**，其中**矩阵乘法**最为常见。详解介绍：[Here](https://www.cnblogs.com/RioTian/p/13529296.html)

### 【运算法则】

#### **(1).【矩阵加法】**

矩阵加法的一般式：$C_{i,j}=A_{i,j} + B_{i,j}$，其中 $A,B$ 均为 $N \times N$ 的矩阵，矩阵 $A + B$ 得到 $N \times N$ 的矩阵 $C$ 。

式子的含义为：矩阵 $C$ 由矩阵 $A,B$ 对应位置上数值相加所得。

**划重点：矩阵加法满足交换律** 。


#### **(2).【矩阵乘法】**

矩阵乘法的一般式：$C_{i,j}=\sum_{k=1}^{K} (A_{i,k} \times B_{k,j})$，其中 $A$ 为 $N \times K$ 的矩阵，$B$ 为 $K*M$ 的矩阵，矩阵 $A \times B$ 得到 $N \times M$ 的矩阵 $C$ 。

式子的含义为：矩阵 $C_{i,j}$ 由矩阵 $A$ 第 $i$ 行上的 $K$ 个数与矩阵 $B$ 第 $j$ 列上的 $K$ 分别相乘并求和得到。

**划重点：矩阵乘法不满足交换律，满足结合律，满足分配律**（在某些特定情况下满足乘法交换律）。

写成代码的话大概长这个样子 (二维数组模拟矩阵

```cpp
struct QAQ {
    LL a[sz][sz];
    inline QAQ() { memset(a, 0, sizeof a); }
    inline QAQ operator-(const QAQ &T) const {
        QAQ res;
        for (int i = 0; i < sz; ++i)
            for (int j = 0; j < sz; ++j) {
                res.a[i][j] = (a[i][j] - T.a[i][j]) % MOD;
            }
        return res;
    }
    inline QAQ operator+(const QAQ &T) const {
        QAQ res;
        for (int i = 0; i < sz; ++i)
            for (int j = 0; j < sz; ++j) {
                res.a[i][j] = (a[i][j] + T.a[i][j]) % MOD;
            }
        return res;
    }
    inline QAQ operator*(const QAQ &T) const {
        QAQ res;
        int r;
        for (int i = 0; i < sz; ++i)
            for (int k = 0; k < sz; ++k) {
                r = a[i][k];
                for (int j = 0; j < sz; ++j)
                    res.a[i][j] += T.a[k][j] * r, res.a[i][j] %= MOD;
            }
        return res;
    }
    inline QAQ operator^(LL x) const {
        QAQ res, bas;
        for (int i = 0; i < sz; ++i) res.a[i][i] = 1;
        for (int i = 0; i < sz; ++i)
            for (int j = 0; j < sz; ++j) bas.a[i][j] = a[i][j] % MOD;
        while (x) {
            if (x & 1) res = res * bas;
            bas = bas * bas;
            x >>= 1;
        }
        return res;
    }
};
```

---

$$
QAQ
$$

---

## 三、如何优化加速

以 [斐波那契数列 $[P1962]$​​](https://www.luogu.org/problemnew/show/P1962) 为例 (之前在牛客的数据范围在 $10^{18}$​，洛谷这题数据范围在 $2^{63}$​ 

大部分人都会简单的求 $Fibonacci$，其递推式为 $f(n)=f(n-1)+f(n-2) (n \geqslant 2$ $且$ $n \in N^{*})$，其中 $f(1)=f(2)=1$ 。

这道题按照正常的递推做法可以过 $60$ 分，对于大一点的 $n$ 就不行了。

由于其递推方程是固定的，决策点只要两个（$n-1$ 和 $n-2$），所以可以考虑用矩阵乘法加速。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210419185017.png)

#### 【构造答案矩阵和累乘矩阵】

注意**决策点数量：两个**，可以先尝试使用二维矩阵，如果不行就试着加一维（辅助递推），大概做法如下：

设 $f(n)$ 为 $Fibonacci$ 数列第 $n$ 项。

先构建一个 $1 \times 2$ 的答案矩阵 $F(n)$：

$F(n) = \begin{vmatrix}f(n)&f(n-1)\end{vmatrix}$​

再构造一个 $2 \times 2$ 的累乘矩阵 $base$：

$base = \begin{vmatrix}a&b\\c&d\end{vmatrix}$，其中 $a,b,c,d$ 均为未知数

我们需要满足：$F(n) \times base = F(n+1)$ 。
即 $\begin{vmatrix}f(n)&f(n-1)\end{vmatrix} \times \begin{vmatrix}a&b\\c&d\end{vmatrix} = \begin{vmatrix}af(n)+cf(n-1)&bf(n)+df(n-1)\end{vmatrix} = \begin{vmatrix}f(n+1)&f(n)\end{vmatrix}$
即 $af(n)+cf(n-1)=f(n+1),bf(n)+df(n-1)=f(n)$
由递推式可知：$a=b=c=1,d=0$
即 $base = \begin{vmatrix}1&1\\1&0\end{vmatrix}$ 。

实际上在做题的时候不需要这么麻烦，只需要按这种思路去模拟一下 $base$ 就出来了。

![分割线QAQ](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210419185015.gif)

#### 【快速幂加速运算】

对 $F(n)$ 稍作转换：$F(n) = F(2) \times base \times base......\times base$，其中 $base$ 一共乘了 $n-2$ 次。

为什么要写 $F(2)$ 呢？在写题时，这是一个致命的细节问题。  
根据定义，$F(1)=\begin{vmatrix}f(1)&f(0)\end{vmatrix}$，其中 $f(0)$ 无法计算（或者说毫无意义），所以要用 $f(2)$ 作为初始矩阵来计算。

由**矩阵乘法结合律**可知： $F(n)=F(2) \times base^{n-2} = \begin{vmatrix}1&1\end{vmatrix} \times \begin{vmatrix}1&1\\1&0\end{vmatrix}^{n-2}$

即 $F(n) = \begin{vmatrix}1&1\end{vmatrix} \times \begin{vmatrix}1&1\\1&0\end{vmatrix}^{n-2} = \begin{vmatrix}f(n)&f(n-1)\end{vmatrix}$

在具体的代码实现中，我们可以将 $F(n)$ 视为 $2 \times 2$ 的矩阵，多余的部分赋值为 $0$，即 $F(n) = \begin{vmatrix}f(n)&f(n-1)\\0&0\end{vmatrix}$，可以发现，不管怎么乘，它还是这样的形式（第二行和 $1 \times 2$ 矩阵的变化相同，第二行依然全为 $0$）。  
求 $base$ 的幂时用一个快速幂，注意快速幂初始值要设为 $F(2)$，如果在算完 $base^{n-2}$ 后再乘上 $F(2)$ 的话，就违背了矩阵乘法不满足交换律的原则。

**时间复杂度为** $\mathcal{O}(logn)$。

【AC Code】

```cpp
// Fib: f[1] = f[2] = 1,f[n] = f[n - 1] + f[n - 2]
// [1 1] 矩阵 init
// [1 0]
struct QAQ { // 还是结构体写的方便一些
    ll a[3][3];
    void clear() {memset(a, 0, sizeof(a)); a[1][1] = a[2][1] = a[1][2] = 1;}
    QAQ operator * (QAQ &T) {
        QAQ ans;
        memset(ans.a, 0, sizeof(ans.a));
        for (int i = 1; i < 3; i++)
            for (int j = 1; j < 3; j++)
                for (int k = 1; k < 3; k++)
                    (ans.a[i][j] += a[i][k] * T.a[k][j] % MOD) %= MOD;
        return ans;
    }
};
ll n;
inline ll solve(ll k) {
    if (k < 1) return 1; // 特判很重要
    if (!k) return 1;
    QAQ s, x;
    x.clear();
    s.a[1][1] = s.a[1][2] = 1, s.a[2][1] = s.a[2][2] = 0; // 初始化 F(2)
    while (k) {
        if (k & 1) s = (s * x);
        x = (x * x); k >>= 1;
    }
    return s.a[1][1] % MOD;
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    cin >> n;
    cout << solve(n - 2);
}   
```

---

$$
QAQ
$$

---

## 四、矩阵加维

### 前言

**缺什么补什么**，有不确定的信息就先**将递推式写出来**，然后根据具体情况**加维**。

下面一共总结了 $3$ 种需要加维的情况（可能不全，欢迎补充）：

---

### 【带常数项 $k$】

递推方程：$f(n)=f(n-1)+f(n-2)+k$ 。

求：$f(n)$。

常数项不可忽略，应当专门加一维来计算常数。**常数项的递推式**是最简单的： $k_n=k_{n-1}+0$ 。 

$F(n) = \begin{vmatrix}f(n)&f(n-1)&k\end{vmatrix}$，$base = \begin{vmatrix}1&1&0\\1&0&0\\1&0&1\end{vmatrix}$ 。

--------

### **【带未知数项 $n$】**

递推方程：$f(n)=f(n-1)+f(n-2)+n$ 。

求：$f(n)$。

先将**未知项的递推式**写出来： $(n)=(n-1)+1$ ，虽然 $f(n)$ 的转移只有四项，但需要加一维来辅助未知项 $n$ 的递推。

$F(n) = \begin{vmatrix}f(n)&f(n-1)&n&1\end{vmatrix}$，$base = \begin{vmatrix}1&1&0&0\\1&0&0&0\\1&0&1&0\\1&0&1&1\end{vmatrix}$ 。

--------

### **【求和】**

递推方程：$f(n)=f(n-1)+f(n-2)$ 。

求：$S(n)=\sum_{i=1}^{n} f(i)$ 。

暴力计算 $n$ 次得出 $f(i)$ 数组肯定是不行的，但可以尝试将 $S(n)$ 放入矩阵跟随着 $f(n)$ 一起递推。

先将**前缀和的递推式**写出来：$S(n)=S(n-1)+f(n)$ 。

$F(n) = \begin{vmatrix}f(n)&f(n-1)&S(n-1)\end{vmatrix}$，$base = \begin{vmatrix}1&1&1\\1&0&0\\0&0&1\end{vmatrix}$ 。

---

$$
QAQ
$$

---

## **五、一些经典题目**

### **1.【连续幂次和】**

给定一个 $n \times n$ 的矩阵 $A$ 和一个整数 $K$，求 $\sum_{i=1}^{K} A^i$ 。[传送门](https://www.luogu.org/problem/UVA11149)

#### **(1).【两次分治】**

对于单个 $A^i$ 可以通过 $log$ 次转移得到，需要计算多个时就需要再次分治。

原理：$A^1+A^2+A^3......A^n=$ $A^1+A^2+A^3...A^{mid}+A^{mid}*(A^1+A^2+A^3...A^{mid})$ 。

每次分治时计算一下 $mid$，递归计算 $\sum_{i=1}^{mid} A^i$，$log$ 次乘法计算 $A^{mid}$，另一半可直接得出（当 $K$ 为奇数时还需计算 $A^K$）。

时间复杂度：$n^3log^2K$ 。

##### **【Code】**1250ms

```cpp
#define Re register ll
const int N = 45;
ll n, K, P = 10;
inline void in(Re &x) {
    Re f = 0; x = 0; char c = getchar();
    while (c < '0' || c > '9')f |= c == '-', c = getchar();
    while (c >= '0' && c <= '9')x = (x << 1) + (x << 3) + (c ^ 48), c = getchar();
    x = f ? -x : x;
}
struct QAQ {
    ll n, a[N][N];
    QAQ() {memset(a, 0, sizeof(a));}
    inline void In() {
        for (Re i = 1; i <= n; ++i)
            for (Re j = 1; j <= n; ++j)
                in(a[i][j]), a[i][j] %= P; //一定要边读边膜
    }
    inline void Out() {
        for (Re i = 1; i <= n; printf("%lld\n", a[i][n]), ++i) //卡输出。。
            for (Re j = 1; j < n; ++j)
                printf("%lld ", a[i][j]);
    }
    inline QAQ operator*(QAQ O)const {
        QAQ ans; ans.n = n;
        for (Re i = 1; i <= n; ++i)
            for (Re j = 1; j <= n; ++j)
                for (Re k = 1; k <= n; ++k)
                    (ans.a[i][j] += a[i][k] * O.a[k][j] % P) %= P;
        return ans;
    }
    inline QAQ operator+(QAQ O)const {
        QAQ ans; ans.n = n;
        for (Re i = 1; i <= n; ++i)
            for (Re j = 1; j <= n; ++j)
                (ans.a[i][j] += (a[i][j] + O.a[i][j]) % P) %= P;
        return ans;
    }
    inline void operator+=(QAQ O) {*this = *this + O;}
    inline void operator*=(QAQ O) {*this = *this * O;}
} A;
inline QAQ mi(QAQ x, Re k) {
    QAQ s = x; --k; //s初始化为一个x，k减1
    while (k) {
        if (k & 1)s *= x;
        x *= x, k >>= 1;
    }
    return s;
}
inline QAQ calc(QAQ A, Re k) {
    if (k == 1) {return A;} //边界直接返回
    QAQ ans, tmp = calc(A, k >> 1); //先算出一小段的
    ans = tmp + (tmp * mi(A, k >> 1)); //算出A^1+A^2+...A^(k/2*2)
    if (k & 1)ans += mi(A, k); //如果k为奇数则再加上一个A^k
    return ans;
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    while (scanf("%lld%lld", &A.n, &K) && A.n)A.In(), calc(A, K).Out(), puts(""); //卡输出。。
}
```

改进写法：20ms

```cpp
const int N = 45, P = 10;
int n, m;
struct QAQ {
    int a[N][N];
    QAQ() {memset(a, 0, sizeof(a));}
} e;
QAQ operator*(const QAQ t1, const QAQ t2) {
    QAQ ans;
    for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j)
            for (int k = 1; k <= n; ++k)
                (ans.a[i][j] += t1.a[i][k] * t2.a[k][j] % P) %= P;
    return ans;
}
QAQ operator+(const QAQ t1, const QAQ t2) {
    QAQ ans;
    for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j)
            (ans.a[i][j] = (t1.a[i][j] + t2.a[i][j]) % P) %= P;
    return ans;
}
int main() {
    // cin.tie(nullptr)->sync_with_stdio(false);
    int _ = 0;
    for (int i = 1; i < N; ++i) e.a[i][i] = 1;
    while (~scanf("%d%d", &n, &m) && (n || m)) {
        QAQ A;
        if (_++) putchar('\n');
        for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j)
                scanf("%d", &A.a[i][j]), A.a[i][j] %= P;
        QAQ ans;
        for (QAQ x = A, tmp = A; m; m >>= 1, x = x * tmp + x, tmp = tmp * tmp)
            if (m & 1) ans = ans * tmp + x;
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j < n; ++j) printf("%d ", ans.a[i][j]);
            printf("%d\n", ans.a[i][n]);
        }
    }
}
```



#### **(2).【倍增】**

$To$ $be$ $continued...$

时间复杂度：$n^3logK$ 。

---

### **2.【有向图中求合法路径方案数】**

给出一张 $n$ 个点（从 $0$ 到 $n-1$ 编号） $m$ 条边有向图，每次询问求从 $st$ 恰好走 $K$ 步到达 $ed$ 的方案数，重边视作一条路径，每条边可重复走。

#### **(1).【分析】**

为方便分析，将点编号都加一，变为 $[1,n]$ 。

设 $f_k(i,j)$ 表示从点 $i$ 到达点 $j$ 恰好走 $k$ 步的方案数。所以 $f_k(i,j)$

当 $K$ 较小时可以使用 $bfs+dp$，若存在一条边 $x,j$，则 $f_{k}(i,j)+=f_{k-1}(i,x)*1$ 。但如果 $K \leqslant 10^9$ 就无法解决了。

若用邻接矩阵 $a(i,j)$ 来表示点 $i$ 与 $j$ 之间是否连边，那么根据上述转移式子可得： $f_{k}(i,j)=\sum_{x=1}^{n}f_{k-1}(i,x)*a(x,j)$ 。

由于询问是不定向的，起点和终点可能是 $[1,n]$ 中的任意一个，所以答案矩阵应包含 $n^2$ 个信息，其中 $F(k)$ 中的第 $i$ 行第 $j$ 列表示 $f_k(i,j)$ ，即用 $F(K)$ 表示恰好走 $K$ 步的答案矩阵。

所以对于每条边 $(x,y)$，使累乘矩阵中第 $x$ 行第 $y$ 列的数为 $1$，最后直接求一个 $K$ 次幂即可。
$F(k) = \begin{vmatrix}f_k(1,1)&f_k(1,2)&...&f_k(1,n)\\f_k(2,1)&f_k(2,2)&...&f_k(2,n)\\...&...&...&...\\f_k(n,1)&f_k(n,2)&...&f_k(n,n)\end{vmatrix}$

$base$ 略。

时间复杂度：$O(n^3logK)$ 。

##### **【Code】**

```cpp
const int N = 25, P = 1000;
struct QAQ {
    int n, a[N][N];
    QAQ() {memset(a, 0, sizeof(a));}
    inline QAQ operator*(QAQ x)const {
        QAQ ans; ans.n = n;
        for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j)
                for (int k = 1; k <= n; ++k)
                    (ans.a[i][j] += a[i][k] * x.a[k][j] % P) %= P;
        return ans;
    }
    inline void operator*=(QAQ x) {*this = *this * x;}
};
inline QAQ qpow(QAQ x, int k) {
    QAQ s = x; k -= 1;
    while (k) {
        if (k & 1) s *= x;
        x *= x, k >>= 1;
    } return s;
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n, m, _;
    while (~scanf("%d%d", &n, &m) && (n || m)) {
        QAQ A; A.n = n;
        int x, y, k;
        while (m--) scanf("%d %d", &x, &y), A.a[x + 1][y + 1] = 1;
        scanf("%d", &_);
        while (_--) {
            scanf("%d %d %d", &x, &y, &k);
            //注意：K=0时要特判，不需要走动时输出1否则输出0
            printf("%d\n", k ? qpow(A, k).a[x + 1][y + 1] : x == y);
        }
    }
}
```



#### **(2).【扩展 1】**

询问改为：求走的步数不超过 $K$ 的方案数。其他条件不变。

求一个 $1$ 至 $K$ 的连续幂次和即可。

设 $S_K(i,j)=\sum_{k=1}^{K}f_k(i,j)$ 。

前缀和 $S_K(i,j)$ 也要记录 $n^2$ 个，要尽量缩减矩阵规模的话，可以把他们一层层地包裹在原 $n \times n$ 的矩阵外面（自己口胡的，没有尝试过），但代码难度较大，也可以将原矩阵扩大一倍变成 $2n \times 2n$，代码难度较小，即：$F(k) = \begin{vmatrix}f_k(1,1)&...&f_k(1,n)&S_k(1,1)&...&S_k(1,n)\\...&...&...&...&...&...\\f_k(n,1)&...&f_k(n,n)&S_k(n,1)&...&S_k(n,n)\\\end{vmatrix}$ 。

$base$ 略。

时间复杂度：$O((2n)^3logK)$

#### **(3).【扩展 2】**

增加一个限制条件：一共有四种物品，每条边上有相同或不同种类的若干个，每次经过时如果拿走这些物品则会多消耗一步（相当于走两步），求走的步数不超过 $K$ 且能将四种物品都拿全的方案数。

先考虑走两步的转移，在答案矩阵中再加入 $n^2$ 个 $f_{k-1}$ 的信息，其中一种加维方案（空白处填 $0$）：

$F(k) = \begin{vmatrix}f_k(1,1)&...&f_k(1,n)&S_k(1,1)&...&S_k(1,n)\\...&...&...&...&...&...\\f_k(n,1)&...&f_k(n,n)&S_k(n,1)&...&S_k(n,n)\\f_k(1,1)&...&f_k(1,n)\\...&...&...\\f_k(n,1)&...&f_k(n,n)\end{vmatrix}$ 

$base$ 略。

再考虑物品限制，假设要求最后只能选 $1,2$，那么在构造矩阵时带有 $3,4$ 的边都不加进去。

如上所述，跑若干次计算搞一下**容斥**即可（我太蒻了，容斥这里只有先咕着了）。

$To$ $be$ $continued...$

-------

### **3.【？】**

$To$ $be$ $continued...$

-------

$$
QAQ
$$

--------

## 六、**题目链接**

### **【简单题】**

- [斐波那契数列 $[P1962]$​](https://www.luogu.org/problemnew/show/P1962) 
  【标签】递推/矩阵乘法

- [【模板】矩阵加速（数列）$[P1939]$​](https://www.luogu.org/problemnew/show/P1939) 
  【标签】递推/矩阵乘法

### **【中档题】**

- [小奇的集合 $[BZOJ4547]$](https://www.lydsy.com/JudgeOnline/problem.php?id=4547) [$[HDU5171]$](http://acm.hdu.edu.cn/showproblem.php?pid=5171) 
  【标签】递推/矩阵乘法 
  【题解】[$hzwer$​​](http://hzwer.com/7734.html)

- [$Power$ $of$ $Matrix$ $[UVA11149]$](https://www.luogu.org/problem/UVA11149) 
  [$Matrix$ $Power$ $Series$ $[Poj3233]$​​](http://poj.org/problem?id=3233) 
  【标签】递推/矩阵乘法/二分

- [$How$ $many$ $ways??$ $[Hdu2157]$](http://acm.hdu.edu.cn/showproblem.php?pid=2157)
  【标签】递推/矩阵乘法

- [牛继电器 $Cow$ $Relays$ $[P2886]$](https://www.luogu.org/problem/P2886) [$[Poj3613]$](https://poj.org/problem?id=3613)
  【标签】递推/矩阵乘法

-------

$$
QAQ
$$

--------

## 七、**参考资料**

* 感谢 [辰星凌](https://www.cnblogs.com/Xing-Ling/)桑 的转载同意！

- [题解 $[P1962]$ 斐波那契数列](https://anguei.blog.luogu.org/solution-p1962)

- [矩阵的十大经典题目，留份做题](https://blog.csdn.net/rowanhaoa/article/details/21024093)

