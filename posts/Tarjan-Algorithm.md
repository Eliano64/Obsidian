---
title: Tarjan Algorithm
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-24 00:00:00
katex: true
---

Tarjan 算法常用于求解：

- 有向图中的强连通分量（SCC, Strongly Connected Components）；
- 无向图中的割点；
- 无向图中的割边，又称桥。

核心思想是 DFS 时间戳和 `low` 数组。

## 1. 核心数组

| 数组 | 含义 |
|---|---|
| `dfn[u]` | 节点 `u` 第一次被 DFS 访问到的时间戳 |
| `low[u]` | 从 `u` 或 `u` 的子树出发，经过树边和返祖边能到达的最小 `dfn` |

可以直观理解为：

- `dfn` 记录“访问顺序”；
- `low` 记录“最多能往祖先回到哪里”。

## 2. 有向图强连通分量 SCC

强连通分量指的是有向图中的一个最大点集，其中任意两个点都可以互相到达。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;

vector<int> g[N];
int dfn[N], low[N], in_stack[N], belong[N];
vector<int> stk;
int timer = 0, scc_cnt = 0;

void tarjan(int u)
{
    dfn[u] = low[u] = ++timer;
    stk.push_back(u);
    in_stack[u] = 1;

    for (int v : g[u])
    {
        if (!dfn[v])
        {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        }
        else if (in_stack[v])
        {
            low[u] = min(low[u], dfn[v]);
        }
    }

    if (dfn[u] == low[u])
    {
        ++scc_cnt;
        while (true)
        {
            int x = stk.back();
            stk.pop_back();
            in_stack[x] = 0;
            belong[x] = scc_cnt;

            if (x == u) break;
        }
    }
}
```

使用时，对每个未访问节点调用：

```cpp
for (int i = 1; i <= n; ++i)
{
    if (!dfn[i]) tarjan(i);
}
```

## 3. 无向图割边（桥）

在无向图中，如果删除一条边后，图的连通分量数量增加，则这条边是桥。

对于 DFS 树边 `u -> v`，如果：

```text
low[v] > dfn[u]
```

说明 `v` 的子树无法通过返祖边回到 `u` 或 `u` 的祖先，因此边 `(u, v)` 是桥。

## 4. 无向图割点

在无向图中，如果删除某个点后，图的连通分量数量增加，则这个点是割点。

判断规则：

- 如果 `u` 不是 DFS 根节点，并且存在子节点 `v` 满足 `low[v] >= dfn[u]`，则 `u` 是割点。
- 如果 `u` 是 DFS 根节点，并且它有两个及以上 DFS 子树，则 `u` 是割点。

## 5. 记忆重点

| 问题 | 关键判断 |
|---|---|
| SCC 根 | `dfn[u] == low[u]` |
| 桥 | `low[v] > dfn[u]` |
| 割点（非根） | `low[v] >= dfn[u]` |
| 割点（根） | DFS 子树数量 ≥ 2 |
