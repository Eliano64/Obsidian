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

**打分。** 用当前解码状态 $\boldsymbol{s}_t \in \mathbb{R}^{d}$ 与每个编码状态 $\boldsymbol{h}_i \in \mathbb{R}^{d}$ 做点积，算出标量相关性得分；将全部 $n$ 个位置的得分排成得分向量 $\boldsymbol{e}_t = \mathbf{H}\boldsymbol{s}_t \in \mathbb{R}^{n}$，其第 $i$ 个分量为：

$$
e_{t,i} = \boldsymbol{s}_t^{\top}\boldsymbol{h}_i
$$

点积要求 $\boldsymbol{s}_t$ 与 $\boldsymbol{h}_i$ 维数相同。经典序列模型中，Bahdanau 注意力（加性注意力）先将 $[\boldsymbol{s}_t; \boldsymbol{h}_i]$ 拼接后过单层前馈网络打分，能够处理查询与键维数不同的情形；而现代 Transformer 采用的点积注意力可直接利用高效的高度并行矩阵乘法实现，因而成为主流。

**归一化。** 将整组得分向量 $\boldsymbol{e}_t$ 送入 Softmax，转化为非负且总和为 $1$ 的注意力权重向量 $\boldsymbol{\alpha}_t = \operatorname{softmax}(\boldsymbol{e}_t) \in \mathbb{R}^{n}$，其第 $i$ 个分量为：

$$
\alpha_{t,i} = \frac{\exp(e_{t,i})}{\sum_{j=1}^{n} \exp(e_{t,j})}
$$

**加权求和。** 用权重向量 $\boldsymbol{\alpha}_t$ 对全部编码位置的状态加权汇总，得到第 $t$ 步的上下文向量 $\boldsymbol{c}_t \in \mathbb{R}^{d}$：

$$
\boldsymbol{c}_t = \sum_{i=1}^{n} \alpha_{t,i}\boldsymbol{h}_i
$$

最后把 $\boldsymbol{c}_t$ 与解码状态拼接，送输出层得到下一个 token 的分布。

> [!note] 权重与词表概率
> 注意力权重描述这一步给各个源位置的分配比例，词表分布由输出层给出。权重最大的位置不必然就是即将生成的那个词所对应的源词。

## 数值示例

演示设定：源句为「我 / 喜欢 / 深度 / 学习」，当前解码步要生成「深度」；为演示直接给定查询向量 $\boldsymbol{q}=[1,0]^{\top}$，不取自身行。以下状态与查询都是未训练的演示取值，四个源位置按行排成 2 维状态矩阵 $\mathbf{H} \in \mathbb{R}^{4 \times 2}$：

$$
\mathbf{H} = \begin{bmatrix}
0.2 & 0.1 \\
0.1 & 0.2 \\
2.5 & 0.3 \\
0.3 & 2.5
\end{bmatrix},
\quad \boldsymbol{h}_1^{\top} \leftrightarrow \text{我},\ \boldsymbol{h}_2^{\top} \leftrightarrow \text{喜欢},\ \boldsymbol{h}_3^{\top} \leftrightarrow \text{深度},\ \boldsymbol{h}_4^{\top} \leftrightarrow \text{学习}
$$

计算查询与四行的点积得分 $\mathbf{H}\boldsymbol{q} = [0.2,0.1,2.5,0.3]^{\top}$。对四个分数取指数并归一化，得到行向量形式权重 $\boldsymbol{\alpha}^{\top}=[0.0770,0.0697,0.7682,0.0851]$，分量之和为 1。

用行权重左乘按行排布的状态矩阵，得上下文向量 $\boldsymbol{c}^{\top} = \boldsymbol{\alpha}^{\top}\mathbf{H} = [1.9684,0.4649]$。

> [!tip] PAD
> 批量计算时短句要补齐，补位符号记作 PAD。掩码标记哪些位置有效；计算时在 softmax 之前把无效位置的分数置为 $-\infty$，再在有效位置上归一化，补齐位置的权重因此为 0。

[html-card height=820](../assets/attention-mechanism-slides.html)

# Q、K、V

定义查询 $\mathbf{Q}$ 提出匹配需求，键 $\mathbf{K}$ 用于匹配，值 $\mathbf{V}$ 参与汇总。

查询描述当前位置在更新自身表示时想寻找什么信息；键描述当前位置提供哪些可以参与匹配的特征；值描述匹配成功后真正提供给其他位置的内容。

表中 $n_q$ 是查询位置数，$n_k$ 是被检索的位置数，$d_k$ 与 $d_v$ 分别是键与值的维数。

| 角色              | 作用                                | 形状              |
| --------------- | --------------------------------- | --------------- |
| 查询 $\mathbf{Q}$ | 由解码器状态产生（本示例取 $\boldsymbol{s}_t$） | $n_q\times d_k$ |
| 键 $\mathbf{K}$  | 与查询算分数                            | $n_k\times d_k$ |
| 值 $\mathbf{V}$  | 按权重汇总的内容                          | $n_k\times d_v$ |

一般形式下三者各自经过一个可学习投影：

$$
\mathbf{Q} = \mathbf{X}_Q\mathbf{W}_Q, \qquad \mathbf{K} = \mathbf{X}_K\mathbf{W}_K, \qquad \mathbf{V} = \mathbf{X}_V\mathbf{W}_V
$$

这里 $\mathbf{X}_Q,\mathbf{X}_K,\mathbf{X}_V$ 是三种角色对应的来源表示，$\mathbf{W}_Q,\mathbf{W}_K,\mathbf{W}_V$ 是学习得到的投影矩阵。即使 $\mathbf{X}_K=\mathbf{X}_V$，$\mathbf{W}_K$ 与 $\mathbf{W}_V$ 也不必相同。

自注意力把三种来源取成同一个输入矩阵 $\mathbf{X}$，投影形式简化为 $\mathbf{Q}=\mathbf{X}\mathbf{W}_Q$，$\mathbf{K}=\mathbf{X}\mathbf{W}_K$，$\mathbf{V}=\mathbf{X}\mathbf{W}_V$。三组投影矩阵仍各自学习。

分数矩阵 $\mathbf{QK^{\top}}$ 的形状是 $n_q\times n_k$，softmax 按每行的 $n_k$ 个候选归一化，再乘 $\mathbf{V}$ 得到 $n_q\times d_v$ 的输出。

> [!note] 软对齐
> 把各解码步的权重按行排开，行对应目标词，列对应源位置，可以看到一个目标词同时参考多个源位置。这种对应称为软对齐（soft alignment），不要求一对一；最大权重的位置不保证是语言学上的对应词，权重高也不等于对输出贡献大。

# 从点积注意力到 Transformer

## 缩放点积注意力

点积分数直接送入 Softmax 在维度较大时会引发梯度消失。设查询向量 $\boldsymbol{q}\in\mathbb{R}^{d_k}$ 与键向量 $\boldsymbol{k}\in\mathbb{R}^{d_k}$ 的各分量独立、零均值、单位方差，则点积 $z = \boldsymbol{q}^{\top}\boldsymbol{k} = \sum_{i=1}^{d_k} q_i k_i$ 的均值为 $0$，方差随维度线性增长：

$$
\operatorname{Var}(z) = \sum_{i=1}^{d_k}\operatorname{Var}(q_i k_i) = d_k
$$

$d_k$ 较大时点积幅值显著拉大，经指数函数放大后，Softmax 输出几乎全部集中在最大分数位置（接近 $1$，其余接近 $0$）。此时任意输出分量的导数 $p_i(1-p_i)$ 与 $-p_i p_j$ 均趋近 $0$（$p_i$ 为第 $i$ 个位置权重），注意力梯度随之消失。

将分数除以 $\sqrt{d_k}$ 后，点积方差拉回 $1$（不再随 $d_k$ 膨胀），使 Softmax 维持在梯度畅通的区间：

$$
\operatorname{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V})=\operatorname{softmax}\left(\frac{\mathbf{QK^{\top}}}{\sqrt{d_k}}\right)\mathbf{V}
$$

[html-card height=430](../assets/attention-score-variance.html)

## 位置信息

无掩码自注意力只比较向量内容相似度，满足**置换等变性**：对输入序列矩阵 $\mathbf{X}$ 乘任意行置换矩阵 $\mathbf{P}$，输出也只作相同重排：

$$
\operatorname{SelfAttention}(\mathbf{P}\mathbf{X}) = \mathbf{P}\operatorname{SelfAttention}(\mathbf{X})
$$

词序被打乱后各词的新表示数值不变，模型无法区分“狗咬人”和“人咬狗”，因此必须额外注入[[Position-Encoding|位置信息]]。

[html-card height=820](../assets/attention-position-scramble.html)

关于上述机制如何在编码器与解码器中落地为自注意力、掩码自注意力与交叉注意力，见 [[Attention-In-Transformer]]。
