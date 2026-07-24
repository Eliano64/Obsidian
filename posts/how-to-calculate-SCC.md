---
title: How to Calculate SCC
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 12:09:30
katex: true
---
[[graph-connectivity-and-components#强连通分量|强连通分量]]的常见求法有 Tarjan 和 Kosaraju。Tarjan 更适合理解代码实现，Kosaraju 更适合手算 SCC 个数。

# Tarjan

Tarjan 算法可以在线性时间内求出有向图的强连通分量。它的核心是 DFS 时间戳 `dfn`、追溯值 `low` 和一个维护当前搜索路径的栈。

## dfn 与 low

| 数组 | 含义 |
|---|---|
| `dfn[u]` | 顶点 `u` 第一次被 DFS 访问到的时间戳 |
| `low[u]` | 从 `u` 出发，沿 DFS 树边向下走，再通过一条返祖边或栈内边能回到的最小 `dfn` |

直观理解：

- `dfn` 记录访问顺序；
- `low` 记录当前顶点所在搜索分支还能向前追溯到哪里；
- 当 `dfn[u] == low[u]` 时，`u` 是某个强连通分量在 DFS 树中的根。

## 栈的作用

Tarjan 只把“已经访问但尚未确定所属 SCC”的顶点留在栈中。

遇到边 `u -> v` 时：

- 若 `v` 没访问过，先 DFS `v`，回来后用 `low[v]` 更新 `low[u]`；
- 若 `v` 仍在栈中，说明 `u` 能到达当前未完成的搜索路径上的某个点，用 `dfn[v]` 更新 `low[u]`；
- 若 `v` 已经出栈，说明 `v` 所在 SCC 已经结束，不能再用它更新 `low[u]`。

## C 代码

下面用链式前向星存有向图。顶点编号为 `1..n`。

```c
#include <stdio.h>

#define MAXV 100005
#define MAXE 200005

typedef struct {
    int to;
    int next;
} Edge;

Edge edges[MAXE];
int head[MAXV], edgeCount;

int dfn[MAXV], low[MAXV], timeStamp;
int stack[MAXV], top;
int inStack[MAXV];
int belong[MAXV], sccCount;

void addEdge(int from, int to) {
    edges[++edgeCount].to = to;
    edges[edgeCount].next = head[from];
    head[from] = edgeCount;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++timeStamp;

    stack[++top] = u;
    inStack[u] = 1;  // u 已访问，但还没有确定所属强连通分量

    for (int e = head[u]; e != 0; e = edges[e].next) {
        int v = edges[e].to;

        if (dfn[v] == 0) {
            tarjan(v);

            if (low[v] < low[u]) {
                low[u] = low[v];  // 子树能追溯到更早的顶点，u 也能追溯过去
            }
        } else if (inStack[v]) {
            if (dfn[v] < low[u]) {
                low[u] = dfn[v];  // v 还在栈内，u 能回到当前搜索路径上的 v
            }
        }
    }

    if (dfn[u] == low[u]) {
        ++sccCount;

        while (1) {
            int x = stack[top--];
            inStack[x] = 0;
            belong[x] = sccCount;

            if (x == u) {
                break;  // 从栈顶弹到 u，刚好得到一个完整 SCC
            }
        }
    }
}

void findSCC(int n) {
    for (int u = 1; u <= n; ++u) {
        if (dfn[u] == 0) {
            tarjan(u);
        }
    }
}
```

## 常见判断

| 问题 | 判断 |
|---|---|
| SCC 根 | `dfn[u] == low[u]` |
| 顶点所属 SCC | `belong[u]` |
| SCC 个数 | `sccCount` |

# Kosaraju 手算 SCC 个数

[html-card height=820](../assets/kosaraju-scc-hand.html)

Kosaraju 分两次 DFS：

1. 在原图 $G$ 上选一个点 DFS，记录每个顶点的**完成时间**（不是**访问时间**！是**退出递归调用接口的时间**）。
2. 把所有弧反向，得到反图 $G^T$。
3. 点按完成时间从大到小，在 $G^T$ 上依次 DFS。
4. 第二次 DFS 中，每启动一次 DFS 所访问到的一整块，就是一个强连通分量。

上面的例子中，按顶点编号从小到大、邻接点也从小到大进行第一次 DFS，可得到完成顺序：

$$
7,\ 5,\ 3,\ 2,\ 6,\ 4,\ 1
$$

因此完成时间从大到小为：

$$
1,\ 4,\ 6,\ 2,\ 3,\ 5,\ 7
$$

把所有弧反向后，按这个顺序在反图中 DFS：

| 入口 | 在反图中本轮能到达的未访问顶点 | 得到的 SCC |
|---|---|---|
| $1$ | $1$ | $\{1\}$ |
| $4$ | $4$ | $\{4\}$ |
| $6$ | $6$ | $\{6\}$ |
| $2$ | $2$ | $\{2\}$ |
| $3$ | $3$ | $\{3\}$ |
| $5$ | $5$ | $\{5\}$ |
| $7$ | $7$ | $\{7\}$ |

所以该图有 $7$ 个强连通分量，每个顶点各自构成一个单点 SCC。

> [!tip] 手算优先用 Kosaraju
> 若题目只要求判断 SCC 个数或划分 SCC，Kosaraju 的两次 DFS 通常比手算 Tarjan 的 `dfn/low` 更直接。若题目给的是代码、要求解释 `low`、栈或 SCC 根，则用 Tarjan。


