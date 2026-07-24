---
title: Huffman Tree
tags:
  - Tree
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:43:00
katex: true
---

# 哈夫曼树与哈夫曼编码

## 路径、权和 WPL

- 路径：从一个结点到另一个结点经过的分支序列。
- 路径长度：路径上的边数。
- 结点的权：赋给结点的数值，常表示频度、代价等。
- 结点的带权路径长度：从根到该结点的路径长度乘以该结点权值。
- 树的带权路径长度 WPL：所有叶子结点带权路径长度之和。

$$
WPL=\sum_{i=1}^{n} w_i l_i
$$

其中 $w_i$ 是第 `i` 个叶子权值，$l_i$ 是该叶子到根的路径长度。

## 哈夫曼树定义

在含有 `n` 个带权叶结点的二叉树中，WPL 最小的二叉树称为哈夫曼树，也称最优二叉树。

同一组权值可以构造出很多棵不同形态的二叉树。比较它们时只看所有叶子的 WPL：例如某组权值在几种树形下可能得到 `26`、`25`、`25`、`34` 等不同 WPL，其中 WPL 为 `25` 的树形都是哈夫曼树。哈夫曼树不要求形态唯一，只要求 WPL 达到最小。

[html-card height=720 step=80](../assets/huffman-tree-build.html)

## 构造算法

给定权值 $w_1,w_2,\ldots,w_n$：

1. 将 `n` 个权值分别作为 `n` 棵只有一个结点的二叉树，构成森林。
2. 从森林中选出根权值最小的两棵树，构造一棵新二叉树；新根权值为两棵树根权值之和。
3. 从森林中删除这两棵树，把新树加入森林。
4. 重复直到森林中只剩一棵树。

以权值 `7, 3, 2, 1, 2` 为例：

| 轮次 | 当前森林根权值 | 选出的两棵树 | 新根 | 合并后森林根权值 |
| --- | --- | --- | --- | --- |
| 初始 | `7, 3, 2, 1, 2` | - | - | `7, 3, 2, 1, 2` |
| 1 | `7, 3, 2, 1, 2` | `1, 2` | `3` | `7, 3, 2, 3` |
| 2 | `7, 3, 2, 3` | `2, 3` | `5` | `7, 3, 5` |
| 3 | `7, 3, 5` | `3, 5` | `8` | `7, 8` |
| 4 | `7, 8` | `7, 8` | `15` | `15` |

同一权值有多种选择时，树形可能不同；左右孩子也可以互换。这些差异会改变具体编码的 `0/1` 形式，但不改变最小 WPL。

如果得到的叶子深度分别为：

| 权值 | 深度 |
| --- | --- |
| `7` | `1` |
| `3` | `2` |
| `2` | `3` |
| `1` | `4` |
| `2` | `4` |

则：

$$
WPL_{\min}=7\times1+3\times2+2\times3+1\times4+2\times4=31
$$

这体现了哈夫曼树的核心直觉：权值大的叶子应尽量靠近根，权值小的叶子可以放得更深。

## 构造算法的 C 实现

顺序存储哈夫曼树时，常把叶子和后来生成的内部结点放在同一个数组中：

- 叶子结点：下标 `0..n-1`。
- 内部结点：下标 `n..2n-2`。
- 根结点：最后生成，位于下标 `2n-2`。
- `parent == -1`：该结点当前仍是森林中某棵树的根。
- `lchild == -1 && rchild == -1`：该结点是叶子结点。

```c
#include <stdio.h>

typedef struct {
    int weight;
    int parent;
    int lchild;
    int rchild;
} HuffNode;

void SelectTwoMin(HuffNode tree[], int limit, int *first, int *second) {
    *first = -1;
    *second = -1;

    for (int i = 0; i < limit; ++i) {
        if (tree[i].parent != -1) {
            continue;  // 已经被合并过，不再是当前森林的根
        }

        if (*first == -1 || tree[i].weight < tree[*first].weight) {
            *second = *first;
            *first = i;
        } else if (*second == -1 || tree[i].weight < tree[*second].weight) {
            *second = i;
        }
    }
}

void BuildHuffmanTree(HuffNode tree[], const int weights[], int n) {
    if (n <= 0) {
        return;
    }

    int total = 2 * n - 1;

    for (int i = 0; i < total; ++i) {
        tree[i].weight = 0;
        tree[i].parent = -1;
        tree[i].lchild = -1;
        tree[i].rchild = -1;
    }

    for (int i = 0; i < n; ++i) {
        tree[i].weight = weights[i];
    }

    for (int i = n; i < total; ++i) {
        int leftRoot;
        int rightRoot;

        SelectTwoMin(tree, i, &leftRoot, &rightRoot);

        tree[i].weight = tree[leftRoot].weight + tree[rightRoot].weight;
        tree[i].lchild = leftRoot;
        tree[i].rchild = rightRoot;

        tree[leftRoot].parent = i;
        tree[rightRoot].parent = i;
    }
}
```

代码要点：

- `SelectTwoMin(tree, i, ...)` 只在下标 `< i` 的已存在结点中选，因为下标 `i` 是本轮即将生成的新结点。
- 只选 `parent == -1` 的结点，因为它们才是当前森林中每棵树的根。
- 每轮生成一个新结点，权值等于两个被合并根结点的权值之和。
- 被合并的两个根结点不需要从数组中删除，只要把它们的 `parent` 改为新结点下标，它们就不再参与后续根结点选择。

计算 WPL 可以从每个叶子向上追溯到根，数出深度：

```c
int HuffmanWPL(const HuffNode tree[], int n) {
    int wpl = 0;

    for (int i = 0; i < n; ++i) {
        int depth = 0;

        for (int parent = tree[i].parent; parent != -1; parent = tree[parent].parent) {
            ++depth;
        }

        wpl += tree[i].weight * depth;
    }

    return wpl;
}
```

这里的 `n` 是叶子结点个数，不是数组总结点数。WPL 只统计叶子结点，内部结点的权值只是构造过程中的合并代价，不直接进入 WPL 求和。

## 性质

- 初始有 `n` 个叶子结点。
- 每次合并减少一棵树，共合并 `n-1` 次。
- 每次合并产生一个新内部结点，所以总结点数为 `2n-1`。且叶节点数比非叶节点数多1.
- 哈夫曼树不存在度为 1 的结点。是满二叉树。
- 哈夫曼树不唯一，但最小 WPL 相同。

## 哈夫曼编码

把字符集中的每个字符作为叶子结点，以出现频度作为权值构造哈夫曼树。通常约定：

- 左分支记为 0。
- 右分支记为 1。
- 从根到某叶子的路径编码即该字符编码。

哈夫曼编码是前缀编码：任何一个字符编码都不是另一个字符编码的前缀。因此解码时可从左到右唯一切分。

## 固定长度编码与可变长度编码

若字符集大小为 `k`，固定长度编码至少需要$\lceil \log_2 k\rceil$位来表示每个字符。

哈夫曼编码是可变长度编码，高频字符用短码，低频字符用长码，从而降低总编码长度。

例如有 4 种字符：

| 字符 | 出现次数 |
| --- | --- |
| `A` | `10` |
| `B` | `8` |
| `C` | `80` |
| `D` | `2` |

固定长度编码需要 `2` 位表示一个字符，总长度为：

$$
(10+8+80+2)\times2=200
$$

按权值构造哈夫曼树：

1. 先合并 `D:2` 和 `B:8`，得到权值 `10` 的子树。
2. 再合并 `A:10` 和上一步得到的 `10`，得到权值 `20` 的子树。
3. 最后合并 `C:80` 和 `20`，得到根权值 `100` 的整棵树。

![1000](../assets/huffman-coding-example.svg)

图中约定左分支为 `0`、右分支为 `1`，所以从根到叶子可读出：

| 字符 | 出现次数 | 编码 | 编码长度 |
| --- | --- | --- | --- |
| `C` | `80` | `0` | `1` |
| `A` | `10` | `10` | `2` |
| `D` | `2` | `110` | `3` |
| `B` | `8` | `111` | `3` |

编码总长度等于这棵哈夫曼树的 WPL：

$$
80\times1+10\times2+2\times3+8\times3=130
$$

按上表编码，字符串 `CAAABD` 可写为：

$$
C\,A\,A\,A\,B\,D \Rightarrow 0\,10\,10\,10\,111\,110
$$

即二进制串 `0101010111110`。由于哈夫曼编码是前缀编码，解码时从左到右沿哈夫曼树走，走到叶子就得到一个字符，再从根重新开始。

压缩率题常用：

$$
压缩率=\frac{压缩后长度}{压缩前长度}
$$

也可按题目要求写成节省比例：

$$
节省比例=1-\frac{压缩后长度}{压缩前长度}
$$

代入上例：

$$
压缩率=\frac{130}{200}=65\%
$$

$$
节省比例=1-\frac{130}{200}=35\%
$$

考试中要注意题目问法：如果问“压缩到原来的多少”，答 `65%`；如果问“压缩了多少”或“节省了多少”，答 `35%`。
