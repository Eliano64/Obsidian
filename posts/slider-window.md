---
title: slider window
tags:
  - DataStructureAndAlgorithm
  - Queue
categories:
  - Data Structure & Algorithm
date: 2025-10-12 00:00:00
katex: true
---

核心思想：使用两个指针`l`和`r`表示窗口的左右边界。`r`表示即将入窗口的元素，`l`表示即将出窗口的元素。由于这种出和入只发生在窗口特定的位置，显然就可以只需检查出/入的单个元素来维护这个窗口里的元素满足的性质。

# 1. 定长滑动窗口

只用`r`即可，`l=r-w+1`。

模板：

```text
for r in (0..n-1); do
    // enter (loop i)
    arr[r] enters the window
    l = r - w + 1
    if l < 0; then
        continue
    fi
    // update (loop i)
    update window state
    // exit, prepare for the next loop (for loop i+1)
    l++
done
```

> 注意：出窗口的操作要在更新窗口状态之后。这样，当`r`指代的元素入窗口时，由`r`推出的`l`所指代的元素会在**下一轮**循环开始之前出窗口。相应的，在下一轮循环`i+1`时，上一个`r`推出的`l`所指代的元素在`i+1`之前就出了窗口。**确保状态更新时`i+1`的出入都发生过了**。

# 2. 变长滑动窗口

需要`l`和`r`两个指针。`r`表示所有即将入窗口的元素，`l`表示被淘汰的、即将出窗口的元素。

```text
l := 0
for r in (0..n-1); do
    // enter (loop i)
    arr[r] enters the window
    // need someone to exit? (loop i)
    if [check_exit]; then
        delete arr[l] from window state
        l++
    fi
    // update (loop i)
    update window state
done
```

# 3. 例题

* [定长滑动窗口](https://leetcode.cn/problems/minimum-discards-to-balance-inventory)

* [变长滑动窗口](https://leetcode.cn/problems/longest-substring-without-repeating-characters)

