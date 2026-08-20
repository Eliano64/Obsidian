---
title: Antiderivative And Riemann Integral
tags:
  - Integral
categories:
  - Calculus
date: 2026-08-18 13:59:39
katex: true
---
# 函数有原函数

函数有原函数，即该函数是另一个函数的导函数 $\iff$ 该函数连续**或**有振荡间断点（导函数的 Darboux 性质）

# 函数可积

函数黎曼可积，即在定义域的一个有限区域黎曼和的极限存在 $\iff$ 区域内函数有界**且**间断点集合的测度[^1]为 0 

# 有原函数是可积的既不充分也不必要条件

1. 有原函数 $\not\implies$ 可积，可积 $\not\implies$ 有原函数
2. 牛顿—莱布尼茨公式的前提条件：函数既可积，又有原函数。



[^1]: 测度：区间的长度。根据$\epsilon-\delta$语言可定义一个点的测度为 0 （对任意的 $\epsilon$, 都可以找到$\delta$, 使得区间$(x-\delta,x+\delta)$的长度小于$\epsilon$）。使用多个区间定义点的集合的测度为 0 ：如果对任意的 $\epsilon$, 都可以找到{$\delta_{i}$}, 使得区间$(x_{i}-\delta_{i},x_{i}+\delta_{i})$的总长度和小于$\epsilon$，那么我们认为这个点集的测度为 0 。有限个点的集合的测度显然为 0 。
