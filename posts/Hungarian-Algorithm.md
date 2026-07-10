---
title: Hungarian Algorithm
tags:
  - DataStructureAndAlgorithm
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-24 00:00:00
katex: true
---

# Hungarian Algorithm Notes

二分图定义：一个图是二分图，当且仅当它不包含奇数长度的环。可以使用两种颜色（通常为黑和白）对二分图的顶点进行染色，使得任意一条边的两个端点的颜色不同。

最大匹配数：从图中选出尽可能多的边，使得每条边连接的两个顶点都只被选中一次，即选出最大匹配的边数（即对数）。

最小点覆盖数：选出尽可能少的顶点，使得每条边的两个顶点中至少有一个顶点被选中。

König 定理：二分图中，最大匹配数等于这个图中的最小点覆盖数。

## 实现匈牙利算法的具体代码

下面代码中，左部点可以理解为“男生”，右部点可以理解为“女生”。`pr[y]` 表示右部点 `y` 当前匹配的左部点编号。

```cpp
const int N = 509;

vector<int> g[N];  // 邻接表表示的图：g[x] 存储左部点 x 能连接到的右部点
int pr[N];         // pr[y] 表示与右部点 y 匹配的左部点编号；0 表示尚未匹配
int vis[N];        // vis[y] == st 表示右部点 y 在第 st 轮中已被访问过

// 深度优先搜索，尝试为左部点 x 寻找匹配
bool dfs(int x, int st)
{
    for (auto& y : g[x])
    {
        // 如果右部点 y 在当前轮次未被访问
        if (vis[y] != st)
        {
            vis[y] = st;

            // 如果 y 还没有匹配对象，或者能让 y 的现任匹配对象重新匹配成功
            if (!pr[y] || dfs(pr[y], st))
            {
                pr[y] = x;
                return true;
            }
        }
    }

    return false;
}
```

## 求最大匹配数

```cpp
int match_count = 0;
for (int x = 1; x <= n_left; ++x)
{
    if (dfs(x, x))
    {
        ++match_count;
    }
}
```

`st` 使用当前左部点编号即可，这样不需要每次清空 `vis` 数组。
