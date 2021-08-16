**数位DP**用于处理一些与**数位**有关的问题，主要是计数问题。我们从一道例题开始：

(**[HDU2089 不要62](https://vjudge.net/problem/HDU-2089)**)

> **Problem Description**
> 杭州人称那些傻乎乎粘嗒嗒的人为62（音：laoer）。
> 杭州交通管理局经常会扩充一些的士车牌照，新近出来一个好消息，以后上牌照，不再含有不吉利的数字了，这样一来，就可以消除个别的士司机和乘客的心理障碍，更安全地服务大众。
> 不吉利的数字为所有含有4或62的号码。例如：
> 62315 73418 88914
> 都属于不吉利号码。但是，61152虽然含有6和2，但不是62连号，所以不属于不吉利数字之列。
> 你的任务是，对于每次给出的一个牌照区间号，推断出交管局今次又要实际上给多少辆新的士车上牌照了。
> **Input**
> 输入的都是整数对n、m（$0<n≤m<1000000$），如果遇到都是0的整数对，则输入结束。
> **Output**
> 对于每个整数对，输出一个不含有不吉利数字的统计个数，该数值占一行位置。

为了简化问题，我们只需要求出 $[0,m]$ 内符合条件的值，以及 $[0,n - 1]$ 内符合条件的值，再把两者相减即可。首先，我们要把数字拆成数位的数组，一般我喜欢从高位到低位存储，这样需要`reverse`一下：

```cpp
cnt = 0;
while (x)
    A[cnt++] = x % 10, x /= 10;
reverse(A, A + cnt);
```

我们一般使用**记忆化搜索**的方法来实现数位DP。例如，如果我们要求 $231756$ 以内符合条件的数，我们只需要知道形如 $0?????、1?????$ 或 $2?????$ ，且小于等于原数的符合条件的数的数量即可。

这样直接递归地搜索下去，会遍历所有范围内的值，所以我们要记忆化。数位DP和其他很多DP一样，减少复杂度的核心就是合并相同状态。例如在本题中，$106???$ 和 $206???$ 的答案是一样的，所以他们可以当作同样的状态。当我们搜到 $206???$ 时，发现这种状态的答案已经被记录过了，于是可以直接返回。

不过你肯定发现，上面的 $2?????$ 和另两种不同，因为它的第一个 $?$  只能取 0,1,2,3。所以我们引入一个常用的状态 $limit$ 。它表示，当前位是最多只能取到原数这一位的值，还是可以任意取 0 ~ 9  。显然，某位 limit 为 1当且仅当上一位  limit 为 1 且上一位已取到最大值。

对于本题而言，除了 $limit$ 和代表正在处理的位数的 $pos$ 外，只需要确定上一位的数字（ $last$  )，就可以确定一个状态，所以代码如下：

```cpp
int A[8], cnt, dp[8][12][2];
int dfs(int pos, int last, bool limit) {
    int ans = 0;
    if (pos == cnt)
        return 1; // 搜索终点
    if (dp[pos][last][limit] != -1)
        return dp[pos][last][limit];
    for (int v = 0; v <= (limit ? A[pos] : 9); ++v) { // 根据是否limit决定循环上界
        if (last == 6 && v == 2 || v == 4) // 舍弃不合法解
            continue;
        ans += dfs(pos + 1, v, limit && v == A[pos]);
    }
    dp[pos][last][limit] = ans;
    return ans;
}
int f(int x) {
    cnt = 0;
    memset(A, 0, sizeof(A));
    memset(dp, -1, sizeof(dp)); // 初始化dp数组为-1
    while (x)
        A[cnt++] = x % 10, x /= 10;
    reverse(A, A + cnt);
    return dfs(0, 11, true);    // 用last为11表示不存在上一位
}
int main() {
    ios::sync_with_stdio(false);
    int x, y;
    while (cin >> x >> y, x || y) {
        int l = f(x - 1), r = f(y);
        cout << r - l << endl;
    }
    return 0;
}
```

再来看一道例题：

(**[洛谷P2602 ZJOI2010\][数字计数](https://vjudge.net/problem/LibreOJ-10169)**)

> **题目描述**
>
> 给定两个正整数 a 和 b，求在 $[a,b]$ 中的所有整数中，每个数码 (digit) 各出现了多少次。
>
> 输入格式
>
> 仅包含一行两个整数 a,b，含义如上所述。
>
> 输出格式
>
> 包含一行 10 个整数，分别表示 0∼9 在 [a,b] 中出现了多少次。

这也是类似的方法，但是需要注意，我们这次要留心处理前导零，否则 $0$ 的出现次数会被多算。我们用一个状态 $lead$ 记录当前位之前是否均为0。如果某位之前均为0，且它自身也为0，那么它就是前导零。

```cpp
using ll = long long;
ll A[22], cnt, digit, dp[22][22][2][2];
ll dfs(int pos, int cntd, bool limit, bool lead) { // cntd表示目前为止已经找到多少个digit
    ll ans = 0;
    if (pos == cnt)
        return cntd;
    if (dp[pos][cntd][limit][lead] != -1)
        return dp[pos][cntd][limit][lead];
    for (int v = 0; v <= (limit ? A[pos] : 9); ++v)
        if (lead && v == 0)
            ans += dfs(pos + 1, cntd, limit && v == A[pos], true);
        else
            ans += dfs(pos + 1, cntd + (v == digit), limit && v == A[pos], false);
    dp[pos][cntd][limit][lead] = ans;
    return ans;
}
ll f(ll x) {
    cnt = 0;
    memset(dp, -1, sizeof(dp));
    memset(A, 0, sizeof(A));
    while (x)
        A[cnt++] = x % 10, x /= 10;
    reverse(A, A + cnt);
    return dfs(0, 0, true, true);
}
int main() {
    ios::sync_with_stdio(false);
    ll x, y;
    cin >> x >> y;
    for (int i = 0; i <= 9; ++i) {
        digit = i;
        ll l = f(x - 1), r = f(y);
        cout << r - l << " ";
    }
    return 0;
}
```

实际上，不仅是十进制可以数位DP，二进制也可以，很多与按位与/或/异或有关的题目，都可以用数位DP解决。

(**[CF276D Little Girl and Maximum XOR](https://codeforces.com/problemset/problem/276/D)**)

~~题目就不贴了~~

这题存在一个非常短的贪心写法，但如果没想到也可以数位DP。需要注意的是，这不是计数问题，所以不能用类似于`f(r)-f(l-1)`的方法。可以对上、下界分别设`limit`标记来解决。同时，因为要枚举两个数，要分别为两个数设上下标记，即一共要设4个标记。

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
ll L[64], R[64], cnt, dp[64][2][2][2][2];
ll dfs(int pos, int limitl1, int limitr1, int limitl2, int limitr2) {
    ll ans = 0;
    if (pos == cnt)
        return 0;
    auto &d = dp[pos][limitl1][limitr1][limitl2][limitr2];
    if (d != -1)
        return d;
    for (ll u = (limitl1 ? L[pos] : 0); u <= (limitr1 ? R[pos] : 1); ++u)
        for (ll v = (limitl2 ? L[pos] : 0); v <= (limitr2 ? R[pos] : 1); ++v) // 要找两个数所以是二重循环
            ans = max(ans, ((u ^ v) << (cnt - pos - 1)) + dfs(pos + 1, limitl1 && u == L[pos], limitr1 && u == R[pos], limitl2 && v == L[pos], limitr2 && v == R[pos]));
    d = ans;
    return ans;
}
ll f(ll x, ll y) {
    int cnt1 = 0, cnt2 = 0;
    memset(dp, -1, sizeof(dp));
    memset(L, 0, sizeof(L));
    memset(R, 0, sizeof(R));
    while (x)
        L[cnt1++] = x & 1, x >>= 1;
    while (y)
        R[cnt2++] = y & 1, y >>= 1;
    cnt = max(cnt1, cnt2);
    reverse(L, L + cnt);
    reverse(R, R + cnt);
    return dfs(0, true, true, true, true);
}
int main() {
    ios::sync_with_stdio(false);
    ll x, y;
    cin >> x >> y;
    cout << f(x, y) << endl;
    return 0;
}
```

[2020年ICPC区域赛上海站的C题](https://ac.nowcoder.com/acm/contest/9925/C)就是跟这比较类似的二进制数位DP。

数位DP乍一看很难，但学会后还是蛮简单的，很多题只需要改改模板。它把数字当作数位的数组处理，只要状态不要太多，效率一般比较优秀。但是要注意避免冗余的状态，例如如果不提前舍弃掉不合法值而是将其当作状态存储，会有很大影响。



## 写在最后

鸣谢：

[数位DP笔记 -  Luckyblock](https://www.cnblogs.com/luckyblock/p/14297825.html)

