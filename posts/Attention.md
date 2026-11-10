---
title: Attention
tags:
  - NLP
categories:
  - Artificial Intelligence
date: 2026-10-27 23:29:21
katex: true
---

# 固定上下文缺什么

[[Encoder-Decoder-And-Seq2Seq#编码器与解码器|编码器-解码器]]的经典实现把整条源句压进一个向量 $\boldsymbol{c}$，解码器每一步都从同一个 $\boldsymbol{c}$ 里取信息。不同生成位置所需的源句内容可能不同，一个固定向量只能折中压缩。

编码器本来就在每个输入位置产生过一个隐藏状态，第 $i$ 个位置的状态 $\boldsymbol{h}_i$ 记录了读到该位置为止的内容。让解码器每一步回到这些状态里按需取用，就不必只依赖一个向量。

设源句长度 $n$，编码器输出列向量 $\boldsymbol{h}_1,\dots,\boldsymbol{h}_n$，每个都是 $d$ 维。将各位置的编码状态按行组成 $\mathbf{H}\in\mathbb{R}^{n\times d}$，第 $i$ 行是 $\boldsymbol{h}_i^{\top}$，解码器保留这些状态供每一步读取。第 $t$ 步的状态 $\boldsymbol{s}_t$ 与每一行做内积。

# 一次注意力计算

## 打分、归一化与加权

**打分。** 用当前状态与每个编码状态算一个标量分数：

$$
e_{t,i} = \boldsymbol{s}_t^{\top}\boldsymbol{h}_i
$$

点积要求 $\boldsymbol{s}_t$ 与 $\boldsymbol{h}_i$ 维数相同，分数可以是任意实数。

**归一化。** 把分数变成一组和为 1 的权重：

$$
\alpha_{t,i} = \frac{\exp(e_{t,i})}{\sum_{j=1}^{n} \exp(e_{t,j})}
$$

softmax 逐行作用在 $n$ 个位置上，每个权重都大于 0，只反映分数的相对大小。

**加权求和。** 用权重把编码状态汇总成上下文向量：

$$
\boldsymbol{c}_t = \sum_{i=1}^{n} \alpha_{t,i}\boldsymbol{h}_i
$$

最后把 $\boldsymbol{c}_t$ 与解码状态拼接，送输出层得到下一个 token 的分布。

> [!note] 权重与词表概率
> 注意力权重描述这一步给各个源位置的分配比例，词表分布由输出层给出。权重最大的位置不必然就是即将生成的那个词所对应的源词。

## 数值示例

演示设定：源句为「我 / 喜欢 / 深度 / 学习」，当前解码步要生成「深度」；为演示直接给定查询 $\boldsymbol{q}=[1,0]$，不取自身行。以下状态与查询都是未训练的演示取值，四个源位置对应四个 2 维状态：

$$
\mathbf{H} = \begin{bmatrix}
0.2 & 0.1 \\
0.1 & 0.2 \\
2.5 & 0.3 \\
0.3 & 2.5
\end{bmatrix},
\quad \boldsymbol{h}_1 \leftrightarrow \text{我},\ \boldsymbol{h}_2 \leftrightarrow \text{喜欢},\ \boldsymbol{h}_3 \leftrightarrow \text{深度},\ \boldsymbol{h}_4 \leftrightarrow \text{学习}
$$

取查询 $\boldsymbol{q}=[1,0]$，与四行的点积为 $[0.2,0.1,2.5,0.3]$。对四个分数取指数并归一化，得到权重 $\boldsymbol{\alpha}=[0.0770,0.0697,0.7682,0.0851]$，分量之和为 1。

再乘 $\mathbf{H}$，得上下文向量 $\boldsymbol{c}=[1.9684,0.4649]$。
 

> [!note] PAD
> 批量计算时短句要补齐，补位符号记作 PAD。掩码标记哪些位置有效；计算时在 softmax 之前把无效位置的分数置为 $-\infty$，再在有效位置上归一化，补齐位置的权重因此为 0。

[html-card height=820](../assets/attention-mechanism-slides.html)

# Q、K、V 的分工

同样的计算可以按角色拆开：查询提出匹配需求，键用于匹配，值参与汇总。

查询描述当前位置在更新自身表示时想寻找什么信息；键描述当前位置提供哪些可以参与匹配的特征；值描述匹配成功后真正提供给其他位置的内容。

表中 $n_q$ 是查询位置数，$n_k$ 是被检索的位置数，$d_k$ 与 $d_v$ 分别是键与值的维数。

| 角色 | 作用 | 形状 |
| --- | --- | --- |
| 查询 $\mathbf{Q}$ | 由解码器状态产生（本示例取 $\boldsymbol{s}_t$） | $n_q\times d_k$ |
| 键 $\mathbf{K}$ | 与查询算分数 | $n_k\times d_k$ |
| 值 $\mathbf{V}$ | 按权重汇总的内容 | $n_k\times d_v$ |

本篇的点积版本把键与值都取成 $\mathbf{H}$，即 $\mathbf{K}=\mathbf{V}=\mathbf{H}$，数值上相同、作用不同。

一般形式下三者各自经过一个可学习投影：

$$
\mathbf{Q} = \mathbf{X}_Q\mathbf{W}_Q, \qquad \mathbf{K} = \mathbf{X}_K\mathbf{W}_K, \qquad \mathbf{V} = \mathbf{X}_V\mathbf{W}_V
$$

这里 $\mathbf{X}_Q,\mathbf{X}_K,\mathbf{X}_V$ 是三种角色对应的来源表示，$\mathbf{W}_Q,\mathbf{W}_K,\mathbf{W}_V$ 是学习得到的投影矩阵。即使 $\mathbf{X}_K=\mathbf{X}_V$，$\mathbf{W}_K$ 与 $\mathbf{W}_V$ 也不必相同。

自注意力把三种来源取成同一个输入矩阵 $\mathbf{X}$，投影形式简化为 $\mathbf{Q}=\mathbf{X}\mathbf{W}_Q$，$\mathbf{K}=\mathbf{X}\mathbf{W}_K$，$\mathbf{V}=\mathbf{X}\mathbf{W}_V$。三组投影矩阵仍各自学习。

分数矩阵 $\mathbf{QK^{\top}}$ 的形状是 $n_q\times n_k$，softmax 按每行的 $n_k$ 个候选归一化，再乘 $\mathbf{V}$ 得到 $n_q\times d_v$ 的输出。

编码器-解码器注意力是跨序列的：查询来自解码器，键与值来自编码器。三个输入都来自同一条序列时 $n_q=n_k=n$，每个位置既发起查询也提供内容，这种机制称为自注意力（self-attention）。计算公式相同，区别在输入的来源。

> [!note] 软对齐
> 把各解码步的权重按行排开，行对应目标词，列对应源位置，可以看到一个目标词同时参考多个源位置。这种对应称为软对齐（soft alignment），不要求一对一；最大权重的位置不保证是语言学上的对应词，权重高也不等于对输出贡献大。


# 从点积注意力到 Transformer

## 缩放

若各分量独立、均值为 0、方差为 1，点积 $\mathbf{QK^{\top}}$ 的方差随 $d_k$ 增大，分数幅度随之变大，softmax 容易过于集中。常见的处理是除以 $\sqrt{d_k}$：

$$
\text{scores} = \frac{\mathbf{QK^{\top}}}{\sqrt{d_k}}
$$

缩放之后，后续的归一化与加权求和不变。

[html-card height=430](../assets/attention-score-variance.html)

## 位置信息

没有位置编码、也没有额外掩码的自注意力只比较向量相似度，不含顺序信息：打乱输入序列，输出按同样的顺序重排。要区分“第 3 个词”与“第 4 个词”，必须另外提供位置信息。

[html-card height=820](../assets/attention-position-scramble.html)

## 并行的边界

Transformer 的编码器在整段输入已知时可以并行：所有位置的查询、键、值同时算出。解码器训练时也一次送入右移后的真实前缀，用因果掩码把每个位置可看的范围限制在当前位置及之前，未来的 token 不参与计算。

推理逐 token 进行，第 $t$ 步的查询依赖已经生成的前缀，只能一步接一步算。

注意力加到循环网络上时，递推仍在，教师强制一次送整段也不改变状态更新的顺序；Transformer 用自注意力替代递推，才得到按位置并行的训练。完整模型还包含前馈网络、残差连接、层归一化与位置编码。


