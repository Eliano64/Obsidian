---
title: sieve of eratosthenes
tags:
  - DataStructureAndAlgorithm
categories:
  - Data Structure & Algorithm
date: 2025-08-24 00:00:00
katex: true
---

埃氏筛是一种简单的筛法，用于求小于等于n的所有质数。

# 流程

```text
Prime(n):
    mark i from 2 to n as prime (initial)
    for i from 2 to n:
            if i is marked as prime:
                for j from i^2 to n with step i:
                    mark j as not prime
    return all marked primes
```

## 解释：

1. 为什么是从$i^2$开始标记非质数？
    - 因为如果一个数不是质数，那么它一定有一个小于等于$\sqrt{n}$的质因子。
    - 所以，我们只需要从$i^2$开始标记，因为小于$i^2$的数，一定已经被小于$i$的数标记过了。
