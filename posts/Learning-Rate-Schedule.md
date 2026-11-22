---
title: Learning Rate Schedule
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-11-22 22:09:57
katex: true
---

# 为什么需要学习率调度

**学习率调度（Learning Rate Schedule）** 将原本恒定的全局学习率替换为随训练步数 $t$ 动态变化的函数 $\eta_t$：初期参数随机须稳步热身，中期放大步长加速穿越平缓坡面，后期收小步长在谷底精细收敛。下表展示了在两层 MLP（`784→128→10`）训练 5 个 epoch 时改变固定学习率的典型现象：

| 固定学习率 | 训练现象 | 5 epoch 后损失 | 5 epoch 后准确率 |
| --- | --- | --- | --- |
| $10$ | 首步发散飙升并卡死，未能收敛 | $2.34$ | 约 10% |
| $1.0$ | 损失可下降，但跨过谷底剧烈震荡 | $1.07$ | 约 59% |
| $0.1$ | 平稳快速下降，处于最优区间 | $0.07$ | 约 97% |
| $10^{-5}$ | 单步位移微乎其微，几乎停滞 | $2.31$ | 约 15% |

## 调度与自适应优化器

即使使用 [[Gradient-Descent#Adam 与 AdamW（双旋钮合体与解耦）|Adam / AdamW]]，二阶矩分母调整的也只是**各参数间的相对步长比例**，全局乘子 $\eta_t$（设第 $t$ 步参数为 $\boldsymbol{\theta}_t$，偏差修正矩为 $\hat{\boldsymbol{m}}_t, \hat{\boldsymbol{s}}_t$，$\delta$ 为防除零小常数）仍完全受外部调度控制：

$$
\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \eta_t \cdot \frac{\hat{\boldsymbol{m}}_t}{\sqrt{\hat{\boldsymbol{s}}_t} + \delta}
$$

# 线性预热（Warmup）

在训练最初若干步（尤其是 Adam / AdamW 初期），二阶矩滑动平均样本数极少、分母开方后方差极大，直接采用峰值步长极易引起步长过冲发散。**线性预热**让学习率在前 $T_{\text{warmup}}$ 步从接近 0 线性升温至峰值 $\eta_{\max}$：

$$
\eta_t = \eta_{\max} \cdot \frac{t}{T_{\text{warmup}}}, \qquad t \le T_{\text{warmup}}
$$

下表对比四层 MLP 配 Adam（$\eta=0.02$）训练 MNIST 时加入 200 步线性预热的稳定效果：

| 设置 | 前 50 batch 最大损失 | 5 epoch 测试准确率 |
| --- | --- | --- |
| 直接采用 $0.02$ | $17.73$ | 91.75% |
| 加入 200 步线性预热 | $2.30$ | 94.74% |

## PyTorch 预热写法

使用 `LinearLR` 通过非零 `start_factor` 控制起步倍率并在到达 `total_iters` 后精准衔接主峰值：

```python
from torch.optim.lr_scheduler import LinearLR

warmup_scheduler = LinearLR(
    optimizer, start_factor=1e-3, end_factor=1.0, total_iters=warmup_steps
)
```

# 主流调度策略

预热完成后按训练预算、数据延续性与验证指标形态，主要分为五类策略：

## 余弦退火（Cosine Annealing）

沿半周期余弦曲线将学习率从峰值 $\eta_{\max}$ 平滑衰减至下界 $\eta_{\min}$（$T$ 为衰减总步数），呈「前平、中陡、后缓」分布减少步长跳变：

$$
\eta_t = \eta_{\min} + \frac{1}{2}\left(\eta_{\max} - \eta_{\min}\right)\left(1 + \cos\left(\frac{t\pi}{T}\right)\right)
$$

注意 PyTorch 的 `CosineAnnealingLR` 在调用超过 `T_max` 后会按余弦周期回弹上升，应与总循环次数严格对齐（若带重启探索其他极小点常与稳态动量/二阶矩失配，大模型较少用）：

```python
from torch.optim.lr_scheduler import CosineAnnealingLR

scheduler = CosineAnnealingLR(optimizer, T_max=100, eta_min=1e-6)
for epoch in range(100):
    train(...)
    scheduler.step()
```

## 预热-恒定-衰减（WSD）

标准余弦退火开训前须锁死总步数 $T$，不便于中途追加语料续训。现代大模型预训练广泛改用 **WSD（Warmup-Stable-Decay）** 三阶段调度：

1. **Warmup**：线性升温至峰值学习率。
2. **Stable**：长期维持高稳态学习率持续训练；任意中间检查点均可无缝延长语料步数直接续训。
3. **Decay**：仅在全部训练尾声（如最后 10% 步数）快速退火至近零完成精调收口。

## 阶梯与指数衰减

记初始学习率为 $\eta_0$、衰减系数为 $\gamma\in(0,1)$：

- **阶梯衰减（Step / MultiStep）**：每满 $T_{\text{step}}$ 步乘 $\gamma$（$\eta_t=\eta_0\gamma^{\lfloor t/T_{\text{step}}\rfloor}$），或每越过预设节点 $\{m_1,\dots,m_N\}$ 时衰减（记越过节点数 $c=\sum_{i=1}^N \mathbf{1}_{\{t\ge m_i\}}$ 则 $\eta_t=\eta_0\gamma^c$），简单但需人工定死节点。
- **指数衰减（Exponential）**：每步固定乘系数 $\eta_t = \eta_0 \gamma^t$。对 $\gamma$ 极敏感（如 $0.99^{1000}\approx 4\times 10^{-5}$ 而 $0.999^{1000}\approx 0.37$）。

## 按验证集反馈：ReduceLROnPlateau

当收敛所需轮数未知或训练易进平台期时，可监控验证指标在连续 `patience` 轮不下降后自动缩减学习率：

```python
from torch.optim.lr_scheduler import ReduceLROnPlateau

scheduler = ReduceLROnPlateau(optimizer, mode="min", factor=0.5, patience=5, min_lr=1e-6)
for epoch in range(100):
    train(...)
    val_loss = evaluate(...)
    scheduler.step(val_loss)  # 显式传入验证指标，连续 5 轮未降减半学习率
```

## 单周期调度：OneCycleLR

`OneCycleLR` 按 **batch** 粒度执行，在前 `pct_start` 段将学习率升至 `max_lr` 随后余弦退火，同时使动量与学习率反向联动（学习率升时动量降），适合固定预算下的紧凑模型快训：

```python
from torch.optim.lr_scheduler import OneCycleLR

scheduler = OneCycleLR(optimizer, max_lr=0.1, total_steps=epochs * len(train_loader), pct_start=0.3)
for epoch in range(epochs):
    for batch in train_loader:
        loss = train_step(batch)
        optimizer.step()
        scheduler.step()      # 在每个 batch 后步进，而非 epoch 末尾
```

# 调度策略选型

大规模预训练首选 **Warmup + Cosine** 或 **WSD**，步数未知的中小规模调优先用 **ReduceLROnPlateau**，固定预算从头快训可用 **OneCycleLR**：

| 策略 | 调度依据与步进粒度 | 典型适用场景 |
| --- | --- | --- |
| Warmup + Cosine Annealing | 预先确定总步数，按 step 推进 | 固定算力预算的大模型与视觉预训练 |
| WSD（Warmup-Stable-Decay） | 恒定主阶段加尾段退火，按 step 推进 | 需随时追加语料续训的持续预训练 |
| Step / MultiStep Decay | 固定步距或指定节点衰减 | 成熟卷积视觉基线复现 |
| Exponential Decay | 每步按 $\gamma$ 连续指数收缩 | 需要平滑指数降步长的短程任务 |
| ReduceLROnPlateau | 验证指标连续 `patience` 轮停滞降速 | 总轮数未知的中小实验与微调 |
| OneCycleLR | 单周期先升后降伴动量反联，按 batch 推进 | 总步数已知、追求快收敛的从头训练 |
