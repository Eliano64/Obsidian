---
title: Attention In Transformer
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-11-09 23:37:00
katex: true
---

# 三类注意力

Transformer 通过注意力更新序列表示：进入注意力前，每个 Token 的向量主要承载自身语义与位置；经过注意力后，各位置向量在不改变序列长度与向量维度的前提下融入上下文信息。通用计算流程见 [[Attention#打分、归一化与加权|一次注意力计算]]。

![注意力前后：向量形状不变，内容改变|631](../assets/attention-before-after.svg)

Transformer 中的三类注意力共享相同的计算流程，在实现上均采用 [[Multi-Head-Attention|多头注意力]]，区别仅在于查询、键与值的来源：

| 类型 | 查询来自 | 键与值来自 | 作用 |
| --- | --- | --- | --- |
| 自注意力 | 输入序列 | 输入序列 | 生成输入序列的上下文表示 |
| 掩码自注意力 | 输出序列 | 输出序列 | 生成当前输出前缀的上下文表示 |
| 交叉注意力 | 解码器 | 编码器 | 根据当前生成状态读取输入序列 |

# 自注意力

编码器使用自注意力处理输入序列。每个位置既发起查询，也提供键和值。因此每个位置都能观察整个序列，并根据与其他位置的关系重新生成表示。

例如在句子「The animal didn't cross the street because it was too tired」中，代词 it 在更新自身表示时，与 animal 的匹配程度最高，与 street 等位置也有一定匹配。

设序列长度为 $N$，查询 $\mathbf{Q}\in\mathbb{R}^{N\times d_k}$、键 $\mathbf{K}\in\mathbb{R}^{N\times d_k}$、值 $\mathbf{V}\in\mathbb{R}^{N\times d_v}$（$d_k$、$d_v$ 为键与值的特征维数），[[Attention#缩放点积注意力|公式仍为以下形式]]：

$$
\operatorname{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V})=\operatorname{softmax}\left(\frac{\mathbf{QK^{\top}}}{\sqrt{d_k}}\right)\mathbf{V}
$$

![代词 it 对各位置的注意力权重|631](../assets/attention-example-lines.svg)

# 掩码自注意力

解码器在训练时一次收到完整的目标序列。不加限制时，前面位置在更新表示时可以直接读取后面的正确答案，训练因此失去意义。

## 因果掩码

解决这个问题的办法是采用**因果掩码（Causal Mask）**。把每个位置可见的范围限制在自身及之前。匹配分数矩阵的右上三角被置为 $-\infty$，对角线与下三角保持不变；第 $i$ 行只有第 $j\le i$ 列参与 Softmax。掩码矩阵的边长等于序列长度。这样，即使一次读入完整序列，每个位置也只能根据自身及之前的前缀预测下一个 Token，与自回归推理语义一致。训练阶段一次并行读入完整目标序列，以及 Decoder-Only 模型推理的预填充（Prefill）阶段并行处理整段提示词时，都必须施加因果掩码防止靠前位置读到靠后位置；仅在启用 KV Cache 后的单步生成（Decode）阶段，查询位于序列末尾且只读取已缓存历史，因果掩码才退化为全放行。

在匹配分数上加入掩码矩阵 $\mathbf{M}$：允许关注的位置取 0，需要遮挡的位置取 $-\infty$。经过 Softmax，被遮挡位置的权重变为 0：

$$
\operatorname{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V})
=
\operatorname{softmax}\left(\frac{\mathbf{QK^{\top}}}{\sqrt{d_k}}+\mathbf{M}\right)\mathbf{V}
$$

[html-card height=560](../assets/attention-causal-mask-slides.html)

# 交叉注意力

掩码自注意力之后，解码器已经形成当前输出序列的上下文表示，需要据此回到编码器输出中读取源序列信息。

设解码器的当前序列表示为 $\mathbf{Z}$，编码器的最终输出为 $\mathbf{H}$，$\mathbf{W}_Q,\mathbf{W}_K,\mathbf{W}_V$ 分别为查询、键与值的可学习投影矩阵，则交叉注意力由 $\mathbf{Z}$ 生成查询 $\mathbf{Q}$，由 $\mathbf{H}$ 生成键 $\mathbf{K}$ 与值 $\mathbf{V}$：

$$
\mathbf{Q}=\mathbf{Z}\mathbf{W}_Q,\qquad
\mathbf{K}=\mathbf{H}\mathbf{W}_K,\qquad
\mathbf{V}=\mathbf{H}\mathbf{W}_V
$$

由于 $\mathbf{H}$ 只在编码阶段计算一次，解码每一步均可复用这组源序列表示；逐步解码的交互演示见 [[Decoder-Workflow#交叉注意力|解码器逐步生成流程]]。

# KV Cache

KV Cache 仅用于推理阶段，训练时并不使用。推理采用自回归生成：每生成一个新的 Token，都将其追加到输出序列末尾并执行新一轮注意力。若每步都重算历史 Token 的键与值，计算量会随生成长度重复累积。

KV Cache 将每一层注意力已经算出的历史键与值缓存在显存中，生成后续 Token 时直接复用；其代价是显存占用随序列长度、层数与 KV 头数增长。现代大模型常以 [[Multi-Head-Attention#MQA、GQA 与 MLA（KV Cache 的结构优化）|GQA 与 MLA]] 削减 KV 缓存规模，并在推理引擎 [[vLLM]] 中结合分页管理复用显存。

两类解码注意力的缓存策略不同：交叉注意力的键与值完全来自固定的编码器输出 $\mathbf{H}$，解码全程只需预计算并缓存一次；掩码自注意力的键与值随已生成序列增长，每步只需计算新位置的查询、键与值，再把新键与新值追加到缓存末尾，无需重算历史位置。

> [!question] 为什么查询不缓存
> 第 $t$ 个位置只产生一个查询 $\boldsymbol{q}_t$，它只用于本次查询。生成下一个 Token 时会产生新的查询，过去的查询不会再次使用。键与值则作为历史信息被后续位置持续读取。

KV Cache 发生在一次生成过程内部。Prompt Cache 跨请求复用相同 Prompt 前缀的计算结果，减少相同前缀被反复处理的时间与计算成本。两者依赖同一个事实：每个位置的中间状态都依赖它之前的内容。前缀越稳定，可以复用的范围越大。因此在 Agent 开发中，稳定的系统提示词、工具定义与公共上下文通常放在 Prompt 前部，经常变化的用户输入放在后部。

# FlashAttention：硬件感知精确注意力

标准自注意力需将完整 $N\times N$ 的分数矩阵与权重矩阵反复写入并读出容量大但较慢的 GPU 主显存（HBM），长序列下形成 $O(N^2)$ 显存占用与访存瓶颈。**FlashAttention** 保持**数学计算结果完全精确等价**，从两方面重构算子：

1. **分块与 SRAM 融合**：把 $\mathbf{Q}$、$\mathbf{K}$、$\mathbf{V}$ 分块载入速度极快的片上高速缓存（SRAM），在片内融合完成点积、掩码、Softmax 与乘 $\mathbf{V}$，不在主显存实例化 $N\times N$ 中间矩阵，只写回最终输出。
2. **Online Softmax 动态重缩放**：逐块扫描 $\mathbf{K},\mathbf{V}$ 时动态跟踪每行当前最大值 $m$ 与分母和 $l$；依据恒等式 $\exp(x - m_{\text{new}})=\exp(x - m_{\text{old}})\cdot\exp(m_{\text{old}} - m_{\text{new}})$，每当遇到更大的局部最大值 $m_{\text{new}}$，将此前累计的分子与分母和统一乘以 $\exp(m_{\text{old}} - m_{\text{new}})$ 即可无损重缩放，扫完全部块后得到与标准 Softmax 完全一致的结果。

由此将注意力额外显存从 $O(N^2)$ 降至 $O(N)$，大幅减少主显存读写并加速长序列计算。
