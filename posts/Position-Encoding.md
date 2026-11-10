---
title: Position Encoding
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-11-09 22:35:00
katex: true
---

# 为什么需要位置编码

循环网络按时间步依次处理 Token，Token 的先后顺序由递推结构隐含表示。Transformer 以并行计算替代递推，所有位置同时参与注意力计算，序列顺序不再进入计算。

但由于[[Attention#位置信息|自注意力并不天然关注顺序信息]]，因此在进入第一层自注意力之前，需要为每个 Token 补充位置信息。词向量表示 Token 的语义，位置向量表示它在序列中的位置，两者相加后作为模型输入：

$$
\mathbf{X}_i = \operatorname{Embedding}(x_i) + \operatorname{PositionalEncoding}(i)
$$

位置向量的维数与词向量相同，都是 $d_{\text{model}}$。逐元素相加之后，输入形状不变，后面的投影矩阵也不需要改动。

同一个 Token 出现在不同位置时，得到的 $\mathbf{X}_i$ 不同，由它生成的查询、键、值也随之不同。注意力因此能同时利用语义与顺序。

位置信息只在输入端加入一次，之后各层沿用更新后的表示。

# 位置向量的构造

位置向量需要与词向量结合后送入模型。结合方式有相加与拼接，位置的表示方式有整数编号与正弦函数。

## 位置信息如何合并

位置向量与词向量可以相加，也可以拼接。相加保持维数不变，两种信息落在同一组维度上。拼接改变维数，把两种信息分到不同的区间。

Transformer 选择了相加，原因在拼接的代价。

> [!question] 为什么相加而不是拼接
> 拼接会把输入维数从 $d_{\text{model}}$ 扩大到 $d_{\text{model}}+d_{\text{pos}}$。输入变宽之后，生成查询、键、值的投影矩阵都要跟着扩大，参数量与计算量一起上涨。
> 拼接也没有把两种信息真正分开。投影之后，位置与语义仍然汇总到同一组输出维度里。
> 相加不改变维数。位置与语义共用每一维，区分两者的任务交给投影矩阵学习。

## 位置向量如何表示

位置可以用一个顺序编号表示，也可以用一个向量表示。编号的数值范围随序列长度增长。

Transformer 选择向量表示，原因与数值范围有关。

> [!question] 为什么不用顺序编号
> 编号是随长度增长的标量。序列越长，编号越大，最终会超过 Embedding 的数值范围。
> 把它归一化到 0 到 1 可以控制范围，但必须预先固定最大长度，更长的序列无法表示。
> 编号也不携带位置之间的关系。第 5 个与第 6 个相差 1，这个差只在编号系统内部成立，进入向量运算后没有对应结构。
> 位置向量每个分量落在 $[-1,1]$，与序列长度无关。不同分量还携带不同频率的变化。两个位置相隔多远，可以体现在向量之间的关系里。

## 正弦位置编码

正弦位置编码采用正弦、余弦函数，具体计算方法如下。

### 公式与频率

Transformer 原论文由位置 $pos$ 与模型维数 $d_{\text{model}}$ 计算位置向量。维度两两一组，第 $i$ 组使用频率 $1/D_i$，其中 $D_i=10000^{2i/d_{\text{model}}}$。这一组算出的两个数，分别写入第 $2i$ 维与第 $2i+1$ 维：

$$
PE(pos,2i)=\sin\left(\frac{pos}{D_i}\right),\qquad
PE(pos,2i+1)=\cos\left(\frac{pos}{D_i}\right),\qquad
i=0,1,\dots,\frac{d_{\text{model}}}{2}-1
$$

$d_{\text{model}}$ 必须为偶数，共有 $d_{\text{model}}/2$ 组。$i=0$ 时 $D_0=1$，这一组变化最快。$i$ 增大时，$D_i$ 按指数增长，波长随之变长。

整条位置向量由这些分组依次排列：

$$
\mathbf{P}_{pos}=\left[\sin\frac{pos}{D_0},\cos\frac{pos}{D_0},\dots,\sin\frac{pos}{D_{m-1}},\cos\frac{pos}{D_{m-1}}\right],\qquad m=\frac{d_{\text{model}}}{2}
$$

索引小的组变化快，适合区分相邻位置。索引大的组变化慢，适合容纳远距离的差异。这与二进制计数类似：低位频繁翻转，高位翻转缓慢。不同频率共同覆盖从邻近到远距离的尺度。

### 相对距离

注意力关注两个 Token 在序列中的距离。取位置 $pos$ 与 $pos+k$ 的两个位置向量，考察第 $i$ 组分量，即第 $2i$ 维与第 $2i+1$ 维。两个向量在该组上的乘积之和为：

$$
\sin\frac{pos}{D_i}\sin\frac{pos+k}{D_i}+\cos\frac{pos}{D_i}\cos\frac{pos+k}{D_i}=\cos\frac{k}{D_i}
$$

两个位置向量的点积等于各维乘积之和。第 $i$ 组贡献 $\cos(k/D_i)$，因此：

$$
\mathbf{P}_{pos}\cdot\mathbf{P}_{pos+k}=\sum_i\cos\left(\frac{k}{D_i}\right)
$$

右边不含 $pos$。两个位置向量的点积只由距离 $k$ 决定。

# 旋转位置编码（RoPE）

正弦编码将位置向量加到词向量上，然后向量与投影矩阵 $\mathbf{X}$ 相乘生成查询、键、值。位置和语义在投影之前已经混合，查询与键的点积因此不能保证只依赖相对距离而与绝对位置无关。

旋转位置编码（Rotary Position Embedding，RoPE）不再生成位置向量，而是对投影后的查询与键按位置旋转，使二者的点积只依赖相对距离而与绝对位置无关，进而可以反映位置信息。

旋转只作用在查询与键上，值保持原样。

> [!question] 为什么相加要同时处理查询、键、值，而 RoPE 只旋转查询与键
> 相加的位置编码加在 Embedding 之后、投影之前。查询、键、值分别由同一个 $\mathbf{X}$ 投影得到，三者的投影矩阵都会看到位置。位置因此同时进入三者：匹配分数带位置，被汇总的内容也带位置。
> RoPE 在投影之后才施加位置。查询与键需要位置，是为了让点积只含相对距离。值不参与打分，它的作用是被按权重汇总。旋转值只会让汇总结果随绝对位置转动，并不提供匹配所需的信息。
> 所以，相加法无法把位置限制在查询与键上；RoPE 可以，因为它在投影之后按角色分别处理。这样，模型不必再从绝对位置中推算相对关系，位置差直接出现在分数里。

## 旋转方式

将查询与键的维度按相邻两维分组。位置 $m$ 的向量，其第 $j$ 组按角度 $\theta_{m,j}=m\omega_j$ 旋转，其中各组频率为：

$$
\omega_j=10000^{-2j/d_{\text{model}}},\qquad j=0,1,\dots,\frac{d_{\text{model}}}{2}-1
$$

$j$ 较小时频率高、波长短，用于分辨邻近位置；$j$ 较大时频率低、波长长，用于表达远距离。位置越靠后，各组的旋转角越大。

记二维旋转矩阵：

$$
\mathbf{R}(\theta)=
\begin{bmatrix}
\cos\theta & -\sin\theta\\
\sin\theta & \cos\theta
\end{bmatrix}
$$

查询 $\mathbf{q}_{m}$ 第 $j$ 组分量 $(q_{2j},q_{2j+1})$ 旋转后得 $(q'_{2j},q'_{2j+1})$ ：

$$
\begin{bmatrix}
q'_{2j}\\
q'_{2j+1}
\end{bmatrix}
=
\mathbf{R}(\theta_{m,j})
\begin{bmatrix}
q_{2j}\\
q_{2j+1}
\end{bmatrix}
$$

键 $\mathbf{k}_{n}$ 同理。每组旋转只改变该组的方向，不改变长度。

## 相对位置

查询 $\mathbf{q}_m$ 与键 $\mathbf{k}_n$ 分别旋转后，第 $j$ 组分量对点积的贡献为：

$$
\left(\mathbf{R}(m\omega_j)
\begin{bmatrix}
q_{2j}\\
q_{2j+1}
\end{bmatrix}\right)^{\top}
\left(\mathbf{R}(n\omega_j)
\begin{bmatrix}
k_{2j}\\
k_{2j+1}
\end{bmatrix}\right)
$$

旋转矩阵满足 $\mathbf{R}(\alpha)^{\top}\mathbf{R}(\beta)=\mathbf{R}(\beta-\alpha)$，因此上式等于：

$$
\begin{bmatrix}
q_{2j}\\
q_{2j+1}
\end{bmatrix}^{\top}
\mathbf{R}\left((n-m)\omega_j\right)
\begin{bmatrix}
k_{2j}\\
k_{2j+1}
\end{bmatrix}
$$

结果只与位置差 $n-m$ 有关，与 $m$ 本身无关。旋转不改变向量长度，注意力分数的量级不受位置影响。


