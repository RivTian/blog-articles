## 定义

### 先复制一则定义

> A\*算法在人工智能中是一种典型的启发式搜索算法
> 启发中的估价是用估价函数表示的：
> h(n)=f(n)+g(n)
> 其中f(n)是节点n的估价函数
> g(n)表示实际状态空间中从初始节点到n节点的实际代价
> h(n)是从n到目标节点最佳路径的估计代价。
> 另外定义h'(n)为n到目标节点最佳路径的实际值。
> 如果h'(n)≥h(n)则如果存在从初始状态走到目标状态的最小代价的解
> 那么用该估价函数搜索的算法就叫A*算法。

有点繁琐，但也看得过去

### 通俗来讲

的核心在于上面所讲到的**估价函数**
他是干什么用的呢
就是我们在搜索的过程中，保证更优的先搜用的
还是有些繁琐对不对，嗯，我也不大会讲啊（没事~~我会加油~~）

------

嘿，认真看下面，~~我可认真了的啊~~。。。

如果一个题目要求我们求**前K个**代价最小的解（只是一个典型，不是所有题目都这样）
假设我们现在有一个状态在
已经要记录到答案里面的代价是(我喜欢用这个)
我们发现如果爆搜的话状态会是乱的对不对，肯定会使搜索搜到太多
而如果直接把状态按照排序的话不能保证答案就会正确（当然，不然就去贪心去）
所以我们引进一个**估价函数状态**
当然要求一般是可以预处理出一个状态到答案状态的**最优解**
回到前面讲到的当前状态
如果我们把与并列的所有状态按排序呢？
**既不影响答案的正确性，又可以减少坏状态的转移**
（因为题目要求是K个最优状态，而这样待决策状态会有序且跑完K个就可以结束，所以会变快）

好吧，还有点蒙对不对，那我们看例题

## 例题

[洛谷P2901 [USACO08MAR\]牛慢跑Cow Jogging](https://www.luogu.org/problemnew/show/P2901)
好像其他很多都有，但是是权限。。。

### 题目简述

要求我们求出从起点n到终点1的最短K条路径的长度
（只能从编号大的点往编号小的点走&边有边权）

### 很裸对吧？

> 1. 预处理估价函数

先跑一遍反向边的预处理出每个点到1的最短路作为估价函数

> 1. 直接跑(这里用实现)

从n号点开始，用堆来代替队列（实现上面所讲的排序）
这时候先到1节点的肯定答案更优（也就是路径更短）
原因很简单吧：**估价函数保证答案合法，而排序之后答案有序**
搜到K个到达1节点的路径就可以结束，快的飞起。。。

### 放个代码？

~~好不容易写一次注释~~

```
#include<bits/stdc++.h>
#define lst long long
#define ldb double
#define N 1050
#define M 10050
#define qw ljl[i].to
using namespace std;
const lst Inf=1e15;
int read()
{
    int s=0,m=0;char ch=getchar();
    while(!isdigit(ch)){if(ch=='-')m=1;ch=getchar();}
    while( isdigit(ch))s=(s<<3)+(s<<1)+(ch^48),ch=getchar();
    return m?-s:s;
}
int n,m,K,Done;
bool in[N];
lst dis[N];
queue<int> Q;
int hd[N],cnt;
struct EDGE{int to,nxt,v;}ljl[M<<1];
void Add(int p,int q,int o){ljl[++cnt]=(EDGE){q,hd[p],o},hd[p]=cnt;}
void SPFA()
{
    for(int i=2;i<=n;++i)dis[i]=Inf;
    while(!Q.empty())Q.pop();
    Q.push(1),dis[1]=0,in[1]=true;
    while(!Q.empty())
    {
        int now=Q.front();Q.pop(),in[now]=false;
        for(int i=hd[now];i;i=ljl[i].nxt)
            if(qw>now&&dis[qw]>dis[now]+ljl[i].v)
            {
                dis[qw]=dis[now]+ljl[i].v;
                if(!in[qw])in[qw]=true,Q.push(qw);
            }
    }
}
//h[i]=g[i]+f[i]---->ans[i]=D+dis[i]
struct NODE{
    lst D;int id;
    bool operator<(const NODE &X) const
        {
            return D+dis[id]>X.D+dis[X.id];
        }
};priority_queue<NODE> H;
void A_star_Bfs()
{
    while(!H.empty())H.pop();
    H.push((NODE){0,n});
    while(!H.empty())
    {
        NODE temp=H.top();
        int now=temp.id;H.pop();
        if(now==1)
        {
            printf("%lld\n",temp.D);
            if(++Done==K)return;continue;
        }
        for(int i=hd[now];i;i=ljl[i].nxt)
            if(qw<now)H.push((NODE){temp.D+ljl[i].v,qw});
    }while(Done<K)++Done,puts("-1");
}
int main()
{
    n=read(),m=read(),K=read();
    for(int i=1;i<=m;++i)
    {
        int p=read(),q=read(),o=read();
        Add(p,q,o),Add(q,p,o);
    }
    SPFA(),A_star_Bfs();
    return 0;
}
/************
1.A*算法在人工智能中是一种典型的启发式搜索算法
启发中的估价是用估价函数表示的：
h(n)=f(n)+g(n)
其中f(n)是节点n的估价函数
g(n)表示实际状态空间中从初始节点到n节点的实际代价
h(n)是从n到目标节点最佳路径的估计代价。
另外定义h'(n)为n到目标节点最佳路径的实际值。
如果h'(n)≥h(n)则如果存在从初始状态走到目标状态的最小代价的解
那么用该估价函数搜索的算法就叫A*算法。
2.第K最短路的算法
我们设源点为s，终点为t，我们设状态f(i)的g(i)为从s走到节点i的实际
距离，h(i)为从节点i到t的最短距离，从而满足A*算法的要求，
当第K次走到f(n-1)时表示此时的g(n-1)为第K最短路长度。
3.这里是kuai的xzy的。。。别怪我。。。
*************/ 
```

## 总结

暂时就将这么多吧
主要是看到网上没有写的那么通俗的搜索
就想自己总结一下（~~其实也不通俗。。。~~）
撤撤撤溜了溜了***_\***