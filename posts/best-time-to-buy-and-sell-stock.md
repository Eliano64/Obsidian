---
title: best time to buy and sell stock
tags:
  - LeetCode
categories:
  - Record
date: 2025-10-12 00:00:00
katex: true
---

[题目](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock)

```C++
int maxProfit(vector<int>& prices) {
    int lo = prices[0]
    int ans = 0
    for(int i=1;i<prices.size();i++){
        ans = max(ans,prices[i]-lo)
        lo = min(lo,prices[i])
    }
    return ans
}
```

这个模板可以用于：寻找不单增数组的元素与该元素前面的所有元素之差的最大值。