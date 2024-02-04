---
title: 【最大流+二分图匹配建图+强连通分量】AcWing 380. 舞动的夜晚
date: 2024-02-04
tag: 网络流
categories: 题解
top_img: https://pic.imgdb.cn/item/65bf0526871b83018a429895.jpg
cover: https://pic.imgdb.cn/item/65bf050e871b83018a42504c.jpg
---

本题即求二分图最大匹配**不可行**边，相当于**可行边**的**补集**。

对于完备匹配，先求出一组最大匹配，有如下判定：

- 若 $(x,y)$ 为必须边：
  - $(x,y)$ 为匹配边
  - 删除 $(x,y)$ 后，不存在 $x$ 到 $y$ 的增广路
- 若 $(x,y)$ 为可行边：
  - $(x,y)$ 为匹配边
  - $(x,y)$ 为非匹配边，设 $(x,v),(u,y)$ 匹配，则连接 $(x,y)$ 后存在 $u$ 到 $v$  的增广路

考虑快速判定，采用如下**建图转化**：

​	可以将该匹配的匹配边 $(x,y)$ 定义方向 $y\rightarrow x$ ，非匹配边方向 $x\rightarrow y$ 。

设上述构造的图为 $G$ ，则上述存在增广路可以化为：$x$ 和 $y$ 否/是 在同一个**强连通分量**。

{% note info modern %}

1. 对于必须边，存在 $G$ 上 $y\rightarrow x$ 的边，若存在 $x\rightarrow y$ 路径则存在增广路，因此不再同一个scc
2. 对于非必须边，存在 $x\rightarrow y$ 的边，此时从 $y$ 出发的边只有 $(u,y)$ 匹配边， 因此存在 $y\rightarrow x$ 的路径即存在 $u\rightarrow v$ 的增广路

{% endnote %}

而对于非完备匹配，对于上述条件2的限制又有所放宽：

-  $x,y$ 之一为非匹配点，连接并断开原来一条匹配仍是最大匹配
- $x,y$ 均为匹配点，设 $(x,v),(u,y)$ 匹配，则 $u$ 找到到达任意非匹配点 $w$ 的增广路也可

{% note danger modern %}

第二条不能任意找两个点的增广路，因为若两个点均为非匹配点，可以使得匹配数量直接 $+1$ ，这与最大匹配矛盾

{% endnote %}

在这种情况下，上述判定无法适用。

为了能够采取上述快速的判定方式，考虑构建**额外的源/汇点** $s,t$ ，

而 $G$ 上边的方向对应**残量网络**上的剩余容量方向，考虑对 $s\rightarrow t$ 做最大流，可以发现：

- 对于 $x$ 为非匹配点，$x\rightarrow t$ 存在 $1$ 的剩余容量
- 对于 $x$ 为匹配点，$t\rightarrow x$ 存在 $1$ 的剩余容量

因此对于上述放宽的两个条件，均相当于 $x,y$ 在同一个**强连通分量**。

对此，可以提出一种新的判定方法：

{% note success modern %}

1. 对原图建立源点汇点，跑一边 $\text{Dinic}$ ，并求出 $scc$
2. 若 $(y,x)$ 剩余容量为 $1$ ，**并**在残量网络上属于**不同** $scc$ ，则为**必须边**
3. 若 $(y,x)$ 剩余容量为 $1$ ，**或**在残量网络上属于**相同** $scc$ ，则为**可行边**

{% endnote %}

取可行边补集即可，时间复杂度 $O(M\sqrt N)$ ，瓶颈在于 $\text{Dinic}$ 。

```cpp
// 二分图不可行边 → 可行边补集
// Dinic 求出残量网络, Tarjan求强连通分量
// 必须边：(y,x)剩余容量为 1，且不在同一个强连通分量；
// 可行边：(y,x)剩余容量为 1，或在同一个强连通分量
// 网络流建图

#include<bits/stdc++.h>
using namespace std;

const int N = 3e5 + 10, inf = 1e9;

int n, m, E, tot = 1, s = 0, t, d[N];
int head[N], ver[N << 1], edge[N << 1], Next[N << 1];

void add_edge(int x, int y)
{
    ver[++tot] = y; edge[tot] = 1; Next[tot] = head[x]; head[x] = tot;
    ver[++tot] = x; edge[tot] = 0; Next[tot] = head[y]; head[y] = tot;
}

bool bfs()
{
    for(int i = 0; i <= n + m + 1; i ++ ) d[i] = 0;
    queue<int> q; q.push(s); d[s] = 1;
    while(q.size())
    {
        int x = q.front(); q.pop();
        for(int i = head[x]; i; i = Next[i])
            if(edge[i] && !d[ver[i]])
            {
                q.push(ver[i]), d[ver[i]] = d[x] + 1;
                if(ver[i] == t) return 1;
            }
    }
    return 0;
}

int dinic(int x, int flow)
{
    if(x == t) return flow;
    int rest = flow, k;
    for(int i = head[x]; i && rest; i = Next[i])
        if(edge[i] && d[ver[i]] == d[x] + 1)
        {
            k = dinic(ver[i], min(rest, edge[i]));
            if(!k) d[ver[i]] = 0;
            edge[i] -= k; edge[i ^ 1] += k;
            rest -= k;
        }
    return flow - rest;
}

int dfn[N], low[N], num, top, ins[N], st[N], scc[N], cnt, e[N], u[N], v[N];

void tarjan(int x)
{
    dfn[x] = low[x] = ++ num; st[++top] = x, ins[x] = 1;
    for(int i = head[x]; i; i = Next[i]) if(edge[i])
        if(!dfn[ver[i]]) tarjan(ver[i]), low[x] = min(low[x], low[ver[i]]);
        else if(ins[ver[i]]) low[x] = min(low[x], dfn[ver[i]]);
    if(dfn[x] == low[x])
    {
        cnt ++ ; int y; do
        {
            y = st[top -- ], ins[y] = 0;
            scc[y] = cnt;
        } while (x != y);
    }
}

vector<int> res;

int main()
{
    scanf("%d%d%d", &n, &m, &E); t = n + m + 1;
    for(int i = 1; i <= n; i ++ ) add_edge(s, i);
    for(int i = n + 1; i <= n + m; i ++ ) add_edge(i, t);
    for(int i = 1, x, y; i <= E; i ++ )
        scanf("%d%d", &x, &y), add_edge(x, n + y), 
    	e[i] = tot, u[i] = x, v[i] = n + y;
    while(bfs()) while(dinic(s, inf));
    for(int i = 0; i <= n + m + 1; i ++ ) if(!dfn[i]) tarjan(i);
    for(int i = 1; i <= E; i ++ ) 
        if(!edge[e[i]] && scc[u[i]] != scc[v[i]]) res.push_back(i);
    printf("%d\n", res.size()); 
    for(auto it : res) printf("%d ", it); putchar('\n');
    return 0;
}
```

