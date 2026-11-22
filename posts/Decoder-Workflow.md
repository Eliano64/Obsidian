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

**解码器（Decoder）** 从起始符 `<BOS>` 出发自回归预测词元，每生成一词便追加到前缀末尾预测下一词，直到输出 `<EOS>` 或触及长度上限。记编码器输出的源序列表示矩阵为 $\mathbf{H} \in \mathbb{R}^{S\times d_{\text{model}}}$（$S$ 为源序列长度，$d_{\text{model}}$ 为隐藏维数，详见 [[Encoder-Workflow]]）。

原始 Encoder-Decoder 解码器层依次包含**掩码自注意力**、读取 $\mathbf{H}$ 的**交叉注意力**与 **FFN**（现代大模型普遍采用 Pre-RMSNorm 排布，见 [[Encoder-Workflow#Pre-Norm|Pre-Norm]]）；现代 Decoder-Only 大语言模型则省略编码器与交叉注意力，将提示词与生成回复统一视作因果序列。

# 推理

设当前处于第 $t$ 个解码步（已生成前缀长 $t$）。未开启 KV Cache 时，每步须把全部前缀矩阵 $\mathbf{X}_D \in \mathbb{R}^{t\times d_{\text{model}}}$ 重新算一遍；开启 [[Attention-In-Transformer#KV Cache|KV Cache]] 复用历史键值对后，第 $t$ 步仅输入最新词元行向量 $\boldsymbol{x}_t \in \mathbb{R}^{1\times d_{\text{model}}}$。

[html-card height=680](../assets/attention-decoder-steps.html)

推理每步仅提取末层**最新一行**输出用于预测下一词，因为注意力已通过加权求和将整段前缀上下文写入该位置表示中。

## 掩码自注意力

生成第 $t$ 步时不能窥探未来词元；训练期一次读入整条目标序列以及 Decoder-Only 推理的预填充（Prefill）阶段并行处理整段提示词时，必须施加 [[Attention-In-Transformer#掩码自注意力|因果掩码]] 将右上三角（$j > i$）打分置为 $-\infty$。仅在启用 KV Cache 的单步 Decode 阶段，当前查询位于序列末端且只读已缓存历史，因果掩码退化为全放行。

## 交叉注意力

经掩码自注意力得到当前输出前缀表示 $\mathbf{Z}_D \in \mathbb{R}^{t\times d_{\text{model}}}$（训练取目标全长 $T$；Decode 开启 KV Cache 时仅取末行 $\boldsymbol{z}_t \in \mathbb{R}^{1\times d_{\text{model}}}$），交叉注意力以它为查询、以编码器输出 $\mathbf{H}$ 为键和值（$\mathbf{W}_Q,\mathbf{W}_K,\mathbf{W}_V$ 为可学习投影矩阵）：

$$
\mathbf{Q}=\mathbf{Z}_D\mathbf{W}_Q,\qquad
\mathbf{K}=\mathbf{H}\mathbf{W}_K,\qquad
\mathbf{V}=\mathbf{H}\mathbf{W}_V
$$

由于 $\mathbf{H}$ 行数为 $S$，训练期交叉注意力权重矩阵为 $T\times S$，单步推理权重为 $1\times S$（行向量）。

## 输出分布

推理时线性分类头仅取解码器末层最后一个位置的行向量 $\boldsymbol{h}_t^{\text{dec}}\in\mathbb{R}^{1\times d_{\text{model}}}$（训练阶段则对 $T\times d_{\text{model}}$ 整矩阵一次右乘并行算出所有步 logits），经词表映射矩阵 $\mathbf{W}_{\text{vocab}}\in\mathbb{R}^{d_{\text{model}}\times |\mathcal{V}|}$（$|\mathcal{V}|$ 为词表大小）得到未归一化得分向量 $\boldsymbol{l}_t\in\mathbb{R}^{1\times|\mathcal{V}|}$：

$$
\boldsymbol{l}_t = \boldsymbol{h}_t^{\text{dec}}\mathbf{W}_{\text{vocab}}, \qquad
p_i = \frac{\exp(l_i/\tau)}{\sum_{j=1}^{|\mathcal{V}|}\exp(l_j/\tau)}
$$

其中 $l_i$ 为 $\boldsymbol{l}_t$ 第 $i$ 个词元原始得分，**温度 $\tau > 0$** 在 Softmax 前缩放 logits 得到分布概率 $p_i$：调高 $\tau$ 使分布平缓增加随机探索，调低 $\tau$ 使高分项更集中。

> [!example] 温度极限 $\tau \to 0^+$
> 当 $\tau\to 0^+$ 时，最大得分项 $i^\ast = \arg\max_j l_j$ 在指数中占据绝对优势，分布收敛为在 $i^\ast$ 上取 $1$ 其余为 $0$ 的 one-hot 分布，等价于[[#贪心采样]]。

[html-card height=620](../assets/decoder-distribution-slides.html)

## 输出采样

先按温度 $\tau$ 调整整体概率锐度，再选定采样规则确定候选范围与随机性：

![三种采样策略|630](../assets/decoder-sampling-diagram.svg)

### 贪心采样

每步直接选取概率最大词元。结果完全确定但无法回溯，早步错选可能导致后续陷入次优或重复循环；翻译等确定性任务常改用保留多分支的束搜索（见 [[Encoder-Decoder-And-Seq2Seq#生成：从 BOS 到 EOS|束搜索]]）。

### Top-K 采样

从词表按概率从高到低截断保留固定前 $K$ 个词构成候选集 $\mathcal{C}$，按 $q_i = p_i / \sum_{j\in\mathcal{C}} p_j$（$i\in\mathcal{C}$）重归一化后随机抽样；缺点是分布极尖锐时仍会强行纳入低概率词。

### Top-P 采样

按概率从高到低累加，取累计概率刚达到阈值 $P$ 的最小集合作为 $\mathcal{C}$ 后依重归一化概率 $q_i$ 随机抽样，候选个数随上下文不确定度动态增减。

# 训练

训练阶段完整目标序列已知，因此无需像推理那样逐词等待。Transformer 通过错开一位输入一次并行算出全句 $T$ 个位置的预测与交叉熵损失（消除序列内部循环依赖，但自回归推理本身的先后顺序不变）。

![训练与推理的差别|630](../assets/decoder-training-vs-inference.svg)

## Teacher Forcing

设纯目标序列为 $y_1,\dots,y_T$：解码器输入在头部拼接 `<BOS>`、目标标签在尾部拼接 `<EOS>`（即完整序列分别错位去尾与去头）。配合 [[Attention-In-Transformer#掩码自注意力|因果掩码]] 保证第 $i$ 位置只能读取自身及之前真实前缀预测第 $i+1$ 词，所有时间步可在单次前向中并行训练并避免早期错误滚雪球累积。

## 批量与填充

处理批次大小 $B$ 的不等长样本时需用 `<PAD>` 补齐，并将补位位置掩码设 $0$（在进入 Softmax 前转为 $-\infty$），与因果掩码叠加使补位权重与损失贡献归零。下表列出单样本与批量张量形状对照（$S$ 为源长、$T$ 为目标长、$|\mathcal{V}|$ 为词表大小）：

| 张量对象 | 单个样本形状 | 批量形状（$B$ 样本） |
| --- | --- | --- |
| 编码器序列表示 | $S\times d_{\text{model}}$ | $B\times S\times d_{\text{model}}$ |
| 解码器序列表示 | $T\times d_{\text{model}}$ | $B\times T\times d_{\text{model}}$ |
| 掩码自注意力权重 | $T\times T$ | $B\times T\times T$ |
| 交叉注意力权重 | $T\times S$ | $B\times T\times S$ |
| 词表线性层输出 | $T\times\lvert\mathcal{V}\rvert$ | $B\times T\times\lvert\mathcal{V}\rvert$ |
