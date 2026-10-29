---
title: API
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-27 11:35:21
katex: true
---

# 数据准备

训练前需要划分数据、估计预处理参数，再将处理后的数组转为张量批次。以下接口分别处理数值表示、预处理和批次装载。

形状记号中，`N` 表示样本数，`D` 表示输入特征数，`K` 表示输出特征数，`C` 表示类别数。`dtype` 表示元素类型，`device` 表示张量所在的计算设备。

## NumPy 构造与属性

```python
import numpy as np
rng = np.random.default_rng(0)
X = rng.normal(size=(4, 3))        # 浮点，形状 [4, 3]
i = np.arange(6).reshape(2, 3)     # 整数，形状 [2, 3]
```

`np.array(data)` 可将列表等序列转为数组，dtype 默认由元素推断：整数列表得到整数数组，含浮点得到浮点数组。`np.zeros`、`np.ones` 可以传 `dtype`；`rng.normal` 不接受 `dtype` 参数，得到 `float64`，需要单精度时用 `.astype(np.float32)`。`shape`、`dtype`、`ndim` 分别给出形状、元素类型与维数。

## 形状、索引与聚合

```python
X.mean(axis=0)      # [3]：同一特征在各样本上取值求平均
X.mean(axis=1)      # [4]：同一样本各特征求平均
X.mean(axis=0, keepdims=True)   # [1, 3]：保留被归约的维
X[:, 0]             # [4]：第一列
X[..., None]        # [4, 3, 1]：在末尾插入一维
```

`axis` 指定参与运算的维，可以是单个整数或元组；`keepdims=True` 保留该维长度为 1，方便与数组广播。`X[:, None]` 在第二维插入，得到 `[4, 1, 3]`；要加在末尾用 `X[..., None]`。

`@` 是矩阵乘法：`[4, 3] @ [3]` 得 `[4]`，`[4, 3] @ [3, 2]` 得 `[4, 2]`。`*` 是逐元素乘，形状需相同或可广播。

广播允许不同形状的数组逐元素运算。比较形状时从末维向前，各维长度必须相同或其中一个为 1，缺少的前置维按 1 处理。例如 `[4, 1] + [3]` 得到 `[4, 3]`，无需手动复制数据。

布尔索引用同一掩码筛特征与标签：

```python
mask = X[:, 0] > 0      # [4]
X[mask].shape           # 取第一列为正的行
y_np = np.array([1, 0, 1, 0])
y_np[mask]              # 标签必须用同一掩码
```

掩码长度要与对应维一致。筛特征行与标签时用同一个掩码，否则样本与标签错位。

## 划分与缩放（scikit-learn）

```python
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

X_s = np.random.default_rng(1).normal(size=(20, 3))
y_s = np.random.default_rng(2).normal(size=20)
X_tr, X_te, y_tr, y_te = train_test_split(X_s, y_s, test_size=0.2, random_state=0)
scaler = StandardScaler()
X_tr_s = scaler.fit_transform(X_tr)   # 训练集估计均值与标准差
X_te_s = scaler.transform(X_te)       # 测试集沿用训练集参数
```

`fit` 估计参数，`transform` 应用参数，`fit_transform` 是组合。测试集只能 `transform`；对测试集 `fit_transform` 会让测试集统计量进入预处理参数，评估流程失去独立性，分数可能偏高或偏低。

## PyTorch 张量

```python
import torch
a = np.random.default_rng(0).normal(size=(4, 3)).astype(np.float32)
t = torch.from_numpy(a)         # 与 a 共享内存
t2 = torch.tensor(a)            # 复制数据
t3 = t.detach().cpu().numpy()   # 转回 NumPy
```

`torch.from_numpy` 接受 dtype 受支持的普通 NumPy 数组（例如 float32/float64/int64），返回与数组共享内存的 CPU 张量，修改任一方都会影响另一方；需要独立副本时用 `torch.tensor`。转回 NumPy 时要求张量在 CPU、dtype 受支持且不需要梯度，带梯度或位于 GPU 的张量先写 `.detach().cpu().numpy()`。NumPy 数组没有 `requires_grad` 属性，梯度只属于张量。

```python
t.view(2, 6)          # 新旧布局兼容时共享存储，否则报错
t.reshape(2, 6)       # 必要时复制，更稳妥
t.transpose(0, 1)     # 交换两维
t.unsqueeze(0)        # 在最前插入一维，[1, 4, 3]
```

`view` 在新旧布局兼容时共享存储地改形状，兼容性不满足时报错；`reshape` 在必要时复制数据。两者不能当作无差别改形状。

## 数据装载

```python
from torch.utils.data import TensorDataset, DataLoader
ds = TensorDataset(t2, torch.tensor([1, 0, 1, 0], dtype=torch.float32))
dl = DataLoader(ds, batch_size=2, shuffle=True)
xb, yb = next(iter(dl))
xb.shape, yb.shape      # [2, 3] 与 [2]
```

`TensorDataset` 按第 0 维把张量配对，长度必须一致。`shuffle=True` 每个 epoch 重新打乱，训练时使用；评估时关闭。最后一个不完整批次默认保留。

# 模型训练

这一节按一次训练实际发生的顺序排列接口：先定义模型，再算前向与损失，然后求梯度、更新参数，最后看完整的训练示例。

## 模型定义

```python
from torch import nn

class Regressor(nn.Module):
    def __init__(self, n_features):
        super().__init__()
        self.linear = nn.Linear(n_features, 1)

    def forward(self, x):
        return self.linear(x).squeeze(-1)

model = Regressor(3)
sum(p.numel() for p in model.parameters())   # 4：3 个权重 + 1 个偏置
```

`nn.Linear(in_features, out_features)` 输入 `[N, in_features]`，输出 `[N, out_features]`；`bias=True` 时添加输出偏置向量。`nn.Linear(D, K)` 的权重形状是 `[K, D]`，偏置是 `[K]` 向量。`model.parameters()` 是注册参数的迭代器，包含冻结参数。只取可训练参数用 `filter(lambda p: p.requires_grad, model.parameters())`。

`squeeze(-1)` 只删除末维且该维为 1 的情况，不会误删批次维。若输出形状是 `[N, 1]`，`squeeze(-1)` 得 `[N]`；写成 `squeeze()` 则在 `N=1` 时会把批次维也删掉。

`model.train()` 与 `model.eval()` 切换训练与推理行为，影响 Dropout 与 BatchNorm；梯度是否记录由 `requires_grad` 与 `torch.no_grad()` 控制。张量的 `.to(device)` 与 `.float()` 返回转换结果，也可能复用原张量，调用时应使用返回值。`model.to(device)` 会迁移注册参数与缓冲区。模型输入与参数需同 dtype、同 device；回归目标用浮点，分类索引目标用整型。

## 损失函数

| 损失 | 输入 | 目标 | 必要条件 |
| --- | --- | --- | --- |
| `MSELoss` | `[N]` 或 `[N, K]` 浮点 | 与输入同形的浮点 | 回归；保持形状一致，避免意外广播 |
| `CrossEntropyLoss` | logits `[N, C]` 浮点 | `[N]` int64 类别索引 | 多分类；输入未归一化，不再套 softmax |
| `BCEWithLogitsLoss` | logits `[N]` 或 `[N, C]` 浮点 | `[0, 1]` 内浮点，与输入同形 | 二分类或多标签；内部含 sigmoid |

```python
logits = torch.randn(8, 5)                       # [N, C]
target = torch.randint(0, 5, (8,))               # [N]，long
loss_ce = nn.CrossEntropyLoss()(logits, target)
```

`CrossEntropyLoss` 在类别索引模式下接收未归一化的 logits `[N, C]` 与 `int64` 类别索引 `[N]`，内部完成 log_softmax 与负对数似然，不要再套 softmax。`MSELoss` 期望逐元素的数值目标，输入与目标同形；形状不匹配时广播可能悄悄产生错误目标，报错与否取决于具体形状，不能依赖报错来发现问题。

## 自动微分

```python
import torch
w = torch.zeros(3, requires_grad=True)
x = torch.tensor([1.0, 2.0, 3.0])
loss = (w * x).sum()
loss.backward()
w.grad          # 与 w 同形
```

`backward()` 默认把梯度累加到 `requires_grad=True` 的叶子张量（例如模型参数）的 `.grad`，中间张量的梯度默认不保留。`optimizer.zero_grad()` 清空上一轮梯度；梯度默认累加，不清零会把多轮结果加在一起。`torch.no_grad()` 在评估与推理时关闭记录。

## 优化器

| 优化器 | 构造 | 常用参数 |
| --- | --- | --- |
| `torch.optim.SGD` | `SGD(model.parameters(), lr=0.1)` | `params`、`lr`；动量可加 `momentum` |
| `torch.optim.Adam` | `Adam(model.parameters(), lr=1e-3)` | `params`、`lr`；`betas`、`eps` 有默认值 |
| `torch.optim.AdamW` | `AdamW(model.parameters(), lr=1e-3)` | `params`、`lr`；解耦权重衰减用 `weight_decay` |

三者共用 `zero_grad()`、`step()`。

## 完整训练示例（回归）

```python
import numpy as np
import torch
from torch import nn
from torch.utils.data import TensorDataset, DataLoader

torch.manual_seed(0)
rng = np.random.default_rng(0)

n, d = 64, 3
X = rng.normal(size=(n, d)).astype(np.float32)
true_w = np.array([2.0, -1.0, 0.5], dtype=np.float32)
y = X @ true_w + 0.1 * rng.normal(size=n).astype(np.float32)

X_t = torch.from_numpy(X)
y_t = torch.from_numpy(y)

X_train, X_val = X_t[:48], X_t[48:]
y_train, y_val = y_t[:48], y_t[48:]

train_dl = DataLoader(TensorDataset(X_train, y_train), batch_size=16, shuffle=True)

model = nn.Linear(d, 1)
loss_fn = nn.MSELoss()
opt = torch.optim.SGD(model.parameters(), lr=0.1)

for epoch in range(50):
    model.train()
    for xb, yb in train_dl:
        opt.zero_grad()
        pred = model(xb).squeeze(-1)     # [16]
        loss = loss_fn(pred, yb)         # yb 也是 [16]
        loss.backward()
        opt.step()

model.eval()
with torch.no_grad():
    val_loss = loss_fn(model(X_val).squeeze(-1), y_val).item()
```

循环顺序与代码一致：清零、前向、算损失、反向、更新。只需在本次反向之前清零一次；`loss.item()` 返回 Python 标量，用于记录，不参与反向。

`MSELoss` 默认对所有元素求平均；多输出时元素数等于样本数乘输出维数，与“按样本平均”不同。

# 评估与保存

训练结束后，评估、推理与保存都针对已经训练好的模型。

## 推理与评估

```python
model.eval()
with torch.no_grad():
    pred_val = model(X_val).squeeze(-1)
    val_mse = ((pred_val - y_val) ** 2).mean().item()
```

`model.eval()` 切换推理行为，`torch.no_grad()` 关闭记录以降低显存占用。`val_mse` 是验证样本的平均平方误差。

`predict` 返回预测值，`predict_proba` 返回各类概率。

`sklearn.metrics` 提供分类指标 `accuracy_score`、`precision_score`、`recall_score`、`f1_score`，以及回归指标 `mean_absolute_error`、`mean_squared_error`。这些函数通常按 `指标函数(y_true, y_pred)` 调用，分别传入真实标签和预测结果。

## 保存与加载

以下代码保存前文完整训练示例中的线性模型，再加载到同结构的新模型中。

```python
torch.save(model.state_dict(), "model.pt")
model2 = nn.Linear(d, 1)
model2.load_state_dict(torch.load("model.pt", map_location="cpu", weights_only=True))
model2.eval()   # 推理前切到评估模式
```

`state_dict` 保存参数与持久缓冲区（例如 BatchNorm 的 running 统计量），不保存类定义；加载前要构造同结构的模型。`map_location="cpu"` 便于在无 GPU 环境加载；`weights_only=True` 表示按权重字典加载，不反序列化任意 Python 对象。需要继续训练时，还要保存优化器状态（`opt.state_dict()`）。`torch.save` 返回 `None`，作用是写文件。

# 接口速查

| 需求 | 调用 | 输入 → 输出 |
| --- | --- | --- |
| 随机数组 | `rng.normal(size=(N, D))` | → `[N, D]` float64 |
| 逐特征均值 | `X.mean(axis=0)` | `[N, D]` → `[D]` |
| 末尾加一维 | `X[..., None]` | `[N, D]` → `[N, D, 1]` |
| 转张量 | `torch.tensor(X)` | ndarray → Tensor（复制） |
| 分批 | `DataLoader(ds, batch_size=B)` | → 每批 `[B, D]` |
| 线性层 | `nn.Linear(D, K)` | `[N, D]` → `[N, K]` |
| 清梯度 | `opt.zero_grad()` | 原地清空 `.grad` |
| 反向 | `loss.backward()` | 梯度写入叶子 `.grad` |
| 更新 | `opt.step()` | 原地更新参数 |
| 保存参数 | `torch.save(state_dict, path)` | 写文件，返回 `None` |

官方文档：[PyTorch 快速入门](https://docs.pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html)、[NumPy 入门](https://numpy.org/doc/stable/user/absolute_beginners.html)、[scikit-learn 常见陷阱](https://scikit-learn.org/stable/common_pitfalls.html)。
