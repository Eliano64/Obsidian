---
title: Counting Sort 
tags:
  - DataStructureAndAlgorithm
  - Sort
categories:
  - Data Structure & Algorithm
date: 2026-06-29 09:05:00 
katex: true
---


# 基本思想

**计数排序**是一种非比较排序。它的做法是：

1. 统计每个关键字出现了多少次。
2. 根据计数直接恢复出有序序列。


> [!example]
```text
A = 2, 5, 3, 0, 2, 3, 1

C[0] = 1
C[1] = 1
C[2] = 2
C[3] = 2
C[4] = 0
C[5] = 1
```
> 从小到大输出下标`i`， 每个下标输出`C[i]`次，即可得到排序后的序列：
```text
0, 1, 2, 2, 3, 3, 5
```

若记录除了关键字还有其他信息，并且要保持稳定性，就需要用前缀和定位。

# 稳定计数排序

设关键字范围为 $0..k$。稳定计数排序通常使用三个数组：

| 数组 | 含义 |
|---|---|
| `a` | 原记录数组 |
| `count` | 计数数组，长度为 $k+1$ |
| `out` | 输出数组 |

[html-card height=760](../assets/counting-sort-process.html)

稳定版本的核心是把 `count[x]` 从“等于 x 的个数”转换为“$\le x$ 的个数”。

前缀和之后：

$$
count[x] = |\{a_i \mid key(a_i) \le x\}|
$$

因此，关键字为 $x$ 的记录最后一个位置是：

$$
count[x]-1
$$

放入一个关键字为 $x$ 的记录后，再执行：

```c
count[x]--;
```

这样下一个关键字同为 $x$ 的记录会放到前一个位置。

# 为什么要从右向左扫描

稳定版本通常从右向左扫描原数组。

例如原序列中有两个相同关键字记录：

```text
2a ... 2b
```

其中 `2a` 在前，`2b` 在后。

从右向左处理时：

1. 先处理 `2b`，把它放到关键字 `2` 的最后一个空位。
2. `count[2]--`。
3. 后处理 `2a`，它会放到 `2b` 前一个位置。

结果仍然是：

```text
2a, 2b
```

所以稳定性被保留下来。

# C 代码

下面的写法假设所有关键字都在 $0..k$ 内。

```c
#include <stdlib.h>

/**
 * Sorts an integer array using stable counting sort.
 *
 * Args:
 *   a: Array to sort in place. Every value must be in [0, max_key].
 *   n: Number of elements in the array.
 *   max_key: Maximum possible key value.
 *
 * Returns:
 *   1 if sorting succeeds, 0 if memory allocation fails or arguments are invalid.
 *
 * Notes:
 *   This implementation is stable because it scans a from right to left when
 *   placing records into out.
 */
int counting_sort(int a[], int n, int max_key) {
    if (n < 0 || max_key < 0) {
        return 0;
    }
    if (n <= 1) {
        return 1;
    }

    int *count = (int *)calloc((size_t)max_key + 1, sizeof(int));
    int *out = (int *)malloc(sizeof(int) * (size_t)n);
    if (count == NULL || out == NULL) {
        free(count);
        free(out);
        return 0;
    }

    for (int i = 0; i < n; i++) {
        if (a[i] < 0 || a[i] > max_key) {
            free(count);
            free(out);
            return 0;
        }
        count[a[i]]++;              // count[x] 先表示关键字 x 出现次数
    }

    for (int x = 1; x <= max_key; x++) {
        count[x] += count[x - 1];   // count[x] 表示 <= x 的元素个数
    }

    for (int i = n - 1; i >= 0; i--) { // 从后往前遍历输入数组
        out[count[a[i]] - 1] = a[i];  // a[i] 当前应放入的位置
        count[a[i]]--;              // 在前面的与a[i]相等的元素应放到更靠前的位置
    }

    for (int i = 0; i < n; i++) {
        a[i] = out[i];
    }

    free(count);
    free(out);
    return 1;
}
```

# 复杂度与适用条件

设元素个数为 $n$，关键字取值范围大小为 $r$。

| 项目     | 结论       |
| ------ | -------- |
| 时间复杂度  | $O(n+r)$ |
| 空间复杂度  | $O(n+r)$ |
| 稳定性    | 稳定版本稳定   |
| 是否基于比较 | 不是       |
| 适用性    | 顺序表      |

计数排序适合：

- 关键字能映射到整数。
- 关键字范围 $r$ 不大。
- 元素个数 $n$ 较大，且 $r$ 与 $n$ 同数量级或明显小于 $n$。

不适合：

- 关键字范围很大但元素很少。
- 关键字不能自然映射到有限整数范围。
- 只允许 $O(1)$ 额外空间的场景。

# 小结

| 考点 | 记法 |
|---|---|
| 排序思想 | 先计数，再用计数恢复位置 |
| 稳定版本关键 | 前缀和定位，从右向左扫描 |
| 位置公式 | `pos = count[key] - 1` |
| 放入后更新 | `count[key]--` |
| 时间复杂度 | $O(n+r)$ |
| 空间复杂度 | $O(n+r)$ |
| 局限 | 关键字范围不能太大 |
