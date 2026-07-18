---
title: Prefix Sum
tags:
  - DataStructureAndAlgorithm
  - Array
categories:
  - Data Structure & Algorithm
date: 2026-06-24 00:00:00
katex: true
---

# 前缀和

区间求和类题，问求和满足某某性质的区间有多少个。

如果直接暴力求解大概率超时。

但是，可以进行预处理，比如，将所有前缀求和，得到一个新的数组。

这样，一个区间和就变成了前缀和数组**少量**的元素间加减的结果。

一维前缀和：[1.k倍区间 - 蓝桥云课](https://www.lanqiao.cn/problems/97/learning/?page=1&first_category_id=1&second_category_id=3&difficulty=20)

二维前缀和：[小美的平衡矩阵__牛客网](https://www.nowcoder.com/questionTerminal/d3a6268bbbf743f6b7441eb7ab30633e)

在构造前缀和数组时候，为了边界处理的方便，数组大小会大一点。然后遍历从idx=1开始

前缀和可以将暴力法的$O(n^{2})$下降到$O(n)$

## 1.1 一维前缀和模板

```cpp
vector<long long> pre(n + 1, 0);
for (int i = 0; i < n; i++) {
    pre[i + 1] = pre[i] + a[i];
}

// 区间 [l, r] 的和
long long sum = pre[r + 1] - pre[l];
```

## 1.2 二维前缀和模板

```cpp
s[i][j] = a[i][j] + s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];

long long ans = s[x2][y2]
              - s[x1 - 1][y2]
              - s[x2][y1 - 1]
              + s[x1 - 1][y1 - 1];
```
