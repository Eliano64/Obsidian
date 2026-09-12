---
title: Probability
tags:
  - Probability
categories:
  - Probability & Statistics
date: 2026-09-05 16:24:47
katex: true
---
# 概率的性质

$$
0\le P(A)\le1,\qquad P(\Omega)=1,\qquad P(\varnothing)=0.
$$


$$
P(\bar{A})=1-P(A),\qquad
P(A-B)=P(A)-P(AB).
$$

$$
P(A\cup B)=P(A)+P(B)-P(AB).
$$

若 $A\subseteq B$，则 $P(A)\le P(B)$；若 $A_1,A_2,\ldots$ 两两互斥，则

$$
P\left(\bigcup_i A_i\right)=\sum_iP(A_i).
$$

处理复杂事件时，先用集合运算改写事件，再套概率公式（正难则反）。例如，至少一个发生改用对立事件一个都没有发生。

# 古典概型

样本空间含有限个等可能的基本结果时，

$$
P(A)=\frac{|A|}{|\Omega|}.
$$
# 几何概型

试验结果在区域 $\Omega$ 上均匀分布时，

$$
P(A)=\frac{m(A)}{m(\Omega)},
$$

其中 $m$ 可表示长度、面积或体积。先选取能使结果均匀的参数，再把事件翻译成该参数空间中的子区域；边界通常测度为零，不影响概率。

# 条件概率与乘法公式

当 $P(B)>0$ 时，

$$
P(A\mid B)=\frac{P(AB)}{P(B)}.
$$

条件竖线后是谁，分母就是谁。

由定义立即得到

$$
P(AB)=P(B)P(A\mid B)=P(A)P(B\mid A).
$$

多个事件依次发生时，

$$
P(A_1A_2\cdots A_n)
=P(A_1)P(A_2\mid A_1)\cdots
P(A_n\mid A_1\cdots A_{n-1}).
$$

# 全概率公式与贝叶斯公式

若 $B_1,\ldots,B_n$ 两两互斥、并为 $\Omega$，且 $P(B_i)>0$，则

$$
P(A)=\sum_{i=1}^{n} P(AB_{i})=\sum_{i=1}^{n}P(B_i)P(A\mid B_i).
$$

当 $A$ 已经发生，反求它来自第 $k$ 种情形的概率时，

$$
P(B_k\mid A)
=\frac{P(B_k)P(A\mid B_k)}
{\displaystyle\sum_{i=1}^{n}P(B_i)P(A\mid B_i)}.
$$
# 事件独立性

$A,B$ 独立，即

$$
P(AB)=P(A)P(B).
$$

等价于 $P(A\mid B)=P(A)$。

独立事件取补后仍独立。
