---
title: Merge Sort 
tags:
  - DataStructureAndAlgorithm
  - Sort
categories:
  - Data Structure & Algorithm
date: 2026-06-29 08:10:00 
katex: true
---


# 归并的基本思想

**归并**：把两个或多个已经有序的序列合并成一个新的有序序列。

归并排序最常用的是**二路归并**：

1. 把待排序序列拆成若干个有序子表。
2. 每次把相邻的两个有序子表归并成一个更长的有序子表。
3. 子表长度逐趟翻倍，直到整个序列有序。

内部排序中通常采用二路归并；多路归并更常出现在外部排序中。

若朴素地做 $m$ 路归并，每次要从 $m$ 个子表的当前元素中选出最小者，最多需要比较 $m-1$ 次。二路归并时 $m=2$，所以每选出一个元素只需要比较 1 次。

# 二路归并

设数组中已有两个相邻有序子表：

$$
A[left, mid),\quad A[mid, right)
$$

二路归并要把它们合成一个有序区间 $A[left, right)$。

[html-card height=640](../assets/merge-two-runs.html)

核心指针含义：

| 指针 | 含义 |
|---|---|
| `i` | 指向左子表当前待比较元素 |
| `j` | 指向右子表当前待比较元素 |
| `k` | 指向原数组中当前写回位置 |

归并时每次比较 `B[i]` 和 `B[j]`：

- 若 `B[i] <= B[j]`，把 `B[i]` 写回 `A[k]`，然后 `i++`、`k++`。
- 若 `B[i] > B[j]`，把 `B[j]` 写回 `A[k]`，然后 `j++`、`k++`。
- 若某个子表先用完，把另一个子表剩余元素依次复制到尾部。

**相等时优先取左子表元素，是归并排序稳定的关键。**

# Merge 的 C 写法

```c
/**
 * Merges two adjacent sorted ranges into one sorted range.
 *
 * Args:
 *   a: Array that stores the two sorted ranges.
 *   aux: Auxiliary array with at least the same length as a.
 *   left: First index of the left range.
 *   mid: First index after the left range, and first index of the right range.
 *   right: First index after the right range.
 *
 * Notes:
 *   Before the call, a[left, mid) and a[mid, right) must already be sorted.
 *   After the call, a[left, right) is sorted.
 *   The condition aux[i] <= aux[j] preserves stability.
 */
static void merge(int a[], int aux[], int left, int mid, int right) {
    for (int p = left; p < right; p++) {
        aux[p] = a[p];  // 先复制，避免写回 A 时覆盖尚未比较的元素
    }

    int i = left;       // 左子表当前元素
    int j = mid;        // 右子表当前元素
    int k = left;       // A 中当前写回位置

    while (i < mid && j < right) {
        if (aux[i] <= aux[j]) {
            a[k++] = aux[i++];  // 相等时取左侧元素，保持稳定
        } else {
            a[k++] = aux[j++];
        }
    }

    while (i < mid) {
        a[k++] = aux[i++];      // 右子表已空，复制左子表剩余元素
    }

    while (j < right) {
        a[k++] = aux[j++];      // 左子表已空，复制右子表剩余元素
    }
}
```

# 归并排序的分治结构

递归版归并排序是典型的分治：

1. **分解**：把 $A[left, right)$ 分成左右两个左闭右开区间。
2. **解决**：分别对左右两半进行归并排序。
3. **合并**：左右两半都有序后，对它们做一次二路归并。

递归到区间只含一个元素时，该区间天然有序。

```c
#include <stdlib.h>

/**
 * Sorts an integer array in nondecreasing order using recursive merge sort.
 *
 * Args:
 *   a: Array to sort in place.
 *   n: Number of elements in the array.
 *
 * Returns:
 *   1 if sorting succeeds, 0 if auxiliary memory allocation fails.
 *
 * Notes:
 *   The auxiliary array is allocated once and reused by all merge operations.
 *   This keeps the extra space O(n), rather than allocating a new array in
 *   every recursive call.
 */
int merge_sort(int a[], int n);

static void merge_sort_range(int a[], int aux[], int left, int right) {
    if (right - left <= 1) {
        return;  // 只有 0 个或 1 个元素，已经有序
    }

    int mid = left + (right - left) / 2;

    merge_sort_range(a, aux, left, mid);
    merge_sort_range(a, aux, mid, right);
    merge(a, aux, left, mid, right);
}

int merge_sort(int a[], int n) {
    if (n <= 1) {
        return 1;
    }

    int *aux = (int *)malloc(sizeof(int) * n);
    if (aux == NULL) {
        return 0;
    }

    merge_sort_range(a, aux, 0, n);
    free(aux);
    return 1;
}
```

递归过程可以看成一棵倒置的二叉树：底层是长度为 1 的有序子表，向上不断两两归并。

# 二路归并过程

对序列：

```text
49  38  65  97  76  13  27
```

按子表长度逐趟归并：

[html-card height=560](../assets/merge-sort-passes.html)

| 归并趟数 | 子表长度 | 结果 |
|---|---:|---|
| 初始 | 1 | `49 38 65 97 76 13 27` |
| 第 1 趟 | 1 -> 2 | `38 49 65 97 13 76 27` |
| 第 2 趟 | 2 -> 4 | `38 49 65 97 13 27 76` |
| 第 3 趟 | 4 -> 8 | `13 27 38 49 65 76 97` |

若某一趟末尾剩余子表不足标准长度，也仍然作为一个有序子表参与归并或直接保留到下一趟。

# 归并的关键字比较次数

## 一次二路归并比较次数


先看一次二路归并。设两个有序子表长度分别为 $p$ 和 $q$：

$$
A[left, mid),\quad A[mid, right)
$$

其中：

$$
p = mid-left,\quad q = right-mid
$$

每次比较只发生在：

```c
if (aux[i] <= aux[j])
```

一旦某个子表用完，另一个子表剩余元素只需要直接复制，不再比较关键字。

| 情况  |        比较次数 | 原因                         |
| --- | ----------: | -------------------------- |
| 最少  | $\min(p,q)$ | 较短子表的元素连续胜出，较短子表先空         |
| 最多  |     $p+q-1$ | 两个子表交替取元素，直到最后只剩 1 个元素无需再比 |
|     |             |                            |

>[!example]
>例如左子表长度 $4$、右子表长度 $3$：>
>- 最少比较 $\min(4,3)=3$ 次。
>- 最多比较 $4+3-1=6$ 次。

若两个子表长度都为 $m$，一次归并的比较次数范围是：

$$
m \le C \le 2m-1
$$

## 整个归并排序的比较次数

取 $n=2^k$，即每次都能均分的情况。

* 递归归并排序的**最坏**比较次数$W(n)$满足：

$$
W(n)=2W(n/2)+(n-1),\quad W(1)=0
$$

展开可得：

$$
W(n)=n\log_2 n-n+1
$$

* 最好情况下，每次归并时某一边很快用完。比较次数$B(n)$有：

$$
B(n)=2B(n/2)+\frac n2,\quad B(1)=0
$$

所以：

$$
B(n)=\frac n2\log_2 n
$$


考试中若给出具体序列并要求手算比较次数，不要直接套公式；应画出[[merge-sort#二路归并过程|具体归并过程]]，按每一次 Merge 的实际比较过程累计。


# 归并排序的效率

对 $n$ 个元素进行二路归并排序，若归并树高度为 $h$，底层需要容纳 $n$ 个叶结点。二叉树第 $h$ 层最多有 $2^{h-1}$ 个结点，因此：

$$
n \le 2^{h-1}
$$

所以：

$$
h - 1 = \lceil \log_2 n \rceil
$$

也就是说，二路归并排序需要 $\lceil \log_2 n \rceil$ 趟归并。

每一趟归并会把所有元素处理一遍，时间为 $O(n)$，所以总时间为：

$$
T(n)=O(n)\cdot \lceil \log_2 n \rceil = O(n\log_2 n)
$$

| 情况 | 时间复杂度 |
|---|---:|
| 最好 | $O(n\log_2 n)$ |
| 平均 | $O(n\log_2 n)$ |
| 最坏 | $O(n\log_2 n)$ |

归并排序的空间复杂度为 $O(n)$，主要来自辅助数组 `aux`。递归版还需要 $O(\log_2 n)$ 的递归栈空间，但通常被 $O(n)$ 主导。

# 稳定性与适用性

归并排序是**稳定排序**。

稳定性来自归并时的相等处理：

```c
if (aux[i] <= aux[j]) {
    a[k++] = aux[i++];
}
```

当左、右子表当前元素相等时，先取左子表元素。由于左子表中的元素在原序列中本来就在右子表元素之前，所以相等关键字的相对次序不会改变。

归并排序适合顺序表，也适合链表：

- 顺序表归并时常用辅助数组，空间复杂度为 $O(n)$。
- 链表归并时可以通过修改指针合并两个有序链表，不需要像数组那样整体移动元素。

# 小结

| 项目     | 结论                       |
| ------ | ------------------------ |
| 核心操作   | 把两个有序子表归并为一个有序子表         |
| 常用归并路数 | 内部排序通常使用二路归并             |
| 归并趟数   | $\lceil \log_2 n \rceil$ |
| 单次二路归并比较次数 | $\min(p,q)$ 到 $p+q-1$ |
| $n=2^k$ 时最坏比较次数 | $n\log_2 n-n+1$ |
| 每趟时间   | $O(n)$                   |
| 总时间复杂度 | $O(n\log_2 n)$           |
| 空间复杂度  | $O(n)$                   |
| 稳定性    | 稳定                       |
| 适用存储   | 顺序表、链表均可                 |
