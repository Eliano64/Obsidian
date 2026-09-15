---
title: Maximum Common Elements
tags:
  - LeetCode
categories:
  - Data Structure & Algorithm
date: 2026-09-15 17:10:19
katex: true
---

# 最大公共元素数量

给定两个多重集 $S$ 和 $T$，把它们之间的元素配成若干对：每一对由一个 $S$ 中的元素与一个 $T$ 中的元素组成，两者数值必须相等；每个元素最多用一次，元素的先后次序不作要求。求最多能配出多少对。

例如 $S=\{1,1,2\}$、$T=\{1,2,2\}$ 时最多能配出 2 对：$(1,1)$ 和 $(2,2)$。

记 $f_S(x)$、$f_T(x)$ 分别为元素 $x$ 在 $S$ 与 $T$ 中的出现次数。元素 $x$ 最多能配出 $\min\bigl(f_S(x),f_T(x)\bigr)$ 对，而各元素的配对互不影响，所以答案是

$$
\sum_x \min\bigl(f_S(x),f_T(x)\bigr)
$$

# 与最长公共子序列的区别

这个问题的表述与最长公共子序列相似，但允许调换顺序，所以不需要维护元素的先后关系，也就不需要动态规划。

| 问题       | 是否要求顺序   | 方法         |
| -------- | -------- | ---------- |
| 最长公共子序列  | 要求保持相对顺序 | 动态规划       |
| 最大公共元素数量 | 不要求顺序    | 哈希表 / 频次统计 |

# 代码

思路是先用两个哈希表分别统计两个多重集中各元素的出现次数，再遍历其中一个频次表；若某个元素在另一个多重集中也出现，就把两个出现次数的较小值累加进答案。

```cpp
#include <bits/stdc++.h>
using namespace std;

int findCommon(vector<int>& S, vector<int>& T)
{
    unordered_map<int, int> freq1, freq2;

    for (const auto& elem : S)
    {
        freq1[elem]++;                  // 统计 S 中各元素的出现次数
    }

    for (const auto& elem : T)
    {
        freq2[elem]++;                  // 统计 T 中各元素的出现次数
    }

    int ans = 0;
    for (const auto& kv : freq1)
    {
        if (freq2.find(kv.first) != freq2.end())
        {
            ans += min(kv.second, freq2[kv.first]);   // 取两边的较小值
        }
    }

    return ans;
}
```

# 复杂度

两个多重集各遍历一次建表，再遍历其中一个频次表，时间复杂度为 $O(|S|+|T|)$；两张哈希表的规模分别不超过各自多重集的长度，额外空间为 $O(|S|+|T|)$。
