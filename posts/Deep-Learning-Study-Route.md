---
title: Deep Learning Study Route
tags:
  - DeepLearning
categories:
  - Artificial Intelligence
date: 2026-10-26 23:42:25
katex: true
---

# 深度学习在机器学习中的位置

深度学习仍是参数化的机器学习分支：核心主线同样是定义损失、计算梯度、沿负梯度迭代更新参数。区别在于函数形式，通过多层可微非线性变换复合，让模型同时从原始数据中学出有效特征表示与决策边界。

## 表示学习与特征工程的分工

传统机器学习依赖人工构造特征，再交由模型完成分类或回归；深度学习则将「构造表示」纳入端到端优化，称为**表示学习**。以视觉模型为例，浅层往往自动习得边缘与局部纹理，中层组合出部件，高层组合出完整物体。

[html-card height=820](../assets/study-route-representation.html)

表示学习并非对所有数据占优：在低维结构化表格数据上，人工特征配合中小规模树模型或线性模型往往更稳健；其优势主要出现在维度高、样本充足且具有明显层次结构的图像、语音与文本任务中。数据清洗、网络拓扑、损失函数与正则化策略仍需人工在训练前配置。

## 非线性与深度

线性映射加偏置称为仿射变换（$\mathbf{W}\boldsymbol{x} + \boldsymbol{b}$，其中 $\mathbf{W}$ 为权重矩阵、$\boldsymbol{x}$ 为输入向量、$\boldsymbol{b}$ 为偏置向量）。若层间不插入非线性激活函数，多层仿射串联在代数上仍坍缩为单次仿射变换，还受最窄中间层秩的限制。深度的价值来自「仿射变换 + 非线性激活」交替复合，数学证明见 [[From-Perceptron-To-MLP]]，激活导数对比见 [[Activation-Function-Comparison]]。

# 最小训练循环

一次标准监督学习更新的核心循环只有四步，外围机制各司其职：

1. **前向传播**：按定义好的结构逐层计算预测输出（初始权重尺度见 [[Weight-Initialization-And-Variance]]，激活与归一化选型见 [[Activation-Function-Comparison]] 与 [[Norm]]）。
2. **计算损失**：用可导损失函数衡量预测与真实标签的差距。
3. **反向传播**：调用自动微分沿计算图求各参数梯度（机制见 [[Backpropagation-Chain-Rule]]）。
4. **更新参数**：按梯度方向与学习率更新权重（优化器见 [[Gradient-Descent]]，步长调节见 [[Learning-Rate-Schedule]]）。

![784 维输入经 128 个 ReLU 隐藏单元到 10 个输出 logits 的示意，省略号表示未画出的单元|390](../assets/study-route-network.svg)

下面给出一个自包含的最小 PyTorch 训练示例（对应上图的 `784 → 128 → 10` 结构）。样本矩阵 `x` 按框架惯例取形状 `[64, 784]`：**行数 `64` 为 batch 内样本数，列数 `784` 为特征维数**（即 $28\times 28$ 图像展平像素）：

```python
import torch
from torch import nn

class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(784, 128)   # 构造时完成默认权重初始化
        self.fc2 = nn.Linear(128, 10)    # 输出 10 维未归一化 logits

    def forward(self, x):
        # 第一层输出经 ReLU 引入非线性，末层直出 logits 供损失函数接收
        return self.fc2(torch.relu(self.fc1(x)))

model = Net()
loss_fn = nn.CrossEntropyLoss()          # 内部融合 log_softmax，直接收 logits
opt = torch.optim.SGD(model.parameters(), lr=0.1)

x = torch.randn(64, 784)                 # 行=样本数(64)，列=特征数(784)
y = torch.randint(0, 10, (64,))          # 64 个样本对应的真实类别编号
for step in range(200):
    logits = model(x)                    # 1. 前向传播
    loss = loss_fn(logits, y)            # 2. 计算损失
    opt.zero_grad()                      # 清空上一轮累积梯度
    loss.backward()                      # 3. 反向传播构建并释放计算图
    opt.step()                           # 4. 优化器更新参数
```

上述示例使用随机张量仅用于验证循环闭环能否正常前向、反传与更新参数；`loss.backward()` 自动追踪张量计算图，因此无需手动指定求导参数列表。

# 前置知识

> [!note] 按需查阅
> 遇到具体工程障碍时再回溯对应数学概念，效率远高于脱离问题通读基础教材：
>
> | 遇到的现象 | 需要补充的数学/工程概念 |
> | --- | --- |
> | 看不懂 $\mathbf{W}\boldsymbol{x} + \boldsymbol{b}$ 形状如何对齐 | 矩阵乘法维度匹配、行列布局与广播 |
> | 不明白反向为什么连乘局部导数 | 复合函数链式法则与雅可比矩阵 |
> | 看不懂交叉熵里的对数形式 | 信息熵、最大似然估计与负对数似然 |
> | 多分类原始分数如何转成概率 | Softmax 指数归一化与数值稳定性 |
> | 深层训练梯度消失或爆炸 | 连乘缩放、激活导数与层间方差传播 |
> | 训练集表现好但验证集变差 | 过拟合识别、偏差方差权衡与正则化 |
