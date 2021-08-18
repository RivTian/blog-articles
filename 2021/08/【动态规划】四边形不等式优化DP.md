> 学习自 Pecco 神的算法[笔记](https://zhuanlan.zhihu.com/p/398419302)

下面这个式子是**区间DP**的经典方程：

$dp[l][r] = \min\limits_{l\le k\le r}(dp[l][k] + dp[k + 1][r]) + w(l,r)$ 

这是一个2D/1D动态规划（一共有约 $n^2$ 个状态，每次状态转移需要 $\mathcal{O}(n)$ 的时间），所以朴素的算法的时间复杂度是 $\mathcal{O}(n^3)$  。然而，如果上面的 $w(l,r)$ 这个二元函数符合一些条件，我们可以在 $\mathcal{O}(n^2)$ 内解决它。

第一是**区间包含单调性**，即若 $l \le l’ \le r’ \le r$ ，则 $w(l’,r’)\le w(l,r)$ 

![被包含者不大于包含者](https://gitee.com/riotian/blogimage/raw/master/img/20210817151935.jpg)



第二是**四边形不等式**，即若 $l \le l’\le r’ \le r$ ，则$w(l,r') + w(l',r) \le w(l’,r’)+ w(l,r)$

![交叉不大于包含](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210817194554.jpg)

为什么叫四边形不等式呢？可能是因为这个不等式的形式与 <u>四边形对边长度和 $≤$ 对角线长度和的</u> 形式非常类似。[^1] (下图中 $dis(A,D) + dis(B,C) \le dis(A,C) + dis(B,D)$ )

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210817194936.jpg)



接下来有两个非常神奇的结论（证明略 [^2]

* 当 $w(l,r)$ 同时满足这两个条件时，$dp[l][r] $ 也将符合四边形不等式。
* 如果 $dp[l][r]$ 满足四边形不等式，假设 $m[l][r]$ 为 $dp[l][r]$ 的最优决策点，那么 $m[l][r - 1] \le m[l][r] \le m[l + 1][r]$.

什么叫最优决策点？考虑 $dp[l][r] = \min\limits_{l\le k,r}(dp[l][k] + dp[k + 1][r]) + w(l,r)$ ，我们把使 $dp[l][k] + dp[k + 1][r]$ 取到最小值的那个 $k$ ，称为 $dp[l][r]$ 的最优决策点。

结论2相当于在说，最优决策点的矩阵**每一行、每一列**都是**单调不减**的。<u>（这非常重要，实际上这才是这个优化的核心。你甚至选择不去证明四边形不等式，而是通过打表猜出这个结论）</u>

利用结论，我们可以在DP的过程中记录最优决策点，这样，我们对于每个状态点，枚举的范围就从 $l\le k < r$ 缩小到了 $m[l][r- 1]\le k\le m[l + 1][r]$ 。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210817195906.jpg)

区间DP的DP顺序是从主对角线开始，一条条对角线地往上转移的。在上图中，对于每一条对角线，考察它枚举的区间，可以发现每两个相邻的区间只有一个元素重叠，所以处理每条对角线需要花费 $\mathcal{O}(n)$ 的时间 ，处理所有对角线也就只需要 $\mathcal{O}(n^2)  $ 的时间。

优化后的代码如下：

```cpp
for (int i = 1; i <= n; ++i)
    m[i][i] = i; // 初始化边界决策点
for (int d = 2; d <= n; ++d)
    for (int l = 1, r = d; r <= n; ++l, ++r) {
        f[l][r] = inf;
        for (int k = m[l][r - 1]; k <= m[l + 1][r]; ++k) // 利用结论，缩小了枚举范围
            if (f[l][k] + f[k + 1][r] + w(l, r) < f[l][r]) {
                f[l][r] = f[l][k] + f[k + 1][r] +  w(l, r); // 更新dp数组
                m[l][r] = k; // 更新决策点
            }
    }
```

那么哪些二元函数符合四边形不等式呢？根据 [OI-wiki](https://link.zhihu.com/?target=https%3A//oi-wiki.org/dp/opt/quadrangle/)，我们有以下结论：

1. 如果 $f(l,r)$ 和 $g(l,r)$  符合四边形不等式/区间包含单调性，则对于任意 $A,B\ge 0$ ，$Af(l,r) + Bg(l,r)$ 也符合四边形不等式/区间包含单调性。
2. 如果存在 $f(x)$   和 $g(x)$  使 $w(l,r) = f(r) - g(l)$ ，则 $w(l,r)$ 符合**四边形恒等式**（即等号总是成立的四边形不等式）。如果 $f,g$ 单增，则 $w$​  还符合区间包含单调性。
3. 若 $h(x)$​ 单增且下凸[^3], $w(l,r)$​ 符合四边形不等式和区间包含单调性,则 $h(w(l,r))$​ 也符合四边形不等式和区间包含单调性。
4. 若 $h(c)$ 下凸，$w(l,r)$ 符合四边形**恒**等式和区间包含单调性,则 $h(w(l,r))$ 也符合四边形不等式。

例如，在经典的[石子合并问题](https://www.cnblogs.com/RioTian/p/15081277.html)中， $w(l,r) = S[r] - S[l - 1]$  ，稍作推导或根据结论2可以知道，它符合四边形恒等式和区间包含单调性，所以可以使用四边形不等式优化DP。

再比如若 $w(l,r) = (r - l)^2$  ，因为 $h(x) = x^2$ 是单增的下凸函数，而 $r - l$ 符合四边形不等式和区间包含单调性，所以 $h(r - l) = (r - l)^2$ 也符合四边形不等式和区间包含单调性。

---

其实，并 不只有上面那种方程可以用这种方法优化，下面这种方程也有类似的结论：
$$
f[i][j] = \min_{1\le k\le j} f[i - 1][k] + w(k,j)
$$
如果 $w(i,k)$  符合区间包含单调性和四边形不等式，则  $f[i][j]$ 符合四边形不等式，且它的最优决策点 $m[i][j]$ 满足 $m[i - 1][j] \le m[i][j] \le m[j][j + 1]$ 。

虽然这个结论的不等式看起来和上面有点不一样，但本质是相同的，都是m矩阵每一行、每一列单调不减。只是因为DP顺序不同，采取了不同的写法。这次的DP顺序不再是按对角线DP，而是按行DP。需要注意，如果要用四边形不等式优化DP，第二层循环需要逆序进行，因为我们要在获知 $m[i][j]$ 前获知 $m[i][j + 1]$。

例题：[CF321E Ciel and Gondolas](https://link.zhihu.com/?target=https%3A//codeforces.com/contest/321/problem/E)

这道题就是很经典的符合上面方程的例子。用 $f[i][j]$ 表示前 $i$ 艘船载前 $j$ 个人的最小陌生值。很容易写出转移方程：

$f[i][j] = \min\limits_{1\le k\le j}f[i - 1][k - 1] + cal(k,j)$​

其中 $cal(k,j)$ 表示从第 $k$ 个人到 $j$ 个人坐在同一艘船上的陌生值，可以用二位前缀和算出，且显然符合四边形不等式与区间包含单调性。你可能注意到方程里是 $f[i - 1][k - 1]$ 而不是前面提到的 $f[i - 1][k]$ ，但是这里并不会影响结论。

**优化后的核心代码**

```cpp
for (int i = 1; i <= N; ++i)
    m[0][i] = 1, m[i][N + 1] = N; // 初始化边界决策点
memset(f, 63, sizeof(f));
f[0][0] = 0;
for (int i = 1; i <= K; ++i)
    for (int j = N; j >= 1; --j) {
        for (int k = m[i - 1][j]; k <= m[i][j + 1]; ++k)
            if (f[i][j] > f[i - 1][k - 1] + cal(k, j)) {
                f[i][j] = f[i - 1][k - 1] + cal(k, j);
                m[i][j] = k;
            }
    }
```

完整AC代码

```cpp
namespace IO {
char buf[1 << 15], *S, *T;
inline char gc() {
    if (S == T) {
        T = (S = buf) + fread(buf, 1, 1 << 15, stdin);
        if (S == T) return EOF;
    }
    return *S++;
}
inline int read() {
    int x; bool f; char c;
    for (f = 0; (c = gc()) < '0' || c > '9'; f = c == '-');
    for (x = c ^ '0'; (c = gc()) >= '0' && c <= '9'; x = x * 10 + (c ^ '0'));
    return f ? -x : x;
}
const int olim = 1 << 20;
char obuf[olim + 5], *OS;
inline void print() {
    fwrite(obuf, 1, OS - obuf, stdout);
    OS = obuf;
}
inline void pc(char c) {
    *OS++ = c;
    if (OS == obuf + olim) print();
}
inline void write(const char *s, char c = '\n') {
    for (const char *i = s; *i != ' '; ++i)  pc(*i);
    pc(c);
}
inline void write(int x, char c = '\n') {
    if (x < 0) pc('-'), x = -x;
    char temp[20], cnt;
    for (temp[cnt = 0] = x % 10; x /= 10; temp[++cnt] = x % 10);
    for (; ~cnt; --cnt) pc(temp[cnt] ^ '0');
    pc(c);
}
struct flusher {
    flusher() { OS = obuf; }
    ~flusher() { print(); }
} __flusher__;
}

const int N = 4005, INF = 0x3f3f3f3f;
int C[N][N], S[N][N], f[N][N], m[N][N];
int cal(int a, int b) { // 二维前缀和计算
    return S[b][b] - S[b][a - 1] - S[a - 1][b] + S[a - 1][a - 1];
}
void cmin(int &x, int y) {if (x < y) x = y;}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n = IO::read(), K = IO::read();
    for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j) {
            int c = IO::read();
            if (i >= j) C[i][j] = c;
        }
    for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j)
            S[i][j] = C[i][j] + S[i - 1][j] + S[i][j - 1] - S[i - 1][j - 1];
    for (int i = 1; i <= n; ++i) m[0][i] = 1, m[i][n + 1] = n;
    memset(f, 63, sizeof(f));
    f[0][0] = 0;
    for (int i = 1; i <= K; ++i)
        for (int j = n; j >= 1; --j) {
            for (int k = m[i - 1][j]; k <= m[i][j + 1]; ++k) {
                if (f[i][j] > f[i - 1][k - 1] + cal(k, j)) {
                    f[i][j] = f[i - 1][k - 1] + cal(k, j);
                    m[i][j] = k;
                }
            }
        }
    cout << f[K][n];
}
```



[^1]:  严格地说，这里不能取等号，因为取等号时四边形会退化.
[^2]:证明可以参见 https://wenku.baidu.com/view/c44cd84733687e21af45a906.html  
[^3]:下凸即 $h'(x)$​ 单增
