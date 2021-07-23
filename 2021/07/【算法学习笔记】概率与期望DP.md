> 本文学习自 [Sengxian](https://blog.sengxian.com/about/) 学长的博客
>
> 之前也在CF上写了一些概率DP的题并做过总结
>
> 建议阅读完本文再去接着阅读这篇文章：[Here](https://www.cnblogs.com/RioTian/p/14117154.html)



## 前言

单纯只用到概率的题并不是很多，从现有的 OI/ACM 比赛中来看，大多数题目需要概率与期望结合起来（期望就是用概率定义的），所以本文主要讲述期望 DP。

期望 DP 有一些固定的方法，这里分多种方法来讲述。

---

## 讲解

### 例一

[#3036. 绿豆蛙的归宿](https://www.acwing.com/problem/content/description/219/)

题意：

给定一个起点为 $1$，终点为 $n$ 的有向无环图。到达每一个顶点时，如果有 $K$ 条离开该点的道路，可以选择任意一条道路离开该点，并且走向每条路的概率为 $\frac 1 K$。问你从 $1$ 出发走到 $n$ 的路径期望总长度是多少。

#### 方法一：直接定义期望状态

这道题的终点很明确，那就是走到 $n$ 即停止。对于期望 DP，我们一般采用**逆序**的方式来定义状态，即考虑从当前状态到达终点的期望代价。因为在大多数情况下，终点不唯一，而起点是唯一的。
我们定义 $dp(i)$为从 $i$ 出发走到终点 $n$ 的路径期望总长度，根据全期望公式，得到（设​ $G_i$为从 $i$ 的边的集合）：
$$
dp(i) = \sum\limits_{e\in G_i}\frac{dp(e_{to}) + e_{const}}{|G_i|}
$$
因为这是一个有向无环图，每个点需要其能到达的点的状态，故我们采用拓扑序的逆序进行计算即可。

【AC Code】

```cpp
const int N = 100000, M = 2 * N;
int n, m;
struct node { int v, w;};
vector<node>e[N];
int d[N];    // 出度
double f[N]; // dp

double dfs(int u) {
    if (f[u] >= 0)return f[u];
    f[u] = 0;
    for (auto [v, w] : e[u]) { // tuple 需开启 C++17
        f[u] += (w + dfs(v)) / d[u];
    }
    return f[u];
}

int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    memset(f, -1, sizeof(f));
    cin >> n >> m;
    for (int i = 0; i < m; ++i) {
        int u, v, w;
        cin >> u >> v >> w;
        e[u].push_back(node{v, w});
        d[u]++;//出度++
    }
    cout << fixed << setprecision(2) << dfs(1);
}
```

#### 方法二：利用期望的线性性质

根据[期望的线性性质](https://zh.wikipedia.org/wiki/%E6%9C%9F%E6%9C%9B%E5%80%BC#%20.E6.80.A7.E8.B4.A8)，$E[aX +bY]=aE[X] + bE[Y]$。所以另外一种求期望的方式是分别求出每一种代价产生的期望贡献，然后相加得到答案。在本题中，路径期望总长度等于每条边产生的期望贡献之和。而每条边产生又等于经过这条边的期望次数乘这条边的代价。所以，我们只需要算每条边的期望经过次数即可。

边 $(u,v,w)$ 的期望经过次数是不好直接算的，但如果我们能算得点 $u$ 的期望经过次数为 $dp(u,v)$，那么边 $(u,v,w)$ 的期望经过次数是 $dp(u)*\frac1{|G_u|}$ ，对答案的贡献就是 $w*dp(u)*\frac1{|G_u|}$

如何计算点 $u$ 的期望经过次数 $dp(u)$呢？我们依然考虑 DP 的方式，首先有 $dp(u) = 1$，转移采取刷表的方式：
$$
dp(e_{to})\leftarrow dp(u)*\frac1{|G_u|},e\in G_u
$$
在用边 $e$ 刷表的同时，边 $e$ 的贡献就可以计算了，十分简洁。因为这种方法计算答案十分的便捷，而且适用范围广，所以这种『利用期望的线性性质，单独计算贡献的方法』是我们计算期望首选的方法。

【AC Code】这里贴 Sengxian 学长的代码

```cpp
typedef long long ll;
inline int readInt() {
    static int n, ch;
    n = 0, ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) n = n * 10 + ch - '0', ch = getchar();
    return n;
}

const int MAX_N = 100000 + 3, MAX_M = MAX_N * 2;
struct edge {
    edge *next;
    int to, cost;
    edge(edge *next = NULL, int to = 0, int cost = 0): next(next), to(to), cost(cost) {}
} pool[MAX_M], *pit = pool, *first[MAX_N];
int n, m, deg[MAX_N], outDeg[MAX_N];
double f[MAX_N];

void solve() {
    static int q[MAX_N];
    int l = 0, r = 0;
    q[r++] = 0;

    double ans = 0;
    f[0] = 1.0;
    while (r - l >= 1) {
        int u = q[l++];
        for (edge *e = first[u]; e; e = e->next) {
            f[e->to] += f[u] / outDeg[u];
            ans += f[u] * e->cost / outDeg[u];
            if (--deg[e->to] == 0) q[r++] = e->to;
        }
    }

    printf("%.2f\n", ans);
}

int main() {
    n = readInt(), m = readInt();
    for (int i = 0, u, v, w; i < m; ++i) {
        u = readInt() - 1, v = readInt() - 1, w = readInt();
        first[u] = new (pit++) edge(first[u], v, w);
        deg[v]++, outDeg[u]++;
    }
    solve();
}
```

### 例二

接着我们考虑 [Codeforces 518D](http://codeforces.com/problemset/problem/518/D) 这道题，以便体会方法二的好处。

**题意：**有 $n$ 个人排成一列，每秒中队伍最前面的人有 $p$ 的概率走上电梯（一旦走上就不会下电梯），或者有 $1-p$ 的概率不动。问你 $T$  秒过后，在电梯上的人的期望。

#### 方法一

在本题这样一个情况中，方法一是用不了的，因为我们的结束状态不明确。

#### 方法三：使用期望的定义计算

如果 $X$ 是离散的随机变量，输出值为 $x_1,x_2,...$，输出值相应的概率为 $p_1,p_2,...$，那么期望值是一个无限数列的和（如果不收敛，那么期望不存在）：
$$
E[x] =\sum\limits_ip_ix_i
$$
在本题中，如果设 $dp(i,j)$ 为 $i$  秒过后，电梯上有  $j$  个人的概率，那么答案是：
$$
\sum\limits_{0\le k\le n}dp(T,K)*K
$$
所以我们只需要求 $dp(i, j)$ 就可以了，初始值 $dp(0, 0) = 1$ 就可以了，仍然是采用刷表法：
$$
dp(i + 1,j + 1) \leftarrow dp(i,j)*p\\
dp(i + 1,j)\leftarrow dp(i,j) * (1 - p)
$$
【AC Code】

```cpp
const int N = 2e3 + 10;
double p, dp[N][N];
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n, t;
    cin >> n >> p >> t;
    dp[0][0] = 1;
    for (int i = 0; i < t; ++i) {
        dp[i + 1][n] += dp[i][n];
        for (int j = 0; j < n; ++j) if (dp[i][j] > 1e-10) {
                dp[i + 1][j + 1] += dp[i][j] * p;
                dp[i + 1][j] += dp[i][j] * (1 - p);
            }
    }
    double ans = 0;
    for (int i = 0; i <= n; ++i) ans += i * dp[t][i];
    cout << fixed << setprecision(6) << ans;
}
```

#### 方法二

那么之前提到的适用范围广的方法二，是否能在这里用呢？答案是肯定的。

延续方法三的 DP，我们不妨将状态之间的转移抽象成边，只不过只有 $dp(i, j)$ 到 $dp(i + 1, j + 1)$ 的边才有为 $1$ 的边权，其余都为 $0$。因为这个 DP 涵盖了所有可能出现的情况，所以我们仍然可以利用期望的线性性质，在刷表的过程中进行计算答案。

本题中，没有直观的边的概念，但是我们可以将状态之间的转移抽象成边，由于 $dp(i, j)$到 $dp(i + 1, j + 1)$ 这一个转移是对答案有 $1$ 的贡献的，所以我们将它们之间的边权赋为 $1$。

这一题将方法二抽象化了，实际上大多数题并非是直观的，而是这种抽象的形式。

```cpp
const int N = 2e3 + 10;
double p, dp[N][N];
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n, t;
    cin >> n >> p >> t;
    dp[0][0] = 1;
    double ans = 0;
    for (int i = 0; i < t; ++i) {
        dp[i + 1][n] += dp[i][n];
        for (int j = 0; j < n; ++j) if (dp[i][j] > 1e-10) {
                dp[i + 1][j + 1] += dp[i][j] * p;
                dp[i + 1][j] += dp[i][j] * (1 - p);
                ans += dp[i][j] * p;
            }
    }
    cout << fixed << setprecision(6) << ans;
}
```

### 例三

[BZOJ 3450](http://www.lydsy.com/JudgeOnline/problem.php?id=3450)

**题意：**给定一个序列，一些位置未确定（是 $\texttt{o}$ 与 $\texttt{x}$ 的几率各占 $\%50%$）。对于一个 $\texttt{ox}$ 序列，连续 $x$ 长度的 $\texttt{o}$ 会得到 $x^2$ 的收益，请问最终得到的序列的期望收益是多少？

#### 分析

这个题如果一段一段的处理，实际上并不是很好做。我们观察到 $(x + 1) ^ 2 - x ^ 2 = 2x + 1$，那么根据期望的线性性质，我们可以单独算每一个字符的贡献。我们设 $dp_i$ 为考虑前 i*i* 个字符的期望得分，$l_i$ 为以 $i$ 为结尾的 comb 的期望长度，$Comb_i$ 为第 $i$个字符，那么有 3 种情况：

1. $s_i = o$ ，则 $dp_i = dp_{i - 1} + l_{i - 1} * 2 + 1,l_i = l_{i - 1} + 1$
2. $s_i = x$ ，则 $dp_i = dp_{i - 1}$
3. $s_i =\ ?$， 则 $dP_i = dp_{i - 1} + \frac{l_i*2 + 1}{2},l_i = \frac{l_{i - 1} + 1}{2}$

对于前两种情况，其实是非常直观的，对于第三种情况，实际上是求了一个平均长度。例如 `?oo`，两种情况的长度 $l_i$ 分别为 $[0,1,2]$ 和 $[1,2,3]$ ，但是求了平均之后，长度 $l_i$ 变成了 $[0.5,1.5,2.5]$ ，这样由于我们的贡献是一个关于长度的一次多项式 $(2x + 1)$ ，所以长度平均之后，贡献也相当于求了一个平均，自然能够求得正确的得分期望。

【AC Code】

```cpp
const int N = 3e5 + 10;
double dp[N], Comb[N];
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n; string s;
    cin >> n >> s;
    for (int i = 0; i < n; ++i) {
        if (s[i] == 'o') {
            dp[i] = dp[i - 1] + Comb[i - 1] * 2 + 1;
            Comb[i] = Comb[i - 1] + 1;
        } else if (s[i] == 'x') {
            dp[i] = dp[i - 1];
            Comb[i] = 0;
        } else {
            dp[i] = dp[i - 1] + (Comb[i - 1] * 2 + 1) / 2;
            Comb[i] = (Comb[i - 1] + 1) / 2;
        }
    }
    cout << setprecision(4) << fixed << dp[n - 1];
}
```

思考：如果长度为 $a$ 的 comb 的贡献为 $a^3$ 时该如何解决？

> Tips：由于 $(a + 1)^3 - a^3 = 3a^3 + 3a + 1$ ，所以我们要维护 $a^2$ 和 $a$ 的期望，注意 $E_{a^2} \not= E^2_a$，所以维护 $a^2$ 的期望是必要的。

---

### 例四

[BZOJ 4318](https://darkbzoj.tk/problem/4318)

**题意：**给定一个序列，每个位置 $o$ 的几率为 $p_i$ ，为 $x$ 的几率为 $1-p_i$ 。对于一个 $\texttt{ox}$ 序列，连续 $x$ 长度的 $\texttt{o}$ 会得到 $x^3$ 的收益，请问最终得到的 $ox$ 序列的期望收益是多少？

#### 分析

延续例三的思路，我们还是分别求每一个位置的贡献。根据 $(a + 1)^3 - a^3 = 3a^3 + 3a + 1$，我们只需要维护 $l(i)$为以 $i$ 为结尾的 comb 的期望长度，$l_2(i)$为以 $i$ 为结尾的 comb 的期望长度的平方。注意 $E[a^2] \not =E^2[a]$，所以维护 $a^2$ 的期望是必要的。

```cpp
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n;
    double p, l1 = 0, l2 = 0, ans = 0;
    cin >> n;
    for (int i = 0; i < n; ++i) {
        cin >> p;
        ans += (3 * l2 + 3 * l1 + 1) * p;
        l2 = (l2 + 2 * l1 + 1) * p;
        l1 = (l1 + 1) * p;
    }
    cout << fixed << setprecision(1) << ans;
}
```

## 总结

期望 DP 一般来说有它固定的模式，一种模式是直接 DP，定义状态为到终点期望，采用逆序计算得到答案。一种模式是利用期望的线性性质，对贡献分别计算，这种模式一般要求我们求出每种代价的期望使用次数，而每种代价往往体现在 DP 的转移之中。最后的两个例题是典型的分离变量，用期望的线性性质计算答案的例子，如果状态过于巨大，那么就得考虑分离随机变量了。

本总结只是解释了概率与期望 DP 的冰山一角，它可以变化多端，但那些实际上并不只属于概率与期望 DP，真正核心的内容，还是逃不出我们几种方法。

> 想要深入了解一些概率的DP的请阅读这篇文章：[Here](https://www.cnblogs.com/RioTian/p/14117154.html)