## 【前言】

数位DP是一种计数用的 ${DP}$ ，一般就是要统计一个区间 $[l,r]$ 内满足一些条件数的个数。

所谓数位DP，我们从字面意思理解就是在数位上，也就是十进制下每一个数位上进行DP。

## 【例题】

### How many 0's

改编自 [POJ - 3286](https://vjudge.net/problem/POJ-3286)

<details open="" class="warn">
<summary> Description </summary> 
<p>输入 $a$ 和 $b$，求 $a$ 到 $b$ 中所有数之中有多少个 $0$ 出现。比如 $1\sim 100$ 中，$10，20，30，...,90$ 出现了 $9$ 个 $0$，$100$ 出现了 $2$ 个 $0$，所以一共有 $11$ 个 $0$。
</p>
</details><br>

这道题作为我们的引入题，照例还是非常简单的。这题严格上来说其实算是一个递推或者说是思维题。举个例子，假如我们想知道 $[0,4132]$ 当中出现了多少个 $0$ ，那我们就可以按位进行统计。先算个位，个位是 $0$ 的情况有 $413$ 种,因为 $0$ 的左边可以是 $1$ 到 $413$，而 $0$ 的右边没有数字，所以有 $413∗1=413$  种。然后是十位，十位有 $0$ 的情况 0 的左边有 $1$ 到 $41$ 共 $41$ 种，右边有 $0$ 到 $9$ 共 $10$ 种，所以一共有 $41∗10=410$ 种。接着是百位，百位有 $0$ 的情况 $0$ 的左边有 $1$ 到 $4$ 共 $4$ 种，右边有 $0$ 到 $99$ 共 $100$ 种，所以一共有 $4∗100=400$ 种。再到千位不能位 $0$，所以没有。最后还剩下 $0$ 的情况没考虑，这时有 $1$ 种。所以总共有 $413+410+400+1=1224$ 个 $0$。

<br>

接下来我们来看一个比较特殊地，假如我们想知道 $[0,4032]$​​​ 当中有多少个 $0$​​​ 出现，那我们就可以按位进行统计。先算个位，个位是 $0$​​ 的情况有 $403$​ 种,因为 $0$​ 的左边可以是 $1$ 到 $403$，而 $0$ 的右边没有数字，所以有 $403∗1=403$ 种。然后是十位，十位有 $0$ 的情况 0 的左边有 $1$ 到 $40$ 共 $40$ 种，右边有 $0$ 到 $9$ 共 $10$ 种，所以一共有 $40∗10=400$ 种。接着是百位，这里比较特殊，百位有 $0$ 的情况 $0$ 的左边有 1 到 4 共 4 种，但是对于左边是 $1,2,3$ 右边才有 0 到 99 共 100 种，当左边是 4 的时候，右边只有 0 到 32 共 33 种所以一共有 $3∗100+33=333$ 种。再到千位不能位 0，所以没有。最后还剩下 0 的情况没考虑，这时有 1 种。所以总共有 $403+400+333+1=1137$ 个 $0$。

我们到这就解决了 $[0,x]$  的统计问题，那题目要问 $[l,r]$ 的 $0$ 的个数，

那我们只要让 $ans = num([0,r]) - num([0,l - 1])$ 就可以了。

**【代码实现】**

```cpp
ll fac[12] = {1, 10, 100, 1000, 10000, 100000, 1000000, 10000000, 100000000, 1000000000, 10000000000, 100000000000};
ll count(ll n) {
    ll left, m, sum = 0;
    for (int i = 1; i < 12; ++i) {
        left = n / fac[i] - 1;
        sum += left * fac[i - 1];
        m = (n % fac[i] - n % fac[i - 1]) / fac[i - 1];
        //考虑是否卡住上界
        if (m > 0) sum += fac[i - 1];
        else sum += n % fac[i - 1] + 1;
        if (fac[i] > n) break;
    }
    return sum;
}
```

当然这道题也可以用数位DP来解决，只不过有些杀鸡用牛刀。

我们设两个数组 $f[i]$ 和 $f0[i]$ 

* 其中 $f[i]$ 存的是 $0\sim 99999...9$ (共 $i$ 个 $9$) 中 $0$ 的个数（不含前导 $0$ ，即 $0$ 就是 $0$, $12$ 就是 $12$）；

* $f0[i]$ 中存的是 $000....0$ (共 $i$ 个 $0$) 到 $99999...9$ (共 $i$ 个 $9$) 中 $0$ 的个数（含前导 $0$ )

  比如在 $f[4]$​ 中 $0$ 是 $0000$ ，$12$ 是 $0012$ 

含前导$0$ 的 $i$ 位数字就是由 $i−1$ 位的数字前面分别加上 $0$ 到 $9$ 构成的，

所以 $f0[i]=f0[i−1]∗10+pow(10,i−1)$​ 。

前面的 $f0[i−1]$ 表示 $i−1$ 位的那些数字被用了 $10$ 次，里面的 $0$ 又被写了 $10$ 次，后面的加上 $10^{i−1}$ 则表示在 $i−1$位数字前面加 0，一共加了 $10^{i−1}$ 个 $0$。

不含前导 $0$ 的 $i$ 位数字是由不含前导 $0$ 的 $i−1$ 位数字和前导 $0$ 的 $i−1$ 位数字前面加 $1$ 到 $9$ 构成的，所以 $f[i]=f[i−1]+f0[i−1]∗9$ 。

但对于一个具体的数字来说，比如对于 $400357$​ 要分成 $1−99999$，$100000−399999$ 和 $400000−400357$。第一部分的答案显然是 $f[5]$ ；第二部分为 $(4−1)$  倍的 $f0[5]$，在含前导 $0$ 的五位数前面分别写 $1,2,3$（注意 $4$ 并不能全部写完，有限制，故需要第三部分单独讨论）；对于第三部分的$400000−400357$，可以再分为三个部分，$400000−400299$，$400300−400349$，$400350−400357$，其中对于 $400000−400299$ 这部分可以写成 $2∗300+(000−299)$ 的方案数。而 $000−299$ 的方案数又写出 $000−099$的方案数和 $100−299$ 的方案数，

所以答案为 $ans = (f0[2]+100)+(f0[2]∗2)$ ，对于拆分的剩下部分也是同理就不过多赘述了。

---

$$
QAQ
$$

---

这里如果写迭代的 `for` 循环，绝对非常麻烦， 所以我们引入 DPS 来递归求解这第三部分我们设 DFS 函数为 `ll dfs(int pos, ll a)`。

如果当前的搜索的这位上是 $0$ ，即 $bit[pos]==0$，

那么就是后面这些位中含 0 的个数 $dfs(pos−1,400357)$​ 加上这个 $0$​ 被写的次数 $(a\%pow(10,pos−1)+1)$​。

但如果当前搜索的这位上不是 0（即 $num[pos]!=0$），那么这个数字要做和上面类似的分解。我们假设算到了 $400357$ 中的 $3$ 这位，那么分解为 $000−299，300−349，350−357$。

* 第一部分就是在含前导 0 的两位数前面写 0，容易知道是 $f0[2]+pow(10,pos−1)$；
* 第二部分是在含前导 $0$​​ 的两位数前面写 $1，2$，易知是  $(3−1)∗f0[2]$；
* 第三部分根据 DFS 递归的定义，就是 $dfs(2,400357)$。

这题用数位 DP 确实没有展现优势，不过这种分解数位的方法还是值得借鉴的。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210419185017.png)

### 送分了QAQ

<details open="" class="warn">
<summary> Description </summary> 
<p>每次给出一个区间 $[n,m]$ ，求包含 $38$ 或者 $4$ 的数字的个数。
</p>
</details><br>
原题链接：[Here](https://ac.nowcoder.com/acm/contest/74/G?&headNav=www)，改编自 [HDU - 2089 不要62](https://vjudge.net/problem/HDU-2089/origin) 

首先，我们求区间 $[n,m]$​ 的包含 38 或者 4 的数字的个数肯定可以转化为求 $[0,m]$​ 的包含 38 或者 4 的数字的个数减去 $[0,n−1]$​ 包含 38 或者 4 的数字的个数。所以我们只需要去考虑单边的限制。

在数位DP时，我们的DP数组一般是从高往低位第 $i$​ 位不受上下界限制时相关的数据，也就是说我们在计算DP数组的时候时脱离我们 $[n,m]$ 的限制的。在这里，我们设 $f[i][st]$ 为从高位往低位第 i 为数对应的情况为 $st$ 的数字个数。 $st=0$  表示既没有 4 也没有 38，$st=1$ 既没有 4 也没有 38但当前位是 3， $st=2$ 表示有 4 或者 38。

接下来我们考虑如何转移。

* 对于 $f[i][0]$，我们可以由既没有 4 也没有 38的后面 0−3 和 5−9 随便填， 但是这样我们就会错过前面有 3 现在填 8 的情况，所以我们还要减掉 $f[i−1][1]$，所以总的转移是 $f[i][0]=9∗f[i−1][0]−f[i−1][1]$；
* $f[i][1]$ 就比较好转移了，直接在既没有 4 也没有 38 的后面加上 3 就好了，所以 $f[i][1]=f[i−1][0]$；
* $f[i][2]$ 可以由原本就已经有 4 或者 38 的后面加上 0−9，也可以从既没有 4 也没有 38 的后面加 4，还可以由既没有 4 也没有 38但前一位是 3 的后面加 8 来组成，所以总的转移是 $f[i][2]=f[i−1][2]∗10+f[i−1][0]+f[i−1][1]$。

当然我们要注意边界是 $f[0][0]=1$​。

----

$$
QAQ
$$

---

那我们有了DP数组以后我们接下来要怎么做呢？没错还是要拆位的，和上一题一样，如果要写 `for `循环就会非常麻烦了，所以我们考虑搜索去找 $[0,x]$ 中满足条件的数。模板如下所示：

```cpp
int dfs(int pos, int st, bool flag) {
    //flag表示当前能否直接返回值，也就是前 pos - 1 位是否和原数不同——
    //相同则当前这位受到限制，需要继续递归求解
    //不同则不受限制，如果之前算过了就可以直接返回
    if (pos == 0) return st == 2;
    if (flag && f[pos][st] != -1) return f[pos][st];
    int x = flag ? 9 : a[pos];
    int ans = 0;
    for (int i = 0; i <= x; ++i) {
        if (i == 4 || st == 2 || (st == 1 && i == 8)) ans += dfs(pos - 1, 2, flag || i < x);
        else if (i == 3) ans += dfs(pos - 1, 1, flag || i < x);
        else ans += dfs(pos - 1, 0, flag || i < x);
    }
    if (flag) f[pos][st] = ans;
    return ans;
}
```



首先我们的数是倒着存的，比如 $x=134$ ，那 $a[1]=4,a[2]=3,a[3]=1$。

然后我们通过函数 `int dfs(int pos, int st, bool flag)` 来计算我们的答案。

* 其中 `pos` 是我们处理到 $a$ 数组第 `pos` 位；

* `st` 就是我们上面讲到的状态，

  也就是 $st=0$ 表示既没有 4 也没有 38，$st=1$ 既没有 4 也没有 38但当前位是 3， $st=2$ 表示有 4 或者 38；

* 而这个 `flag`  就很有意思了，代表的是前 $pos−1$ 位是否和原数不同，也就是有没有卡着上界。如果没有卡着上界，那我这一位是 $0\sim 9$​ 都可以填的，但是如果卡住了上界，我只能填 $0 \sim a[pos]$ 。

  其中 1 代表没有卡着上界， 0 则代表卡着上界。

  所以 `flag==1` 的时候，我们就能直接返回我们的 $f$ 数组。当然我们的 $f$ 数组开始时空的，我们也没必要单独计算一下 $f$，就类似记忆化搜索就可以了。



然后我们一行一行看程序，

* 首先开头的 `if(pos == 0) return st == 2;` 表示的是我们已经填完最后一位了，这时我们就看如果这时 $st==2$ 也就是有 4 或者 38 那就是一个合法的解我们返回 1，否则就不是合法的解我们返回 0。

* 然后是 `if(flag && f[pos][st] != -1) return f[pos][st];` 说的是假如我们没有卡着上界，也就是后面都是任填可以从 $000……0$ 填到 $999……9$，那就是我们 $f$ 数组，如果这个值已经被算过了那我们就直接返回它的值，没错就是一个记忆化搜索！接下来的 `int x = flag ? 9 : a[pos];` 是在处理我们这一位能填的数的范围，还是一样地，如果没有卡着上界，那我这一位是 $0−9$ 都可以填的，但是如果卡住了上界，我只能填 $0−a[pos]$​。
* 过后我们进 for 循环，`if(x == 4 || st == 2 || (st == 1 && i == 0))` 表示的是假如我们这一位填了 4，或者填完前面一位已经有 4 或者 38 了，或者我们上一位是 3 并且这一位填了 8，那我们就走向了一个合法的局面，所以我们 `ans += dfs(pos - 1, 2, flag || i < x);` 其中的 `flag || i < x` 只有我们这一位之前卡着上界了并且我这一位也卡着上界我们下一位才会是一个卡着上界的状态；`if(i == 3) ans += dfs(pos - 1, 1, flag || i < x)` 说的是我不管前面填了什么，但是我现在填了一个 3，那就导向了一个 $st=1$ 的局面；最后的就是填了其他的阿猫阿狗，那还是只能走 $st=0$，所以有 `ans += dfs(pos - 1, 0, flag || i < x);`。
* 倒数第二行我们的 `if(flag) f[pos][st] = ans;` 就是我们的记忆化搜索了，如果当前不是卡着上界的，也就是我们现在算出的 $ans$ 是可以存入 $f$​ 数组的，那我们就果断记忆化将其存进去。
* 最后返回 $ans$ 就可以了。其实我们看这个程序的本质如果没有记忆化其实就是相当于暴力搜索每一位放什么，而记忆化搜索就是将那些可以直接填 $000……0$ 填到 $999……9$​ 的存下来来加速我们的过程。

那其实我们计算 $[0,x]$ 的过程就可以用下面这个程序来调用计算：

```cpp
int calc(int x) {
    memset(a, 0, sizeof(a));
    int pos = 0;
    while (x) {
        a[++pos] = x % 10;
        x /= 10;
    }
    return dfs(pos, 0, 0);
}
```

【完整代码】因为记忆化搜索的原因，运行时间从 $240ms \to 3ms$

```cpp
int f[9][3], a[9];
int dfs(int pos, int st, bool flag) {
    //flag表示当前能否直接返回值，也就是前 pos - 1 位是否和原数不同——
    //相同则当前这位受到限制，需要继续递归求解
    //不同则不受限制，如果之前算过了就可以直接返回
    if (pos == 0) return st == 2;
    if (flag && f[pos][st] != -1) return f[pos][st];
    int x = flag ? 9 : a[pos];
    int ans = 0;
    for (int i = 0; i <= x; ++i) {
        if (i == 4 || st == 2 || (st == 1 && i == 8)) ans += dfs(pos - 1, 2, flag || i < x);
        else if (i == 3) ans += dfs(pos - 1, 1, flag || i < x);
        else ans += dfs(pos - 1, 0, flag || i < x);
    }
    if (flag) f[pos][st] = ans;
    return ans;
}

int calc(int x) {
    memset(a, 0, sizeof(a));
    int pos = 0;
    while (x) {
        a[++pos] = x % 10;
        x /= 10;
    }
    return dfs(pos, 0, 0);
}

int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    ll n, m;
    while (cin >> n >> m && (n || m)) {
        memset(f, -1, sizeof(f));
        cout << calc(m) - calc(n - 1) << "\n";
    }
}
```

最后再提供一个前缀和写暴力AC这道题的代码，$22ms$ 

```cpp
int n, m, sum[1000005];
bool check(int x) {
    while (x) {
        if (x % 10 == 4 || x % 100 == 38) return 1;
        x /= 10;
    }
    return 0;
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    sum[0] = 0;
    for (int i = 1; i <= 1000000; ++i) sum[i] = sum[i - 1] + check(i);
    while (cin >> n >> m && (n || m)) {
        cout << sum[m] - sum[n - 1] << "\n";
    }
}
```

### 诡异数字

<details open="" class="warn">
<summary> Description </summary> 
<p>给定你一个区间 $[l,r]$ 和多个约束，约束表示为数字 $x(0\le x\le 9)$ 在数字串中出现的次数不超过 $len$，请你给出在这个区间内满足这些约束的数字个数（不含前导 $0$）。
</p>
</details><br>

原题链接：[Here](https://ac.nowcoder.com/acm/contest/214/E?&headNav=www) ，From [牛客小白月赛8](https://ac.nowcoder.com/acm/contest/214) 

其实这道题和上一道题是类似的，有了上一题的基础，这一道题只要改一改状态就好了。

* 首先我们还是先将求 $[l,r]$ 的问题转化为求 $[0,x]$ 的问题。

* 我们设 $f[i][x][cnt]$  表示从高往低第 $i$ 位数字是 $x$，且 $x$ 已经连续出现了 $cnt$ 次的合法数字个数。记住我们这里的数字个数是后面的数字个数，也就是我们后面还没填的几位所有填写情况能产生的合法数字个数。比如 $233$，那我们的，那我们的 $f$ 里面存的其实是里面存的其实是​ 所有填写可能下能产生的合法数字的填法。

* 那我们还是对于当前位从 $0$ 到可能的最大值枚举，这个可能的最大值取决于有没有卡着上界。我们现在已经填好了前 $i$ 位且第 $i$ 位填了 $x$，我们考虑第 $i+1$ 位填的 $num$ 是否与 $x$ 相等。

  如果是我们这时就去看有没有超过 $x$ 的相关限制——超出则直接返回 $0$，没超过则通过递归累加 `dfs(i + 1, x, cnt, flag')` 的答案就可以了；

  如果不是，则我们递归累加 `dfs(i + 1, num, 1, flag')` 的答案就可以了。但其实最后代码实现是从从低位到高位第 $i−1$ 位贡献到 $i$ 位，这本质上是一致的。

```cpp
int dfs(int pos, int x, int num, bool flag) {
    if (pos == 0) return 1;
    if (flag && f[pos][x][num] != -1) return f[pos][x][num];
    int maxi = flag ? 9 : a[pos];
    int ans = 0;
    for (int i = 0; i <= maxi; i++) {
        if (i == x) {
            if (num + 1 > limit[i]) continue;
            ans = (ans + dfs(pos - 1, x, num + 1, flag || i < maxi)) % mod;
        } else {
            ans = (ans + dfs(pos - 1, i, 1, flag || i < maxi)) % mod;
        }
    }
    if (flag) f[pos][x][num] =  ans;
    return ans;
}
```

聪明的你一定已经发现了，这其实就是个板子，我们只要把我们的变量一改，DP方程往里面一套，我们的代码就出来了。

【AC Code】

```cpp
const int mod = 20020219;
ll book[20], f[20][20][20], a[11];
ll dfs(ll pos, ll num, ll cnt, bool flag) {
    if (cnt > book[num]) return 0;
    if (pos == 0) return 1;
    if (flag && f[pos][num][cnt] != -1)return f[pos][num][cnt];
    ll x = flag ? 9 : a[pos];
    ll ans = 0;
    for (int i = 0; i <= x; ++i) {
        if (i == num) ans = (ans + dfs(pos - 1, i, cnt + 1, flag || i < x)) % mod;
        else ans = (ans + dfs(pos - 1, i, 1, flag || i < x)) % mod;
    }
    if (flag) f[pos][num][cnt] = ans;
    return ans;
}
ll calc(ll x) {
    int pos = 0;
    while (x) {
        a[++pos] = x % 10;
        x /= 10;
    }
    return dfs(pos, 0, 0, 0);
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int _; for (cin >> _; _--;) {
        memset(f, -1, sizeof(f));
        memset(a, 0, sizeof(a));
        memset(book, 0x3f3f3f3f, sizeof(book));
        ll l, r, n;
        cin >> l >> r >> n;
        while (n--) {
            ll num, cnt;
            cin >> num >> cnt;
            book[num] = min(book[num], cnt);
        }
        cout << (calc(r) - calc(l - 1) + mod) % mod << "\n";
    }
}
```

### 7的意志

<details open="" class="warn">
<summary> Description </summary> 
<p>定义一个序列 $a:7,77,777...,7777777$ (数字全为 $7$ 的正整数，且长度可以无限大）<br>
RioTian 需要从含有 $7$ 的意志的数里获得力量，如果一个整数能被序列 $a$ 中的任意一个数字整除，并且其数位之和为序列 $a$ 中任意一个数字的倍数，那么这个数字就含有 $7$ 的意志，现在给你一个范围 $[n,m] (1\le n\le m\le 10^{18}$ ，问这个范围里有多少个数字含有 $7$ 的意志。
</p>
</details><br>

题目链接：[Here](https://ac.nowcoder.com/acm/problem/20665)

这道题给出的限制很强，我们先要对题目进行一个化简。首先若一个整数 $x$ 能被序列 $a$ 中的任意一个数字整除，序列 $a$ 中的任意一个数字能被 $7$ 整除，那其实就代表 $x$ 是 $7$ 的倍数。所以我们只需要重点关心第二个限制——其数位之和为序列 $a$ 中任意一个数字的倍数。我们还是先把 $[l,r]$ 中的计数问题转化为 $[0,x]$ 中计数的问题。

我们设 $f[pos][pre][sum]$ 表示从高到底前 $pos$ 位的数位的和 $\%7$ 的余数是 $pre$，数位和 %7 为 $sum$ 的个数。那假如我们第 $pos+1$ 位填了 $x$，那 $pre′=(10∗pre+x)$ 且 $sum′=(sum+x)$。都是可以维护的，那我们还是枚举每一位填什么，修改一下我们模板里的 `for` 循环：

```cpp
for (int i = 1; i <= maxi; i++) {
    ans += dfs(i - 1, (pre * 10 + i) % 7, (sum + i) % 7, flag || i < maxi);
}
```

剩下的照抄就可以了。其实我们大部分数位DP也是这样的。

完整代码：[Here](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=48625420&returnHomeType=1&uid=612647183)

### Beautiful Numbers

<details open="" class="warn">
<summary> Description </summary> 
<p>我们设 $F(x)$ 为 $x$ 各个位数的和，求 $[l,r](0\le l\le r\le 10^12)$ 中满足 x % F(x) == 0 的数的个数。
</p>
</details><br>

题目链接：[Here](https://ac.nowcoder.com/acm/problem/17385)

我们还是先把 $[l,r]$ 中的计数问题转化为 $[0,x]$ 中计数的问题。

我们这道题的难点是我们用什么来代表我们的 $x\%F(x)$，同时维护 $x$ 和 $F(x)$ 或者维护 $x\%F(x)$ 是很难的。但是我们发现数据范围 $0≤l≤r≤1012$，所以我们可以在 $0$ 到 $12∗9$ 的范围里枚举 $F(x)$。这样我们就设 $f[pos][x][mod][sum]$ 表示前 $pos$ 位数除以 $x$ 的余数为 $mod$，且前 $pos$ 位的和为 $sum$ 的个数。那么我们的 $x$ 就可以从 $0$ 到 $12∗9$ 的范围里枚举。

所以我们的代码大致就和下面这样：

```cpp
int dfs(int pos, int mod, int x, int sum, bool flag) {
    if (pos == 0) return (x == sum && mod % sum == 0);
    if (flag && f[pos][mod][x][sum] != -1) return f[pos][mod][x][sum];
    int maxi = flag ? 9 : a[pos];
    int ans = 0;
    for (int i = 0; i <= maxi; i++) {
        int temp = (mod * 10 + i) % x;
        ans += dfs(pos - 1, temp, x, sum + i, flag || i < maxi);
    }
    if (flag) f[pos][mod][x][sum] =  ans;
    return ans;
}
```

完整代码：[Here](https://ac.nowcoder.com/acm/contest/view-submission?submissionId=48625479&returnHomeType=1&uid=612647183)

### Beautiful Numbers2

<details open="" class="warn">
<summary> Description </summary> 
<p>求 $[l,r](0\le l\le r\le 10^{18})$ 中有多少数能够整除他自身各个位数。
</details><br>

我们还是先把 $[l,r]$ 中的计数问题转化为 $[0,x]$ 中计数的问题。然后我们若一个数 $x$ 能整除他自身各个位数，那它一定可以整除它自身各个位数的最小公倍数。那我们还能去枚举我们的最小公倍数吗？我们看 $lcm(1,2,3,4,5,6,7,8,9) = 2520$ 有点大，枚举起来可能会T 。那怎么办呢？

其实很简单，由于，我们只需要记录我们 $lcm(1,2,3,4,5,6,7,8,9) = 2520$ 的这个数对 DP 的模就可以了。我们最后只要看这个数对 $2520$ 的模是否整除我们选出来的数的最小公倍数就能判断出它能否整除它自身各个位数了。

所以我们设 $f[pos][mod][lcm]$ 表示 pos 是当前位，$mod$ 为前面那些位对 $2520$ 的模，$lcm$ 为前面那些数位的最小公倍数，这样一来我们的 $f$ 就开到了 $19*2520*2520$ 明显是 $MLE$ 了。考虑我们的最小公倍数其实是离散的，比如 $11,13...$ 什么的就不可能成为 $1\sim 10$ 组合的最小公倍数。$1\sim 2520$ 中真正有可能成为最小公倍数的其实只有 $48$ 个，经过离散化处理后(也可以`map`)，我们 $f$ 的最后一维可以降到 ，这样就不会超了。

代码大致如下：

```cpp
ll dfs(int pos, int mod, int lcm, bool flag) {
    if (pos == 0) return mod % num[lcm] == 0;
    if (flag && f[pos][mod][num[lcm]] != -1) return f[pos][mod][num[lcm]];
    int maxi = flag ? 9 : a[pos];
    ll ans = 0;
    for (int i = 0; i <= maxi; i++) {
        int tlcm = (i == 0) ? lcm : match[num[lcm] * i / gcd(num[lcm], i)];
        int tmod = (mod * 10 + i) % 2520;
        ans += dfs(pos - 1, tmod, tlcm, flag || i < maxi);
    }
    if (flag) f[pos][mod][num[lcm]] =  ans;
    return ans;
}
```

~~完整代码：[Here](#)~~

```cpp
ll f[20][2521][50];
ll num[] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
             12, 14, 15, 18, 20, 21, 24, 28, 30, 35,
             36, 40, 42, 45, 56, 60, 63, 70, 72, 84,
             90, 105, 120, 126, 140, 168, 180, 210, 252, 280,
             315, 360, 420, 504, 630, 840, 1260, 2520
           };
int a[20];
ll gcd(ll a, ll b) { return b == 0 ? a : gcd(b, a % b);}
ll lc(ll a, ll b) { return a * b / gcd(a, b);}
ll dp(ll pos, ll mod, ll lcm, bool flag) {
    if (pos == 0) return mod % lcm == 0;
    ll tmp = lower_bound(num, num + 48, lcm) - num;
    if (flag && f[pos][mod][tmp] != -1) return f[pos][mod][tmp];
    int x = flag ? 9 : a[pos];
    ll ans = 0;
    for (int i = 0; i <= x; i++) {
        if (i == 0) ans += dp(pos - 1, (mod * 10 + i) % 2520, lcm, flag || i < x);
        else ans += dp(pos - 1, (mod * 10 + i) % 2520, lc(lcm, i), flag || i < x);
    }
    if (flag) f[pos][mod][tmp] = ans;
    return ans;
}
ll cul(ll x) {
    int pos = 0;
    while (x) {
        a[++pos] = x % 10;
        x /= 10;
    }
    return dp(pos, 0, 1, false);
}
int main() {
    cin.tie(nullptr)->sync_with_stdio(false);
    int t;
    cin >> t;
    memset(f, -1, sizeof(f));
    while (t--) {
        ll l, r;
        cin >> l >> r;
        cout << cul(r) - cul(l - 1) << endl;
    }
}
```



## 结语

其实大部分的数位 DP 都是个板子题，因为出的太难我们选手也不会做（误）。但就算是个板子，你也要会写会推会理解。可能有的选手到退役都不理解这个板子在干什么也能写题，不过我还是希望通过这一篇 BLOG 你能对数位DP大概有点感觉。最后希望你喜欢这篇 BLOG！
