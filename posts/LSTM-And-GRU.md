---
title: LSTM And GRU
tags:
  - NLP
categories:
  - Artificial Intelligence
date: 2026-10-27 23:29:21
katex: true
---

# 门控要解决的问题

普通循环网络每读入一个词元就重写一次全部隐藏状态，梯度在 [[Recurrent-Neural-Networks#梯度传播|时间反向传播]] 时不得不连续乘以饱和 $\tanh'$ 与循环权重矩阵 $\mathbf{W}_h$，导致长距离梯度迅速衰减。**LSTM** 与 **GRU** 没有取消时间步递推，而是通过可学习的**门控机制**构造加法更新通道，使旧记忆与梯度得以近似无损地跨时间传递。

# LSTM：细胞状态与加法直接路径

**长短期记忆网络（LSTM）** 将状态拆为两条 $H$ 维通道：$\boldsymbol{c}_t \in \mathbb{R}^H$ 为专职跨时间保存长期记忆的**细胞状态**，$\boldsymbol{h}_t \in \mathbb{R}^H$ 为向任务头与下一步门控提供的**隐藏输出**。设 $D$ 为输入嵌入维数，第 $t$ 步输入为 $\boldsymbol{x}_t \in \mathbb{R}^D$，将 $[\boldsymbol{h}_{t-1}; \boldsymbol{x}_t] \in \mathbb{R}^{H+D}$ 送入四组线性投影（权重 $\mathbf{W}_f, \mathbf{W}_i, \mathbf{W}_c, \mathbf{W}_o \in \mathbb{R}^{H\times(H+D)}$，偏置 $\boldsymbol{b}_f, \boldsymbol{b}_i, \boldsymbol{b}_c, \boldsymbol{b}_o \in \mathbb{R}^H$），以逐元素 Sigmoid 函数 $\sigma$ 产生连续比例门控，由 $\tanh$ 生成候选写入向量 $\tilde{\boldsymbol{c}}_t$（记 $\odot$ 为逐元素乘法）：

$$
\begin{aligned}
\boldsymbol{f}_t &= \sigma\left(\mathbf{W}_f [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_f\right), &
\boldsymbol{i}_t &= \sigma\left(\mathbf{W}_i [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_i\right), \\
\tilde{\boldsymbol{c}}_t &= \tanh\left(\mathbf{W}_c [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_c\right), &
\boldsymbol{o}_t &= \sigma\left(\mathbf{W}_o [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_o\right)
\end{aligned}
$$

随后通过加法更新细胞状态，再按输出门缩放出隐藏输出：

$$
\boldsymbol{c}_t = \boldsymbol{f}_t \odot \boldsymbol{c}_{t-1} + \boldsymbol{i}_t \odot \tilde{\boldsymbol{c}}_t, \qquad \boldsymbol{h}_t = \boldsymbol{o}_t \odot \tanh(\boldsymbol{c}_t)
$$

其中**遗忘门 $\boldsymbol{f}_t$** 决定保留多少旧记忆 $\boldsymbol{c}_{t-1}$，**输入门 $\boldsymbol{i}_t$** 控制写入多少候选信息 $\tilde{\boldsymbol{c}}_t$，**输出门 $\boldsymbol{o}_t$** 控制向外部读出多少当前记忆；注意 $\boldsymbol{c}_t$ 本身仅经历加法更新而不被 $\tanh$ 反复压扁。

## 直接路径

考察沿细胞状态主通道传播的主导梯度路径（暂固定各门输出与 $\boldsymbol{h}_{t-1}$）：由于加法更新第一项仅乘逐元素遗忘门 $\boldsymbol{f}_t$，相邻两步主通道偏导简化为：

$$
\frac{\partial \boldsymbol{c}_t}{\partial \boldsymbol{c}_{t-1}} = \operatorname{diag}(\boldsymbol{f}_t)
$$

沿时间回溯 $k$ 步得到 $\prod_{\tau=t-k+1}^{t}\operatorname{diag}(\boldsymbol{f}_\tau)$。与普通 RNN 每步硬乘固定权重 $\mathbf{W}_h$ 不同，遗忘门 $\boldsymbol{f}_\tau \in (0, 1)$ 由上下文动态决定：需要长期保留记忆的维度只需把 $\boldsymbol{f}_\tau$ 学到接近 $1$，即可形成近似恒等直通通道（实践中常将遗忘门偏置 $\boldsymbol{b}_f$ 初始化为正值以防止训练初期遗忘过多）。这种加法直通思想与后续深层网络中的残差连接一脉相承。

[html-card height=820](../assets/lstm-direct-path.html)

# GRU 与两种单元对照

**门控循环单元（GRU）** 将细胞状态与隐藏输出合二为一（仅保留 $\boldsymbol{h}_t$），并把三个门精简为两个：

- **更新门 $\boldsymbol{z}_t \in (0, 1)^H$**：用互补权重在旧状态与新候选之间做线性插值，即 $\boldsymbol{h}_t = \boldsymbol{z}_t \odot \boldsymbol{h}_{t-1} + (1 - \boldsymbol{z}_t) \odot \boldsymbol{n}_t$（$\boldsymbol{n}_t$ 为候选新状态）。
- **重置门 $\boldsymbol{r}_t \in (0, 1)^H$**：控制计算候选 $\boldsymbol{n}_t$ 时参考多少上一时刻状态 $\boldsymbol{h}_{t-1}$。

下表汇总两种门控单元的对应关系：注意二者仅为功能近似而非逐门一一等价（LSTM 的遗忘与输入独立开关可同时趋近 1，而 GRU 由互补系数分配新旧比例），下方卡片演示同一步内的状态更新流转：

| 对比维度 | LSTM | GRU |
| --- | --- | --- |
| 状态向量 | $(\boldsymbol{h}_t, \boldsymbol{c}_t)$ 两套状态 | $\boldsymbol{h}_t$ 单一状态 |
| 门控构成 | 遗忘门、输入门、输出门（独立控制） | 更新门、重置门（互补插值与重置） |
| 状态更新方式 | 保留项与写入项直接相加 | 旧状态与新候选加权插值 |
| 每步线性投影组数 | 4 组 | 3 组（参数更省） |

[html-card height=580](../assets/lstm-gru-update.html)

# 门控仍未解决的瓶颈与后续演进

门控单元虽然缓解了长距离梯度消失，但未打破循环结构本身的两道限制：时间维依然严格串行（计算 $\boldsymbol{h}_t$ 仍须等待 $\boldsymbol{h}_{t-1}$，无法并行训练长文本），且单步隐藏状态维度有限，长程记忆依然面临容量竞争。这一局限促使 NLP 架构转向全序列两两直连、可高度并行的 [[Attention|自注意力]] 与 [[Transformer-Architecture|Transformer]]。

在最新长序列架构演化中，**状态空间模型（SSM，如 Mamba）** 保留了固定维度状态的递推骨架，但移除了状态更新主链上的非线性激活使递推满足线性结合律，从而兼具训练阶段的并行扫描能力与推理阶段 $O(1)$ 恒定状态显存开销。
