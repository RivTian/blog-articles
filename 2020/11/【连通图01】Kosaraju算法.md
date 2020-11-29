`Kosaraju`算法一看这个名字很奇怪就可以猜到它也是一个根据人名起的算法，它的发明人是`S. Rao Kosaraju`，这是一个在图论当中非常著名的算法，可以用来拆分**有向图当中的强连通分量**。

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183424.gif"/></div>

## 背景知识

这里有两个关键词，一个是有向图，另外一个是强连通分量。有向图是它的使用范围，我们只能使用在有向图当中。对于无向图其实也存在强连通分量这个概念，但由于无向图的连通性非常强，只需要用一个集合维护就可以知道连通的情况，所以也没有必要引入一些算法。



有向图我们都了解，那么什么叫做强连通分量呢？强连通分量的英文是**strongly connected components**。这是一个很直白的翻译，要理解它我们首先需要理解强连通的概念。在有向图当中，如果两个点之间**彼此存在一条路径相连**，那么我们称这两个点强连通。那么推广一下，如果一张图当中的**一个部分中的每两个点都连通**，那么这个部分就称为强连通分量。



强连通分量一般是一张完整的图的一个部分，比如下面这张图当中的{1, 2, 3, 4}节点就可以被看成是一个强连通分量。



![img](https://gitee.com//riotian/blogimage/raw/master/img/20201123184254.webp)



其实求解强连通分量的算法并不止一种，除了**Kosaraju**之外还有大名鼎鼎的**Tarjan**算法可以用来求解。但相比**Tarjan**算法，**Kosaraju**算法更加==直观==，更加==容易理解==。



## 算法原理

**Kosaraju**算法的原理非常简单，简单到**只有三个步骤**：

1. 我们通过**后序遍历**的方式遍历整个有向图，并且维护每个点的出栈顺序
2. 我们将有向图反向，根据出栈顺序从大到小再次遍历反向图
3. 对于点u来说，在遍历反向图时所有能够到达的v都和u在一个强连通分量当中

怎么样，是不是很简单？

下面我们来详细阐述一下细节，首先后序遍历和维护出栈顺序是一码事。也就是在递归的过程当中当我们遍历完了u这个节点所有连通的点之后，再把u加入序列。其实也就是**u在递归出栈的时候才会被加入序列**，那么序列当中存储的也就是每个点的出栈顺序。

<div align=center><img src="https://gitee.com//riotian/blogimage/raw/master/img/20201123183415.png"/></div>

这里我用一小段代码（python）演示一下，看完也就明白了。

```python
popped = [] # 存储出栈节点

def dfs(u):
    for v in Graph[u]:
        dfs(v)
        popped.append(u)
```

我们在访问完了所有的v之后再把u加入序列，这也就是后序遍历，和二叉树的后序遍历是类似的。

反向图也很好理解，由于我们求解的范围是有向图，如果原图当中存在一条边从u指向v，那么反向图当中就会有一条边从v指向u。也就是把所有的边都调转反向。

我们用上面的图举个例子，对于原图来说，它的出栈顺序我们用红色笔标出。



![](https://gitee.com//riotian/blogimage/raw/master/img/20201123184258.webp)



也就是[6, 4, 2, 5, 3, 1]，我们按照出栈顺序**从大到小排序**，也就是将它反序一下，得到[1, 3, 5, 2, 4, 6]。1是第一个，也就是最后一个出栈的，也意味着1是遍历的起点。

我们将它反向之后可以得到：



![](https://gitee.com//riotian/blogimage/raw/master/img/20201123184302.webp)



我们再次从1出发可以遍历到2，3， 4，说明{1, 2, 3, 4}是一个强连通分量。

怎么样，整个过程是不是非常简单？

我们将这段逻辑用代码实现，也并不会很复杂。

```cpp
// Cpp
// g 是原图，g2 是反图
void dfs1(int u) {
    vis[u] = true;
    for (int v : g[u])
        if (!vis[v])
            dfs1(v);
    s.push_back(u);
}

void dfs2(int u) {
    color[u] = sccCnt;
    for (int v : g2[u])
        if (!color[v])
            dfs2(v);
}

void kosaraju() {
    sccCnt = 0;
    for (int i = 1; i <= n; ++i)
        if (!vis[i])
            dfs1(i);
    for (int i = n; i >= 1; --i)
        if (!color[s[i]]) {
            ++sccCnt;
            dfs2(s[i]);
        }
}
```

```python
# python
N = 7
graph, rgraph = [[] for _ in range(N)], [[] for _ in range(N)]
used = [False for _ in range(N)]
popped = []


# 建图
def add_edge(u, v):
    graph[u].append(v)
    rgraph[v].append(u)


# 正向遍历
def dfs(u):
    used[u] = True
    for v in graph[u]:
        if not used[v]:
            dfs(v)
    popped.append(u)


# 反向遍历
def rdfs(u, scc):
    used[u] = True
    scc.append(u)
    for v in rgraph[u]:
        if not used[v]:
            rdfs(v, scc)
            
# 建图，测试数据         
def build_graph():
    add_edge(1, 3)
    add_edge(1, 2)
    add_edge(2, 4)
    add_edge(3, 4)
    add_edge(3, 5)
    add_edge(4, 1)
    add_edge(4, 6)
    add_edge(5, 6)


if __name__ == "__main__":
    build_graph()
    for i in range(1, N):
        if not used[i]:
            dfs(i)

    used = [False for _ in range(N)]
    # 将第一次dfs出栈顺序反向
    popped.reverse()
    for i in popped:
        if not used[i]:
            scc = []
            rdfs(i, scc)
            print(scc)
```

## 思考

算法讲完，代码也写了，但是并没有结束，仍然有一个很大的疑惑没有解开。算法的原理很简单，很容易学会，但问题是**为什么这样做就是正确的呢**？这其中的原理是什么呢？我们似乎仍然没有弄得非常清楚。

这里面的原理其实很简单，我们来思考一下，如果我们在正向dfs的时候，u点出现在了v点的后面，也就是u点后于v点出栈。有两种可能，**一种可能是u点可以连通到v点，说明u是v的上游**。**还有一种可能是u不能连通到v**，说明图被分割成了多个部分。对于第二种情况我们先不考虑，因为这时候u和v一定不在一个连通分量里。对于第一种情况，u是v的上游，说明u可以连通到v。

这时候，我们将图反向，如果我们从u还可以访问到v，那说明了什么？很明显，**说明了在正向图当中v也有一条路径连向u**，不然反向之后u怎么连通到v呢？所以，u和v显然是一个强连通分量当中的一个部分。我们再把这个结论推广，所有u可以访问到的，第一次遍历时在它之前出栈的点，都在一个强连通分量当中。

如果你能理解了这一点，那么整个算法对你来说也就豁然开朗了，相信剩下的细节也都不足为虑了。

到这里，整个算法流程的介绍就算是结束了，希望大家都可以enjoy今天的内容。