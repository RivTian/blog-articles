> 本文转载自 Pecco桑的博文：[Here](https://zhuanlan.zhihu.com/p/402195848)
>
> 悬线法前文：[Here](https://www.cnblogs.com/RioTian/p/15039583.html)

**悬线法**是一种比较简单的DP技巧，主要用于在 $\mathcal{O}(nm)$​  的时间内解决最大矩形/最大正方形问题，即，给出一个 $n\times m$​  的方格矩阵，其中某些方格是不可选的，求可选的最大矩形/正方形。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210824151340.jpg)

其实这个问题用单调栈也是可以 $\mathcal{O}(nm)$ 解决的，但悬线法是一种思路更简单的方法。我们定义“**悬线**”是从某一格出发，一直向上能延申的最长竖线（如下图)。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210824151431.jpg)

我们定义**极大矩形**是无法再向上、下、左或右拓展的矩形，很明显，每一个**最大矩形**，都一定是一个**极大矩形**（否则拓展后的矩形比它更大，它就不是最大的了）。

一个重要的事实是：每个极大矩形都一定可以由某条**悬线**“**拓展**”（即把悬线尽可能地往左、往右平移）得到。实际上，在矩形上边界上找到一个不能再向上的方格（作为极大矩形，一定存在这样的方格），把它跟矩形下边界的对应格连线，即可得到这样的悬线。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210824151452.jpg)

由于一个可选的格子对应一条悬线，所以悬线最多只有 $nm$  条。因此，我们只需要枚举最多  $nm$ 条悬线就可以枚举所有的极大矩形，最大矩形一定就在其中。

接下来是十分简单的DP环节。定义 $u,l,r$ 分别表示从某格出发向上、左、右最多能走多少格（包括自身)，则：


$$
u[x][y]=\left\{\begin{array}{ll}
u[x-1][y]+1, & \text { 若 }(x-1, y) \text { 可选 } \\
1, & \text { 若 }(x-1, y) \text { 不可选 }
\end{array}\right.\\
l[x][y]=\left\{\begin{array}{ll}
l[x][y-1]+1, & \text { 若 }(x, y-1) \text { 可选 } \\
1, & \text { 若 }(x, y-1) \text { 不可选 }
\end{array}\right.\\
r[x][y]=\left\{\begin{array}{ll}
r[x][y+1]+1, & \text { 若 }(x, y+1) \text { 可选 } \\
1, & \text { 若 }(x, y+1) \text { 不可选 }
\end{array}\right.
$$
然后我们定义 $L、R$  分别表示从当前格向上的悬线最多能向左、向右拓展多少格，则：
$$
\begin{array}{l}
L[x][y]=\left\{\begin{array}{ll}
\min (l[x][y], L[x-1][y]), & \text { 若 }(x-1, y) \text { 可选 } \\
l[x][y], & \text { 若 }(x-1, y) \text { 不可选 }
\end{array}\right. \\
R[x][y]=\left\{\begin{array}{ll}
\min (r[x][y], R[x-1][y]), & \text { 若 }(x-1, y) \text { 可选 } \\
r[x][y], & \text { 若 }(x-1, y) \text { 不可选 }
\end{array}\right.
\end{array}
$$
实际上我们发现 $l$ 和 $L、r$ 和 $R$ 分别可以用同一个数组来存，可以节省很多空间。对于每一格来说，它对应的悬线左右拓展能得到的最大的矩形面积为：

$u[x][y] \times (L[x][y]+R[x][y] - 1)$ 

主要代码如下：

```cpp
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= m; ++j)
        if (A[i][j])
            l[i][j] = l[i][j - 1] + 1;
for (int i = 1; i <= n; ++i)
    for (int j = m; j >= 1; --j)
        if (A[i][j])
            r[i][j] = r[i][j + 1] + 1;
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= m; ++j)
        if (A[i][j]) {
            u[i][j] = u[i - 1][j] + 1;
            if (A[i - 1][j]) cmin(l[i][j], l[i - 1][j]), cmin(r[i][j], r[i - 1][j]);
            cmax(ans, (r[i][j] + l[i][j] - 1) * u[i][j]);
        }
```

现在我们解决了最大矩形问题，那么**最大正方形**呢？很简单，因为最大正方形一定是最大矩形的**子正方形**。所以我们只需枚举所有悬线，计算每条悬线所对应矩形的最大子正方形面积：

$\min(u[x][y],L[x][y] + R[x]][y] - 1)^2$​

---

$$
QAQ
$$

---

来看几道稍微有些变式的例题：

**[[ZJOI2007\]棋盘制作](https://vjudge.net/problem/%E9%BB%91%E6%9A%97%E7%88%86%E7%82%B8-1057)**

> **题目描述**
> 国际象棋是世界上最古老的博弈游戏之一，和中国的围棋、象棋以及日本的将棋同享盛名。据说国际象棋起源于易经的思想，棋盘是一个 $8\times 8$ 大小的黑白相间的方阵，对应八八六十四卦，黑白对应阴阳。
> 而我们的主人公`小Q`，正是国际象棋的狂热爱好者。作为一个顶尖高手，他已不满足于普通的棋盘与规则，于是他跟他的好朋友`小W`决定将棋盘扩大以适应他们的新规则。
> `小Q`找到了一张由 $N\times M$ 个正方形的格子组成的矩形纸片，每个格子被涂有黑白两种颜色之一。`小Q`想在这种纸中裁减一部分作为新棋盘，当然，他希望这个棋盘尽可能的大。
> 不过`小Q`还没有决定是找一个正方形的棋盘还是一个矩形的棋盘（当然，不管哪种，棋盘必须都黑白相间，即相邻的格子不同色），所以他希望可以找到最大的正方形棋盘面积和最大的矩形棋盘面积，从而决定哪个更好一些。
> 于是`小Q`找到了即将参加全国信息学竞赛的你，你能帮助他么？
> **输入格式**
> 包含两个整数 $N$ 和 $M$ ，分别表示矩形纸片的长和宽。接下来的 $N$ 行包含一个 $N\times M$ 的 $01$ 矩阵，表示这张矩形纸片的颜色（ $0$ 表示白色， $1$ 表示黑色）。
> **输出格式**
> 包含两行，每行包含一个整数。第一行为可以找到的最大正方形棋盘的面积，第二行为可以找到的最大矩形棋盘的面积（注意正方形和矩形是可以相交或者包含的)。

这个题换成了找最大的棋盘格，但其实只需对定义稍作修改，上面那些分析就基本可以沿用。甚至上面的状态转移方程都继续成立，只不过某格可不可选不再只依赖于它自身，而是取决于它与相邻格的关系。

【AC Code】

```cpp
const int N = 2e3 + 10;
int a[N][N];
int H[N][N], L[N][N], R[N][N];
void chmin(int &a, int b) { return (void)(a > b ? a = b : a = a); }
void chmax(int &a, int b) { return (void)(a < b ? a = b : a = a); }
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int n, m; cin >> n >> m;
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            cin >> a[i][j], L[i][j] = R[i][j] = j, H[i][j] = 1;

    for (int i = 1; i <= n; ++i) {
        for (int j = 2; j <= m; ++j)
            L[i][j] = a[i][j] == a[i][j - 1] ? j : L[i][j - 1];
        for (int j = m - 1; j >= 1; --j)
            R[i][j] = a[i][j] == a[i][j + 1] ? j : R[i][j + 1];
    }

    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            if (i > 1 and a[i][j] != a[i - 1][j]) {
                H[i][j] = H[i - 1][j] + 1;
                chmax(L[i][j], L[i - 1][j]);
                chmin(R[i][j], R[i - 1][j]);
            }

    int ans1 = 0, ans2 = 0;
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j) {
            int a = H[i][j], b = R[i][j] - L[i][j] + 1;
            if (a > b) swap(a, b);
            chmax(ans1, a * a);
            chmax(ans2, a * b);
        }
    cout << ans1 << "\n" << ans2;
}
```

（[**HDU6957 Maximal submatrix**](https://vjudge.net/problem/HDU-6957)）From 2021杭电多校1

> **Problem Description**
> Given a matrix of n rows and m columns,find the largest area submatrix which is non decreasing on each column
> **Input**
> The first line contains an integer  $T(1\le T\le10)$ representing the number of test cases.
> For each test case, the first line contains two integers $n,m(1\le n,m\le 2\times10^3)$ representing the size of the matrix
> the next  $n$ line followed. the $i$-th line contains  $m$ integers $v_{i,j}$ ( $1\le v_{i,j}\le 5\times10^3$  )representing the value of matrix
> It is guaranteed that there are no more than 2 testcases with $nm > 10000$
> **Output**
> For each test case, print a integer representing the Maximal submatrix

这题要求最大的每列均为不下降序列的矩阵，在稍作转换后可以变成最大矩形问题。我们比较原来矩阵中所有上下相邻的数，如果下面不小于上面，则在两者间插入一个 $1$ ，否则插入一个 $0$ 。插入完成后，把原来矩阵的数都附为 $0$。这样，我们得到一个 $01$ 矩阵，而且这个01矩阵的最大矩形和原矩阵是对应的。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210824152103.png)

注意计算面积时要把高度除以2（向上取整）。另外这个题有点卡空间，也许需要用`short`代替`int`之类的卡空间技巧。

【AC Code】

```cpp
const int N = 2e3 + 10;
// a为原矩阵，b转为01矩阵
int a[N][N], b[N][N];
int H[N], Q[N];

int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int _; for (cin >> _; _--;) {
        int n, m;
        cin >> n >> m;
        for (int i = 1; i <= n; ++i)
            for (int j = 1; j <= m; ++j) {
                cin >> a[i][j];
                b[i][j] = 0; // init
                if (i > 1)b[i][j] = (a[i][j] >= a[i - 1][j]);
            }

        for (int i = 1; i <= m; ++i) H[i] = 0;

        int ans = 0;
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j <= m; ++j) {
                if (b[i][j] == 0) H[j] = 1;
                else H[j]++;
            }
            int cnt = 0;
            H[m + 1] = 0;
            for (int j = 1; j <= m + 1; ++j) {
                while (cnt and H[Q[cnt]] > H[j]) {
                    ans = max(ans, (j - Q[cnt - 1] - 1) * H[Q[cnt]]);
                    --cnt;
                }
                Q[++cnt] = j;
            }
        }
        cout << ans << '\n';
    }
}
```

