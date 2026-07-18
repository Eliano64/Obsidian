---
title: binary constraint matching
tags:
  - DataStructureAndAlgorithm
  - Search
categories:
  - Data Structure & Algorithm
date: 2025-11-12 00:00:00
katex: true
---

# 1. 抽象

受到力扣[1. 两数之和](https://leetcode.cn/problems/two-sum/description/)以及类似问题的启发，抽象出如下模板：

**给定一个整数数组arr,令$x,y \in arr$,且$x,y$不能是同一个元素。若$f(x,y)=K$，给定$f,K$，求所有满足条件的$\{x,y\}$。**

伪代码如下
```plaintext
//input : arr, f, K
//output : all pairs of x, y in arr such that f(x, y) = K
//assumption : f(x,y)=k <=> g(x)=h(y), and g,h both are well-defined functions
algorithm binary-constraint-matching(arr, K, f):
    hash table := empty hash table, value -> list
    ans := empty set
    for each x in arr; do
      if h(x) in hash table; then
          for j in hash[h(x)]; do
            add (arr[j], x) to ans
          end loop
      end if
      attempt to add index of x to hash table[g(x)]
    end loop
    return ans
```

一个易错点在于：**应该先尝试加入ans，再尝试加入h(x)到哈希表中。**这个顺序是为了避免`ans`加入重复的元素对。

# 2. 例

[两数之和](https://leetcode.cn/problems/two-sum/description/)

[和可被 K 整除的子数组](https://leetcode.cn/problems/subarray-sums-divisible-by-k/description/)

[和为奇数的子数组数目](https://leetcode.cn/problems/number-of-sub-arrays-with-odd-sum/description/)

[统计美丽子数组数目](https://leetcode.cn/problems/count-the-number-of-beautiful-subarrays/description/)
