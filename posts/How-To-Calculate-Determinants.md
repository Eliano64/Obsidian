---
title: How To Calculate Determinants
tags:
  - Determinant
categories:
  - Linear Algebra
date: 2026-08-30 17:41:04
katex: true
---
# 定义与初等变换

一般先用初等变换制造零，化为主对角或副对角形式；结构足够稀疏时再按行、列展开。

定义展开时，每一项都从每行、每列各取一个元素；所取列标的逆序数决定正负号：

$$
\det\mathbf{A}=\sum_{\sigma\in S_n}(-1)^{\tau(\sigma)}
\prod_{i=1}^n a_{i,\sigma(i)}.
$$

实际计算通常只使用它的两个结果：

- 按含零最多的行或列展开；
- 主对角三角形行列式为 $\prod_{i=1}^n a_{ii}$，副对角三角形行列式为
  $$
  (-1)^{\frac{n(n-1)}2}\prod_{i=1}^n a_{i,n+1-i}.
  $$

## 按行或列展开

记 $M_{ij}$ 为余子式，$A_{ij}=(-1)^{i+j}M_{ij}$ 为代数余子式。沿第 $i$ 行或第 $j$ 列展开：

$$
\det\mathbf{A}=\sum_{k=1}^n a_{ik}A_{ik},
\qquad
\det\mathbf{A}=\sum_{k=1}^n a_{kj}A_{kj}.
$$

**适用条件：某行或某列的非零元很少，或经过一两次简单变换就能使其只剩少量非零元；同时，删去该行、列后得到的低阶行列式容易计算。** 若展开后仍保留原来的排列规律，则转为递推关系。

固定第 $i$ 行的代数余子式，用第 $k$ 行元素与它们对应相乘，则

$$
\sum_{j=1}^n a_{kj}A_{ij}
=\begin{cases}
\det\mathbf{A},&k=i,\\
0,&k\ne i.
\end{cases}
$$

列的情形相同。

> [!example] 首尾相接的双对角行列式
> 设
> $$
> D_n=\begin{vmatrix}
> a_1&b_1&&&0\\
> 0&a_2&b_2&&\\
> &\ddots&\ddots&\ddots&\\
> &&0&a_{n-1}&b_{n-1}\\
> b_n&0&\cdots&0&a_n
> \end{vmatrix}.
> $$
> 沿第一行展开：
> $$
> \begin{aligned}
> D_n
> &=a_1
> \begin{vmatrix}
> a_2&b_2&&0\\
> 0&a_3&\ddots&\\
> \vdots&\ddots&\ddots&b_{n-1}\\
> 0&\cdots&0&a_n
> \end{vmatrix}\\
> &\quad-b_1
> \begin{vmatrix}
> 0&b_2&&0\\
> 0&a_3&\ddots&\\
> \vdots&\ddots&\ddots&b_{n-1}\\
> b_n&0&\cdots&a_n
> \end{vmatrix}.
> \end{aligned}
> $$
> 第一个低一阶行列式为上三角形。第二个低一阶行列式沿第一列展开，再得到以 $b_2,\ldots,b_{n-1}$ 为主对角元的三角形：
> $$
> \begin{aligned}
> D_n
> &=a_1a_2\cdots a_n
> -b_1(-1)^n b_n
> \begin{vmatrix}
> b_2&0&\cdots&0\\
> a_3&b_3&\ddots&\vdots\\
> 0&\ddots&\ddots&0\\
> 0&\cdots&a_{n-1}&b_{n-1}
> \end{vmatrix}\\
> &=a_1a_2\cdots a_n
> +(-1)^{n-1}b_1b_2\cdots b_n.
> \end{aligned}
> $$
> 即
> $$
> D_n=\prod_{i=1}^n a_i
> +(-1)^{n-1}\prod_{i=1}^n b_i.
> $$

## 初等变换

行变换对行列式的影响：

- 交换两行，变号；
- 某行乘 $k$，行列式乘 $k$；
- 某行加上另一行的倍数，行列式不变。

列变换完全相同。消元时尽量不把主元归一化，以免过早引入分式；若主元含参数，不要在未分类时除以它。

# 化为三角行列式

## 行和或列和相同

**适用条件：所有行的元素和相同，或所有列的元素和相同。** 这与各元素的具体排列无关。

若各行和均为 $s$，把所有列加到某一列，该列的每个元素都变成 $s$；若各列和均为 $s$，则把所有行加到某一行。$s=0$ 时行列式直接为 $0$；$s\ne0$ 时先提出 $s$，再作行差或列差。

> [!example] 对角元相同，非对角元相同
> 设
> $$
> D_n=\begin{vmatrix}
> a&b&\cdots&b\\
> b&a&\cdots&b\\
> \vdots&\vdots&\ddots&\vdots\\
> b&b&\cdots&a
> \end{vmatrix}.
> $$
> 记 $s=a+(n-1)b$。把所有列加到第一列：
> $$
> \begin{aligned}
> D_n
> &=
> \begin{vmatrix}
> s&b&b&\cdots&b\\
> s&a&b&\cdots&b\\
> s&b&a&\cdots&b\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> s&b&b&\cdots&a
> \end{vmatrix}
> \end{aligned}
> $$
> 
> $s=0$时显然结果为0。若$s\neq{0}$,则
> $$\begin{align}
> D_{n}=s\begin{vmatrix}
> 1&b&b&\cdots&b\\
> 1&a&b&\cdots&b\\
> 1&b&a&\cdots&b\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> 1&b&b&\cdots&a
> \end{vmatrix}
> \end{align}$$
> 
> 进而：
> $$
> \begin{aligned}
> D_n
> &=
> s\begin{vmatrix}
> 1&0&0&\cdots&0\\
> 1&a-b&0&\cdots&0\\
> 1&0&a-b&\cdots&0\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> 1&0&0&\cdots&a-b
> \end{vmatrix}\\
> &=s(a-b)^{n-1}\\
> &=\bigl(a+(n-1)b\bigr)(a-b)^{n-1}.
> \end{aligned}
> $$

## 高斯消元

这是一般行列式的通法，不要求相邻行或相邻列具有某种固定形式。

先选元素简单、便于消元的一行或一列，把同列或同行的其他元素消成零，再对低一阶行列式重复。若题中存在重复元素、成比例的部分或简单的行列关系，先利用这些关系成批制造零，再继续消元。

> [!example] 元素为 $\min(i,j)$
> $$
> D_n=
> \begin{vmatrix}
> 1&1&1&\cdots&1\\
> 1&2&2&\cdots&2\\
> 1&2&3&\cdots&3\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> 1&2&3&\cdots&n
> \end{vmatrix}.
> $$
> 从第 $n$ 行开始，依次把上一行从下一行减去：
> $$
> \begin{aligned}
> D_n
> &=
> \begin{vmatrix}
> 1&1&1&\cdots&1\\
> 0&1&1&\cdots&1\\
> 0&0&1&\cdots&1\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> 0&0&0&\cdots&1
> \end{vmatrix}\\
> &=1.
> \end{aligned}
> $$

含参数时可采用不含除法的消元：

$$
R_i\leftarrow a_{11}R_i-a_{i1}R_1.
$$

它能消去第 $i$ 行第一列的元素，又不需要假定 $a_{11}\ne0$。若对第 $2$ 至第 $n$ 行都这样变换，新行列式是原行列式的 $a_{11}^{\,n-1}$ 倍，最后应除去这个因子。

## 加边法

**适用条件：每行的大部分元素都有同一个公因子；除去各行的公因子后，剩下的字母排列相同，而主对角线上各自还多一项。**

新增第一行取 $(1,0,\ldots,0)$，右下角保留原行列式。此时沿第一行展开，新增第一列中 $1$ 以下的元素不参与计算，所以可以任意填写，实际取值由下一步准备消去的项反推。

> [!example] 新增第一列的确定
> 计算
> $$
> D_n=
> \begin{vmatrix}
> a_1+x_1y_1&x_1y_2&\cdots&x_1y_n\\
> x_2y_1&a_2+x_2y_2&\cdots&x_2y_n\\
> \vdots&\vdots&\ddots&\vdots\\
> x_ny_1&x_ny_2&\cdots&a_n+x_ny_n
> \end{vmatrix}.
> $$
> 先补第一行、第一列。第一列中的 $\lambda_1,\ldots,\lambda_n$ 目前任意：
> $$
> D_n=
> \begin{vmatrix}
> 1&0&0&\cdots&0\\
> \lambda_1&a_1+x_1y_1&x_1y_2&\cdots&x_1y_n\\
> \lambda_2&x_2y_1&a_2+x_2y_2&\cdots&x_2y_n\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> \lambda_n&x_ny_1&x_ny_2&\cdots&a_n+x_ny_n
> \end{vmatrix}.
> $$
> 希望通过列变换，把右下角化成主对角线为 $a_1,\ldots,a_n$、其余元素全为零的行列式。
> 
> 对于一般的第 $j+1$ 列：
> $$
> C_{j+1}=
> \begin{pmatrix}
> 0\\
> x_1y_j\\
> \vdots\\
> a_j+x_jy_j\\
> \vdots\\
> x_ny_j
> \end{pmatrix},
> \qquad
> C_1=
> \begin{pmatrix}
> 1\\
> \lambda_1\\
> \vdots\\
> \lambda_j\\
> \vdots\\
> \lambda_n
> \end{pmatrix}.
> $$
> 作
> $$
> C_{j+1}\leftarrow C_{j+1}+y_jC_1,
> $$
> 得
> $$
> C_{j+1}=
> \begin{pmatrix}
> y_j\\
> y_j(x_1+\lambda_1)\\
> \vdots\\
> a_j+y_j(x_j+\lambda_j)\\
> \vdots\\
> y_j(x_n+\lambda_n)
> \end{pmatrix}.
> $$
> 要使这一列变为
> $$
> \begin{pmatrix}
> y_j&0&\cdots&a_j&\cdots&0
> \end{pmatrix}^{\top},
> $$
> 必须取
> $$
> \lambda_1=-x_1,\quad
> \lambda_2=-x_2,\quad\ldots,\quad
> \lambda_n=-x_n.
> $$
> 这就是新增第一列的确定过程。代回后，
> $$
> D_n=
> \begin{vmatrix}
> 1&0&0&\cdots&0\\
> -x_1&a_1+x_1y_1&x_1y_2&\cdots&x_1y_n\\
> -x_2&x_2y_1&a_2+x_2y_2&\cdots&x_2y_n\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> -x_n&x_ny_1&x_ny_2&\cdots&a_n+x_ny_n
> \end{vmatrix}.
> $$
> 对每个 $j=1,2,\ldots,n$ 作
> $$
> C_{j+1}\leftarrow C_{j+1}+y_jC_1.
> $$
> 上面已经逐项算出第 $j+1$ 列的变化，因此变换后
> $$
> D_n=
> \begin{vmatrix}
> 1&y_1&y_2&\cdots&y_n\\
> -x_1&a_1&0&\cdots&0\\
> -x_2&0&a_2&\cdots&0\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> -x_n&0&0&\cdots&a_n
> \end{vmatrix}.
> $$
> 因为 $a_i\ne0$ ，作
> $$
> R_1\leftarrow
> R_1-\sum_{i=1}^n\frac{y_i}{a_i}R_{i+1}.
> $$
> 第一行除首项外全部变为零，得到三角形行列式：
> $$
> \begin{aligned}
> D_n
> &=
> \begin{vmatrix}
> 1+\displaystyle\sum_{i=1}^n\frac{x_i y_i}{a_i}
> &0&0&\cdots&0\\
> -x_1&a_1&0&\cdots&0\\
> -x_2&0&a_2&\cdots&0\\
> \vdots&\vdots&\vdots&\ddots&\vdots\\
> -x_n&0&0&\cdots&a_n
> \end{vmatrix}\\
> &=\left(1+\sum_{i=1}^n\frac{x_i y_i}{a_i}\right)
> \prod_{i=1}^n a_i\\
> &=\prod_{i=1}^n a_i
> +\sum_{i=1}^n x_i y_i
> \prod_{\substack{1\le j\le n\\j\ne i}}a_j.
> \end{aligned}
> $$

# 范德蒙德行列式

标准范德蒙德行列式为

$$
\begin{vmatrix}
1&x_1&x_1^2&\cdots&x_1^{n-1}\\
1&x_2&x_2^2&\cdots&x_2^{n-1}\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&x_n&x_n^2&\cdots&x_n^{n-1}
\end{vmatrix}
=\prod_{1\le i<j\le n}(x_j-x_i).
$$

**适用条件：每一行都由同一个数 $x_i$ 代入若干多项式得到，而且各列多项式的次数依次为 $0,1,\ldots,n-1$。** 用前面的列消去后面各列的低次项，最后只剩 $1,x_i,x_i^2,\ldots,x_i^{n-1}$。

> [!example] 二项式系数行列式
> 计算四阶行列式
> $$
> D=
> \begin{vmatrix}
> 1&x_1&\binom{x_1}{2}&\binom{x_1}{3}\\
> 1&x_2&\binom{x_2}{2}&\binom{x_2}{3}\\
> 1&x_3&\binom{x_3}{2}&\binom{x_3}{3}\\
> 1&x_4&\binom{x_4}{2}&\binom{x_4}{3}
> \end{vmatrix}.
> $$
> 由
> $$
> \binom{x}{2}=\frac{x^2-x}{2},
> \qquad
> \binom{x}{3}=\frac{x^3-3x^2+2x}{6},
> $$
> 先作 $C_3\leftarrow C_3+\dfrac12C_2$：
> $$
> D=
> \begin{vmatrix}
> 1&x_1&\dfrac{x_1^2}{2}&\binom{x_1}{3}\\
> 1&x_2&\dfrac{x_2^2}{2}&\binom{x_2}{3}\\
> 1&x_3&\dfrac{x_3^2}{2}&\binom{x_3}{3}\\
> 1&x_4&\dfrac{x_4^2}{2}&\binom{x_4}{3}
> \end{vmatrix}.
> $$
> 再作 $C_4\leftarrow C_4+C_3-\dfrac13C_2$：
> $$
> \begin{aligned}
> D
> &=
> \begin{vmatrix}
> 1&x_1&\dfrac{x_1^2}{2}&\dfrac{x_1^3}{6}\\
> 1&x_2&\dfrac{x_2^2}{2}&\dfrac{x_2^3}{6}\\
> 1&x_3&\dfrac{x_3^2}{2}&\dfrac{x_3^3}{6}\\
> 1&x_4&\dfrac{x_4^2}{2}&\dfrac{x_4^3}{6}
> \end{vmatrix}\\
> &=\frac1{12}
> \begin{vmatrix}
> 1&x_1&x_1^2&x_1^3\\
> 1&x_2&x_2^2&x_2^3\\
> 1&x_3&x_3^2&x_3^3\\
> 1&x_4&x_4^2&x_4^3
> \end{vmatrix}\\
> &=\frac1{12}\prod_{1\le i<j\le4}(x_j-x_i).
> \end{aligned}
> $$

# 矩阵的行列式

## 特殊矩阵的行列式

对 $n$ 阶矩阵，

$$
\det\mathbf{A}^{\top}=\det\mathbf{A},
\qquad
\det(\mathbf{A}^{-1})=(\det\mathbf{A})^{-1},
$$

$$
\det(k\mathbf{A})=k^n\det\mathbf{A},
\qquad
\det(\mathbf{A}\mathbf{B})=\det\mathbf{A}\det\mathbf{B}.
$$

$$
\begin{gathered}
\mathbf{A}^*\mathbf{A}=(\det\mathbf{A})\mathbf{E},\\
\det(\mathbf{A}^*)=(\det\mathbf{A})^{n-1}.
\end{gathered}
$$

## 矩阵加法的行列式

行列式对矩阵加法不满足分配律。若 $\mathbf{A},\mathbf{B}$ 可逆，应先提取 $\mathbf{A},\mathbf{B}$：

$$
\begin{align}
\det(\mathbf{A}+\mathbf{B})&=\det\mathbf{A}\,\det(\mathbf{E}+\mathbf{A}^{-1}\mathbf{B}) \\
&=\det \mathbf{A} \det(\mathbf{B}^{-1}+\mathbf{A}^{-1})\det \mathbf{B}
\end{align}

$$
## 分块矩阵求行列式

**适用条件：矩阵能自然分成若干方块，并且某个非对角块为零，或某个对角块容易求逆。** 这时按块消元，可以把高阶行列式降成几个低阶行列式。

分块三角形直接取对角块行列式之积：

$$
\det\begin{pmatrix}\mathbf{A}&\mathbf{B}\\0&\mathbf{D}\end{pmatrix}
=\det\mathbf{A}\det \mathbf{D}.
$$

若 $\mathbf{A}$ 可逆，先把左下角的 $\mathbf{C}$ 消成零：

$$
\begin{aligned}
\det\begin{pmatrix}\mathbf{A}&\mathbf{B}\\\mathbf{C}&\mathbf{D}\end{pmatrix}
&=
\det\left[
\begin{pmatrix}\mathbf{E}&0\\-\mathbf{C}\mathbf{A}^{-1}&\mathbf{E}\end{pmatrix}
\begin{pmatrix}\mathbf{A}&\mathbf{B}\\\mathbf{C}&\mathbf{D}\end{pmatrix}
\right]\\
&=
\det\begin{pmatrix}
\mathbf{A}&\mathbf{B}\\
0&\mathbf{D}-\mathbf{C}\mathbf{A}^{-1}\mathbf{B}
\end{pmatrix}\\
&=\det\mathbf{A}\,\det(\mathbf{D}-\mathbf{C}\mathbf{A}^{-1}\mathbf{B}).
\end{aligned}
$$

若 $\mathbf{D}$ 更容易求逆，则改用

$$
\det\begin{pmatrix}\mathbf{A}&\mathbf{B}\\\mathbf{C}&\mathbf{D}\end{pmatrix}
=\det\mathbf{D}\,\det(\mathbf{A}-\mathbf{B}\mathbf{D}^{-1}\mathbf{C}).
$$

> [!example] 对角块为单位矩阵的倍数
> 各块均为 $n$ 阶且 $a\ne0$ 时，
> $$
> \begin{aligned}
> \det\begin{pmatrix}a\mathbf{E}&\mathbf{B}\\\mathbf{C}&d\mathbf{E}\end{pmatrix}
> &=
> \det\begin{pmatrix}
> a\mathbf{E}&\mathbf{B}\\
> 0&d\mathbf{E}-\dfrac1a\mathbf{C}\mathbf{B}
> \end{pmatrix}\\
> &=a^n\det\left(d\mathbf{E}-\frac1a\mathbf{C}\mathbf{B}\right)\\
> &=\det(ad\mathbf{E}-\mathbf{C}\mathbf{B}).
> \end{aligned}
> $$

## 特征值之积

**适用条件：全部特征值比行列消元更容易得到。** 常见情形包括循环移位、每行由同一规律平移得到、已知 $\mathbf{A}^2$ 或 $\mathbf{A}^k$，或能直接找出几类向量，使 $\mathbf{A}$ 对每一类向量都只相当于乘同一个数。

若 $\mathbf{A}$ 的全部特征值为 $\lambda_1,\ldots,\lambda_n$，则

$$
\det\mathbf{A}=\prod_{i=1}^n\lambda_i.
$$

重复的特征值按重复次数相乘。

> [!example] 循环移位矩阵
> 设
> $$
> \mathbf{P}=
> \begin{pmatrix}
> 0&0&\cdots&0&1\\
> 1&0&\cdots&0&0\\
> 0&1&\ddots&0&0\\
> \vdots&\ddots&\ddots&\vdots&\vdots\\
> 0&0&\cdots&1&0
> \end{pmatrix}.
> $$
> 若 $\mathbf{P}z=\lambda z$，则
> $$
> z_n=\lambda z_1,\quad
> z_1=\lambda z_2,\quad\ldots,\quad
> z_{n-1}=\lambda z_n.
> $$
> 将这些等式相乘，得到 $\lambda^n=1$。因此 $\mathbf{P}$ 的特征值依次为全部 $n$ 次单位根 $\omega_0,\ldots,\omega_{n-1}$，从而
> $$
> \det(\mathbf{E}-t\mathbf{P})
> =\prod_{k=0}^{n-1}(1-t\omega_k)
> =1-t^n.
> $$

# 代数余子式之和

## 同一行或同一列

固定第 $i$ 行时，$A_{i1},\ldots,A_{in}$ 均与第 $i$ 行的元素无关。因此

$$
\lambda_1A_{i1}+\cdots+\lambda_nA_{in}
$$

等于把第 $i$ 行换成 $(\lambda_1,\ldots,\lambda_n)$ 后的新行列式。固定一列时同理。不要逐个计算代数余子式；换行或换列后，继续用初等变换、范德蒙德公式等结构计算。

> [!example] 化为范德蒙德行列式
> 设
> $$
> D=
> \begin{vmatrix}
> a_1&a_2&\cdots&a_n\\
> 1&1&\cdots&1\\
> x_1&x_2&\cdots&x_n\\
> \vdots&\vdots&\ddots&\vdots\\
> x_1^{n-2}&x_2^{n-2}&\cdots&x_n^{n-2}
> \end{vmatrix},
> $$
> $A_{1j}$ 是第一行元素的代数余子式。求
> $$
> S=\sum_{j=1}^n x_j^{n-1}A_{1j}.
> $$
> 将第一行换成 $(x_1^{n-1},\ldots,x_n^{n-1})$：
> $$
> \begin{aligned}
> S
> &=
> \begin{vmatrix}
> x_1^{n-1}&x_2^{n-1}&\cdots&x_n^{n-1}\\
> 1&1&\cdots&1\\
> x_1&x_2&\cdots&x_n\\
> \vdots&\vdots&\ddots&\vdots\\
> x_1^{n-2}&x_2^{n-2}&\cdots&x_n^{n-2}
> \end{vmatrix}\\
> &=(-1)^{n-1}
> \begin{vmatrix}
> 1&1&\cdots&1\\
> x_1&x_2&\cdots&x_n\\
> \vdots&\vdots&\ddots&\vdots\\
> x_1^{n-2}&x_2^{n-2}&\cdots&x_n^{n-2}\\
> x_1^{n-1}&x_2^{n-1}&\cdots&x_n^{n-1}
> \end{vmatrix}\\
> &=(-1)^{n-1}\prod_{1\le p<q\le n}(x_q-x_p).
> \end{aligned}
> $$

## 跨行、跨列

若所求式含有不同行、不同列的代数余子式，先把系数排成矩阵 $\boldsymbol{\Lambda}=(\lambda_{ij})$。

由$A_{ij}=\mathbf{A}^{*}_{ji}$可得

$$
\sum_{i=1}^n\sum_{j=1}^n\lambda_{ij}A_{ij}
=\operatorname{tr}\!\left(\boldsymbol{\Lambda}\mathbf{A}^{*}\right).
$$

常见的系数排列为：

- 只取主对角线：$\boldsymbol{\Lambda}=\mathbf{E}$，故
  $$
  \sum_{i=1}^n A_{ii}=\operatorname{tr}(\mathbf{A}^{*}).
  $$
- 取全部代数余子式：令 $\mathbf{e}=(1,\ldots,1)^{\top}$，则 $\boldsymbol{\Lambda}=\mathbf{e}\mathbf{e}^{\top}$，故
  $$
  \sum_{i=1}^n\sum_{j=1}^n A_{ij}
  =\mathbf{e}^{\top}\mathbf{A}^{*}\mathbf{e}.
  $$
- 系数可写成 $\lambda_{ij}=u_i v_j$：$\boldsymbol{\Lambda}=\mathbf{u}\mathbf{v}^{\top}$，故
  $$
  \sum_{i=1}^n\sum_{j=1}^n u_i v_j A_{ij}
  =\mathbf{v}^{\top}\mathbf{A}^{*}\mathbf{u}.
  $$


> [!example] 两种代数余子式之和
> 设
> $$
> \mathbf{A}=
> \begin{pmatrix}
> a+b&b&\cdots&b\\
> b&a+b&\cdots&b\\
> \vdots&\vdots&\ddots&\vdots\\
> b&b&\cdots&a+b
> \end{pmatrix},
> \qquad a(a+nb)\ne0.
> $$
> 分别求
> $$
> S_1=\sum_{i=1}^n\sum_{j=1}^n A_{ij},
> \qquad
> S_2=\sum_{i=1}^n A_{ii}.
> $$
>
> 令 $\mathbf{e}=(1,\ldots,1)^{\top}$。在 $\mathbf{e}$ 所在方向上，$\mathbf{A}$ 的特征值为 $a+nb$；在与 $\mathbf{e}$ 垂直的方向上，特征值为 $a$，重数为 $n-1$。因此
> $$
> \det\mathbf{A}=a^{n-1}(a+nb).
> $$
> 伴随矩阵在这两个方向上的特征值分别为
> $$
> \frac{\det\mathbf{A}}{a+nb}=a^{n-1},
> \qquad
> \frac{\det\mathbf{A}}a=a^{n-2}(a+nb).
> $$
> 由 $\mathbf{A}^{*}\mathbf{e}=a^{n-1}\mathbf{e}$，得
> $$
> S_1=\mathbf{e}^{\top}\mathbf{A}^{*}\mathbf{e}
> =na^{n-1}.
> $$
> $S_2$ 是 $\mathbf{A}^{*}$ 的迹，即其全部特征值之和：
> $$
> \begin{aligned}
> S_2
> &=a^{n-1}+(n-1)a^{n-2}(a+nb)\\
> &=na^{n-2}\bigl(a+(n-1)b\bigr).
> \end{aligned}
> $$

# 递推关系

**适用条件：求稀疏矩阵的行列式。且按$n$行或$n$列展开后，余下的行列式与原式具有相同形式，只是阶数变成 $n-1,n-2,\ldots$。** 求三对角行列式就是典型例子。


> [!example] 常系数三对角行列式
> 设
> $$
> D_n=\begin{vmatrix}
> a&b&&&\\
> c&a&b&&\\
> &c&a&\ddots&\\
> &&\ddots&\ddots&b\\
> &&&c&a
> \end{vmatrix}.
> $$
> 沿第一行展开：
> $$
> \begin{aligned}
> D_n
> &=a
> \begin{vmatrix}
> a&b&&\\
> c&a&\ddots&\\
> &\ddots&\ddots&b\\
> &&c&a
> \end{vmatrix}
> -b
> \begin{vmatrix}
> c&b&&\\
> 0&a&\ddots&\\
> \vdots&\ddots&\ddots&b\\
> 0&\cdots&c&a
> \end{vmatrix}\\
> &=aD_{n-1}-bcD_{n-2}\qquad(n\geqslant3).
> \end{aligned}
> $$
> 初值为
> $$
> D_1=a,\qquad D_2=a^2-bc.
> $$
> 由递推式对应的特征方程，按 $a^2-4bc$ 的符号分三种情况。
>
> 当 $a^2>4bc$ 时，两个实根分别为
> $$
> \frac{a+\sqrt{a^2-4bc}}2,
> \qquad
> \frac{a-\sqrt{a^2-4bc}}2.
> $$
> 代入递推式的通解，并由 $D_1,D_2$ 确定系数，得
> $$
> D_n=
> \frac{
> \left(\dfrac{a+\sqrt{a^2-4bc}}2\right)^{n+1}
> -\left(\dfrac{a-\sqrt{a^2-4bc}}2\right)^{n+1}
> }{
> \sqrt{a^2-4bc}
> }.
> $$
> 当 $a^2=4bc$ 时，特征方程有重根，故
> $$
> D_n=(n+1)\left(\frac a2\right)^n.
> $$
> 当 $a^2<4bc$ 时，必有 $bc>0$。两个根为
> $$
> \frac{a\pm i\sqrt{4bc-a^2}}2.
> $$
> 将共轭复根的幂化为正弦，得
> $$
> D_n=
> \frac{2(bc)^{(n+1)/2}}{\sqrt{4bc-a^2}}
> \sin\!\left(
> (n+1)\arccos\frac{a}{2\sqrt{bc}}
> \right).
> $$
