起因：在一场训练赛上。有这么一题没做出来。

题目链接：http://acm.hdu.edu.cn/showproblem.php?pid=6829

题目大意：有三个人，他们分别有$X，Y，Z$块钱（$1<=X,Y,Z<=1e6$)，钱数最多的（如果不止一个那么随机等概率的选一个）随机等可能的选另一个人送他一块钱。直到三个人钱数相同为止。输出送钱轮数的期望，如果根本停不下来，输出-1。

根据题目的意思，其实就是每次向包里随机加入一枚钱币，直到包里某种钱币数量达到 100。本题的核心是如何计算期望。本题属于标准的动态规划求期望问题。直接套用模板即可。

> 一道”简单“概率DP题，没怎么了解概率DP导致做不出

当理解基础的知识以后发现的确比较简单

**DP 数组定义**

定义 $DP[i][j][k]$，表示有 i 枚金币， j 枚银币， k 枚铜币的期望。

**初值**

所有的期望都为零。

**递推方法**

使用逆推。

**状态转移方程：**
$$
s = X + Y + Z\\
dp(i,j,k) = \frac{i}{s}*(dp(i + 1,j,k) + 1) + \frac{j}{s}*dp(i,j + 1,k) + 1) \\+ \frac{j}{s}*dp(i,j,k + 1) + 1))
$$
**AC代码：**

```cpp
// Author : RioTian
// Time : 20/12/09
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 1e2 + 10;
double dp[N][N][N];
int main() {
    // freopen("in.txt", "r", stdin);
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int a, b, c;
    cin >> a >> b >> c;
    for (int i = 99; i >= a; i--)
        for (int j = 99; j >= b; j--)
            for (int k = 99; k >= c; k--) {
                // 令 t = x + y + z，减少代码量
                double t = i + j + k;
                dp[i][j][k] = i / t * (dp[i + 1][j][k] + 1) +
                              j / t * (dp[i][j + 1][k] + 1) +
                              k / t * (dp[i][j][k + 1] + 1);
            }
    // 关于 C++ 的输出控制可以在我的以前博客找到
    cout << fixed << setprecision(9) << dp[a][b][c] << endl;
}
```

**时间和空间复杂度：**

$O(n^3)$

当然在训练赛的题解我也提到也可以用**蒙特卡洛方法模拟**在$O(n)$解决，这里就不再解释了，有兴趣的话可以去[题解报告](https://www.cnblogs.com/RioTian/p/14110922.html#e---probability)的那篇博客查阅

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183415.png"/></div>

做完上面那道概率DP题，算是入了门，但仅仅入门不够的需要加强，所以开始学习各位大神的博客。

下面先说一下[9974](https://jiong.blog.csdn.net/)dalao的总结： 

> 很多概率题总逃不开用dp转移。
>
> 期望题总是倒着推过来的，概率是正着推的，多做题就会理解其中的原因
>
>   有些期望题要用到有关 概率 或 期望的常见$\color{red}公式或思想$
>
> ​		 遇到dp转移方程(组)中有环的，多半逃不出$\color{red}高斯消元$（手动 和 写代码 两种）
>
> ​             这套题中还有道树上的dp转移，还用dfs对方程迭代解方程， 真是大开眼界了 
>
> ​                  当然还有与各种算法结合的题，关键还是要$\color{red}学会分析$
>
> ​                        当公式或计算时有除法时， 特别要注意$\color{red}分母是否为零$

下面几个题型来自Oi wiki 和 [kuangbin](https://www.cnblogs.com/kuangbin/archive/2012/10/02/2710606.html) 的总结

## DP 求概率

这类题目采用顺推，也就是从初始状态推向结果。同一般的 DP 类似的，难点依然是对状态转移方程的刻画，只是这类题目经过了概率论知识的包装。

### "例题 [Codeforces 148 D Bag of mice](https://codeforces.com/problemset/problem/148/D)"

​    题目大意：袋子里有 w 只白鼠和 b 只黑鼠，公主和龙轮流从袋子里抓老鼠。谁先抓到白色老鼠谁就赢，如果袋子里没有老鼠了并且没有谁抓到白色老鼠，那么算龙赢。公主每次抓一只老鼠，龙每次抓完一只老鼠之后会有一只老鼠跑出来。每次抓的老鼠和跑出来的老鼠都是随机的。公主先抓。问公主赢的概率。

设 $f_{i,j}$ 为轮到公主时袋子里有 $i$ 只白鼠， $j$ 只黑鼠，公主赢的概率。初始化边界， $f_{0,j}=0$ 因为没有白鼠了算龙赢， $f_{i,0}=1$ 因为抓一只就是白鼠，公主赢。
考虑 $f_{i,j}$ 的转移：

- 公主抓到一只白鼠，公主赢了。概率为 $\frac{i}{i+j}$ ；
- 公主抓到一只黑鼠，龙抓到一只白鼠，龙赢了。概率为 $\frac{j}{i+j}\cdot \frac{i}{i+j-1}$ ；
- 公主抓到一只黑鼠，龙抓到一只黑鼠，跑出来一只黑鼠，转移到 $f_{i,j-3}$ 。概率为 $\frac{j}{i+j}\cdot\frac{j-1}{i+j-1}\cdot\frac{j-2}{i+j-2}$ ；
- 公主抓到一只黑鼠，龙抓到一只黑鼠，跑出来一只白鼠，转移到 $f_{i-1,j-2}$ 。概率为 $\frac{j}{i+j}\cdot\frac{j-1}{i+j-1}\cdot\frac{i}{i+j-2}$ ；

考虑公主赢的概率，第二种情况不参与计算。并且要保证后两种情况合法，所以还要判断 $i,j$ 的大小，满足第三种情况至少要有 3 只黑鼠，满足第四种情况要有 1 只白鼠和 2 只黑鼠。

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183415.png"/></div>

<details>
<summary>AC 代码(建议独立思考以后再阅读)</summary>
<pre><code class="language-cpp">// Author : RioTian
// Time : 20/12/10
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 1e3 + 10;
int w, b;
double dp[N][N];
int main() {
    // freopen("in.txt", "r", stdin);
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    cin >> w >> b;
    memset(dp, 0.0, sizeof dp);
    for (int i = 1; i <= w; ++i)
        dp[i][0] = 1;
    for (int j = 1; j <= b; ++j)
        dp[0][j] = 0;
    for (int i = 1; i <= w; ++i)
        for (int j = 1; j <= b; ++j) {
            dp[i][j] += (double)i / (i + j);
            if (j >= 3)
                dp[i][j] += (double)j / (i + j) * (j - 1) / (i + j - 1) *
                            (j - 2) / (i + j - 2) * dp[i][j - 3];
            if (i >= 1 && j >= 2)
                dp[i][j] += (double)j / (i + j) * (j - 1) / (i + j - 1) * i /
                            (i + j - 2) * dp[i - 1][j - 2];
        }
    cout << fixed << setprecision(9) << dp[w][b] << endl;
}
</code></pre>
</details>

### 相关习题

-  [POJ3071 Football](http://poj.org/problem?id=3071) 
-  [CodeForces 768 D Jon and Orbs](https://codeforces.com/problemset/problem/768/D) 

## DP 求期望

### " 例题 [POJ2096 Collecting Bugs](http://poj.org/problem?id=2096)"

​    题目大意：一个软件有 s 个子系统，会产生 n 种 bug。某人一天发现一个 bug，这个 bug 属于某种 bug 分类，也属于某个子系统。每个 bug 属于某个子系统的概率是 $\frac{1}{s}$ ，属于某种 bug 分类的概率是 $\frac{1}{n}$ 。求发现 n 种 bug，且 s 个子系统都找到 bug 的期望天数。

令 $f_{i,j}$ 为已经找到 $i$ 种 bug 分类， $j$ 个子系统的 bug，达到目标状态的期望天数。这里的目标状态是找到 $n$ 种 bug 分类， $j$ 个子系统的 bug。那么就有 $f_{n,s}=0$ ，因为已经达到了目标状态，不需要用更多的天数去发现 bug 了，于是就以目标状态为起点开始递推，答案是 $f_{0,0}$ 。

考虑 $f_{i,j}$ 的状态转移：

-  $f_{i,j}$ ，发现一个 bug 属于已经发现的 $i$ 种 bug 分类， $j$ 个子系统，概率为 $p_1=\frac{i}{n}\cdot\frac{j}{s}$ 
-  $f_{i,j+1}$ ，发现一个 bug 属于已经发现的 $i$ 种 bug 分类，不属于已经发现的子系统，概率为 $p_2=\frac{i}{n}\cdot(1-\frac{j}{s})$ 
-  $f_{i+1,j}$ ，发现一个 bug 不属于已经发现 bug 分类，属于 $j$ 个子系统，概率为 $p_3=(1-\frac{i}{n})\cdot\frac{j}{s}$ 
-  $f_{i+1,j+1}$ ，发现一个 bug 不属于已经发现 bug 分类，不属于已经发现的子系统，概率为 $p_4=(1-\frac{i}{n})\cdot(1-\frac{j}{s})$ 

再根据期望的线性性质，就可以得到状态转移方程：

$$
\begin{aligned}
f_{i,j} &= p_1\cdot f_{i,j}+p_2\cdot f_{i,j+1}+p_3\cdot f_{i+1,j}+p_4\cdot f_{i+1,j+1} + 1\\
&=(p_2\cdot f_{i,j+1}+p_3\cdot f_{i+1,j}+p_4\cdot f_{i+1,j+1}+1)/(1-p_1)
\end{aligned}
$$

AC 代码

```cpp
// Author : RioTian
// Time : 20/12/10
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;
const int N = 1e3 + 10;
int n, s;
double dp[N][N];
int main() {
    while (cin >> n >> s) {
        dp[n][s] = 0;
        for (int i = n; i >= 0; i--)
            for (int j = s; j >= 0; j--) {
                if (i == n && j == s)  //跳过初始值
                    continue;
                double p1, p2, p3, p4;
                p1 = (double)i * j / (n * s);        // dp[i][j];
                p2 = (double)(n - i) * j / (n * s);  // dp[i+1][j];
                p3 = (double)i * (s - j) / (n * s);  // dp[i][j+1];
                p4 = (double)(n - i) * (s - j) / (n * s);
                dp[i][j] = (1 + p2 * dp[i + 1][j] + p3 * dp[i][j + 1] +
                            p4 * dp[i + 1][j + 1]) /
                           (1 - p1);
            }
        printf("%.4f\n", dp[0][0]);
    }
}
```



### "例题 [「NOIP2016」换教室](http://uoj.ac/problem/262)"

​    题目大意：牛牛要上 $n$ 个时间段的课，第 $i$ 个时间段在 $c_i$ 号教室，可以申请换到 $d_i$ 号教室，申请成功的概率为 $p_i$ ，至多可以申请 $m$ 节课进行交换。第 $i$ 个时间段的课上完后要走到第 $i+1$ 个时间段的教室，给出一张图 $v$ 个教室 $e$ 条路，移动会消耗体力，申请哪几门课程可以使他因在教室间移动耗费的体力值的总和的期望值最小，也就是求出最小的期望路程和。

对于这个无向连通图，先用 Floyd 求出最短路，为后续的状态转移带来便利。以移动一步为一个阶段（从第 $i$ 个时间段到达第 $i+1$ 个时间段就是移动了一步），那么每一步就有 $p_i$ 的概率到 $d_i$ ，不过在所有的 $d_i$ 中只能选 $m$ 个，有 $1-p_i$ 的概率到 $c_i$ ，求出在 $n$ 个阶段走完后的最小期望路程和。
定义 $f_{i,j,0/1}$ 为在第 $i$ 个时间段，连同这一个时间段已经用了 $j$ 次换教室的机会，在这个时间段换（1）或者不换（0）教室的最小期望路程和，那么答案就是 $max \{f_{n,i,0},f_{n,i,1}\} ,i\in[0,m]$ 。注意边界 $f_{1,0,0}=f_{1,1,1}=0$ 。

考虑 $f_{i,j,0/1}$ 的状态转移：

- 如果这一阶段不换，即 $f_{i,j,0}$ 。可能是由上一次不换的状态转移来的，那么就是 $f_{i-1,j,0}+w_{c_{i-1},c_{i}}$ , 也有可能是由上一次交换的状态转移来的，这里结合条件概率和全概率的知识分析可以得到 $f_{i-1,j,1}+w_{d_{i-1},c_{i}}\cdot p_{i-1}+w_{c_{i-1},c_{i}}\cdot (1-p_{i-1})$ ，状态转移方程就有

$$
\begin{aligned}
f_{i,j,0}=min(f_{i-1,j,0}+w_{c_{i-1},c_{i}},f_{i-1,j,1}+w_{d_{i-1},c_{i}}\cdot p_{i-1}+w_{c_{i-1},c_{i}}\cdot (1-p_{i-1}))
\end{aligned}
$$

- 如果这一阶段交换，即 $f_{i,j,1}$ 。类似地，可能由上一次不换的状态转移来，也可能由上一次交换的状态转移来。那么遇到不换的就乘上 $(1-p_i)$ ，遇到交换的就乘上 $p_i$ ，将所有会出现的情况都枚举一遍出进行计算就好了。这里不再赘述各种转移情况，相信通过上一种阶段例子，这里的状态转移应该能够很容易写出来。

AC代码    
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 2010;
int n, m, v, e;
int f[maxn][maxn], c[maxn], d[maxn];
double dp[maxn][maxn][2], p[maxn];

int main() {
    scanf("%d %d %d %d", &n, &m, &v, &e);
    for (int i = 1; i <= n; i++)
        scanf("%d", &c[i]);
    for (int i = 1; i <= n; i++)
        scanf("%d", &d[i]);
    for (int i = 1; i <= n; i++)
        scanf("%lf", &p[i]);
    for (int i = 1; i <= v; i++)
        for (int j = 1; j < i; j++)
            f[i][j] = f[j][i] = 1e9;

    int u, V, w;
    for (int i = 1; i <= e; i++) {
        scanf("%d %d %d", &u, &V, &w);
        f[u][V] = f[V][u] = min(w, f[u][V]);
    }

    for (int k = 1; k <= v; k++)
        for (int i = 1; i <= v; i++)
            for (int j = 1; j < i; j++)
                if (f[i][k] + f[k][j] < f[i][j])
                    f[i][j] = f[j][i] = f[i][k] + f[k][j];

    for (int i = 1; i <= n; i++)
        for (int j = 0; j <= m; j++)
            dp[i][j][0] = dp[i][j][1] = 1e9;

    dp[1][0][0] = dp[1][1][1] = 0;
    for (int i = 2; i <= n; i++)
        for (int j = 0; j <= min(i, m); j++) {
            dp[i][j][0] =
                min(dp[i - 1][j][0] + f[c[i - 1]][c[i]],
                    dp[i - 1][j][1] + f[c[i - 1]][c[i]] * (1 - p[i - 1]) +
                        f[d[i - 1]][c[i]] * p[i - 1]);
            if (j != 0) {
                dp[i][j][1] =
                    min(dp[i - 1][j - 1][0] + f[c[i - 1]][d[i]] * p[i] +
                            f[c[i - 1]][c[i]] * (1 - p[i]),
                        dp[i - 1][j - 1][1] +
                            f[c[i - 1]][c[i]] * (1 - p[i - 1]) * (1 - p[i]) +
                            f[c[i - 1]][d[i]] * (1 - p[i - 1]) * p[i] +
                            f[d[i - 1]][c[i]] * (1 - p[i]) * p[i - 1] +
                            f[d[i - 1]][d[i]] * p[i - 1] * p[i]);
            }
        }

    double ans = 1e9;
    for (int i = 0; i <= m; i++)
        ans = min(dp[n][i][0], min(dp[n][i][1], ans));
    printf("%.2lf", ans);

    return 0;
}
```

比较这两个问题可以发现，DP 求期望题目在对具体是求一个值或是最优化问题上会对方程得到转移方式有一些影响，但无论是 DP 求概率还是 DP 求期望，总是离不开概率知识和列出、化简计算公式的步骤，在写状态转移方程时需要思考的细节也类似。

### 习题

-  [HDU3853 LOOPS](http://acm.hdu.edu.cn/showproblem.php?pid=3853) 
-  [HDU4035 Maze](http://acm.hdu.edu.cn/showproblem.php?pid=4035) 
-  [「SCOI2008」奖励关](https://www.luogu.com.cn/problem/P2473) 

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183415.png"/></div>

## 有后效性 DP

### 例题 "[CodeForces 24 D Broken robot](https://codeforces.com/problemset/problem/24/D)"

​    题目大意：给出一个 $n*m$ 的矩阵区域，一个机器人初始在第 $x$ 行第 $y$ 列，每一步机器人会等概率地选择停在原地，左移一步，右移一步，下移一步，如果机器人在边界则不会往区域外移动，问机器人到达最后一行的期望步数。

在 $m=1$ 时每次有 $\frac{1}{2}$ 的概率不动，有 $\frac{1}{2}$ 的概率向下移动一格，答案为 $2\cdot (n-x)$ 。
设 $f_{i,j}$ 为机器人机器人从第 i 行第 j 列出发到达第 $n$ 行的期望步数，最终状态为 $f_{n,j}=0$ 。
由于机器人会等概率地选择停在原地，左移一步，右移一步，下移一步，考虑 $f_{i,j}$ 的状态转移：

-  $f_{i,1}=\frac{1}{3}\cdot(f_{i+1,1}+f_{i,2}+f_{i,1})+1$ 
-  $f_{i,j}=\frac{1}{4}\cdot(f_{i,j}+f_{i,j-1}+f_{i,j+1}+f_{i+1,j})+1$ 
-  $f_{i,m}=\frac{1}{3}\cdot(f_{i,m}+f_{i,m-1}+f_{i+1,m})+1$ 

在行之间由于只能向下移动，是满足无后效性的。在列之间可以左右移动，在移动过程中可能产生环，不满足无后效性。
将方程变换后可以得到：

-  $2f_{i,1}-f_{i,2}=3+f_{i+1,1}$ 
-  $3f_{i,j}-f_{i,j-1}-f_{i,j+1}=4+f_{i+1,j}$ 
-  $2f_{i,m}-f_{i,m-1}=3+f_{i+1,m}$ 

由于是逆序的递推，所以每一个 $f_{i+1,j}$ 是已知的。
由于有 $m$ 列，所以右边相当于是一个 $m$ 行的列向量，那么左边就是 $m$ 行 $m$ 列的矩阵。使用增广矩阵，就变成了 m 行 m+1 列的矩阵，然后进行 [高斯消元](../math/gauss.md) 即可解出答案。

AC代码：这个有点绕，代码博主没有自己写，copy下dalao的代码了（侵权删）。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1e3 + 10;

double a[maxn][maxn], f[maxn];
int n, m;

void solve(int x) {
    memset(a, 0, sizeof a);
    for (int i = 1; i <= m; i++) {
        if (i == 1) {
            a[i][i] = 2;
            a[i][i + 1] = -1;
            a[i][m + 1] = 3 + f[i];
            continue;
        } else if (i == m) {
            a[i][i] = 2;
            a[i][i - 1] = -1;
            a[i][m + 1] = 3 + f[i];
            continue;
        }
        a[i][i] = 3;
        a[i][i + 1] = -1;
        a[i][i - 1] = -1;
        a[i][m + 1] = 4 + f[i];
    }

    for (int i = 1; i < m; i++) {
        double p = a[i + 1][i] / a[i][i];
        a[i + 1][i] = 0;
        a[i + 1][i + 1] -= a[i][i + 1] * p;
        a[i + 1][m + 1] -= a[i][m + 1] * p;
    }

    f[m] = a[m][m + 1] / a[m][m];
    for (int i = m - 1; i >= 1; i--)
        f[i] = (a[i][m + 1] - f[i + 1] * a[i][i + 1]) / a[i][i];
}

int main() {
    scanf("%d %d", &n, &m);
    int st, ed;
    scanf("%d %d", &st, &ed);
    if (m == 1) {
        printf("%.10f\n", 2.0 * (n - st));
        return 0;
    }
    for (int i = n - 1; i >= st; i--) {
        solve(i);
    }
    printf("%.10f\n", f[ed]);
    return 0;
}
```
### 习题

-  [HDU Time Travel](http://acm.hdu.edu.cn/showproblem.php?pid=4418) 
-  [「HNOI2013」游走](https://loj.ac/problem/2383) 

## 参考文献

 [kuangbin 概率 DP 总结](https://www.cnblogs.com/kuangbin/archive/2012/10/02/2710606.html) 

[ACM里的期望和概率问题 从入门到精通](https://www.cnblogs.com/idyllic/p/13460122.html)

**论文学习：**

**[《信息学竞赛中概率问题求解初探》](http://wenku.baidu.com/view/1c41152de2bd960590c677a8.html)**

[**《浅析竞赛中一类数学期望问题的解决方法》**](http://wenku.baidu.com/view/90adb02acfc789eb172dc8a8.html)

[**《有关概率和期望问题的研究 》**](http://wenku.baidu.com/view/56147518a8114431b90dd81e.html)

关于一些练习题：

**[概率dp总结](https://www.zybuluo.com/01010101/note/1308805)**