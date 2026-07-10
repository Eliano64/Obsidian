---
title: Selection Sorts 
tags:
  - DataStructureAndAlgorithm
  - Sort
categories:
  - Data Structure & Algorithm
date: 2026-06-29 07:12:00 
katex: true
---



# 选择类排序的共同思想

**选择排序**的共同动作是：每一趟从待排序元素中选择一个关键字最小或最大的元素，把它加入有序子序列。

两种典型算法：

- **简单选择排序**：直接在线性表中扫描，找出当前待排序区的最小元素。
- **堆排序**：用堆快速维护当前待排序区的最大或最小元素。

# 简单选择排序

简单选择排序每一趟在待排序区 $A[i..n-1]$ 中找最小元素，把它与 $A[i]$ 交换。第 $i$ 趟结束后，$A[0..i]$ 是最终有序区。

[html-card height=540](../assets/selection-sort.html)

### 简单选择排序的 C 写法

```c
/**
 * Sorts an integer array in nondecreasing order using simple selection sort.
 *
 * Args:
 *   a: Array to sort in place.
 *   n: Number of elements in the array.
 *
 * Notes:
 *   After pass i, a[0..i] contains the i+1 smallest elements in final order.
 *   The algorithm does not stop early when the input is already sorted.
 */
void selection_sort(int a[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int min_pos = i;  // 记录当前待排序区中最小元素的位置

        for (int j = i + 1; j < n; j++) {
            if (a[j] < a[min_pos]) {
                min_pos = j;
            }
        }

        if (min_pos != i) {
            int temp = a[i];
            a[i] = a[min_pos];
            a[min_pos] = temp;
        }
    }
}
```

### 简单选择排序的效率与稳定性

对 $n$ 个元素，简单选择排序需要 $n-1$ 趟。

关键字比较次数固定为：

$$
(n-1)+(n-2)+\dots+1=\frac{n(n-1)}{2}
$$

**所以无论初始序列有序、逆序还是乱序，时间复杂度都是 $O(n^2)$。**

空间复杂度为 $O(1)$。

交换次数小于等于 $n-1$，因为每一趟最多交换一次。

简单选择排序**不稳定**。例如：

```text
2a 2b 1
```

第 1 趟会把 `1` 与 `2a` 交换，得到：

```text
1 2b 2a
```

`2a` 与 `2b` 的相对次序改变。

简单选择排序既可以用于顺序表，也可以用于链表。链表不能随机访问，但每一趟只需要顺着链扫描最小结点，再把结点或数据移动到有序区。

# 堆的基本概念

**堆**首先是一棵按顺序存储表示的[[../posts/complete-binary-tree|完全二叉树]]。设关键字序列为 $L[0..n-1]$，若满足下列任一性质，就称为堆：

- **大根堆**：$L(i)\ge L(2i+1)$ 且 $L(i)\ge L(2i+2)$。
- **小根堆**：$L(i)\le L(2i+1)$ 且 $L(i)\le L(2i+2)$。

其中 $2i+1<n$，也就是只需要检查有孩子的分支结点。

堆与二叉排序树不同：

- 堆只要求**父子之间**满足大小关系。
- 二叉排序树要求**整棵左子树、根、整棵右子树**满足大小关系。

顺序存储完全二叉树时，有以下几个下标关系：

| 关系               | 0 下标写法, 合法下标取值$[0,n)$         |
| ---------------- | ----------------------------- |
| 左孩子              | `2 * i + 1`                   |
| 右孩子              | `2 * i + 2`                   |
| 父结点              | `(i - 1)/ 2`                  |
| 是否有左孩子           | `2 * i + 1 < n`               |
| 是否有右孩子           | `2 * i + 2 < n`               |
| 是否为分支结点(即是否有左孩子) | 同“是否有左孩子”写法                   |
| 是否为叶子结点          | `2 * i + 1 >= n`              |
| 结点 `i` 所在层次$h$   | $2^{h-1}\leqslant i+1 <2^{h}$ |
| `n` 个结点的树高$H$    | $2^{H-1}\leqslant n <2^{H}$   |

## 建立大根堆

建立大根堆的核心是：从最后一个分支结点 $\lfloor (n-2)/2\rfloor$ 开始，依次向前检查每个分支结点是否满足“大于等于孩子”。C 语言整数除法中，这个位置常写成 `n / 2 - 1`。若不满足，就让较小元素不断向**下坠**。

[html-card height=650](../assets/max-heap-build.html)

向下调整时的规则：

1. 比较当前结点的左右孩子，选出关键字更大的孩子。
2. 若当前结点小于这个更大的孩子，则交换。
3. 交换后，原当前结点下坠到下一层，可能继续破坏下一层堆性质，所以继续同样调整。
4. 若当前结点已经不小于孩子，或已经下坠到叶子，就停止。

## 堆排序

用大根堆进行递增排序：

1. 先把整个待排序序列建立成大根堆。
2. 堆顶元素是当前最大值，把它与当前待排序区最后一个元素交换。
3. 最大值进入最终位置，待排序区长度减一。
4. 对新的堆顶做向下调整，使待排序区重新成为大根堆。
5. 重复直到只剩一个待排序元素。

仅适用于

[html-card height=720](../assets/heap-sort-max.html)

> [!info]
> 基于大根堆的堆排序得到**递增序列**；基于小根堆的堆排序得到**递减序列**。

### 堆排序的 C 写法

下面采用 0 下标写法：`a[0..n-1]` 存放待排序元素。若当前结点下标为 `i`，左孩子为 `2 * i + 1`，右孩子为 `2 * i + 2`，父结点为 `(i - 1) / 2`。

```c
static void swap_int(int *x, int *y) {
    int temp = *x;
    *x = *y;
    *y = temp;
}

/**
 * Restores the max-heap property in a subtree whose root may be too small.
 *
 * Args:
 *   a: Zero-based array. The valid heap interval is a[0..heap_size-1].
 *   root: Root index of the subtree to adjust.
 *   heap_size: Number of elements in the current heap.
 *
 * Notes:
 *   Before the call, the left and right subtrees of root are already max-heaps.
 *   Only a[root] may violate the max-heap property, so it is pushed downward.
 */
void sift_down_max(int a[], int root, int heap_size) {
    int parent = root;

    while (2 * parent + 1 < heap_size) {
        int child = 2 * parent + 1;  // 先假设左孩子更大

        if (child + 1 < heap_size && a[child + 1] > a[child]) {
            child++;            // 右孩子存在且更大，改选右孩子
        }

        if (a[parent] >= a[child]) {
            break;              // 父结点已经不小于较大的孩子
        }

        swap_int(&a[parent], &a[child]);
        parent = child;         // 较小元素下坠后，继续检查下一层
    }
}

/**
 * Builds a max heap from a zero-based array.
 *
 * Args:
 *   a: Zero-based array to transform in place.
 *   n: Number of elements in a[0..n-1].
 *
 * Notes:
 *   Leaves are already heaps. Adjustment starts from the last internal node.
 */
void build_max_heap(int a[], int n) {
    for (int i = n / 2 - 1; i >= 0; i--) {
        sift_down_max(a, i, n);
    }
}

/**
 * Sorts a zero-based array in nondecreasing order using heap sort.
 *
 * Args:
 *   a: Zero-based array to sort in place.
 *   n: Number of elements in a[0..n-1].
 */
void heap_sort(int a[], int n) {
    build_max_heap(a, n);

    for (int heap_size = n; heap_size > 1; heap_size--) {
        swap_int(&a[0], &a[heap_size - 1]);  // 最大值进入最终位置
        sift_down_max(a, 0, heap_size - 1);  // 恢复剩余待排序区的大根堆
    }
}
```

### 堆排序的效率

| 指标     | 结论                   |
| ------ | -------------------- |
| 建堆时间   | $O(n)$               |
| 排序调整时间 | $O(n\log_2 n)$       |
| 总时间复杂度 | $O(n\log_2 n)$       |
| 空间复杂度  | $O(1)$，仅需常数个辅助单元     |
| 稳定性    | 不稳定                  |
| 适用性    | 依赖随机访问，因此**仅适用于顺序表** |

### 时间复杂度分析

堆排序的最好、平均、最坏时间复杂度都为 $O(n\log_2 n)$。它没有快速排序那种因划分极端不均匀而退化到 $O(n^2)$ 的问题。

堆排序的时间分为两部分：

$$
T(n)=T_{\text{build}}(n)+T_{\text{sort}}(n)
$$

其中 $T_{\text{build}}(n)$ 是初始建堆，$T_{\text{sort}}(n)$ 是反复交换堆顶并向下调整。

#### 单个结点下坠的比较次数

一个结点每下坠一层，关键字比较次数最多为 2：

1. 若有两个孩子，先比较左右孩子，选出较大者。
2. 再比较父结点与较大的孩子，判断是否需要交换。

若只有一个孩子，则这一层只需比较 1 次。因此用“每层最多 2 次比较”作为上界是成立的。

设完全二叉树高度为 $h$，根结点在第 1 层。若某个结点在第 $i$ 层，则它最多下坠到第 $h$ 层，所以最多下坠：

$$
h-i
$$

层。于是该结点一次向下调整的关键字比较次数不超过：

$$
2(h-i)
$$

#### 建堆阶段时间复杂度

建堆从最后一个分支结点开始，依次向前做向下调整。只有第 $1$ 层到第 $h-1$ 层的结点可能下坠，第 $h$ 层叶子不需要调整。

第 $i$ 层最多有$2^{i-1}$个结点。第 $i$ 层每个结点最多下坠 $h-i$ 层，每层最多比较 2 次，所以建堆总比较次数 $C_{\text{build}}$ 满足：

$$
C_{\text{build}}
\le
\sum_{i=1}^{h-1} 2^{i-1}\cdot 2(h-i)
$$

化简得：

$$
C_{\text{build}}
\le
\sum_{i=1}^{h-1} 2^i(h-i)
$$

令 $j=h-i$，则 $i=h-j$。当 $i=1$ 时 $j=h-1$，当 $i=h-1$ 时 $j=1$，所以：

$$
\sum_{i=1}^{h-1} 2^i(h-i)
=
\sum_{j=1}^{h-1} 2^{h-j}j
=
2^h\sum_{j=1}^{h-1}\frac{j}{2^j}
$$

而级数：

$$
\sum_{j=1}^{\infty}\frac{j}{2^j}=2
$$

所以：

$$
\sum_{j=1}^{h-1}\frac{j}{2^j}<2
$$

于是：

$$
C_{\text{build}}<2^{h+1}
$$

又因为 $n$ 个结点的完全二叉树高度为：

$$
h=\lfloor \log_2 n\rfloor+1
$$

所以：

$$
2^h=2\cdot 2^{\lfloor \log_2 n\rfloor}\le 2n
$$

因此：

$$
C_{\text{build}}<2^{h+1}\le 4n
$$

建堆阶段的关键字比较次数被线性函数控制，所以：

$$
T_{\text{build}}(n)=O(n)
$$

> [!important]
> 建堆不是对 $n$ 个结点都做一次 $O(\log n)$ 调整。大量结点靠近叶子，最多只下坠很少层；按层求和后总成本是 $O(n)$。

#### 排序阶段时间复杂度

建好大根堆后，排序阶段重复 $n-1$ 趟。每一趟做两件事：

1. 交换堆顶与当前堆底，让最大值进入最终位置。
2. 对新的堆顶做一次向下调整，让剩余待排序区恢复为大根堆。

交换是 $O(1)$。向下调整从根开始，最多下坠到叶子。若当前堆大小为 $m$，完全二叉树高度为：

$$
\lfloor \log_2 m\rfloor+1
$$

根结点最多下坠：

$$
\lfloor \log_2 m\rfloor
$$

层，每层最多比较 2 次，所以一次调整比较次数不超过：

$$
2\lfloor \log_2 m\rfloor
$$

排序阶段的总比较次数 $C_{\text{sort}}$ 满足：

$$
C_{\text{sort}}
\le
\sum_{m=2}^{n}2\lfloor \log_2 m\rfloor
$$

由于对所有 $2\le m\le n$ 都有：

$$
\lfloor \log_2 m\rfloor\le \lfloor \log_2 n\rfloor
$$

所以：

$$
C_{\text{sort}}
\le
2(n-1)\lfloor \log_2 n\rfloor
$$

因此：

$$
T_{\text{sort}}(n)=O(n\log_2 n)
$$

最终：

$$
T(n)=O(n)+O(n\log_2 n)=O(n\log_2 n)
$$


### 堆排序为什么不稳定

堆排序不稳定，原因是建堆、下坠、堆顶与堆底交换都可能让相等关键字发生远距离跨越。

例如初始序列：

```text
1 2a 2b
```

建立大根堆时，若左右孩子相等时优先与左孩子交换，可能得到：

```text
2a 1 2b
```

随后堆顶与堆底交换，可能变成：

```text
2b 1 2a
```

相同关键字 `2a`、`2b` 的相对顺序已经改变，所以堆排序不稳定。

## 堆的插入与删除

堆排序本身主要用到“建堆”和“删除堆顶最大值”的思想。单独考堆的基本操作时，还要掌握插入与删除。

[html-card height=650](../assets/max-heap-insert-delete.html)

### 插入新元素

以大根堆为例：

1. 把新元素放到表尾，保持完全二叉树形态。
2. 与父结点比较。
3. 若新元素大于父结点，就与父结点交换。
4. 重复向上，直到新元素不大于父结点，或到达根。

这叫**上升调整**。一次插入最多沿树高上升，时间复杂度为 $O(\log_2 n)$。

例子中，插入 `76` 从表尾开始，依次与 `32`、`45`、`87` 比较，共比较 3 次；插入 `70` 时先与 `17` 交换，再与 `76` 比较后停止。

### 删除堆中元素

以大根堆删除某个结点为例：

1. 删除目标结点。
2. 用堆底元素补到空位，保持完全二叉树形态。
3. 若补上来的元素破坏堆性质，就与更大的孩子交换。
4. 重复向下，直到它不小于孩子，或到达叶子。

这叫**下坠调整**。一次删除最多沿树高下坠，时间复杂度为 $O(\log_2 n)$。

例子中，删除 `76` 后用堆底 `17` 补位，`17` 需要在孩子 `45`、`70` 中选更大者，再与该孩子比较；交换后 `17` 到达叶子，调整结束。

```c
/**
 * Inserts a value into a max heap.
 *
 * Args:
 *   heap: Zero-based max heap. heap[0..*size-1] is valid before insertion.
 *   size: Pointer to the current heap size. It is increased by 1.
 *   value: New value to insert.
 */
void max_heap_insert(int heap[], int *size, int value) {
    int child = *size;
    heap[child] = value;
    (*size)++;

    while (child > 0) {
        int parent = (child - 1) / 2;

        if (heap[parent] >= heap[child]) {
            break;
        }

        swap_int(&heap[parent], &heap[child]);
        child = parent;  // 新元素继续向上
    }
}

/**
 * Restores the max-heap property after a root or internal replacement.
 *
 * Args:
 *   heap: Zero-based max heap.
 *   root: Root index of the subtree to adjust.
 *   heap_size: Number of elements in the heap.
 */
void sift_down_max_from(int heap[], int root, int heap_size) {
    int parent = root;

    while (2 * parent + 1 < heap_size) {
        int child = 2 * parent + 1;

        if (child + 1 < heap_size && heap[child + 1] > heap[child]) {
            child++;  // 大根堆要选择更大的孩子
        }

        if (heap[parent] >= heap[child]) {
            break;
        }

        swap_int(&heap[parent], &heap[child]);
        parent = child;
    }
}

/**
 * Deletes the element at index pos from a max heap.
 *
 * Args:
 *   heap: Zero-based max heap.
 *   size: Pointer to the current heap size. It is decreased by 1.
 *   pos: Index of the element to delete.
 *
 * Returns:
 *   The deleted value.
 */
int max_heap_delete_at(int heap[], int *size, int pos) {
    int deleted = heap[pos];

    if (pos == *size - 1) {
        (*size)--;
        return deleted;
    }

    heap[pos] = heap[*size - 1];
    (*size)--;

    if (pos > 0 && heap[pos] > heap[(pos - 1) / 2]) {
        while (pos > 0 && heap[pos] > heap[(pos - 1) / 2]) {
            int parent = (pos - 1) / 2;
            swap_int(&heap[pos], &heap[parent]);
            pos = parent;
        }
    } else {
        sift_down_max_from(heap, pos, *size);
    }

    return deleted;
}
```

## Top-K 问题

Top-K 问题通常是：在 $n$ 个元素中找出最大的 $k$ 个或最小的 $k$ 个，其中 $k\ll n$。

若要求**最小的 $k$ 个元素**，可以维护一个大小为 $k$ 的大根堆：

1. 先用前 $k$ 个元素建立大根堆。
2. 从第 $k+1$ 个元素开始扫描。
3. 若当前元素不小于堆顶，说明它不可能进入最小的 $k$ 个，跳过。
4. 若当前元素小于堆顶，就用当前元素替换堆顶，再向下调整。
5. 扫描结束后，堆中保留的就是最小的 $k$ 个元素。

这里使用大根堆的原因是：堆顶保存“当前候选集合中最大的那个”。一旦遇到更小的元素，就淘汰堆顶。

时间复杂度：

$$
O(k)+O((n-k)\log_2 k)=O(n\log_2 k)
$$

空间复杂度为：

$$
O(k)
$$

对偶地，若要求**最大的 $k$ 个元素**，应维护一个大小为 $k$ 的小根堆；堆顶保存当前候选集合中最小的那个，遇到更大的元素就替换堆顶。

```c
/**
 * Keeps the k smallest values in a max heap.
 *
 * Args:
 *   a: Input array.
 *   n: Number of elements in a.
 *   k: Number of smallest values to keep.
 *   heap: Output buffer with room for at least k elements.
 *
 * Notes:
 *   heap[0] is the largest value among the current k smallest candidates.
 */
void top_k_smallest(const int a[], int n, int k, int heap[]) {
    for (int i = 0; i < k; i++) {
        heap[i] = a[i];
    }

    build_max_heap(heap, k);

    for (int i = k; i < n; i++) {
        if (a[i] < heap[0]) {
            heap[0] = a[i];
            sift_down_max(heap, 0, k);
        }
    }
}
```

# 小结

| 算法 | 最好 | 平均 | 最坏 | 空间 | 稳定性 | 核心记忆 |
|---|---:|---:|---:|---:|---|---|
| 简单选择排序 | $O(n^2)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 不稳定 | 每趟扫描待排序区，选最小值 |
| 堆排序 | $O(n\log_2 n)$ | $O(n\log_2 n)$ | $O(n\log_2 n)$ | $O(1)$ | 不稳定 | 建堆 $O(n)$，每趟取堆顶后下坠 |
