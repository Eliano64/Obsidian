---
title: Deep Learning Training Pipeline
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

神经网络训练（以监督学习为例）由可微前向模型、可导损失函数、反向传播与优化器更新组成；强化学习与无监督预训练沿用同一骨架，差别仅在数据来源与目标函数。

# 从输入到预测

**输入形态决定网络入口结构**：图像以像素张量进入线性层或卷积层，文本则先通过嵌入查表把离散词元编号映射为稠密向量，再进入后续矩阵运算。

以多层全连接网络为例，设 $l$ 为层序号，$\mathbf{A}^{(l-1)} \in \mathbb{R}^{n_{l-1} \times B}$ 为第 $l-1$ 层的输出激活矩阵（$n_{l-1}$ 为输入单元数，$B$ 为一个 batch 内的样本数），$\mathbf{W}^{(l)} \in \mathbb{R}^{n_l \times n_{l-1}}$ 与 $\boldsymbol{b}^{(l)} \in \mathbb{R}^{n_l \times 1}$ 分别为第 $l$ 层的权重矩阵与偏置列向量，$n_l$ 为当前层单元数，$\sigma$ 为逐元素激活函数，则前向传播写作：

$$
\mathbf{A}^{(l)} = \sigma\left(\mathbf{W}^{(l)} \mathbf{A}^{(l-1)} + \boldsymbol{b}^{(l)}\right)
$$

其中 $\boldsymbol{b}^{(l)}$ 在加法时沿 batch 维自动广播。

注意数据布局对照：普通表格数据与 PyTorch 惯例采用「**行是样本数、列是特征数**」（输入张量形状为 $[B, D]$，$D$ 为特征维数，线性变换写作 $\mathbf{X}\mathbf{W}^{\top} + \boldsymbol{b}$，$\mathbf{X}$ 为输入矩阵），而上式及下方演示卡片采用列样本排布 $[n_{l-1}, B]$，两者互为转置、数学上完全等价。

[html-card height=680](../assets/dl-forward-matrix-card.html)

# 配置与训练循环

工程上先区分训练前确定的**静态配置**与反复执行的**动态训练循环**，避免把超参数配置混同为单步运算：

- **静态配置**：包含数据划分、网络结构（层宽、激活、[[Norm|归一化层]]与 Dropout 位置）、参数初始化（见 [[Weight-Initialization-And-Variance]]）、损失函数、优化器与正则化策略（见 [[Regularization]]）。其中归一化层与 Dropout 在前向阶段生效，不放在更新步骤之后。
- **动态循环四步**：每次迭代依次执行「前向计算预测 → 计算标量损失 → 反向传播求梯度 → 优化器按梯度更新参数」。

数据集通常划分为训练集（更新参数）、验证集（筛选超参数与早停）与测试集（最终只评估一次）；样本较少时保留独立测试集后在剩余数据上做交叉验证（详见 [[Model-Evaluation]]）。正则化、学习率调度与早停均属可选项，需结合过拟合程度与算力预算选用。

[html-card height=820](../assets/training-pipeline-overview.html)

# 损失、指标与输出层

训练阶段**损失函数必须可导**，用以提供参数更新梯度；**评价指标不需要可导**（如准确率、F1、ROC-AUC），仅用于衡量任务表现。损失函数数学推导见 [[Logistic-Regression]]，分类指标体系与不平衡数据处理见 [[Model-Evaluation]]。

输出层形态与损失函数需严格配对：多分类或二分类输出层的未归一化实数分数称为 logits，直接传入框架融合损失即可；无约束回归任务则不在输出层施加压缩激活。

> [!important] 分类损失默认接收 logits
> PyTorch 的 `CrossEntropyLoss` 内部先计算 `log_softmax` 再取目标类别负对数似然，`BCEWithLogitsLoss` 内部将 `sigmoid` 与二元交叉熵融合以保证数值稳定。训练阶段直接传入 logits，不可提前手加 `Softmax` 或 `Sigmoid`；推理需要概率分布时再显式转换。

# 模块顺序

网络模块顺序没有唯一模板：卷积与全连接常见排布为 `Linear → BatchNorm → ReLU → Dropout`，而残差网络常采用前置归一化，Transformer 前馈层则写为 `Linear → GELU → Linear`。

判断任意模块排布是否合理，只需核对三项条件：归一化在哪些维度上统计均值与方差、激活是否改变张量正负范围与尺度、残差连接相加两路的数值尺度是否匹配。

[html-card height=820](../assets/module-order-comparison.html)

# epoch、iteration 与 batch

- **batch**：单次前向与反向计算所处理的样本子集，其样本个数称为 batch size（记作 $B$）。
- **iteration（迭代）**：完成一个 batch 的计算记为一次迭代；不启用梯度累积时每次迭代执行一次参数更新，启用梯度累积时每累积若干个 batch 再执行一次更新。
- **epoch**：把全部训练样本完整遍历一轮。

设训练样本总数为 $N_{\text{train}}$，保留末尾不足一批的数据时，每个 epoch 的 batch 数为：

$$
\text{每个 epoch 的 batch 数} = \left\lceil \frac{N_{\text{train}}}{B} \right\rceil
$$

例如 $N_{\text{train}} = 50{,}000$、$B = 64$ 时，每个 epoch 共有 $\lceil 50000 / 64 \rceil = 782$ 个 batch；若设置 `drop_last=True` 丢弃尾批则为 781 个；若每 4 个 batch 累积更新一次，则对应 $\lceil 782 / 4 \rceil = 196$ 次参数更新。增大 batch 可降低梯度方差但抬高显存占用，减小 batch 则噪声略高且更新更频繁。

[html-card height=360](../assets/dl-batch-epoch-card.html)

# 参数量与显存账本

记 $\text{fan\_in}^{(l)}$ 与 $\text{fan\_out}^{(l)}$ 分别为第 $l$ 层单输出单元接收的输入个数与输出单元个数，多层全连接网络的总参数量为：

$$
\text{参数量} = \sum_{l} \left( \text{fan\_in}^{(l)} \times \text{fan\_out}^{(l)} + \text{fan\_out}^{(l)} \right)
$$

以 `784 → 256 → 10` 两层网络为例，第一层含权重 $784 \times 256 = 200{,}704$ 与偏置 $256$，第二层含权重 $256 \times 10 = 2{,}560$ 与偏置 $10$，共 $203{,}530$ 个参数。卷积层在所有空间位置共享卷积核，参数量只取决于核大小与输入输出通道数（如 $3\times 3$ 卷积、64 通道转 128 通道共有 $3\times 3\times 64\times 128 + 128 = 73{,}856$ 个参数），与输入图像宽高无关。

## 混合精度训练与显存账本

参数量不等于训练显存。GPU 显存由**模型参数、参数梯度、优化器状态**与**前向中间激活**四部分组成。现代大模型常采用**自动混合精度（AMP）**，在 16 位精度下完成前向与反向矩阵乘，同时在优化器侧保留 32 位（`fp32`）状态防微小更新下溢。下表以 AdamW 优化器为例对比每参数静态显存开销：

| 存储项 | 纯 `fp32`（每参数） | AMP `bf16`/`fp16`（每参数） | 说明 |
| --- | --- | --- | --- |
| 模型参数 | 4 字节 | 2 字节 | 权重与偏置本体；AMP 以 16 位参与前向与反向计算 |
| 参数梯度 | 4 字节 | 2 字节 | 存放 `loss.backward()` 计算得到的参数梯度 |
| 优化器状态（AdamW） | 8 字节 | 8 字节 | 保存 `fp32` 一阶矩与二阶矩两份动量估计 |
| 主权重副本（Master Weights） | 0 字节 | 4 字节 | AMP 专属：`fp32` 精度下执行步长更新后再回写 16 位副本 |
| 静态合计与动态激活 | 静态 16 字节 | 静态 16 字节（激活减半） | **前向中间激活**随 batch 与序列长度增长，16 位下显存减半；长上下文下可借 [[Backpropagation-Chain-Rule#自动微分的记录与释放|梯度检查点]]以算换存 |

### 精度格式取舍：fp16 与 bf16

- **`fp16`（5 位指数、10 位尾数）**：动态范围较窄，梯度过小时易下溢为零、过大时易上溢，需配合 `GradScaler` 在反向前放大损失、更新前再缩回梯度。
- **`bf16`（8 位指数、7 位尾数）**：保留与 `fp32` 相同的指数位宽，动态范围一致，仅尾数精度略降，通常无需 `GradScaler` 即可稳定训练，是现代 GPU 上大模型预训练与微调的默认格式。

# 验证与早停

训练集损失下降只说明模型在拟合已知样本；当验证集损失持续上升而训练损失继续下降时，表明出现过拟合。早停策略保存验证集损失最低一轮的权重，而非最后一轮权重；小批量梯度带有随机噪声，观察趋势应看平滑窗口均值而非单步波动，且测试集绝不参与超参数筛选或早停决策（详见 [[Model-Evaluation]] 与 [[Regularization]]）。
