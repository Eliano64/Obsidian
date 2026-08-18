---
title: Minimum Spanning Tree
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 12:24:30
katex: true
---

# 最小生成树定义

最小生成树（Minimum Spanning Tree, MST）讨论的是：在一个**带权无向连通图**中，既要让所有顶点连通，又要让选中边的总代价尽可能低。


![最小生成树概念](../assets/minimum-spanning-tree-concepts.svg)



设带权无向连通图为 $G=(V,E)$，$R$ 为 $G$ 的所有[[Spanning-Tree#生成树|生成树]]的集合。对任意生成树 $T$，它的权值是 $T$ 中所有边权值之和：

$$
w(T)=\sum_{e\in T}w(e)
$$

若某棵生成树 $T$ 满足：

$$
w(T)=\min_{T_i\in R}w(T_i)
$$

则 $T$ 称为 $G$ 的**最小生成树**，也称**最小代价树**。

> [!important] MST 的对象
> 最小生成树在**带权连通无向图**的前提下讨论。
> 对于非联通图，则是 **最小生成森林** 。

# MST 性质

| 条件 | 说明 |
|---|---|
| 覆盖全部顶点 | 不能只连接一部分顶点 |
| 连通 | 任意两个顶点之间都有路径 |
| 无环 | 有环就能删去环上的某条边并仍保持连通，不是生成树的最简形态 |
| 边数为 $n-1$ | 这是生成树的固定边数 |
| 边权和最小 | 在所有生成树中总代价最低 |

这些条件中，前四个保证它是生成树，最后一个保证它是**最小**生成树。

# 一个连通图的 MST 是否唯一

MST 的**边集**可能不唯一，但MST 的**最小权值和**是唯一的，因为“最小值”本身是确定的。

于是判断 MST 是否唯一，等价于判断边集是否唯一。

> [!info] 怎样算是不同的 MST ？
> 只看边集是否相同。而判断边是否相同，即等价于看两个端点是否相同（不计顺序）。

## MST 唯一的充分条件

- 当原图为树时，MST 是唯一的。
- 当原图没有权值相同的边， MST 是唯一的。

## MST 唯一的充要条件


**每条非树边$(u,v)$，权值都必须严格大于在树上的路径$u\to v$中的权值最大的边。**

# 计算 MST 的算法

## Kruskal 算法

Kruskal 的思路是“按边扩张”：把所有边按权值从小到大排序，每次选择当前最短且不会形成回路的边。

[html-card height=760 step=80](../assets/mst-kruskal.html)

核心过程：

1. 将所有边按权值从小到大排序。
2. 从小到大枚举边 $(u,v,w)$。
3. 若 `u` 和 `v` 已在同一连通块中，加入这条边会成环，跳过。
4. 若 `u` 和 `v` 不在同一连通块中，选择这条边，并合并两个连通块。
5. 选够 $n-1$ 条边后停止。

Kruskal 常用[并查集](Union-Find.md)判断是否会成环。

```c
#include <stdio.h>
#include <stdlib.h>

#define MAXE 1000
#define MAXV 100

typedef struct {
    int u;       // 边的一个端点
    int v;       // 边的另一个端点
    int weight;  // 边权
} Edge;

int parent[MAXV];  // 并查集 parent[x] 表示 x 的父节点或集合代表

int cmpEdge(const void *a, const void *b) {
    const Edge *x = (const Edge *)a;
    const Edge *y = (const Edge *)b;

    // qsort 需要比较函数返回负数、0、正数。
    // 这里按边权从小到大排序，保证 Kruskal 总是先尝试较短的边。
    return x->weight - y->weight;
}

int findRoot(int x) {
    if (parent[x] != x) {
        parent[x] = findRoot(parent[x]);  // 路径压缩：让 x 直接指向集合代表
    }
    return parent[x];
}

int unionSet(int a, int b) {
    int rootA = findRoot(a);
    int rootB = findRoot(b);

    if (rootA == rootB) {
        return 0;  // 已在同一集合中，再连会形成回路
    }

    parent[rootA] = rootB;  // 合并两个连通块
    return 1;
}

int kruskal(Edge edges[], int vertexCount, int edgeCount) {
    int selectedEdges = 0;  // 已经选入生成树的边数
    int totalWeight = 0;    // 已选边的权值总和

    for (int i = 0; i < vertexCount; ++i) {
        parent[i] = i;  // 初始时每个顶点自成一个连通块
    }

    qsort(edges, edgeCount, sizeof(Edge), cmpEdge);  // 先看权值最小的边

    for (int i = 0; i < edgeCount && selectedEdges < vertexCount - 1; ++i) {
        Edge edge = edges[i];  // 当前正在尝试加入生成树的边

        if (unionSet(edge.u, edge.v)) {
            // 两端原本不连通：选入这条边不会形成回路。
            totalWeight += edge.weight;
            ++selectedEdges;
        }
    }

    if (selectedEdges != vertexCount - 1) {
        return -1;  // 原图不连通，无法得到覆盖全图的生成树
    }

    return totalWeight;
}
```

时间复杂度主要来自边排序：

$$
O(\lvert E\rvert\log\lvert E\rvert)
$$

按边表实现时，需要先把边按权值排序，然后最多检查每条边一次。检查一条边时，要判断两个端点是否已经属于同一连通块；常见写法使用并查集完成这个判断与合并。

Kruskal 适合边较少的稀疏图。

## Prim 算法

Prim 的思路是“按点扩张”：从一个起点出发，维护当前生成树到每个外部顶点的最小边权，每次把距离当前树最近的顶点加入树。

[html-card height=840 step=80](../assets/mst-prim.html)

核心过程：

1. 任取一个顶点加入生成树集合。
2. 对每个未加入顶点，记录它到当前树集合的最小边权 `lowCost[v]`。
3. 选择 `lowCost` 最小的未加入顶点 `v`，把它加入生成树。
4. 用 `v` 的邻接边更新其他未加入顶点的 `lowCost`。
5. 重复直到所有顶点都加入。

朴素 Prim 常用两个数组表达这个过程：

| 数组 | 含义 | 更新时机 |
|---|---|---|
| `isJoin[v]` 或 `inTree[v]` | 顶点 `v` 是否已经加入当前生成树集合 | 每轮选出新的最近顶点后置为 `true` |
| `lowCost[v]` | 顶点 `v` 到当前生成树集合的最小接入代价 | 新顶点加入后，用它的邻接边尝试更新树外顶点 |

每一轮有两个扫描动作：

1. 扫描所有顶点，找到 `lowCost` 最小且尚未加入树的顶点。
2. 再扫描所有顶点，用新加入顶点的邻接边更新树外顶点的 `lowCost`。

邻接矩阵版本适合稠密图。

```c
#include <stdio.h>

#define MAXV 100
#define INF 1000000000

int prim(int graph[MAXV][MAXV], int vertexCount) {
    int inTree[MAXV] = {0};  // inTree[v] == 1 表示顶点 v 已加入生成树
    int lowCost[MAXV];       // lowCost[v] 表示 v 到当前生成树集合的最小边权
    int totalWeight = 0;     // 已选生成树边的权值总和

    for (int v = 0; v < vertexCount; ++v) {
        lowCost[v] = graph[0][v];  // 初始树只含顶点 0，记录各点到顶点 0 的边权
    }

    inTree[0] = 1;  // 顶点 0 作为起点加入生成树

    for (int step = 1; step < vertexCount; ++step) {
        int nextVertex = -1;  // 本轮要加入生成树的顶点
        int minCost = INF;    // 当前找到的最小接入边权

        for (int v = 0; v < vertexCount; ++v) {
            if (!inTree[v] && lowCost[v] < minCost) {
                minCost = lowCost[v];
                nextVertex = v;
            }
        }

        if (nextVertex == -1) {
            return -1;  // 找不到能接入树的顶点，说明原图不连通
        }

        inTree[nextVertex] = 1;
        totalWeight += minCost;  // 选中连接 nextVertex 与当前树的最短边

        for (int v = 0; v < vertexCount; ++v) {
            if (!inTree[v] && graph[nextVertex][v] < lowCost[v]) {
                lowCost[v] = graph[nextVertex][v];  // 用新顶点更新外部点到树的最近距离
            }
        }
    }

    return totalWeight;
}
```

邻接矩阵朴素 Prim 的时间复杂度为：

$$
O(\lvert V\rvert^2)
$$

因为总共进行 $\lvert V\rvert-1$ 轮，每轮大约两次线性扫描，所以总时间复杂度写作 $O(\lvert V\rvert^2)$。Prim 适合稠密图。

## Kruskal 与 Prim 对比

| 算法 | 扩张对象 | 关键结构 | 常见实现 | 适合图 |
|---|---|---|---|---|
| Kruskal | 边 | 并查集 | 边集数组排序 | 稀疏图 |
| Prim | 点 | `lowCost` 数组 | 邻接矩阵或优先队列 | 稠密图 |

两者都基于贪心思想：每一步选一条当前看来最安全的边。差别在于 Kruskal 从全局最小边开始筛选，Prim 从一个已连通顶点集合向外扩张。

当图中存在多条等权边时，同一带权图用 Prim 和 Kruskal 可能得到边集不同的 MST；即使都使用 Kruskal，不同的等权边排序也可能得到不同的 MST。但只要都是最小生成树，最终权值和相同。
