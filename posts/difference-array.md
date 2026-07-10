---
title: difference array
tags:
  - Difference
  - DataStructureAndAlgorithm
categories:
  - Data Structure & Algorithm
date: 2025-09-26 00:00:00
katex: true
---

# 1. 简介

对于一个数组a，构造的这样的一个数组d称为a的差分数组：

<script type="math/tex; mode=display">
d[i] =
\begin{cases}
0, & \text{if } i = 0 \\
a[i] - a[i-1], & \text{otherwise}
\end{cases}
</script>

# 2. 性质

1. 对d构造前缀和数组得到a
2. 对于a的一个下标区间[i,j)对应的元素统一加上一个数`c`后，则对应的d的变化为：`d[i](now)=>d[i](prev)+c` `d[j](now)=>d[j](prev)-c`。

也就是说，**差分数组的构造可以将一个区间的变化映射为两个元素的变化**。

> 在实际操作时，还要注意j==a.length，此时超出d索引范围（因为在前面的构造中d.length==a.length）。因此为了方便，通常让d.length=a.length+1，即设置一个dummy。

# 3. 例题

[3355.零数组变换I](https://leetcode.cn/problems/zero-array-transformation-i/description/)

go参考解法：（by 灵神）

```go
func isZeroArray(nums []int, queries [][]int) bool {
	diff := make([]int, len(nums)+1)
	for _, q := range queries {
		diff[q[0]]++
		diff[q[1]+1]--
	}

	sumD := 0
	for i, x := range nums {
		sumD += diff[i]
		// 此时 sumD 表示 x=nums[i] 要减掉多少
		if x > sumD { // x 无法变成 0
			return false
		}
	}
	return true
}
```