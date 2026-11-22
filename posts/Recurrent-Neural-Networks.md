---
title: Recurrent Neural Networks
tags:
  - NLP
categories:
  - Artificial Intelligence
date: 2026-10-27 23:29:21
katex: true
---

# 定长输入与序列依赖

用前馈网络处理语言时，只能把固定窗口内的词向量拼接为定长输入：窗口之外的前文无法访问，而扩大窗口会令输入维度与参数量线性膨胀。**循环神经网络（RNN）** 用隐藏状态概括已读历史，每读入一个词元就用当前输入与上一状态算出新状态，所有时间步共享同一套权重，从而能够逐步处理任意长度的序列。

# 循环结构与形状

设序列长度为 $T$，$D$ 为词元嵌入维数，$H$ 为隐藏状态维数。第 $t$ 个词元的嵌入向量为 $\boldsymbol{x}_t \in \mathbb{R}^D$，第 $t$ 步的隐藏状态为 $\boldsymbol{h}_t \in \mathbb{R}^H$（初值 $\boldsymbol{h}_0$ 常取零向量），输入权重 $\mathbf{W}_x \in \mathbb{R}^{H\times D}$、状态转移权重 $\mathbf{W}_h \in \mathbb{R}^{H\times H}$ 与偏置 $\boldsymbol{b} \in \mathbb{R}^H$ 在全体时间步共享，$\tanh$ 逐元素作用于预激活：

$$
\boldsymbol{h}_t = \tanh\left(\mathbf{W}_x \boldsymbol{x}_t + \mathbf{W}_h \boldsymbol{h}_{t-1} + \boldsymbol{b}\right)
$$

> [!note] 单向与双向结构
> 单向 $\boldsymbol{h}_t$ 只汇总到位置 $t$ 为止的前缀。当任务允许预先读入完整输入（如句子分类或序列标注）时，可采用双向结构分别沿正反方向各算一遍状态并拼接输出（这也是编码器 / BERT 路线的基础）；**自回归生成**因生成第 $t$ 步时尚不存在右侧词元，只能采用单向递推（对应解码器 / GPT 路线）。

# 三类任务的读数位置

RNN 按同一套递推公式更新隐藏状态，下游仅因任务头读取的位置不同而区分：语言模型在训练时把输入序列与目标错开一位以逐位置预测下一词（下表中 logits 指未经 Softmax 归一化的类别分数，$B$ 为批大小，$C$ 为分类类别数，$V$ 为词表大小）。

| 任务 | 任务头读数位置 | logits 形状 | 预测目标 |
| --- | --- | --- | --- |
| 整段分类 | 末位置状态 $\boldsymbol{h}_T$ | $(B, C)$ | 每段序列一个类别 |
| 序列标注 | 每个位置的 $\boldsymbol{h}_t$ | $(B, T, C)$ | 每个位置一个标签 |
| 语言模型 | 每个位置的 $\boldsymbol{h}_t$ | $(B, T, V)$ | 下一个词元分布 |

[html-card height=780](../assets/rnn-unfolded.html)

# 时间反向传播与长序列限制

## 梯度传播

**时间反向传播（BPTT）** 要求将末尾损失沿时间链逆向传回早先步骤。以标量简化递推 $h_t = w\,h_{t-1} + x_t$ 为例，回溯 $k$ 步须连乘 $k$ 次因子 $w$（如 $w=0.5$ 五步缩至约 $0.03$，$w=1.6$ 五步放大至约 $10.5$）。

[html-card height=820](../assets/rnn-bptt-gradient.html)

推广到向量状态 $\boldsymbol{h}_t \in \mathbb{R}^H$，令 $\operatorname{diag}(1 - \boldsymbol{h}_\tau^2)$ 表示第 $\tau$ 步 $\tanh$ 的局部导数对角矩阵，沿时间回传 $k$ 步产生雅可比矩阵连乘：

$$
\frac{\partial \boldsymbol{h}_t}{\partial \boldsymbol{h}_{t-k}} = \prod_{\tau = t-k+1}^{t} \operatorname{diag}\left(1 - \boldsymbol{h}_\tau^2\right)\mathbf{W}_h
$$

每回溯一步既乘以不超过 $1$ 的 $\tanh'$（激活饱和时趋近 $0$），又重复右乘同一转移矩阵 $\mathbf{W}_h$。当 $\mathbf{W}_h$ 最大奇异值大于 $1$ 时容易发生**梯度爆炸**，可利用 [[Backpropagation-Chain-Rule#梯度裁剪与数值稳定|梯度裁剪]] 抑制尖峰；当连乘因子持续小于 $1$ 时发生**梯度消失**，无法靠裁剪恢复，必须通过 [[LSTM-And-GRU|门控加法通道]] 或 [[Attention|自注意力直连]] 重构传递路径。

## 走向 Transformer 的两大动因

1. **时间维严格串行**：计算 $\boldsymbol{h}_t$ 必须等待 $\boldsymbol{h}_{t-1}$ 完成，同一条序列内无法跨位置并行，限制了大规模语料训练吞吐量。
2. **固定维度状态瓶颈**：无论序列多长，全部历史信息都必须不断被新输入重写进同一维度的 $\boldsymbol{h}_t$ 中，远距离细节极易在递推中被覆盖丢失。
