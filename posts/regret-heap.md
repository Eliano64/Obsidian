---
title: regret heap
tags:
  - Greedy
  - Heap
  - DataStructureAndAlgorithm
categories:
  - Data Structure & Algorithm
date: 2025-09-26 00:00:00
katex: true
---

反悔是贪心的一种策略，用于在当前可能不是全局最优解时，“反思”已做过的选择。

而对于这样的策略，常使用堆来辅助实现。具体来说就是将已做的选择按照某种规则放入最大堆中。“反悔”就是“撤销”堆顶的选择，即代价最大的选择。

**用最大堆在约束触发时“撤销/替换”当前集合里最差的那个已选项。**

# 1. 示例

[LCP30. 魔塔游戏](https://leetcode.cn/problems/p0NxJO/description/)

```plaintext
magicTower(nums[0..n-1], hp){
    declare a empty max heap, named heap.
    ans := 0
    for num in nums; do
        if num < 0; then
            heap.push(num) // push the negative number into the heap, so that we can recover the choice later if needed
        fi
        hp += num
        if hp <= 0; then
            if heap.isEmpty(); then //cannot recover
                return -1
            fi
            ans++
            hp -= heap.pop() // recover the smallest negative number, namely the most costly choice
        fi
    end
    return ans
}
```