---
title: Encoder Decoder And Seq2Seq
tags:
  - NLP
categories:
  - Artificial Intelligence
date: 2026-10-27 23:29:21
katex: true
---

# 编码器与解码器

普通分类只需末状态输出单标签，序列标注则在每个输入位置对齐输出标签；而机器翻译、文本摘要等任务的目标句长度与词序均独立于源句，称为**序列到序列（Sequence-to-Sequence, Seq2Seq）**任务，需要一条独立的生成时间轴逐步决定每个目标词元与终止时机。

经典 Seq2Seq 架构拆为串联的两部分：记源句长度为 $T$，**编码器**读完整条源序列后将末状态 $\boldsymbol{h}_T^{\text{enc}}$ 作为定长**上下文向量 $\boldsymbol{c}$** 交给**解码器**。解码器将初始状态置为 $\boldsymbol{s}_0 = \boldsymbol{c}$，每步结合上一步目标词元 $y_{t-1}$ 与上一步状态 $\boldsymbol{s}_{t-1}$ 经循环单元 $f$ 更新当前状态 $\boldsymbol{s}_t$，再由输出矩阵 $\mathbf{W}$ 映射为词表得分经 Softmax 得到概率分布 $\boldsymbol{p}_t$：

$$
\boldsymbol{c} = \boldsymbol{h}_T^{\text{enc}}, \qquad \boldsymbol{s}_0 = \boldsymbol{c}, \qquad \boldsymbol{s}_t = f(y_{t-1}, \boldsymbol{s}_{t-1}), \qquad \boldsymbol{p}_t = \operatorname{softmax}(\mathbf{W} \boldsymbol{s}_t)
$$

[html-card height=820](../assets/seq2seq-topology.html)

# 生成与训练

## 生成：从 BOS 到 EOS

自回归推理从起始标记 `<bos>` 启动，至生成结束标记 `<eos>` 或触达最大长度上限为止：

1. 编码器读入完整源句计算上下文向量 $\boldsymbol{c}$，用于初始化解码器状态 $\boldsymbol{s}_0$。
2. 以 `<bos>` 作为首步输入算出 $\boldsymbol{p}_1$，从中选出第 1 个目标词元 $y_1$。
3. 将 $y_1$ 送入下一步输入更新状态得出 $y_2$，如此逐词滚动直至输出 `<eos>`。

解码每步可选取概率最大的**贪心策略**，或使用保留多条高分候选路径的**束搜索（Beam Search）**以减少局部误选，开放式文本生成则常采用带随机性的 [[Decoder-Workflow#输出采样|采样策略]]。

## 训练：Teacher Forcing 与暴露偏差

训练时完整目标句已知。**教师强制（Teacher Forcing）** 将目标句整体错开一位构造解码器每步输入，无论模型上一位置真实预测如何，下一步一律送入真实标注前缀，并对有效位置计算逐词交叉熵损失；以下以目标句“学生 喜欢 深度 学习”为例展示输入与监督标签对齐关系：

| 解码位置 | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | --- |
| 解码器输入 | `<bos>` | 学生 | 喜欢 | 深度 | 学习 |
| 监督标签 | 学生 | 喜欢 | 深度 | 学习 | `<eos>` |

[html-card height=820](../assets/seq2seq-teacher-forcing.html)

由于训练期输入全部来自真实标注、而推理期输入来自模型自己上一步生成的预测（某步选错会沿后续前缀滚雪球传播），两者输入分布存在落差，这一现象称为**暴露偏差（Exposure Bias）**。

> [!note] PAD 与损失屏蔽
> 批量训练时较短序列需用占位符 `<pad>` 补齐至相同长度。计算交叉熵损失时必须通过掩码屏蔽 `<pad>` 位置，避免模型去拟合无效填充符号。

# 固定上下文模型的局限

经典循环 Seq2Seq 存在两项制约大规模语言建模的结构瓶颈：

1. **串行计算依赖**：编码器与解码器的隐藏状态 $\boldsymbol{s}_t$ 严格依赖前一步状态，即便训练时采用教师强制一次给出整句目标前缀，时间维循环依然无法并行。
2. **单一向量信息瓶颈**：无论源句多长、信息密度多高，编码器只能将全部内容强行压进固定维度的单一向量 $\boldsymbol{c}$，源句越长细节损失越严重。

解决思路是彻底放弃“只交出一个定长向量”：保留编码器每个位置的表示状态，允许解码器在每一步生成时直接回溯全部源位置按需动态加权读取，由此诞生 [[Attention|注意力机制]]，并最终演进为全序列并行的 [[Transformer-Architecture|Transformer]]。
