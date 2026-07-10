---
title: Cpp STL Algorithms
tags:
  - DataStructureAndAlgorithm
  - STL
  - DataStructureAndAlgorithm
categories:
  - Cpp
date: 2026-06-24 00:00:00
katex: true
---

# Cpp STL Algorithms Notes

## 1. sort()


`sort()` 是 C++ STL 中常用的排序函数，定义于 `<algorithm>`。它可以高效地对数组、`vector`、`deque` 等支持随机访问迭代器的范围排序。

### 1.1 基本用法

默认从小到大：

```cpp
sort(vec.begin(), vec.end());
```

自定义比较函数，例如降序：

```cpp
sort(vec.begin(), vec.end(), [](int a, int b) {
    return a > b;
});
```

### 1.2 复杂度、稳定性与适用容器

- 时间复杂度：`O(n log n)`。
- `sort()` 不是稳定排序，相等元素的原始顺序可能改变。
- 如果需要稳定排序，使用 `stable_sort()`。
- `sort()` 需要随机访问迭代器，适合数组、`vector`、`deque`，不适合 `list` 或 `forward_list`。

比较函数应满足严格弱序，不要写成 `<=`。

## 2. lower_bound() 与 upper_bound()


`lower_bound()` 和 `upper_bound()` 都定义于 `<algorithm>`，常用于**有序范围**上的二分查找。它们返回的是迭代器，查找范围写作 `[first, last)`。

> 前提很重要：输入范围必须已经按同一个比较规则排好序；如果在无序序列上使用，结果不保证正确。

### 2.1 lower_bound()

`lower_bound(first, last, val)` 返回第一个**不小于** `val` 的元素位置，也就是第一个 `>= val` 的位置。

如果所有元素都小于 `val`，返回 `last`。如果存在多个等于 `val` 的元素，返回其中第一个。

示例代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 7, 9};
    int val = 5;

    // 查找有序容器中第一个大于或等于 val 的元素
    auto it = lower_bound(v.begin(), v.end(), val);

    if (it != v.end())
        cout << "Found at index: " << it - v.begin();
    else
        cout << "Not found";

    return 0;
}
```

### 2.2 upper_bound()

`upper_bound(first, last, val)` 返回第一个**大于** `val` 的元素位置，也就是第一个 `> val` 的位置。

如果所有元素都不大于 `val`，返回 `last`。如果存在多个等于 `val` 的元素，它会跳过这些相等元素，返回下一个更大的元素。

示例代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 7, 9};
    int val = 5;

    // 查找有序容器中第一个大于 val 的元素
    auto it = upper_bound(v.begin(), v.end(), val);

    if (it != v.end())
        cout << "First element greater than " << val
             << " is at index: " << it - v.begin();
    else
        cout << "No elements are greater than " << val;

    return 0;
}
```

### 2.3 对照表

复杂度对比：

| 项目 | `lower_bound()` | `upper_bound()` |
|---|---|---|
| 功能 | 查找第一个不小于给定值的元素 | 查找第一个大于给定值的元素 |
| 返回值 | 迭代器 | 迭代器 |
| 条件要求 | 输入序列必须有序 | 输入序列必须有序 |
| 时间复杂度 | `O(log n)` | `O(log n)` |
| 返回元素条件 | 第一个 `>= val` 的元素 | 第一个 `> val` 的元素 |
| 返回 `end()` 的条件 | 所有元素都 `< val` | 所有元素都 `<= val` |

常见组合用法：在有序序列中统计 `val` 出现次数。

```cpp
auto l = lower_bound(v.begin(), v.end(), val);
auto r = upper_bound(v.begin(), v.end(), val);
int cnt = r - l;
```

## 3. next_permutation() 与 prev_permutation()


`next_permutation()` 和 `prev_permutation()` 是处理排列的重要工具。

### 2.1 next_permutation()

作用：生成当前排列的下一个字典序排列。

```cpp
next_permutation(a.begin(), a.end());
```

返回值：

- 若存在下一个排列，返回 `true`，并修改原序列；
- 若不存在下一个排列，返回 `false`。

枚举全部排列：

```cpp
sort(a.begin(), a.end());

do {
    // 使用当前排列
} while (next_permutation(a.begin(), a.end()));
```

### 2.2 prev_permutation()

作用：生成当前排列的上一个字典序排列。

```cpp
prev_permutation(a.begin(), a.end());
```

它同样会修改原排列，并返回是否存在上一个排列。

## 4. reverse()


`reverse()` 用于反转指定范围内的元素顺序。它接收两个迭代器，表示 `[first, last)`。

```cpp
reverse(v.begin(), v.end());
```

示例代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5}; // 定义一个整数类型的向量 v，并初始化为 {1,2,3,4,5}

    reverse(v.begin(), v.end());     // 使用 reverse() 函数将向量 v 的元素顺序反转

    for (int i : v) {
        cout << i << " ";            // 遍历向量 v，输出每个元素并在元素之间添加空格
    }

    // 输出结果：5 4 3 2 1
    return 0;
}
```

迭代器要求：标准库 `reverse` 实际要求双向迭代器；数组、`vector`、`string`、`deque`、`list` 都可以使用。竞赛里最常见的是对数组、`vector`、`string` 使用。

## 5. swap()


`swap()` 用于交换两个同类型对象的值。

示例代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a = 5, b = 10;       // 定义两个整数变量 a 和 b，分别赋值为 5 和 10

    swap(a, b);              // 调用 swap() 函数交换 a 和 b 的值

    cout << "a = " << a << ", b = " << b; // 输出交换后的 a 和 b，即 a = 10, b = 5

    return 0;
}
```

注意：`swap()` 只交换两个对象的值，而不改变它们在内存中的位置。

复杂度理解：

- 对基本类型，通常是 `O(1)`；
- 对标准库容器，通常会交换内部指针/元数据，通常也很快；
- 对自定义类型，复杂度取决于其移动/交换实现。
