---
title: Data Preprocessing
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-08 19:04:09
katex: true
---

> [!cite]
> Garbage in, garbage out.

真实数据普遍混杂缺失值、极端离群点、离散文本与量纲差异。整条预处理流程只守一条防线：**所有变换规则与统计量只能从训练集估计，再原样应用到验证集与测试集**（下文结合泰坦尼克数据集示例说明工程取舍）。

# 划分与防泄露边界

数据划分为三个互斥子集（完整评估指标与交叉验证详见 [[Model-Evaluation]]）：

| 子集 | 常见比例 | 核心职责 |
| --- | --- | --- |
| 训练集 | 60% 到 80% | 拟合模型参数并估计全部预处理统计量 |
| 验证集 | 10% 到 20% | 比较候选方案、调整超参数与特征工程 |
| 测试集 | 10% 到 20% | 模拟线上未见数据，仅作最终一次泛化评估 |

> [!danger] 数据泄露（Data Leakage）
> 若用包含验证集或测试集的全体样本估计缺失值、编码清单或均值方差，测试信息就会提前渗入训练。铁律是**训练集调用 `fit`（或 `fit_transform`），验证与测试集只能调用 `transform`**（K 折交叉验证时必须在每一折的训练子集内部重新 `fit`）：

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")
imputer.fit(X_train[["Age"]])                  # 仅用训练集估计中位数
X_val[["Age"]] = imputer.transform(X_val[["Age"]])
X_test[["Age"]] = imputer.transform(X_test[["Age"]])
```

# 探索性检查（EDA）

正式清洗前通过三项快速检查确定加工方案：

1. **结构盘点（`.info()`）**：表格数据中**行数就是样本数，列数就是特征数**；识别高唯一值无意义标识列（姓名、票号）、文本类别列与缺失列。
2. **分布诊断（`.describe()`）**：对照均值与中位数、75% 分位与最大值；均值远高于中位数或标准差极高提示右偏长尾（如票价 `Fare`），后续应改用抗极值的中位数填充与稳健缩放。
3. **关系排查**：检查分组统计与非单调关系；即使线性相关系数接近 0（如 `Age`），也不能直接武断删列。

# 数据清洗

## 重复行与索引同步

删除重复观测改变**行数（样本数）**，而特征表 `X_train` 与标签 `y_train` 靠行索引对齐，删行后必须同步对齐标签索引（后文独热编码与特征构造只改列数、不改行数）：

```python
if X_train.duplicated().sum() > 0:
    X_train = X_train.drop_duplicates()
    y_train = y_train[X_train.index]  # 按保留索引同步对齐标签
```

## 缺失值分档处理

1. **极低比例随机缺失（< 5%）**：可在训练集内部删行，但验证/测试与推理期绝不能删样本，流水线仍须配置填充兜底。
2. **中等比例缺失**：近似对称数值列填均值或中位数，右偏长尾列选中位数，类别列填众数或 `"Unknown"`。
3. **极高比例缺失（> 70%）**：信息极其稀疏时整列删除；若“是否缺失”本身携带业务信号，额外保留二值缺失指示列。

## 异常值检测与处理

箱线图利用四分位数与须外点快速标出候选偏离样本：

![[../assets/data-preprocessing-boxplot.svg]]

处理关键看**是否能被业务逻辑解释**：

- 明确属于录入错误的极少数观测直接修正或删除。
- 属于真实极端分布（如高价头等舱票）不应乱删，可直接交给对单调尺度不敏感的树模型（[[Decision-Tree]]、[[Ensemble-Learning]]）；若用线性模型或神经网络，则做对数变换（先处理零值）或**截尾（Winsorization）**把超阈值极值压回分位边界。

# 类别编码、特征构造与缩放

## 类别特征编码

类别编码依据「**是否存在真实大小顺序**」与「**类别基数高低**」选型（例如 `OneHotEncoder(handle_unknown="ignore")` 在推理遇未见类别时自动编码全 0）：

| 编码方式 | 映射规则与特点 | 适用场景 |
| --- | --- | --- |
| 序数编码（Ordinal） | 映射为保留有序等级的整数（低=0、中=1、高=2） | 明确有序的类别列（如舱位等级） |
| 独热编码（One-Hot） | 每类占一列 $0/1$，消除虚假数值大小关系 | 类别数较少、无顺序的名义类别（如港口） |
| 标签编码（Label） | 按字典序编成 $0, 1, 2$，隐含虚假等距假设 | 仅用于目标标签 `y` 或二分类转为 $0/1$，禁止用于无序多类特征 |
| 频数 / 目标编码 / Embedding | 频数统计、目标统计均值、低维嵌入或树原生分裂 | 高基数类别列（邮编、商户 ID），避免独热维度爆炸 |

> [!warning] 目标编码的防泄露边界
> 用同组全体训练样本标签均值做目标编码会把样本自身标签泄漏进特征。工程上除加入全局平滑项外，必须配合**折外估计（Out-of-Fold）**或**有序目标统计量（只用排序靠前样本估计）**。

## 特征构造

主要包含三类动作：**合并重组**强信号列（如家庭人数 `SibSp + Parch + 1`、是否独自出行标记）；从文本中**提取结构**（如从姓名提取头衔映射，字典只由训练集归纳）；提前**删减**推理期不可见或隐含未来结果的列防止目标泄露。

## 特征缩放

基于距离的模型（KNN、K-Means、SVM）、梯度优化模型（线性模型、[[Logistic-Regression|逻辑回归]]、神经网络）及权重衰减都受量纲差距影响。记某数值列原值为 $x$，训练集均值与标准差为 $\mu,\sigma$，极小极大值为 $x_{\min},x_{\max}$，下四分位、中位与上四分位分别为 $Q_1,Q_2,Q_3$，三类主流缩放定义如下：

$$
z = \frac{x - \mu}{\sigma}, \qquad
x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}}, \qquad
x_{\text{rob}} = \frac{x - Q_2}{Q_3 - Q_1}
$$

不同分布场景按抗极端值能力选用对应缩放器（全部统计量仅在训练集估计）：

| 缩放方法 | 输出数值范围 | 对异常值敏感度 | 适用场景 |
| --- | --- | --- | --- |
| 标准化（StandardScaler） | 均值 $0$、标准差 $1$，无界 | 中等（极值拉偏 $\mu,\sigma$） | 分布近似对称的线性模型、SVM、神经网络 |
| 归一化（MinMaxScaler） | 严格压入 $[0, 1]$ | 极高（单点极值压扁其余全部样本） | 像素（天然 $[0,255]$）等已知有界特征 |
| 稳健缩放（RobustScaler） | 中位数归 $0$、四分位距归 $1$，无界 | 极低（分位数抗长尾） | 含真实极端高值但需保留样本的表格数值列 |

# 预处理流水线（Pipeline）

将多步变换封装进 `ColumnTransformer` 与 `Pipeline`，可从代码结构上强制封闭训练/验证统计边界，并在交叉验证每一折内自动重估统计量：

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import OneHotEncoder, RobustScaler

# 长尾列使用 RobustScaler，全部 fit 统计量仅在 model.fit(X_train, y_train) 时产生
numeric = Pipeline([("imputer", SimpleImputer(strategy="median")), ("scaler", RobustScaler())])
categorical = Pipeline([("imputer", SimpleImputer(strategy="most_frequent")), ("onehot", OneHotEncoder(handle_unknown="ignore"))])
preprocess = ColumnTransformer([("num", numeric, ["Age", "Fare", "SibSp", "Parch"]), ("cat", categorical, ["Sex", "Embarked"])])

model = Pipeline([("preprocess", preprocess), ("clf", clf)])
model.fit(X_train, y_train)
score = model.score(X_val, y_val)  # 验证与测试集严格只触发 transform
```
