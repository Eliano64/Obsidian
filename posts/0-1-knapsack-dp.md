---
title: 0 1 knapsack dp
tags:
  - DynamicProgramming
categories:
  - Data Structure & Algorithm
date: 2025-09-26 00:00:00
katex: true
---

本文的内容是0-1背包的空间优化、倒序遍历，以及至多/恰好/至少变体。一些[[Dynamic-Programming#2. 0-1 背包|基本定义]]不再赘述。

# 1. 空间优化

由于它的递推表达式：

$$
dp[i][c] = \max\{\, dp[i-1][c],\; dp[i-1][c-w[i]] + v[i] \,\}
$$

dp\[i\]\[c\]: 遍历到第 i 个物品时，若总容量为 c 时能获得的最大价值

可以发现每次更新值时，都是 i-1 更新 i 的，所以完全可以写为：

$$
dp[c] = \max\{\, dp[c],\; dp[c-w[i]] + v[i] \,\}
$$

# 2. 倒序遍历

在使用一维数组进行空间优化时，容量 c 必须“倒序遍历”（从最大容量 C 递减到 w[i]）。

*Why?*

- 若正序遍历（c 从小到大），在同一轮 i 的更新中，`dp[c-w[i]]` 已经被本轮 i 更新过，会导致同一物品 i 被复用多次，等价成“完全背包”的效果；
- 倒序遍历能保证本轮计算依赖的是“上一轮 i-1 的结果”，从而保持 0-1 约束。

更形式化的解释：维护不变式“在处理物品 i 时，所有用于计算的 `dp[*]` 均来自 i-1 的状态”。倒序使得 `dp[c-w[i]]` 在本轮尚未更新，仍是 i-1 的值。

一个反例（正序导致复用）：容量 `c = w[i]` 更新了 `dp[w[i]]`，随后 `c = 2*w[i]` 又读到刚更新过的 `dp[w[i]]`，把 i 用了两次。

# 3. 初始化与变体

根据目标约束不同，`dp` 的初始化与转移边界需要调整：

- 0-1 背包（至多容量 C）：
  - 初始化：`dp[0..C] = 0`；
  - 转移：倒序 `for c = C..w[i]`，`dp[c] = max(dp[c], dp[c - w[i]] + v[i])`；
  - 结果：`max(dp[0..C])`，常规题通常取 `dp[C]`。

- 恰好容量的方案数（计数问题）：
  - 目标：选择若干件物品使总容量恰好为 `C` 的子集数（不计顺序）。
  - 初始化：`dp[0] = 1`，其余 `dp[1..C] = 0`；
  - 转移（倒序）：`dp[c] += dp[c - w[i]]`；
  - 注意：倒序是 0-1 语义（每件物品至多用一次）；若写成正序会变成“序列计数/可重复使用”，语义错误。

- 至少容量（at least C）：
  - 两种做法：
    1) 把答案定义为达到“任意 c >= C”的最大价值，遍历所有 c>=C 的 dp；
    2) 通过容量截断/偏移构造辅助数组（如果题目有额外结构，有时更高效）。
  - 常见实现：仍按至多容量的转移，最终取 `max(dp[C..C_max])`。

# 4. 例题

[3685. 含上限元素的子序列和](https://leetcode.cn/problems/subsequence-sum-after-capping-elements/description/)


> 对 nums 升序排序，分层处理“值为 x 的元素”。这样当我们处理到 x 时，dp 中包含了“所有小于等于 x 的原始元素”的子集和可达性。
>  维护一个布尔 bitset dp，dp[s]=1 表示和 s 可达；插入一个值 v 时做 dp |= (dp << v)。
> 大于 x 的元素都被 cap 成 x，剩余数量 remain = n - cur。检查是否存在 0 ≤ i ≤ min(remain, ⌊k/x⌋) 使得 dp[k - i*x] 为真。

```go
func subsequenceSumAfterCapping(nums []int, k int) []bool {
	slices.Sort(nums)
	n := len(nums)
	ans := make([]bool, n)

	// f[s]==1 表示和 s 可达
	f := big.NewInt(1)

	// 掩码 u = (1<<(k+1)) - 1，确保位长 <= k+1
	u := new(big.Int).Lsh(big.NewInt(1), uint(k+1))
	u.Sub(u, big.NewInt(1))

	// 复用一个临时 big.Int，避免每次 new（微优化）
	var t big.Int

	i := 0
	for x := 1; x <= n; x++ {
		// 插入所有值等于 x 的原始元素：f |= f << x
		for i < n && nums[i] == x {
			t.Lsh(f, uint(nums[i])) // t = f << v
			f.Or(f, &t)             // f |= t
			f.And(f, u)             // 截断到 k+1 位
			i++
		}

		// 大于 x 的部分都被 cap 成 x，最多还能取 j = min(n-i, k/x) 个
		limit := n - i
		if k/x < limit {
			limit = k / x
		}

		ok := false
		for j := 0; j <= limit; j++ {
			// target >= 0 恒成立，因为 j <= k/x
			if f.Bit(k-j*x) == 1 {
				ok = true
				break
			}
		}
		ans[x-1] = ok
	}
	return ans
}
```






