---
title: Deque
tags:
  - Deque
categories:
  - Data Structure & Algorithm
date: 2026-06-27 15:24:00
katex: true
---
# 定义

双端队列允许在两端插入和删除元素。它比普通 [[Queue-Definition-And-Operations|队列]] 更灵活，也能模拟 [[Stack-Definition-And-Operations|栈]] 的行为。

# 受限双端队列

- 输入受限双端队列：只允许从一端插入，允许从两端删除。
- 输出受限双端队列：允许从两端插入，只允许从一端删除。

# 与栈、队列的关系

- 只在一端插入和删除时，双端队列表现得像栈。
- 固定一端插入、另一端删除时，双端队列表现得像队列。
- 普通栈的合法输出序列，在相应的双端队列模型中通常仍合法，但双端队列还可能产生更多序列。
