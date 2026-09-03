---
title: Cesaro Mean Theorem
tags:
  - Sequence
categories:
  - Calculus
date: 2026-08-25 14:45:11
katex: true
---
# Cesàro 平均定理

若数列 $a_n\to A$，则其前 $n$ 项的算术平均也收敛于 $A$：

$$
\lim_{n\to\infty}\frac{\sum_{k=1}^{n}a_k}{n}=A.
$$

# 证明

由 $a_n\to A$，存在 $M>0$，使

$$
|a_k-A|\le M\qquad(k\ge1).
$$

对每个 $n$，取 $m=\sqrt n$。这样既有 $m\to\infty$，又有 $m/n\to0$。再记后 $n-m$ 项与 $A$ 差距为

$$
d_n=\max_{m<k\le n}|a_k-A|.
$$

因为 $m\to\infty$ 且 $a_k\to A$，所以 $d_n\to0$。

前 $m$ 项满足 $A-M\le a_k\le A+M$；后 $n-m$ 项满足 $A-d_n\le a_k\le A+d_n$。将这些不等式分别相加，再除以 $n$：

$$
\begin{aligned}
\frac{\sum_{k=1}^n a_k}{n}
&\ge \frac{m(A-M)+(n-m)(A-d_n)}{n},\\[4pt]
\frac{\sum_{k=1}^n a_k}{n}
&\le \frac{m(A+M)+(n-m)(A+d_n)}{n}.
\end{aligned}
$$

其中

$$
\frac mn\le\frac1{\sqrt n}\to0,
\qquad
d_n\to0.
$$

右侧的下界与上界都趋于 $A$。由夹逼定理，

$$
\lim_{n\to\infty}\frac{\sum_{k=1}^{n}a_k}{n}=A.
$$
