---
title: Decoder Workflow
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-11-10 11:45:00
katex: true
---

# 解码器的任务

[[Encoder-Workflow|编码器]]把输入序列转换成一份可以反复读取的表示 $\mathbf{H}$，形状为 $S\times d_{\text{model}}$。这里 $S$ 是编码器输入序列长度（源序列长度）。$d_{\text{model}}$ 是 token 向量的表示维度数。

解码器从一个起始 Token `<BOS>` 开始，不断预测下一个 Token。它每生成一个 Token，就把这个 Token 追加到已生成的前缀后面，再预测下一个，直到生成 `<EOS>`。

# 推理

设目标序列总长度为 $T$，当前生成第 $t$ 个 Token，前缀长度为 $t$。第 $t$ 步的输入是已经生成的前缀。前缀经过输出侧 Embedding 与位置编码后，得到解码器表示 $\mathbf{X}_D$，形状为 $t\times d_{\text{model}}$。

解码器的一层依次执行掩码自注意力、交叉注意力、FFN。每个模块后面都跟着 Add & Norm，作用与编码器中完全相同。一层解码器只完成一轮历史理解、编码器信息融合和特征加工；完整的解码器会堆叠多个结构相同的层，每层有自己独立的注意力与 FFN 参数。

[html-card height=680](../assets/attention-decoder-steps.html)

推理时每步只取最新位置的一行输出，而不使用整个矩阵。

> [!question] 为什么只用最新一行
> 注意力按权重汇总前面位置的 Value，因此最新一行的表示融合了整段前缀的信息，生成下一个 Token 只需要这一行。

## 掩码自注意力

解码器在生成第 $t$ 个 Token 时，只能读取已经生成的前缀，不能看到未来内容。训练时一次送入完整目标序列，如果不加限制，前面的位置就能直接读到后面的正确答案。

[[Attention-In-Transformer#掩码自注意力|因果掩码]]把匹配分数矩阵的右上三角置为 $-\infty$，第 $i$ 行只允许第 $j\le i$ 列参与 Softmax。这样，训练与推理的可见范围一致。推理时未来 Token 尚未产生，本来就读不到；掩码主要解决训练阶段一次读入完整目标序列时的信息泄露。

[html-card height=680](../assets/decoder-masked-attention-slides.html)


## 交叉注意力

解码器先由掩码自注意力得到当前前缀的上下文表示 $\mathbf{Z}_T$。交叉注意力用它去读取编码器输出：

$$
\mathbf{Q}=\mathbf{Z}_T\mathbf{W}_Q,\qquad
\mathbf{K}=\mathbf{H}\mathbf{W}_K,\qquad
\mathbf{V}=\mathbf{H}\mathbf{W}_V
$$

其中 $\mathbf{W}_Q$、$\mathbf{W}_K$、$\mathbf{W}_V$ 是可学习投影矩阵。查询来自解码器，键与值来自编码器，这是交叉注意力与自注意力唯一的区别。

编码器输出 $\mathbf{H}$ 的形状为 $S\times d_{\text{model}}$，投影后键与值的行数仍为 $S$。训练时整段目标序列有 $T$ 个位置，查询有 $T$ 行，注意力权重矩阵为 $T\times S$；推理第 $t$ 步只有最新一行查询，权重矩阵为 $1\times S$。

## 输出分布

解码器最后一层的输出仍是一个 $T\times d_{\text{model}}$ 的矩阵。每个位置是一个 $d_{\text{model}}$ 维向量，但向量的每一维并不直接对应某个 Token。为了预测下一个 Token，线性层只取最后一行的向量，把它映射到词表空间：

$$
\mathbf{z}_T=\mathbf{X}_D\mathbf{W}_{\text{vocab}}
$$

其中 $\mathbf{X}_D$ 取最后一行的表示，$\mathbf{W}_{\text{vocab}}\in\mathbb{R}^{d_{\text{model}}\times|\mathcal{V}|}$ 是可学习映射矩阵，$|\mathcal{V}|$ 是词表大小。输出的 $\mathbf{z}_T$ 是词表中每个 Token 的原始分数，称为 logits，**还不是概率**。

设第 $i$ 个 Token 的 logit 为 $l_i$。Softmax 把 logits 转换成概率分布。温度参数 $\tau$ 在 Softmax 之前缩放 logits：

$$
p_i=\frac{\exp(l_i/\tau)}{\sum_{j=1}^{|\mathcal{V}|}\exp(l_j/\tau)}
$$

> [!example] 温度为 0
> $\tau=0$ 时取 $\tau\to0^+$ 的极限：最大项 $\exp(l_{\max}/\tau)$ 在分子与分母中占主导，Softmax 收敛为在最大 logit 对应 Token 上取 1、其余取 0 的 one-hot 分布，等价于[[#贪心采样]]。设 $i^\ast=\arg\max_j l_j$，则 $\lim_{\tau\to0^+}p_{i^\ast}=1$，其余 $p_i=0$。
> 若有多个 Token 并列最大，概率在并列项之间均分。
> 代码实现使用很小的正数，或直接取最大 logit 的 Token。

$\tau$ 调高时分布更平缓，低概率 Token 也有机会被选中；$\tau$ 调低时分布更尖锐，生成更保守。

[html-card height=620](../assets/decoder-distribution-slides.html)

## 输出采样

得到概率分布后，还需要决定从哪些候选中、以什么方式选出 Token。不同策略的区别在于候选范围与随机性。

![三种采样策略|630](../assets/decoder-sampling-diagram.svg)

常使用用温度调整分布，再用 Top-K 或 Top-P 缩小候选并采样。

### 贪心采样

每步直接取概率最大的 Token。结果确定，不引入随机性。但它只比较当前这一步的候选，不会比较整条序列的联合概率，也不能回退修改已生成的 Token。前面一旦选错，后面只能沿这个前缀继续，因此可能错过整体概率更高的序列，也容易生成重复或过于常见的句子。

### Top-K 采样

从词表中保留概率最高的 $K$ 个候选，记候选集合为 $S$。归一化得到新的概率$q$

$$
q_i=\frac{p_i}{\sum_{j\in S}p_j},\qquad i\in S
$$

然后从 $S$ 中**随机**抽一个 Token，该 Token 被抽中的概率正是对应的 $q$ 。显然，概率高的更容易被抽到，但其他候选仍有机会。

分布很集中时，$K$ 可能把低概率 Token 也放进候选。

### Top-P 采样

从词表按概率从高到低累加，取累计概率首次达到 $P$ 的最小集合 $S$。$S$ 内的概率同样按上式重归一化，再依概率 $q_i$ 从 $S$ 中随机抽一个 Token。

候选数量随分布动态变化。

# 训练

推理与训练的区别集中在可见范围与并行方式：推理逐 Token 生成，训练一次并行预测所有位置。

推理时下一个 Token 尚未产生，各步必须依次进行。训练时正确的目标序列已经存在，可以把整句右移一位当作解码器输入，一次算出 $T$ 个位置的预测。Transformer 消除的是已知序列内部的循环依赖，不是自回归生成的先后顺序。

![训练与推理的差别|630](../assets/decoder-training-vs-inference.svg)

## Teacher Forcing

Teacher Forcing 指训练时把真实目标序列右移一位作为解码器输入：每个位置用真实前缀预测下一个 Token，而不是用模型自己生成的前缀。这样，所有位置可以在一次前向传播中并行计算，不必等前一个 Token 生成。

具体做法是：输入为 `<BOS>` 加上目标序列去掉最后一个 Token，预测目标为目标序列去掉第一个 Token 后加上 `<EOS>`。每个位置都预测它的下一个 Token，并使用[[Attention-In-Transformer#掩码自注意力|因果掩码]]保证位置 $i$ 只读取前 $i$ 个输入位置，不会看到答案。

Teacher Forcing 避免了错误前缀在训练中逐步累积。

损失函数逐位置比较预测分布与正确 Token，通常用交叉熵。

## 批量与填充

实际实现会同时处理 $B$ 个样本。不同样本长度不同，需要使用 `<PAD>` 补齐到同一长度。补齐的 `<PAD>` 位置不参与损失，也不参与注意力，利用掩码标记：有效 Token 记 1，`<PAD>` 记 0。

下表列出单个样本与批量处理时的形状，其中 $S$ 是源序列长度，$T$ 是目标序列长度，$\lvert\mathcal{V}\rvert$ 是词表大小，$B$ 是批量大小。

| 数据 | 单个样本 | 批量处理 |
| --- | --- | --- |
| 编码器序列表示 | $S\times d_{\text{model}}$ | $B\times S\times d_{\text{model}}$ |
| 解码器序列表示 | $T\times d_{\text{model}}$ | $B\times T\times d_{\text{model}}$ |
| 掩码自注意力权重 | $T\times T$ | $B\times T\times T$ |
| 交叉注意力权重 | $T\times S$ | $B\times T\times S$ |
| 线性层输出 | $T\times\lvert\mathcal{V}\rvert$ | $B\times T\times\lvert\mathcal{V}\rvert$ |


