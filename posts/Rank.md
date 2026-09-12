---
title: Rank
tags:
  - Matrices
  - Vectors
categories:
  - Linear Algebra
date: 2026-09-03 10:42:01
katex: true
---
# 向量组的秩

## 向量

$n$ 维实向量由 $n$ 个有序实数组成，通常写成列向量 $\mathbf{\alpha}=(a_1,\ldots,a_n)^\top$。同维向量按分量相加、数乘。

若 $\mathbf{\beta}=c_1\mathbf{\alpha}_1+\cdots+c_s\mathbf{\alpha}_s$，则称 $\mathbf{\beta}$ 可由向量组 $\mathbf{\alpha}_1,\ldots,\mathbf{\alpha}_s$ 线性表示。

## 向量组的秩

向量组中，线性无关且能线性表示该组全部向量的部分组，称为**极大线性无关组**。各极大线性无关组所含向量个数相同，这个数称为向量组的**秩**，也就是该组中线性无关向量的最大个数。全零向量组的秩为 $0$。

## 向量组线性相关

当 $\mathbf{A}$ 是由 $n$ 个 $n$ 维列向量组成的方阵时，列向量线性相关的等价条件为 $\det\mathbf{A}=0$，也即$r(\mathbf{A})<n$。

## 向量组线性无关

$n$ 个列向量线性无关，等价于 $r(\mathbf{A})=n$ 。

证明 $\mathbf{\alpha}_1,\ldots,\mathbf{\alpha}_n$ 线性无关，可以从定义出发：设 $k_1\mathbf{\alpha}_1+\cdots+k_n\mathbf{\alpha}_n=\mathbf{0}$，利用题设证明所有系数均为零。

### 已知 $\mathbf{A}\mathbf{\alpha}_i=\mathbf{\alpha}_{i+1}\ (i<n)$，$\mathbf{A}\mathbf{\alpha}_n=\mathbf{0}$，且 $\mathbf{\alpha}_n\ne\mathbf{0}$，证明 $\mathbf{\alpha}_1,\ldots,\mathbf{\alpha}_n$ 线性无关

在所设线性关系两边左乘 $\mathbf{A}^{n-1}$，只有第一项可能非零，故 $k_1\mathbf{\alpha}_n=\mathbf{0}$，得到 $k_1=0$。消去第一项后，再左乘 $\mathbf{A}^{n-2}$，得到 $k_2=0$；依次进行，所有系数均为零。



### 已知 $\mathbf{\alpha}_1,\ldots,\mathbf{\alpha}_n$ 线性无关，记 $\mathbf{A}=(\mathbf{\alpha}_1,\ldots,\mathbf{\alpha}_n)$，且 $\mathbf{\beta}_i=\mathbf{A}\mathbf{b}_i$。则 $\mathbf{\beta}_1,\ldots,\mathbf{\beta}_n$ 线性无关的充要条件是 $\mathbf{b}_1,\ldots,\mathbf{b}_n$ 线性无关。

设 $b_{ij}$ 为 $\mathbf{b}_i$ 的第 $j$ 个分量。由 $\mathbf{\beta}_i=\sum_{j=1}^n b_{ij}\mathbf{\alpha}_j$，得：

$$
\begin{aligned}
\sum_{i=1}^n k_i\mathbf{\beta}_i=\mathbf{0}
&\iff \mathbf{A}\left(\sum_{i=1}^n k_i\mathbf{b}_i\right)=\mathbf{0}\\
&\iff \sum_{j=1}^n\left(\sum_{i=1}^n k_i b_{ij}\right)\mathbf{\alpha}_j=\mathbf{0}\\
&\iff \sum_{i=1}^n k_i b_{ij}=0\quad(j=1,\ldots,n)\\
&\iff \sum_{i=1}^n k_i\mathbf{b}_i=\mathbf{0}.
\end{aligned}
$$

## 向量组的关系

设 $\mathbf{A},\mathbf{B}$ 分别由两组向量按列组成，两组向量的分量个数相同。

1. **线性表示。** $\mathbf{A}$ 的列向量组能线性表示 $\mathbf{B}$ 的列向量组，当且仅当 $r([\mathbf{A}\ \mathbf{B}])=r(\mathbf{A})$。
2. **向量组等价。** 两组向量可以相互表示，等价于 $r(\mathbf{A})=r(\mathbf{B})=r([\mathbf{A}\ \mathbf{B}])$。


# 矩阵的秩

$r(\mathbf{A})=r$ 表示 $\mathbf{A}$ 的列向量组的极大无关组含 $r$ 个向量，其余列均可由这组向量线性表示。矩阵的行秩与列秩相等。

若$\mathbf{A}\neq \mathbf{O}$,则一定有$r(\mathbf{A})>0$

## 矩阵等价

同型矩阵 $\mathbf{A},\mathbf{B}$ 等价，是指一者可经有限次初等行变换和列变换化为另一者，充要条件为 $r(\mathbf{A})=r(\mathbf{B})$。矩阵等价不蕴含列向量组等价。

## 乘积的秩

$\mathbf{A}\mathbf{B}$ 的每一列都是 $\mathbf{A}$ 的列向量的线性组合，每一行都是 $\mathbf{B}$ 的行向量的线性组合，因此：

$$
r(\mathbf{A}\mathbf{B})\le\min\{r(\mathbf{A}),r(\mathbf{B})\}.
$$

1. $r(\mathbf{A}\mathbf{B})=r(\mathbf{A})$，当且仅当 $\mathbf{A}\mathbf{B}$ 与 $\mathbf{A}$ 的列向量组等价。特别地，$\mathbf{B}$ 满行秩时成立。
2. $r(\mathbf{A}\mathbf{B})=r(\mathbf{B})$，当且仅当 $\mathbf{B}$ 的列向量能表示的向量中，满足 $\mathbf{A}\mathbf{y}=\mathbf{0}$ 的只有 $\mathbf{y}=\mathbf{0}$。此时 $\mathbf{A}\mathbf{B}\mathbf{x}=\mathbf{0}$ 必能推出 $\mathbf{B}\mathbf{x}=\mathbf{0}$，即 $\mathbf{A}\mathbf{B}\mathbf{x}=\mathbf{0}$ 与 $\mathbf{B}\mathbf{x}=\mathbf{0}$ 同解。特别地，$\mathbf{A}$ 满列秩时成立。

$r(\mathbf{A})\le r(\mathbf{B})$ 时，乘积秩达到上界的条件为第一条；$r(\mathbf{B})\le r(\mathbf{A})$ 时为第二条。特别地，左乘或右乘可逆矩阵不改变秩。


特别地，若$\mathbf{AB=O}$，则$$r(\mathbf{A})+r(\mathbf{B})\leqslant n$$
这是因为 $\mathbf{B}$ 的列向量在齐次方程 $\mathbf{Ax=0}$ 的[[#解空间]]里。

## 拼接矩阵的秩

设 $\mathbf{A},\mathbf{B}$ 行数相同，则拼接矩阵 $[\mathbf{A}\ \mathbf{B}]$ 的秩满足：

$$
\max\{r(\mathbf{A}),r(\mathbf{B})\}
\le r([\mathbf{A}\ \mathbf{B}])
\le r(\mathbf{A})+r(\mathbf{B}).
$$

1. **下界：** $\mathbf{A},\mathbf{B}$ 均为 $[\mathbf{A}\ \mathbf{B}]$ 的子矩阵，故拼接矩阵的秩不小于二者的秩。若 $r(\mathbf{A})\ge r(\mathbf{B})$，下界取等当且仅当 $\mathbf{A}$ 的列向量能表示 $\mathbf{B}$ 的所有列；反之交换两者。
2. **上界：** 两组极大无关组的并集能表示全部列。上界取等当且仅当两组列向量能共同表示的向量只有零向量；若存在两组向量均能表示的非零向量，两种表示式相减即得到两个极大无关组的并集的非平凡线性关系。

纵向拼接时，对行向量作同样判断。对分块矩阵的两个块行应用上述结论，得：

$$
\mathbf{M}=\begin{pmatrix}\mathbf{A}&\mathbf{B}\\\mathbf{C}&\mathbf{D}\end{pmatrix},
$$

$$
\max\{r([\mathbf{A}\ \mathbf{B}]),r([\mathbf{C}\ \mathbf{D}])\}
\le r(\mathbf{M})
\le r([\mathbf{A}\ \mathbf{B}])+r([\mathbf{C}\ \mathbf{D}]).
$$

## 和的秩

设 $\mathbf{A},\mathbf{B}$ 同型。$\mathbf{A}+\mathbf{B}$ 的每一列都能由 $[\mathbf{A}\ \mathbf{B}]$ 的列向量表示，故：

$$
r(\mathbf{A}+\mathbf{B})
\le r([\mathbf{A}\ \mathbf{B}])
\le r(\mathbf{A})+r(\mathbf{B}).
$$

再由 $\mathbf{A}=(\mathbf{A}+\mathbf{B})-\mathbf{B}$ 及交换两者后的同类关系，得到：

$$
|r(\mathbf{A})-r(\mathbf{B})|
\le r(\mathbf{A}+\mathbf{B})
\le r(\mathbf{A})+r(\mathbf{B}).
$$

矩阵和的秩可能小于两个加数的秩，例如 $\mathbf{B}=-\mathbf{A}$ 时，$r(\mathbf{A}+\mathbf{B})=0$。

## 分块矩阵

已知 $r(\mathbf{A})=r>0$，就能取出一个可逆的 $r$ 阶子矩阵。交换行、列将它放在左上角，写成：

$$
\mathbf{A}=
\begin{pmatrix}\mathbf{C}&\mathbf{D}\\\mathbf{E}&\mathbf{F}\end{pmatrix},
\qquad \mathbf{C}\text{ 为 }r\text{ 阶可逆矩阵}.
$$

通过可逆的分块行、列变换，有：

$$
\begin{pmatrix}\mathbf{I}&\mathbf{0}\\-\mathbf{E}\mathbf{C}^{-1}&\mathbf{I}\end{pmatrix}
\mathbf{A}
\begin{pmatrix}\mathbf{I}&-\mathbf{C}^{-1}\mathbf{D}\\\mathbf{0}&\mathbf{I}\end{pmatrix}
=
\begin{pmatrix}
\mathbf{C}&\mathbf{0}\\
\mathbf{0}&\mathbf{F}-\mathbf{E}\mathbf{C}^{-1}\mathbf{D}
\end{pmatrix}.
$$

由 $r(\mathbf{A})=r(\mathbf{C})=r$，得 $r(\mathbf{F}-\mathbf{E}\mathbf{C}^{-1}\mathbf{D})=0$，故 $\mathbf{F}=\mathbf{E}\mathbf{C}^{-1}\mathbf{D}$。若 $r=0$，则 $\mathbf{A}=\mathbf{0}$。


# 线性方程组

设 $\mathbf{A}$ 为 $m\times n$ 实矩阵，$r(\mathbf{A})=r$。

## 齐次方程组

$\mathbf{A}\mathbf{x}=\mathbf{0}$ 描述列向量之间的线性关系，总有零解。有非零解当且仅当 $r<n$。

行化简后有 $r$ 个主元变量，其余 $n-r$ 个变量可以自由取值。依次令一个自由变量为 $1$、其余自由变量为 $0$，就得到 $n-r$ 个线性无关的解向量。

这$n-r$个线性无关的解向量，组成一个向量组。这个向量组称为**基础解系**。

记它们按列组成的矩阵为 $\mathbf{H}$，则：

$$
r(\mathbf{A})+r(\mathbf{H})=n.
$$

所以通解为 $\mathbf{x}=\mathbf{H}\mathbf{c}$，其中 $\mathbf{c}$ 是任意系数向量。

$r=n$ 时，齐次方程组只有零解，基础解系为空。

## 非齐次方程组

$\mathbf{A}\mathbf{x}=\mathbf{b}$ 描述右端向量的线性表示。

若 $\mathbf{x}_p$ 是一个特解，则 $\mathbf{x}$ 也是特解，当且仅当：

$$
\mathbf{A}(\mathbf{x}-\mathbf{x}_p)=\mathbf{0}.
$$

所以非齐次方程的全部解为：

$$
\mathbf{x}=\mathbf{x}_p+\mathbf{H}\mathbf{c}.
$$

| 解的情况 | 充要条件                                          | 向量关系                 |
| ---- | --------------------------------------------- | -------------------- |
| 无解   | $r([\mathbf{A}\ \mathbf{b}])>r(\mathbf{A})$   | $\mathbf{b}$ 不能由各列表示 |
| 唯一解  | $r([\mathbf{A}\ \mathbf{b}])=r(\mathbf{A})=n$ | 可以表示，且各列线性无关         |
| 无穷多解 | $r([\mathbf{A}\ \mathbf{b}])=r(\mathbf{A})<n$ | 可以表示，但各列之间存在线性关系     |

## 解空间


![Strang：The Big Picture，原书图 3.5](../../assets/strang-big-picture-original.svg)

图中 $C(\mathbf{A}^\top)$、$C(\mathbf{A})$ 分别是行空间与列空间，$N(\mathbf{A})$、$N(\mathbf{A}^\top)$ 分别是两个齐次方程的解空间；$\mathrm{dim}$ 表示维数，$\mathbf{R}$ 是 $\mathbf{A}$ 的行最简形矩阵。

1. **列空间与有解性。** $\mathbf{A}\mathbf{x}=\mathbf{b}$ 有解，意味着 $\mathbf{b}$ 位于列空间中。列空间的维数等于 $r(\mathbf{A})$。
2. **行空间与解的自由度。** 任意 $\mathbf{x}\in\mathbb{R}^n$ 均可唯一分解为 $\mathbf{x}=\mathbf{x}_r+\mathbf{x}_0$，其中 $\mathbf{x}_r$ 在行空间中，$\mathbf{A}\mathbf{x}_0=\mathbf{0}$。于是 $\mathbf{A}\mathbf{x}=\mathbf{A}\mathbf{x}_r$。对列空间中的每个 $\mathbf{b}$，行空间内存在唯一的 $\mathbf{x}_r$ 满足 $\mathbf{A}\mathbf{x}_r=\mathbf{b}$，全部解为 $\mathbf{x}_r+\mathbf{x}_0$，其中 $\mathbf{x}_0$ 为任意齐次解。
3. **正交关系。** $\mathbf{A}\mathbf{x}=\mathbf{0}$ 表示 $\mathbf{x}$ 与每一行正交，$\mathbf{A}^\top\mathbf{y}=\mathbf{0}$ 表示 $\mathbf{y}$ 与每一列正交。左右两侧的维数分别满足 $r+(n-r)=n$、$r+(m-r)=m$。

因此，$\mathbf{A}\mathbf{x}=\mathbf{b}$ 有解还等价于：对每个满足 $\mathbf{A}^\top\mathbf{y}=\mathbf{0}$ 的 $\mathbf{y}$，都有 $\mathbf{y}^\top\mathbf{b}=0$。必要性来自 $\mathbf{y}^\top\mathbf{b}=\mathbf{y}^\top\mathbf{A}\mathbf{x}=0$；充分性来自两个空间正交且维数互补，与全部这样的 $\mathbf{y}$ 正交的向量恰好组成列空间。

## $\mathbf{A}^{\top}\mathbf{Ax=A^{\top}b}$

对于实矩阵 $\mathbf{A}$，$\mathbf{A}^\top\mathbf{A}$ 与 $\mathbf{A}$ 的齐次方程同解：

$$
\mathbf{A}^\top\mathbf{A}\mathbf{x}=\mathbf{0}
\ \Longrightarrow\
\mathbf{x}^\top\mathbf{A}^\top\mathbf{A}\mathbf{x}
=\|\mathbf{A}\mathbf{x}\|^2=0
\ \Longrightarrow\
\mathbf{A}\mathbf{x}=\mathbf{0}.
$$

反向显然成立。两者列数相同、基础解系相同，故 $r(\mathbf{A}^\top\mathbf{A})=r(\mathbf{A})$。

也可直接用子式证明。设 $r(\mathbf{A})=r>0$，取 $r$ 个无关列组成 $\mathbf{B}$。对任意非零 $\mathbf{x}$，都有 $\mathbf{x}^\top\mathbf{B}^\top\mathbf{B}\mathbf{x}=\|\mathbf{B}\mathbf{x}\|^2>0$，所以 $\mathbf{B}^\top\mathbf{B}$ 可逆。它是 $\mathbf{A}^\top\mathbf{A}$ 的一个 $r$ 阶主子矩阵，因而：

$$
r\le r(\mathbf{A}^\top\mathbf{A})
\le\min\{r(\mathbf{A}^\top),r(\mathbf{A})\}=r.
$$

$r=0$ 时结论显然成立。

方程 $\mathbf{A}^\top\mathbf{A}\mathbf{x}=\mathbf{A}^\top\mathbf{b}$ 总有解。对任意相容维数的实向量 $\mathbf{b}$，可直接比较系数矩阵与增广矩阵的秩：

$$
\begin{aligned}
r(\mathbf{A}^\top\mathbf{A})
&\le r([\mathbf{A}^\top\mathbf{A}\ \ \mathbf{A}^\top\mathbf{b}])\\
&=r\bigl(\mathbf{A}^\top[\mathbf{A}\ \mathbf{b}]\bigr)\\
&\le r(\mathbf{A}^\top)=r(\mathbf{A})
=r(\mathbf{A}^\top\mathbf{A}).
\end{aligned}
$$

两端相等，系数矩阵与增广矩阵秩相等，故方程总有解；唯一解的条件是 $r(\mathbf{A})=n$。

从原图看，$\mathbf{b}$ 可以分成列空间内的部分与垂直于列空间的部分。方程 $\mathbf{A}^\top(\mathbf{A}\mathbf{x}-\mathbf{b})=\mathbf{0}$ 要求误差垂直于列空间，只需使 $\mathbf{A}\mathbf{x}$ 等于前一部分即可。因此，即使原方程无解，这个方程仍有解。

# 解集关系

设两组方程具有相同的未知向量 $\mathbf{x}\in\mathbb{R}^n$，并记：

$$
\mathbf{C}=\begin{pmatrix}\mathbf{A}\\\mathbf{B}\end{pmatrix}.
$$

## 齐次解集

1. **有公共非零解。** 两组方程的公共解集为 $\mathbf{C}\mathbf{x}=\mathbf{0}$ 的解集，故存在公共非零解的充要条件为 $r(\mathbf{C})<n$。
2. **解集包含。** $\mathbf{A}\mathbf{x}=\mathbf{0}$ 的解集包含于 $\mathbf{B}\mathbf{x}=\mathbf{0}$ 的解集，当且仅当 $r(\mathbf{C})=r(\mathbf{A})$。等价条件为 $\mathbf{B}$ 的行向量组可由 $\mathbf{A}$ 的行向量组线性表示，或 $n-r(\mathbf{C})=n-r(\mathbf{A})$。
3. **同解。** 两组解集相同，当且仅当 $r(\mathbf{A})=r(\mathbf{B})=r(\mathbf{C})$，即两个解集相互包含。

## 非齐次解集

对于 $\mathbf{A}\mathbf{x}=\mathbf{a}$ 与 $\mathbf{B}\mathbf{x}=\mathbf{b}$，记：

$$
\mathbf{d}=\begin{pmatrix}\mathbf{a}\\\mathbf{b}\end{pmatrix},\qquad
\widehat{\mathbf{A}}=[\mathbf{A}\ \mathbf{a}],\qquad
\widehat{\mathbf{B}}=[\mathbf{B}\ \mathbf{b}],\qquad
\widehat{\mathbf{C}}=[\mathbf{C}\ \mathbf{d}].
$$

1. **有公共解。** 两组方程的公共解集为 $\mathbf{C}\mathbf{x}=\mathbf{d}$ 的解集，故有公共解的充要条件为 $r(\mathbf{C})=r(\widehat{\mathbf{C}})$。
2. **解集包含。** 第一组无解时，其空解集包含于第二组解集。第一组有解时，其解集包含于第二组解集的充要条件为 $r(\widehat{\mathbf{C}})=r(\widehat{\mathbf{A}})$，即存在矩阵 $\mathbf{D}$ 使 $[\mathbf{B}\ \mathbf{b}]=\mathbf{D}[\mathbf{A}\ \mathbf{a}]$。若第一组的通解为 $\mathbf{x}_p+\mathbf{H}\mathbf{c}$，则等价条件为 $\mathbf{B}\mathbf{x}_p=\mathbf{b}$ 且 $\mathbf{B}\mathbf{H}=\mathbf{0}$，即 $\mathbf{x}_p$ 为公共特解，$\mathbf{H}$ 的各列均为第二组对应齐次方程的解。
3. **同解。** 两组都无解时同解；至少一组有解时，同解的充要条件为 $r(\widehat{\mathbf{A}})=r(\widehat{\mathbf{B}})=r(\widehat{\mathbf{C}})$。从解的表达看，这等价于具有公共特解，且对应的齐次方程同解。
