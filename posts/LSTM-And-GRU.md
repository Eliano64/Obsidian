---
title: LSTM And GRU
tags:
  - NLP
categories:
  - Artificial Intelligence
date: 2026-10-27 23:29:21
katex: true
---

# 时间链上的信息与梯度

循环网络每读一个词就重写一次状态，早期信息要到达后面的位置，必须经过多次矩阵乘法与非线性。反向传播沿同一条链回退，每回退一步乘一个因子；回退 $k$ 步就是 $k$ 个因子相乘，长期小于 1 时梯度趋近 0。

LSTM 与 GRU 没有消除时间链，改的是状态更新的路径：让旧信息沿一条不经过复杂变换的直接通道传递。

# LSTM：分开存储与输出

## 门与候选

设序列长 $T$，$t$ 是当前步，$\boldsymbol{x}_t\in\mathbb{R}^{D}$ 是第 $t$ 个 token 的嵌入向量，$D$ 是嵌入维度；$\boldsymbol{h}_t\in\mathbb{R}^{H}$ 与 $\boldsymbol{c}_t\in\mathbb{R}^{H}$ 是成对的状态，$H$ 是隐藏维度。把上一步状态与当前输入拼成 $[\boldsymbol{h}_{t-1};\boldsymbol{x}_t]\in\mathbb{R}^{H+D}$，四个线性变换各给出一个 $H$ 维向量。$\sigma$ 指 sigmoid，$\sigma(v)=1/(1+e^{-v})$，逐元素作用：

$$
\begin{aligned}
\boldsymbol{f}_t &= \sigma(\mathbf{W}_f [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_f)\\
\boldsymbol{i}_t &= \sigma(\mathbf{W}_i [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_i)\\
\tilde{\boldsymbol{c}}_t &= \tanh(\mathbf{W}_c [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_c)\\
\boldsymbol{o}_t &= \sigma(\mathbf{W}_o [\boldsymbol{h}_{t-1};\boldsymbol{x}_t] + \boldsymbol{b}_o)
\end{aligned}
$$

状态更新与读出为：

$$
\boldsymbol{c}_t = \boldsymbol{f}_t \odot \boldsymbol{c}_{t-1} + \boldsymbol{i}_t \odot \tilde{\boldsymbol{c}}_t,\qquad \boldsymbol{h}_t = \boldsymbol{o}_t \odot \tanh(\boldsymbol{c}_t)
$$

$\mathbf{W}$ 的形状都是 $H\times(H+D)$，偏置是 $H$ 维向量 $\boldsymbol{b}$，$\odot$ 是逐元素乘法。$\boldsymbol{c}_t$ 与 $\boldsymbol{h}_t$ 是成对的状态：$\boldsymbol{c}$ 负责跨时间保存记忆，$\boldsymbol{h}$ 是这一步交给任务与下一步门计算的表示；两者都是 $H$ 维，不是把两个向量相加。

$\boldsymbol{f}_t$ 是遗忘门，控制旧记忆 $\boldsymbol{c}_{t-1}$ 保留多少；$\boldsymbol{i}_t$ 是输入门，控制写入多少候选；$\boldsymbol{o}_t$ 是输出门，控制从 $\boldsymbol{c}_t$ 读出多少。候选 $\tilde{\boldsymbol{c}}_t$ 由 tanh 生成，取值在 $-1$ 到 1 之间，可正可负。

三个门都经 sigmoid 落在 0 到 1 之间，是连续比例而不是开关；$\boldsymbol{c}_t$ 只经加法更新，不受 tanh 限制，$\boldsymbol{h}_t$ 才是 $\boldsymbol{c}_t$ 经 tanh 后再由输出门缩放的结果。

[html-card height=820](../assets/lstm-cell-slides.html)

## 直接路径

对 $\boldsymbol{c}_{t-1}$ 求导，固定门值与 $\boldsymbol{h}_{t-1}$，只有第一项直接依赖 $\boldsymbol{c}_{t-1}$：

$$
\frac{\partial \boldsymbol{c}_t}{\partial \boldsymbol{c}_{t-1}} = \operatorname{diag}(\boldsymbol{f}_t)
$$

沿时间回退 $k$ 步，这条路径的因子是逐维门值连乘。$\boldsymbol{f}$ 接近 1 时，直接路径接近恒等映射，早期状态的信息与梯度更容易穿过；门值小，连乘仍然衰减。完整梯度还包括绕经各门与隐藏状态的路径，直接路径是主要通道，不是全部梯度。

[html-card height=820](../assets/lstm-direct-path.html)

# GRU：两个门与一份状态

GRU 不再单独维护细胞状态，跨时间信息与对外表示都放在 $\boldsymbol{h}_t$ 中。PyTorch 的实现为：

$$
\begin{aligned}
\boldsymbol{z}_t &= \sigma(\mathbf{W}_{iz}\boldsymbol{x}_t + \boldsymbol{b}_{iz} + \mathbf{W}_{hz}\boldsymbol{h}_{t-1} + \boldsymbol{b}_{hz})\\
\boldsymbol{r}_t &= \sigma(\mathbf{W}_{ir}\boldsymbol{x}_t + \boldsymbol{b}_{ir} + \mathbf{W}_{hr}\boldsymbol{h}_{t-1} + \boldsymbol{b}_{hr})\\
\boldsymbol{n}_t &= \tanh\big(\mathbf{W}_{in}\boldsymbol{x}_t + \boldsymbol{b}_{in} + \boldsymbol{r}_t\odot(\mathbf{W}_{hn}\boldsymbol{h}_{t-1} + \boldsymbol{b}_{hn})\big)\\
\boldsymbol{h}_t &= (1-\boldsymbol{z}_t)\odot \boldsymbol{n}_t + \boldsymbol{z}_t\odot \boldsymbol{h}_{t-1}
\end{aligned}
$$

$\mathbf{W}_{iz},\mathbf{W}_{ir},\mathbf{W}_{in}$ 为 $H\times D$，作用于输入；$\mathbf{W}_{hz},\mathbf{W}_{hr},\mathbf{W}_{hn}$ 为 $H\times H$，作用于上一步状态。$\boldsymbol{n}_t$ 是候选新状态，$\boldsymbol{r}_t$ 缩放旧状态在候选中的参与，$\boldsymbol{z}_t$ 是保留旧状态的比例，$1-\boldsymbol{z}_t$ 分配给新候选。

[html-card height=820](../assets/gru-cell-slides.html)

> [!note] 另一种书写顺序
> 有些实现先把 $\boldsymbol{r}_t$ 与 $\boldsymbol{h}_{t-1}$ 逐元素相乘，再与 $\boldsymbol{x}_t$ 拼接后一起投影。两种写法数学上不等价，读别的实现时需要确认。

# 两种单元的对照

| 项目 | LSTM | GRU |
| --- | --- | --- |
| 状态 | $(\boldsymbol{h}_t,\boldsymbol{c}_t)$ 一对 | $\boldsymbol{h}_t$ 一份 |
| 门 | 遗忘、输入、输出 | 更新、重置 |
| 更新方式 | 保留与写入相加 | 旧状态与新候选插值 |
| 每步线性变换 | 4 组 | 3 组 |

对应关系是功能上的，不是逐个门一一对等。LSTM 的遗忘门与输入门是两个独立系数，可以同时接近 1；GRU 的更新门用 $\boldsymbol{z}_t$ 与 $1-\boldsymbol{z}_t$ 两个互补系数分配新旧状态。GRU 的重置门只影响候选的生成方式，保留比例由更新门负责。

[html-card height=580](../assets/lstm-gru-update.html)


> [!note] 串行与固定状态
> 时间维仍然串行：$\boldsymbol{h}_t$ 依赖 $\boldsymbol{h}_{t-1}$，同一序列无法一次算出所有位置。状态维度固定：序列越长，同样的维度要概括的内容越多。注意力改变的是可读取的编码位置，循环的时间递推仍在。
