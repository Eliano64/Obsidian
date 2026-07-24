---
title: Insertion Sorts
tags:
  - Sort
categories:
  - Data Structure & Algorithm
date: 2026-06-29 06:08:23
katex: true
---

# Insertion Sorts

## 插入排序的共同思想

插入类排序的核心是：

1. 把一部分元素看作已经有序。
2. 每次取出一个待排序元素。
3. 在已有序部分中找到它应该插入的位置。
4. 移动元素，腾出位置，把该元素放进去。

直接插入、折半插入、希尔排序都属于这个思路，但“已有序部分”的定义和“插入位置”的查找方式不同。

![1000](../assets/insertion-sort-overview.svg)

## 直接插入排序

**直接插入排序**每一趟都把 $A[i]$ 插入到前面已经有序的子序列 $A[0..i-1]$ 中。


[html-card height=620](../assets/direct-insertion-sort.html)

### 直接插入排序的 C 写法

```c
/**
 * Sorts an integer array in nondecreasing order using direct insertion sort.
 *
 * Args:
 *   a: Array to sort in place.
 *   n: Number of elements in the array.
 *
 * Notes:
 *   The prefix a[0..i-1] is already sorted before each outer-loop iteration.
 *   The algorithm is stable because equal elements are not moved past temp.
 */
void insertion_sort(int a[], int n) {
    for (int i = 1; i < n; i++) {
        int temp = a[i];      // 待插入元素
        int j = i - 1;        // 从有序前缀的最右端开始比较

        while (j >= 0 && a[j] > temp) {
            a[j + 1] = a[j];  // 较大元素后移，给 temp 腾位置
            j--;
        }

        a[j + 1] = temp;      // j 停在第一个 <= temp 的位置
    }
}
```

这里的稳定性来自条件 `a[j] > temp`，而不是 `a[j] >= temp`。

- 若 `a[j] > temp`，较大元素后移。
- 若 `a[j] == temp`，停止移动，新的相等元素会插在旧的相等元素之后。

### 带哨兵的直接插入排序

有些教材使用 `A[0]` 存放临时变量，真正元素从 `A[1]` 到 `A[n]`。这样可以少写一次 `j >= 0` 或 `j >= 1` 的边界判断。

```c
/**
 * Sorts a[1..n] using insertion sort with a sentinel at a[0].
 *
 * Args:
 *   a: Array where a[1..n] stores records and a[0] is temporary storage.
 *   n: Number of actual records.
 */
void insertion_sort_with_sentinel(int a[], int n) {
    for (int i = 2; i <= n; i++) {
        if (a[i] < a[i - 1]) {
            a[0] = a[i];      // 哨兵位保存待插入元素
            int j = i - 1;

            while (a[j] > a[0]) {
                a[j + 1] = a[j];
                j--;
            }

            a[j + 1] = a[0];
        }
    }
}
```

`A[0]` 的作用是临时保存待插入元素，并让循环不用每次检查是否越界。但这种写法要求数组额外保留 0 号位置。

### 直接插入排序的效率

| 情况 | 输入特点 | 比较与移动 | 时间复杂度 |
|---|---|---|---|
| 最好 | 原本有序 | 每趟只比较一次，不移动元素 | $O(n)$ |
| 最坏 | 原本逆序 | 第 $i$ 趟要向前比较并移动很多元素 | $O(n^2)$ |
| 平均 | 一般随机输入 | 比较和移动数量级均为平方级 | $O(n^2)$ |

直接插入排序的空间复杂度为 $O(1)$，稳定。

若采用带哨兵的一基下标写法，在逆序的最坏情况下，第 $i$ 趟常按如下方式估算：

- 关键字比较约为 $i-1$ 次。
- 元素移动约为 $i-1$ 次。

求总量时对 $i=1,2,\dots,n-1$ 累加，所以数量级为 $O(n^2)$。

对于链表，插入时不需要像顺序表那样整体移动元素，但查找插入位置仍要从前向后比较，整体时间复杂度仍是 $O(n^2)$。

## 折半插入排序

**折半插入排序**是对直接插入排序的改进：在前面有序子序列中，用折半查找先确定插入位置，再统一移动元素。

它只优化了“找位置”的比较次数，没有优化“移动元素”的次数。因此整体时间复杂度仍是 $O(n^2)$。

[html-card height=620](../assets/binary-insertion-sort.html)

### 折半插入排序的 C 写法

查找实现的详细解释见[[../posts/binary-search|binary-search]]

```c
/**
 * Sorts an integer array using binary insertion sort.
 *
 * Args:
 *   a: Array to sort in place.
 *   n: Number of elements in the array.
 *
 * Notes:
 *   Binary search reduces key comparisons, but elements still need shifting.
 *   Equal keys are inserted after existing equal keys to keep stability.
 */
void binary_insertion_sort(int a[], int n) {
    for (int i = 1; i < n; i++) {
        int temp = a[i];
        int low = 0;
        int high = i;
        int ins = i;

        while (low < high) {
            int mid = low + (high - low) / 2;

            if (a[mid] > temp) { //为了稳定性，规定插入位置是在有序表中第一个**大于**新元素的元素的位置，所以相等时mid不是我们期望的解，不取等号
                ins = mid;   
                high = mid;
            } else {
                low = mid + 1;
            }
        }

        for (int j = i - 1; j >= low; j--) {
            a[j + 1] = a[j];     // 移动次数没有减少
        }

        a[ins] = temp;
    }
}
```

### 折半插入排序与直接插入排序

| 项目      | 直接插入排序          | 折半插入排序           |
| ------- | --------------- | ---------------- |
| 查找插入位置  | 顺序向前比较          | 折半查找             |
| 关键字比较次数 | 较多              | 较少               |
| 元素移动次数  | 取决于插入位置         | 没有减少             |
| 整体时间复杂度 | 平均、最坏为 $O(n^2)$ | 平均、最坏仍为 $O(n^2)$ |
| 空间复杂度   | $O(1)$          | $O(1)$           |
| 稳定性     | 稳定              | 稳定，前提是相等时向右找     |

折半插入排序要把“比较次数”和“移动次数”分开看：

- 比较次数：每趟用折半查找，数量级约为 $O(\log i)$，总比较次数约为 $O(n\log n)$。
- 移动次数：插入位置之后的元素仍要整体右移，最坏和平均情况下仍是 $O(n^2)$。
- 原表有序时，移动很少，但仍要做折半定位，因此不能把它简单理解为直接插入排序那种 $O(n)$ 最好情况。

## 希尔排序

**希尔排序**又叫缩小增量排序。它不是直接对整个表做一次插入排序，而是先按增量 $d$ 把表分成若干个子表：

$$
L[i], L[i+d], L[i+2d], \dots
$$

每个子表内部做直接插入排序。然后缩小增量 $d$，重复这一过程，直到 $d=1$。最后一趟 $d=1$ 时，整个表就是一个子表，相当于对全表做直接插入排序。

[html-card height=610](../assets/shell-sort.html)

### 希尔排序的理解

直接插入排序在“基本有序”的序列上效率较高。希尔排序利用这一点：

1. 先用较大的增量让远距离元素提前移动。
2. 每一趟都让表更接近整体有序。
3. 最后一趟增量为 1 时，直接插入排序面对的是一个比较接近有序的表。

例如初始序列：

```text
49 38 65 97 76 13 27 49
```

若增量序列为 $4,2,1$：

```text
d = 4: 49 13 27 49 76 38 65 97
d = 2: 27 13 49 38 65 49 76 97
d = 1: 13 27 38 49 49 65 76 97
```

考试中可能给出不同增量序列。只要最后一个增量是 1，就能保证最终整体有序。

### 希尔排序的 C 写法

```c
/**
 * Sorts an integer array using Shell sort with gap halving.
 *
 * Args:
 *   a: Array to sort in place.
 *   n: Number of elements in the array.
 *
 * Notes:
 *   For a fixed gap, this is insertion sort on gap-separated subsequences.
 *   This implementation uses n/2, n/4, ..., 1 as the gap sequence.
 */
void shell_sort(int a[], int n) {
    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i++) {
            int temp = a[i];
            int j = i - gap;

            while (j >= 0 && a[j] > temp) {
                a[j + gap] = a[j];  // 同一子表内后移 gap 个位置
                j -= gap;
            }

            a[j + gap] = temp;
        }
    }
}
```

这段代码可以看作直接插入排序的“跨步版本”：

- 直接插入排序：比较 `a[j]` 与 `temp`，移动到 `a[j+1]`。
- 希尔排序：比较 `a[j]` 与 `temp`，移动到 `a[j+gap]`。

### 希尔排序的性质

| 性质 | 结论 |
|---|---|
| 空间复杂度 | $O(1)$ |
| 时间复杂度 | 与增量序列有关；最坏可为 $O(n^2)$ |
| 常见经验复杂度 | 某些规模和增量下可接近 $O(n^{1.3})$ |
| 稳定性 | 不稳定 |
| 适用结构 | 仅适合顺序表，不适合链表 |

希尔排序不稳定的原因是：相同关键字可能被分到不同子表中，跨组移动时可能改变相对次序。

例如：

```text
65 49a 49b
```

若第一趟增量为 2，`65` 与 `49b` 同组，排序后可能得到：

```text
49b 49a 65
```

相同关键字 `49a`、`49b` 的相对位置改变，所以希尔排序不稳定。

希尔排序不适合链表，是因为它依赖按下标快速访问 `i, i+d, i+2d` 这些位置；链表虽然插入方便，但无法高效地按增量跳到指定结点。

## 小结

| 算法     | 核心动作       |                最好情况 |     平均情况 |     最坏情况 |     空间 | 稳定性 | 适用性  |
| ------ | ---------- | ------------------: | -------: | -------: | -----: | --- | ---- |
| 直接插入排序 | 顺序找位置，元素后移 |              $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定  | 线性表  |
| 折半插入排序 | 折半找位置，元素后移 | 比较 $O(n\log n)$，移动少 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定  | 仅顺序表 |
| 希尔排序   | 增量分组，多趟插入  |               与增量有关 |    与增量有关 | $O(n^2)$ | $O(1)$ | 不稳定 | 仅顺序表 |

