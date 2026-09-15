---
title: Boyer Moore Majority Vote
tags:
  - Array
categories:
  - Data Structure & Algorithm
date: 2026-09-15 17:10:19
katex: true
---

# 寻找多数元素

给定长度为 $n$ 的数组，求其中出现次数严格大于 $\lfloor n/2 \rfloor$ 的元素，它称为多数元素。

# 思路

若某元素的出现次数超过总数的一半，则它比其他所有元素的出现次数之和还多，即

$$
count_{ans}-count_{others}>0
$$

所以可以把不同元素两两抵消：遍历数组，维护候选元素 `candidate` 与它的净票数 `count`。

- `count` 为 0 时，把当前元素设为新的 `candidate`；
- 当前元素与 `candidate` 相同，`count` 加一，不同则减一。

抵消到最后仍然留下的 `candidate` 就是多数元素。

# 代码

```cpp
int BoyerMoore(int* array, int arraySize)
{
    int candidate = array[0];
    int count = 0;
    for (int i = 0; i < arraySize; i++)
    {
        if (count == 0)
        {
            candidate = array[i];   // 票数清零，换用当前元素作为候选
        }
        if (array[i] == candidate)
        {
            count++;                // 与候选相同，票数加一
        }
        else
        {
            count--;                // 与候选不同，两票抵消
        }
    }
    return candidate;
}
```

# 示例

$array=\{1,1,1,2,3,3,3,3,3\}$ 最终候选是 3。

| i   | array[i] | candidate | count |
| --- | -------- | --------- | ----- |
| 0   | 1        | **1**     | 1     |
| 1   | 1        | 1         | 2     |
| 2   | 1        | 1         | 3     |
| 3   | 2        | 1         | 2     |
| 4   | 3        | 1         | 1     |
| 5   | 3        | 1         | **0** |
| 6   | 3        | **3**     | 1     |
| 7   | 3        | 3         | 2     |
| 8   | 3        | 3         | 3     |

# 复杂度

只有一次遍历，时间 $O(n)$；除 `candidate`、`count` 外不需要额外空间，空间 $O(1)$。
