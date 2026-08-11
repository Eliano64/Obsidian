---
title: Union Find
tags:
  - Tree
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:44:00
katex: true
---

# 并查集

并查集是集合逻辑结构的一种实现，用互不相交的树表示多个集合，只支持两类核心操作：

- `Find`：查找元素属于哪个集合。
- `Union`：合并两个互不相交的集合。

[html-card height=880 step=80](../assets/union-find.html)

# 存储结构

并查集通常使用数组形式的双亲表示法：

- 根结点用负数表示，其绝对值表示树的总结点数。
- 非根结点保存双亲结点的数组下标。

```c
#define SIZE 100
int parent[SIZE];

void InitSet(int parent[], int n) {
    for (int i = 0; i < n; ++i) {
        parent[i] = -1;
    }
}
```

初始时每个元素各自构成一个集合，因此每个位置都是根。

# Find 操作

```c
int Find(int parent[], int x) {
    while (parent[x] >= 0) {
        x = parent[x];
    }
    return x;
}
```

从指定元素一路沿双亲指针向上，直到找到根。判断两个元素是否属于同一集合，只需比较它们的根是否相同。

未优化时，`Find` 最坏时间复杂度等于树高，可能达到 $O(n)$。

# Union 操作

```c
void Union(int parent[], int x, int y) {
    int rootX = Find(parent, x);
    int rootY = Find(parent, y);

    if (rootX == rootY) return;
    parent[rootY] = rootX;
}
```

合并两个集合，本质上是让一棵树的根成为另一棵树根的孩子。因此可见主要的时间开销是两次 `Find` 操作寻找两集合的代表，即两树的根。

# 优化

未经优化的 `Union` 不考虑两棵树的规模，合并顺序不利时会形成很高的树，使 `Find` 沿较长的双亲链查找。下面分别从合并时控制树高、查找后缩短路径两个方向优化。

## 按规模合并

优化思路：让小树合并到大树下面，尽量避免树变高。

```c
void UnionBySize(int parent[], int x, int y) {
    int rootX = Find(parent, x);
    int rootY = Find(parent, y);

    if (rootX == rootY) return;

    if (parent[rootX] > parent[rootY]) {
        int temp = rootX;
        rootX = rootY;
        rootY = temp;
    }

    parent[rootX] += parent[rootY];
    parent[rootY] = rootX;
}
```

因为根结点保存的是负数，`parent[rootX]` 越小，集合规模越大。

按规模合并后，树高不超过 $O(\log_2 n)$，`Find` 最坏复杂度降为 $O(\log_2 n)$。

## 路径压缩

路径压缩在 `Find` 找到根后，把查找路径上的所有结点都直接挂到根结点下。

```c
int FindCompress(int parent[], int x) {
    if (parent[x] < 0) {
        return x;
    }
    parent[x] = FindCompress(parent, parent[x]);
    return parent[x];
}
```

结合按规模合并与路径压缩后，操作复杂度可达到 $O(\alpha(n))$ 级别。$\alpha(n)$ 增长极慢，常见规模下通常不超过 4。

# 时间复杂度

| 实现 | Find 与一次 Union 最坏复杂度 |
| --- | ---: |
| 不优化 | $O(n)$ |
| 按规模合并 | $O(\log_2 n)$ |
| 按规模合并 + 路径压缩 | $O(\alpha(n))$ |
