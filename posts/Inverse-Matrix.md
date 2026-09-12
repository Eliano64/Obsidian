---
title: Inverse Matrix
tags:
  - Matrices
categories:
  - Linear Algebra
date: 2026-09-07 18:37:28
katex: true
---
# 可逆的条件

$n$ 阶方阵 $\mathbf{A}$ 可逆，当且仅当下列任一条件成立：

- $\det\mathbf{A}\ne0$；
- $r(\mathbf{A})=n$，即行、列向量组均满秩；

若可逆，逆矩阵唯一，记为 $\mathbf{A}^{-1}$，满足 $\mathbf{A}\mathbf{A}^{-1}=\mathbf{A}^{-1}\mathbf{A}=\mathbf{E}$。


# 求逆矩阵

## 二维矩阵法

对二维矩阵，伴随矩阵法最为简洁。设

$$
\mathbf{A}=\begin{pmatrix}a&b\\c&d\end{pmatrix},
$$

则 $\det\mathbf{A}=ad-bc$，且

$$
\mathbf{A}^*=\begin{pmatrix}d&-b\\-c&a\end{pmatrix},
$$

故

$$
\mathbf{A}^{-1}
=\frac1{ad-bc}
\begin{pmatrix}d&-b\\-c&a\end{pmatrix}.
$$


$2$ 阶矩阵几乎总用它。对 $3$ 阶及更高阶，计算 $n^2$ 个代数余子式过于繁琐，通常改用初等变换法。只有当矩阵很稀疏、多数代数余子式容易求得时，才会对高阶矩阵使用伴随矩阵法。

## 初等变换法

对 $3$ 阶或更高阶的具体矩阵，用行初等变换把 $\mathbf{A}$ 化为单位矩阵，同时对右侧随行的单位矩阵作相同变换：

$$
[\mathbf{A}\mid\mathbf{E}]
\xrightarrow{\text{行变换}}
[\mathbf{E}\mid\mathbf{A}^{-1}].
$$

**原理**：每个行初等变换都等价于左乘一个初等矩阵。设依次左乘 $\mathbf{E}_1,\ldots,\mathbf{E}_k$ 后 $\mathbf{A}$ 化为 $\mathbf{E}$，即

$$
\mathbf{E}_k\cdots\mathbf{E}_1\mathbf{A}=\mathbf{E}.
$$

于是 $\mathbf{E}_k\cdots\mathbf{E}_1=\mathbf{A}^{-1}$。把同一序列作用到 $\mathbf{E}$ 上，右侧得到

$$
\mathbf{E}_k\cdots\mathbf{E}_1\mathbf{E}=\mathbf{A}^{-1}.
$$

所以右侧累积的就是 $\mathbf{A}^{-1}$。

只能作行变换，不能作列变换。若行变换过程中左侧出现一整行为零，说明 $\mathbf{A}$ 不可逆，即 $\det\mathbf{A}=0$。

与行列式消元一样，尽量不把主元归一化，以免过早引入分式。

> [!example] 三阶矩阵的逆
> 求
> $$
> \mathbf{A}=\begin{pmatrix}1&2&3\\2&4&5\\3&5&6\end{pmatrix}
> $$
> 的逆。写出增广矩阵并作行变换：
> $$
> \left[\begin{array}{ccc|ccc}
> 1&2&3&1&0&0\\
> 2&4&5&0&1&0\\
> 3&5&6&0&0&1
> \end{array}\right].
> $$
> 先消去第二列的 $2$ 与第三列的 $3$，并把第二、第三行分别归零后交换：
> $$
> \left[\begin{array}{ccc|ccc}
> 1&2&3&1&0&0\\
> 0&1&3&3&0&-1\\
> 0&0&1&2&-1&0
> \end{array}\right].
> $$
> 由第三行回代消去第二、第一行的第三列，再由第二行消去第一行的第二列，化为标准形：
> $$
> \left[\begin{array}{ccc|ccc}
> 1&0&0&1&-3&2\\
> 0&1&0&-3&3&-1\\
> 0&0&1&2&-1&0
> \end{array}\right].
> $$
> 故
> $$
> \mathbf{A}^{-1}=\begin{pmatrix}1&-3&2\\-3&3&-1\\2&-1&0\end{pmatrix}.
> $$

## 符号矩阵

符号矩阵的行、列没有具体数值，不能作行变换。此时把目标矩阵表示成已知可逆矩阵的乘积或组合，再对各个因子求逆。


若 $\mathbf{A},\mathbf{B}$ 都可逆，则 $\mathbf{A}\mathbf{B}$ 可逆，且$(\mathbf{A}\mathbf{B})^{-1}=\mathbf{B}^{-1}\mathbf{A}^{-1}.$更一般地，$(\mathbf{A}_1\cdots\mathbf{A}_k)^{-1}=\mathbf{A}_k^{-1}\cdots\mathbf{A}_1^{-1}$ 。

对于求 $\mathbf{A+B}$ 的逆，若 $\mathbf{A}$ 可逆，可提取公因子转化为乘积：$\mathbf{A}+\mathbf{B}=\mathbf{A}\bigl(\mathbf{E}+\mathbf{A}^{-1}\mathbf{B}\bigr),$ 故 $(\mathbf{A}+\mathbf{B})^{-1}=\bigl(\mathbf{E}+\mathbf{A}^{-1}\mathbf{B}\bigr)^{-1}\mathbf{A}^{-1}.$


若存在 $k$ 使 $\mathbf{B}^k=\mathbf{0}$，则 $\mathbf{E}+\mathbf{B}$ 可逆，且由几何级数 $(\mathbf{E}+\mathbf{B})^{-1}=\mathbf{E}-\mathbf{B}+\mathbf{B}^2-\cdots+(-1)^{k-1}\mathbf{B}^{k-1}.$ 

## 分块矩阵

设

$$
\mathbf{A}=\begin{pmatrix}\mathbf{P}&\mathbf{Q}\\\mathbf{R}&\mathbf{S}\end{pmatrix},
$$

其中 $\mathbf{P},\mathbf{S}$ 为方阵。求其逆时，设未知逆矩阵

$$
\mathbf{B}=\begin{pmatrix}\mathbf{X}&\mathbf{Y}\\\mathbf{Z}&\mathbf{W}\end{pmatrix},
$$

并解矩阵方程 $\mathbf{A}\mathbf{B}=\mathbf{E}$。按块相乘，得到四个块方程：

$$
\begin{gathered}
\mathbf{P}\mathbf{X}+\mathbf{Q}\mathbf{Z}=\mathbf{E},\qquad
\mathbf{P}\mathbf{Y}+\mathbf{Q}\mathbf{W}=\mathbf{0},\\
\mathbf{R}\mathbf{X}+\mathbf{S}\mathbf{Z}=\mathbf{0},\qquad
\mathbf{R}\mathbf{Y}+\mathbf{S}\mathbf{W}=\mathbf{E}.
\end{gathered}
$$

从中解出 $\mathbf{X},\mathbf{Y},\mathbf{Z},\mathbf{W}$。

通常因为某一块为零或某一块易求逆而大大简化。例如下面的情况。

> [!example] 分块上三角
> 设 $\mathbf{R}=\mathbf{0}$，即
> $$
> \mathbf{A}=\begin{pmatrix}\mathbf{P}&\mathbf{Q}\\\mathbf{0}&\mathbf{S}\end{pmatrix}.
> $$
> 由 $\mathbf{R}\mathbf{X}+\mathbf{S}\mathbf{Z}=\mathbf{0}$ 得 $\mathbf{S}\mathbf{Z}=\mathbf{0}$，故 $\mathbf{Z}=\mathbf{0}$；由 $\mathbf{R}\mathbf{Y}+\mathbf{S}\mathbf{W}=\mathbf{E}$ 得 $\mathbf{W}=\mathbf{S}^{-1}$。代入第一、第二式：
> $$
> \mathbf{P}\mathbf{X}=\mathbf{E}\ \Rightarrow\ \mathbf{X}=\mathbf{P}^{-1},
> $$
> $$
> \mathbf{P}\mathbf{Y}+\mathbf{Q}\mathbf{S}^{-1}=\mathbf{0}\ \Rightarrow\ \mathbf{Y}=-\mathbf{P}^{-1}\mathbf{Q}\mathbf{S}^{-1}.
> $$
> 因此
> $$
> \begin{pmatrix}\mathbf{P}&\mathbf{Q}\\\mathbf{0}&\mathbf{S}\end{pmatrix}^{-1}
> =\begin{pmatrix}\mathbf{P}^{-1}&-\mathbf{P}^{-1}\mathbf{Q}\mathbf{S}^{-1}\\\mathbf{0}&\mathbf{S}^{-1}\end{pmatrix}.
> $$
> 分块下三角同理。
