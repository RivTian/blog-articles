> 前置芝士：[Here](https://www.cnblogs.com/RioTian/p/13939316.html)
>
> 本文是基于 OI wiki 上的文章加以修改完成，感谢社区的转载支持和其他方面的支持

树形 DP，即在树上进行的 DP。由于树固有的递归性质，树形 DP 一般都是递归进行的。

## 基础

以下面这道题为例，介绍一下树形 DP 的一般过程。

例题 [洛谷 P1352 没有上司的舞会](https://www.luogu.com.cn/problem/P1352)

<details open="" class="warn">
<summary>题目描述 </summary> 
<p>某大学有 $n$ 个职员，编号为 $1 \sim N$。他们之间有从属关系，也就是说他们的关系就像一棵以校长为根的树，父结点就是子结点的直接上司。现在有个周年庆宴会，宴会每邀请来一个职员都会增加一定的快乐指数 $a_i$，但是呢，如果某个职员的上司来参加舞会了，那么这个职员就无论如何也不肯来参加舞会了。所以，请你编程计算，邀请哪些职员可以使快乐指数最大，求最大的快乐指数。
</p>
</details><br>

我们可以定义 $f(i,0/1)$ 代表以 $i$ 为根的子树的最优解（第二维的值为 0 代表 $i$ 不参加舞会的情况，1 代表 $i$ 参加舞会的情况）。

显然，我们可以推出下面两个状态转移方程（其中下面的 $x$ 都是 $i$ 的儿子）：

- $f(i,0) = \sum\max \{f(x,1),f(x,0)\}$（上司不参加舞会时，下属可以参加，也可以不参加）
- $f(i,1) = \sum{f(x,0)} + a_i$（上司参加舞会时，下属都不会参加）

我们可以通过 DFS，在返回上一层时更新当前结点的最优解。

<details> 
<summary>代码</summary>
<pre><code class="language-cpp">const int N = 1e4 + 10;
vector<int> tr[N];
int f[N][2], v[N], Happy[N], n;
void dfs(int u) {
    f[u][0] = 0; f[u][1] = Happy[u];
    for (auto v : tr[u]) {
        dfs(v);
        f[u][0] += max(f[v][0], f[v][1]);
        f[u][1] += f[v][0];
    }
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> Happy[i];
    for (int i = 1, x, y; i < n; ++i) {
        cin >> x >> y;
        v[x] = 1;// x has a father
        tr[y].push_back(x);
    }
    int root;
    for (int i = 1; i <= n; ++i)
        if (!v[i]) {root = i; break;}
    dfs(root);
    cout << max(f[root][0], f[root][1]) << "\n";
}
</code></pre>
</details><br>

### 相关练习

- [HDU 2196 Computer](http://acm.hdu.edu.cn/showproblem.php?pid=2196)，[题解](https://www.cnblogs.com/RioTian/p/13751877.html)

- [POJ 1463 Strategic game](http://poj.org/problem?id=1463)

- [POI2014 FAR-FarmCraft](https://www.luogu.com.cn/problem/P3574) ，[题解](https://www.cnblogs.com/RioTian/p/15111199.html)

## 树上背包

树上的背包问题，简单来说就是背包问题与树形 DP 的结合。

例题 [洛谷 P2014 CTSC1997 选课](https://www.luogu.com.cn/problem/P2014)

<details open="" class="warn">
<summary>题目描述 </summary> 
<p> 现在有 $n$ 门课程，第 $i$ 门课程的学分为 $a_i$，每门课程有零门或一门先修课，有先修课的课程需要先学完其先修课，才能学习该课程。<br> 一位学生要学习 $m$ 门课程，求其能获得的最多学分数。
<br>
$n,m \le 300$
</p>
</details><br>

每门课最多只有一门先修课的特点，与有根树中一个点最多只有一个父亲结点的特点类似。

因此可以想到根据这一性质建树，从而所有课程组成了一个森林的结构。为了方便起见，我们可以新增一门 $0$ 学分的课程（设这个课程的编号为 $0$），作为所有无先修课课程的先修课，这样我们就将森林变成了一棵以 $0$ 号课程为根的树。

我们设 $f(u,i,j)$ 表示以 $u$ 号点为根的子树中，已经遍历了 $u$ 号点的前 $i$ 棵子树，选了 $j$ 门课程的最大学分。

转移的过程结合了树形 DP 和背包 DP 的特点，我们枚举 $u$ 点的每个子结点 $v$，同时枚举以 $v$ 为根的子树选了几门课程，将子树的结果合并到 $u$ 上。

记点 $x$ 的儿子个数为 $s_x$，以 $x$ 为根的子树大小为 $siz_x$，很容易写出下面的转移方程：

$$
f(u,i,j)=\max_{v,k \leq j,k \leq siz_v} f(u,i-1,j-k)+f(v,s_v,k)
$$

注意上面转移方程中的几个限制条件，这些限制条件确保了一些无意义的状态不会被访问到。

$f$ 的第二维可以很轻松地用滚动数组的方式省略掉，注意这时需要倒序枚举 $j$ 的值。

我们可以证明，该做法的时间复杂度为 $O(nm)$[^note1]。

<details> 
<summary>代码</summary>
<pre><code class="language-cpp">const int N = 310;
vector<int>e[N];
int f[N][N], s[N], n, m;
void dfs(int x) {
    f[x][0] = 0;
    for (int v : e[x]) { // 循环子节点（物品）
        dfs(v);
        for (int t = m; t >= 0; --t)     // 倒序循环当前选课总门数（当前背包体积）
            for (int j = 0; j <= t; ++j) // 循环更深子树上的选课门数（组内物品）
                f[x][t] = max(f[x][t], f[x][t - j] + f[v][j]);
        /* 或者
            for (int j = t; j >= 0; j--)
                if (t + j <= m)
                    f[x][t+j] = max(f[x][t+j], f[x][t] + f[y][j]);
            这两种写法j分别用了正序和倒序循环
            是为了正确处理组内体积为0的物品（本题正序倒序都可以AC是因为体积为0的物品价值恰好也为0）
            请读者结合0/1背包问题中DP的“阶段”理论思考 */
    }
    if (x != 0) // x不为0时，选修x本身需要占用1门课，并获得相应学分
        for (int t = m; t > 0; t--) f[x][t] = f[x][t - 1] + s[x];
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    cin >> n >> m;
    for (int i = 1, x; i <= n; ++i) {
        cin >> x >> s[i];
        e[x].push_back(i);
    }
    memset(f, 0xcf, sizeof(f)); // -inf
    dfs(0);
    cout << f[0][m] << "\n";
}
</code></pre>
</details><br>

### 相关练习

- [「CTSC1997」选课](https://www.luogu.com.cn/problem/P2014)
- [「JSOI2018」潜入行动](https://loj.ac/problem/2546)
- [「SDOI2017」苹果树](https://loj.ac/problem/2268)



## 换根 DP

树形 DP 中的换根 DP 问题又被称为二次扫描，通常不会指定根结点，并且根结点的变化会对一些值，例如子结点深度和、点权和等产生影响。

通常需要两次 DFS，第一次 DFS 预处理诸如深度，点权和之类的信息，在第二次 DFS 开始运行换根动态规划。

接下来以一些例题来带大家熟悉这个内容。

例题 [[POI2008]STA-Station](https://www.luogu.com.cn/problem/P3478)

<details open="" class="warn">
<summary>题目描述 </summary> 
<p>给定一个 $n$ 个点的树，请求出一个结点，使得以这个结点为根时，所有结点的深度之和最大。
</p>
</details><br>

> 注意题目的样例给的输出是错误，正确的输出是 $24$ 

不妨令 $u$ 为当前结点，$v$ 为当前结点的子结点。首先需要用 $s_i$ 来表示以 $i$ 为根的子树中的结点个数，并且有 $s_u=1+\sum s_v$。显然需要一次 DFS 来计算所有的 $s_i$，这次的 DFS 就是预处理，我们得到了以某个结点为根时其子树中的结点总数。

考虑状态转移，这里就是体现＂换根＂的地方了。令 $f_u$ 为以 $u$ 为根时，所有结点的深度之和。

$f_v\leftarrow f_u$ 可以体现换根，即以 $u$ 为根转移到以 $v$ 为根。显然在换根的转移过程中，以 $v$ 为根或以 $u$ 为根会导致其子树中的结点的深度产生改变。具体表现为：

- 所有在 $v$ 的子树上的结点深度都减少了一，那么总深度和就减少了 $s_v$；

- 所有不在 $v$ 的子树上的结点深度都增加了一，那么总深度和就增加了 $n-s_v$；

根据这两个条件就可以推出状态转移方程 $f_v = f_u - s_v + n - s_v=f_u + n - 2 \times s_v$。

于是在第二次 DFS 遍历整棵树并状态转移 $f_v=f_u + n - 2 \times s_v$，那么就能求出以每个结点为根时的深度和了。最后只需要遍历一次所有根结点深度和就可以求出答案。



<details> 
<summary>代码</summary>
<pre><code class="language-cpp">using pii = pair<int, int>;
const int N = 2e5 + 10;
vector<pii>e[N];
int f[N], d[N], ff[N];
void dfs(int u, int fa) {
    for (auto [v, w] : e[u]) {
        if (v == fa) continue;
        dfs(v, u);
        if (d[v] == 1) f[u] += w;
        else f[u] += min(f[v], w);
    }
}
void dfs1(int u, int fa) {
    ff[u] = f[u];
    for (auto [v, w] : e[u]) {
        if (v == fa)continue;
        if (d[v] == 1) {
            f[u] -= w;
            f[v] += min(f[u], w);
        } else {
            f[u] -= min(w, f[v]);
            f[v] += min(w, f[u]);
        }
        dfs1(v, u);
    }
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int _; for (cin >> _; _--;) {
        int n; cin >> n;
        for (int i = 1; i <= n; ++i) {
            e[i].clear();
            d[i] = f[i] = ff[i] = 0;
        }
        for (int i = 1, u, v, w; i < n; ++i) {
            cin >> u >> v >> w;
            e[u].push_back({v, w});
            e[v].push_back({u, w});
            d[u]++, d[v]++;
        }
        dfs(1, -1);
        dfs1(1, -1);
        int ans = 0;
        for (int i = 1; i <= n; ++i) ans = max(ans, ff[i]);
        cout << ans << "\n";
    }
}
</code></pre>
</details><br>



### 相关练习

- [POJ 3585 Accumulation Degree](http://poj.org/problem?id=3585)

- [POI2008 STA-Station](https://www.luogu.com.cn/problem/P3478)

- [USACO10MAR Great Cow Gathering G](https://www.luogu.com.cn/problem/P2986)

- [CodeForce 708C Centroids](http://codeforces.com/problemset/problem/708/C)



## 参考资料与注释

[^note1]: [子树合并背包类型的 dp 的复杂度证明 - LYD729 的 CSDN 博客](https://blog.csdn.net/lyd_7_29/article/details/79854245)