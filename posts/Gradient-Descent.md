---
title: Gradient Descent
tags:
  - MachineLearning
categories:
  - Artificial Intelligence
date: 2026-10-08 16:49:56
katex: true
---

# 线性回归模型的训练

对于[[Linear-Regression|线性回归模型]]，$n$ 个样本、$d$ 个特征写成设计矩阵 $\mathbf{X} \in \mathbb{R}^{n \times d}$，第 $i$ 行是第 $i$ 个样本的特征 $\boldsymbol{x}_i^{\top}$；标签是向量 $\boldsymbol{y} \in \mathbb{R}^{n}$。参数是权重 $\boldsymbol{\theta} \in \mathbb{R}^{d}$ 与偏置 $\boldsymbol{b} \in \mathbb{R}^{n}$，预测写成

$$\hat{\boldsymbol{y}} = \mathbf{X} \boldsymbol{\theta} + \boldsymbol{b}$$

偏置向量的每个分量都是同一个数 $b$，即 $\boldsymbol{b} = b\boldsymbol{1}$（$\boldsymbol{1} \in \mathbb{R}^{n}$ 是全 1 向量），加到每个样本上的是同一个值。

> [!note] 为什么偏置的分量都是同一个数
>
> 如果让这些值各不相同，等于给每个样本单独配一个截距。样本一共只有 $n$ 个，于是总能让每个样本的残差都变成零（取 $\boldsymbol{b} = \boldsymbol{y} - \mathbf{X}\boldsymbol{\theta}$），损失直接降到 0，任意 $\boldsymbol{\theta}$ 都算最优，模型什么都没学到，极小点也不唯一。
>
> 所以偏置按所有样本共享的同一个平移量处理，长度是 1，相加时复制到每个样本上（广播），待求参数一共 $d+1$ 个。

训练做的就是找一个 $(\boldsymbol{\theta}, b)$，让预测与标签整体上最接近。损失函数用来量化这个距离，训练的目标就是让该值降到最小。

![[../assets/gradient-descent-training-loop.svg]]

# 损失函数

回归任务最常用的是**均方误差（MSE）**。MSE 建立在残差向量之上：

$$\boldsymbol{r} =\hat{\boldsymbol{y}} - \boldsymbol{y}= \mathbf{X} \boldsymbol{\theta} + \boldsymbol{b}-\boldsymbol{y}=\begin{pmatrix}
\mathbf{X}&\boldsymbol{1}
\end{pmatrix}\begin{pmatrix}
\boldsymbol{\theta} \\
b
\end{pmatrix}-\boldsymbol{y}$$

损失函数 $L$ 为：

$$L(\boldsymbol{\theta}, b) =\frac{1}{n}\boldsymbol{r}^{\top}\boldsymbol{r}= \frac{1}{n}\begin{pmatrix} \boldsymbol{\theta} \\ b \end{pmatrix}^{\top}\begin{pmatrix} \mathbf{X}^{\top}\mathbf{X} & \mathbf{X}^{\top}\boldsymbol{1} \\ \boldsymbol{1}^{\top}\mathbf{X} & n \end{pmatrix}\begin{pmatrix} \boldsymbol{\theta} \\ b \end{pmatrix} - \frac{2}{n}\begin{pmatrix} \boldsymbol{\theta} \\ b \end{pmatrix}^{\top}\begin{pmatrix} \mathbf{X}^{\top}\boldsymbol{y} \\ \boldsymbol{1}^{\top}\boldsymbol{y} \end{pmatrix} + \frac{1}{n}\boldsymbol{y}^{\top}\boldsymbol{y}$$

二次型的系数矩阵为半正定，所以 $L$ 是凸函数。当增广设计矩阵 $[\mathbf{X},\boldsymbol{1}]$ 列满秩时，$L$ 严格凸，最优解唯一。

理论上这个极小点可以直接解出来，但当特征维度很高、样本量很大，或 $\mathbf{X}^{\top}\mathbf{X}$ 接近奇异时，求解不可行，也不稳定，这时可以改用梯度下降法。梯度下降法不求解方程，而是从某个初值出发，每一步让参数沿损失下降的方向更新一点，反复迭代逼近极小点。

MSE 不需要假设误差服从正态分布就能使用。若误差是独立、零均值、同方差的正态噪声，最小化平方误差等价于最大似然估计。误差分布具有重尾时，大残差对平方损失的影响较大。

MSE 有三条局限。

- 对离群点敏感：误差取平方，一个标错的样本会把整个批次的梯度带偏。真值 100 被错标成 10000，单个样本的贡献就是 $9900^2$ 的量级。标签噪声大时改用 Huber 损失（小误差按平方、大误差按绝对值）更稳。
- 假设目标连续可比：平方默认“差 2 比差 1 严重 4 倍”，这对房价、温度成立，对类别标签不成立。把猫预测成狗和预测成飞机，在分类正误上一样错。
- 与 sigmoid 一起用会梯度消失：误差很大时输出端的梯度反而趋近于零，因此分类任务通常改用交叉熵损失。

# 梯度下降法

## 损失函数的梯度

偏导数衡量固定其余参数时，$L$ 沿单个参数方向的变化率：

$$\frac{\partial L}{\partial \theta_j} = \lim_{h \to 0} \frac{L(\theta_1, \dots, \theta_j + h, \dots, \theta_d, b) - L(\boldsymbol{\theta}, b)}{h}$$

把 $d$ 个偏导按分量排成向量，再把偏置的偏导接在后面，就是梯度：

$$\nabla_{\boldsymbol{\theta},b} L = \left( \frac{\partial L}{\partial \theta_1}, \dots, \frac{\partial L}{\partial \theta_d}, \frac{\partial L}{\partial b} \right)^{\top} \in \mathbb{R}^{d+1}$$

把残差写成分量形式，第 $i$ 个样本的残差是

$$r_i = \boldsymbol{x}_i^{\top}\boldsymbol{\theta} + b - y_i$$

损失就是 $\displaystyle L = \frac{1}{n}\sum_{i=1}^{n} r_i^2$。对单个权重 $\theta_j$ 求导，得到

$$\frac{\partial L}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^{n} 2 r_i \cdot x_{ij} = \frac{2}{n}\sum_{i=1}^{n} r_i x_{ij}$$

所以有：

$$\frac{\partial L}{\partial \boldsymbol{\theta}} = \frac{2}{n}\sum_{i=1}^{n} r_i \boldsymbol{x}_i = \frac{2}{n} \mathbf{X}^{\top}\boldsymbol{r}$$

对偏置求导：

$$\frac{\partial L}{\partial b} = \frac{1}{n}\sum_{i=1}^{n} 2 r_i \cdot 1 = \frac{2}{n}\sum_{i=1}^{n} r_i$$

两个梯度都是加权和，但权重不同：权重的梯度里每个残差乘上了对应样本的特征，该特征取值大的样本影响更大；偏置的梯度只用到残差之和，与特征无关。

## 参数更新

在某一点处，梯度方向是 $L$ 增长最快的方向，反方向是下降最快的方向，所以参数沿负梯度方向更新。

但这是局部成立的，只描述该点附近的趋势，步长太大就会越出这个范围。步长取值由[[#学习率]]决定。

沿负梯度方向更新参数时，两个参数用同一个学习率 $\eta$：

$$\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \frac{ \partial L }{ \partial \boldsymbol{\theta} }, \qquad b \leftarrow b - \eta \frac{\partial L}{\partial b}$$

$\displaystyle\frac{ \partial L }{ \partial \boldsymbol{\theta} }$ 与 $\displaystyle \frac{\partial L}{\partial b}$ 都在更新前的参数上取值。

[html-card height=720 step=80](../assets/gradient-descent-surface.html)

## 学习率

**学习率**决定每次参数更新的幅度，也同时决定收敛的快慢和会不会发散。三种情形的表现完全不同：

- 太小：每步都在下降，但沿平缓方向推进极慢，迭代很多次也到不了极小点。
- 合适：损失稳定下降，若干步内就接近极小点。
- 太大：损失在陡方向被反复放大，先震荡，再冲出可表示的范围。

![[../assets/gradient-descent-learning-rate.svg]]

学习率不必固定。这里说的是单个学习率的量级，学习率在训练过程中的变化单独成一节。

批量很大时梯度噪声小，可以配更大的学习率；批量小时相反。这一点决定目标学习率的基准值，而基准值之外还有一条坐标：训练进行到哪一步。

步长怎么随训练阶段变化，见[[#学习率调度]]。

## 迭代过程的伪代码

```text
输入 X, y, 学习率 η, 阈值 ε, 最大迭代次数 K
初始化 θ 与 b（全零或小随机数）
for k = 1 to K:
    r  = Xθ + b - y          // 残差向量，n 维；b 广播到每个样本
    gθ = (2/n) Xᵀ r           // ∂L/∂θ，d 维
    gb = (2/n) 1ᵀ r           // ∂L/∂b，一个数
    θ = θ - η gθ             // 两个梯度都在更新前的参数上取得，所以同时更新
    b = b - η gb
    if ‖∇L(θ, b)‖_2 < ε:        // 梯度已足够小
        break
返回 θ, b
```

> [!important] 本例的停止条件
> 本例检查梯度范数是否满足 $\lVert \nabla L(\theta,b)\rVert < \varepsilon$。范数小于阈值，意味着每个分量的绝对值都小于同一阈值；反向推断不成立。梯度小只能说明当前点附近的一阶变化较弱，一般不能据此断定全局最优。最大迭代次数 $K$ 限制计算开销，防止阈值过严时持续迭代。

## 每次用多少样本

| 做法                      | 每次用到的样本量      | 特点                                                                                                              |
| ------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| 全量梯度下降 BGD          | $n$                   | 每步都要把全部样本读一遍，梯度最准、轨迹稳定；样本量大时单步耗时与显存占用都随 $n$ 增长                           |
| 随机梯度下降 SGD          | 1                     | 每次只读 1 个样本，单步最省；单个样本的梯度是全量梯度的无偏估计，但方差大，轨迹抖动明显，接近极小点后仍在附近跳动 |
| 小批量梯度下降 Mini-batch | 一小批，例如 32 或 64 | 每步读一小批，梯度方差介于前两者之间；一批样本可以拼成矩阵一次算完，能利用 GPU 并行，是当前的主流选择             |

全量梯度下降每步使用全部样本；随机梯度下降每步使用一个样本；小批量版本在两者之间取值。

> [!tip] 批大小与学习率要一起看
> 批大小是关键超参数。太小则梯度噪声大、收敛慢；太大则逐渐逼近全量梯度下降，梯度噪声降低，但不保证泛化更好。改变批大小时通常也需要重新验证学习率。

今天说 SGD 通常就指小批量版本，几乎所有优化器都是在这个框架上做改动。

# 优化器

标准梯度下降有两个缺陷：沿平缓方向推进慢，沿陡方向来回折返。下面每个优化器都在改其中一件事：Momentum 给方向加上历史，AdaGrad 与 RMSProp 让每个参数有自身的步长，Adam 把两者合起来。再往后，AdamW 将权重衰减与自适应梯度更新分开，Muon 把参数当成矩阵而不是一堆独立的标量。

下文把当前点的梯度记作 $\boldsymbol{g}$，记 $\odot$ 为逐分量相乘。

## Momentum

![[../assets/gradient-descent-optimizer-momentum.svg]]

梯度下降每步只看当前点。在陡方向上一步就跨过极小点，下一步又跨回来，来回的步子方向相反，净进展很小。如果把前几步的梯度累加起来，连续同向的部分会变大，来回摆动的那部分则相互抵消。

要这样做，需要一个向量把历史梯度累积起来，还需要一个系数决定每一步保留多少旧值。记 $\boldsymbol{v}$ 为速度向量（与参数同维），$\beta$ 是动量系数，表示旧值的保留比例，取值范围 $[0, 1)$，常用 0.9；$\eta$ 是学习率。更新分两步：

$$\boldsymbol{v} \leftarrow \beta \boldsymbol{v} + \boldsymbol{g}, \qquad \boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \boldsymbol{v}$$

第一步把当前梯度加进速度向量；第二步用速度向量代替当前梯度去更新参数。因此改变的不只是方向，还有步长：连续多步同向时 $\lVert \boldsymbol{v} \rVert$ 比单个梯度大，步子跟着变长；来回摆动时相反的部分相消，$\lVert \boldsymbol{v} \rVert$ 变小。

Momentum 在曲率小的方向很长、需要连续更新很多步的损失上较适用：同向梯度被累积，等效步长随连续步数增大；方向反复摆动时前后梯度相反，累积后抵消，摆动被压下去。瞬时梯度接近零的平坦区，靠已积累的速度也能越过。

代价是多一个超参数 $\beta$，且与学习率相互影响，要一起调。$\beta$ 越大，速度转向越慢，容易冲过极小点再折回来；太小则积累不明显。实践中 0.9 几乎是稳妥的默认值。SGD 训练常加入动量项（`torch.optim.SGD(momentum=0.9)`），VGG、ResNet（以图像为输入的卷积网络）这类卷积网络的大规模训练，标准配方就是小批量 SGD 加动量。

## AdaGrad

![[../assets/gradient-descent-optimizer-adagrad.svg]]

梯度下降对所有参数用同一个学习率。θ 与 b 两个方向的曲率相差约 70 倍时，一个步长满足不了两边：取到陡方向能接受的量级，平缓方向就慢得几乎不动；取到平缓方向需要的量级，陡方向又会跨过极小点来回跳出。既然两个方向的需求差这么多，就让每个参数按自身的历史情况决定步长。

一个典型场景是文本：高频词（例如 the）几乎每个批次都出现，梯度一直很大；低频词（例如 serendipity）整轮训练只出现几次，梯度很小。同一个学习率下，低频词对应的参数几乎永远学不动，让步长按参数各自的历史缩放正好解决这件事。

记 $\boldsymbol{s}$ 为与参数同维的向量，累计该参数历史上梯度平方的和，每个分量的更新是“旧值 + 当前梯度的平方”（$\boldsymbol{g} \odot \boldsymbol{g}$ 表示逐分量平方）。$\sqrt{\boldsymbol{s}}$ 放在分母上，所以历史梯度大的分量步长自动变小。$\delta$ 是很小的常数，避免分母为零。$\boldsymbol{s}$ 初值取零。更新分两步：

$$\boldsymbol{s} \leftarrow \boldsymbol{s} + \boldsymbol{g} \odot \boldsymbol{g}, \qquad \boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \boldsymbol{g}/(\sqrt{\boldsymbol{s}} + \delta)$$

先累加梯度平方，再按缩放后的步长更新参数。

AdaGrad 在梯度量级相差很大、或多数参数长期没有梯度（梯度稀疏）的问题上较适用：步长按各自的历史自动缩放，不必手工为不同方向设定步长。

代价是分母只增不减，等效步长随时间单调衰减，训练后段几乎不再前进；方向上也没有累积。AdaGrad 的设计初衷是提出，是自适应学习率的开山之作；今天在广告点击率预估、传统自然语言处理（NLP）这类梯度稀疏、又常常需要在线更新的场景里仍在用。

## RMSProp

![[../assets/gradient-descent-optimizer-rmsprop.svg]]

AdaGrad 的问题出在“累加”这个动作上，历史只会越积越多，早期的信息永远压在分母里。要让分母只反映近期的梯度水平，就不能再把历史一路累加，而要让旧梯度逐渐被弱化，也就是把“累加”换成“滑动平均”。

$\boldsymbol{s}$ 的含义与 AdaGrad 相同，但更新方式变了；$\beta_2$ 是新旧信息各占多少的系数，新值占 $1 - \beta_2$，旧值占 $\beta_2$，常用 0.9。更新分两步：

$$\begin{align}
&\boldsymbol{s} \leftarrow \beta_2 \boldsymbol{s} + (1 - \beta_2)\boldsymbol{g} \odot \boldsymbol{g},  \\
&\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \boldsymbol{g}/(\sqrt{\boldsymbol{s}} + \delta)
\end{align}$$

$\boldsymbol{s}$ 稳定在近期梯度平方的量级上，等效步长不随时间单调衰减，适合梯度水平随时间变化（非平稳）的损失。

代价是多一个超参数 $\beta_2$，且方向不带惯性，需要沿同一方向连续更新多步的损失上不如动量法。

RMSProp 使用近期梯度平方的滑动平均，缓解 AdaGrad 有效步长持续减小的问题，也适用于梯度统计随训练变化的深层网络和循环神经网络。

## Adam

![[../assets/gradient-descent-optimizer-adam.svg]]

Adam（Adaptive Moment Estimation）结合动量与按参数调整步长。梯度本身的滑动平均估计一阶矩，提供更新方向；梯度平方的滑动平均估计二阶矩，用于缩放步长。两个统计量配合使用，在更新公式中共同决定参数变化。

记 $\boldsymbol{m}$ 为梯度的滑动平均，管方向。$\boldsymbol{s}$ 是梯度平方的滑动平均，管步长。$\beta_1$ 与 $\beta_2$ 分别是两者的系数，常用 0.9 与 0.999。两者都从零开始累积，$\boldsymbol{m}$ 的前几步会明显偏小，$\boldsymbol{s}$ 也一样，所以各自除以 $1 - \beta_1^k$ 与 $1 - \beta_2^k$ 把尺度拉回正常（$k$ 是当前第几步）。修正后的值记作 $\hat{\boldsymbol{m}}$ 与 $\hat{\boldsymbol{s}}$。更新分两步：

$$\boldsymbol{m} \leftarrow \beta_1 \boldsymbol{m} + (1 - \beta_1)\boldsymbol{g}, \qquad \boldsymbol{s} \leftarrow \beta_2 \boldsymbol{s} + (1 - \beta_2)\boldsymbol{g} \odot \boldsymbol{g}$$

$$\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta \frac{\hat{\boldsymbol{m}}}{\sqrt{\hat{\boldsymbol{s}}} + \delta}$$

先更新两个滑动平均并做修正，再按修正后的方向与步长更新参数。

需要结合动量与自适应步长时，可以将 Adam 作为初始选择。学习率仍需结合任务验证。

Adam 为每个参数额外维护两个状态量，内存开销随参数数量线性增加，还需要设置两个衰减率。滑动平均有滞后，梯度统计快速变化时，更新可能无法立即适应。

Adam 的自适应缩放可以减少各参数之间步长差异的影响，但不能替代全局学习率和权重衰减的选择。

## AdamW

Adam 里的 L2 正则与解耦权重衰减不是同一件事：直接加 L2 项会被自适应步长缩放，衰减幅度随梯度二阶矩变化；AdamW 把衰减从梯度里拆出来单独施加。在 SGD 里，“每步把参数往零的方向缩一点”与“在损失上加一项 L2 正则”等价，所以早期 Adam 直接把 L2 项加进梯度。但 Adam 的每一步还要除以 $\sqrt{\hat{\boldsymbol{s}}}$，这个缩放同时作用在正则项上：梯度大的参数衰减不足，梯度小的参数过度衰减，正则强度因此偏离设定值。

AdamW 把这两部分拆开，让权重衰减直接作用在参数上，不经过梯度通道：

$$\boldsymbol{\theta} \leftarrow \boldsymbol{\theta} - \eta\left( \frac{\hat{\boldsymbol{m}}}{\sqrt{\hat{\boldsymbol{s}}} + \delta} + \lambda \boldsymbol{\theta} \right)$$

$\lambda$ 是权重衰减系数。AdamW 常用于 Transformer、BERT、ViT 和大语言模型的训练；ViT 是将图像切成小块后输入 Transformer 的视觉模型。具体优化器仍需结合任务与验证结果选择。

## Muon

前面所有方法都把每个标量参数当成独立的一维来处理。Muon 则换了一个视角：网络里真正起作用的是权重矩阵，一个二维对象，权重矩阵的“方向”应当指矩阵整体的方向，而不是逐个元素各自的方向。

做法是在动量之上加一次正交化：先把累积的更新矩阵用 Newton-Schulz 迭代（不显式分解矩阵、靠反复矩阵乘法逼近的迭代法）近似正交化，把奇异值拉向同一量级，再作为这一步的更新方向。效果是所有奇异方向获得均衡的更新幅度，少数几个很大的奇异值不会主导整步更新。

$$\mathbf{M} \leftarrow \beta \mathbf{M} + \nabla L(\mathbf{W}), \qquad \mathbf{W} \leftarrow \mathbf{W} - \eta \, \mathrm{orth}(\mathbf{M})$$

$\mathbf{W}$ 是某一层的权重矩阵，$\mathrm{orth}(\cdot)$ 表示上面那次正交化。此后的大模型预训练实验里，Muon 得到比 AdamW 更高的样本效率，是当前受关注的方向，但还没有成为通用默认。Muon 适用于隐藏层的二维权重矩阵；偏置、嵌入与归一化层的参数通常仍用 AdamW 等逐元素自适应方法。上面提到的样本效率提升来自特定预训练实验，不能推广为普遍结论。

## 怎么选

| 场景                 | 选择                              | 原因                                                                                         |
| -------------------- | --------------------------------- | -------------------------------------------------------------------------------------------- |
| 视觉任务，卷积网络   | 带动量的小批量 SGD 加学习率衰减   | 最终卷积网络训练中的常见参照方案，精度需实测比较                                                                   |
| 语言模型与大模型     | AdamW                             | BERT、GPT、LLaMA、Stable Diffusion 等模型常用；Muon 在部分预训练任务上报告了更好的结果   |
| 快速试验与原型       | Adam                              | 默认学习率往往就能得到可用的基线                                                             |
| 稀疏、在线、短周期   | AdaGrad                           | 例如广告点击率预估与传统 NLP                                                                 |

优化器只决定每步如何更新，不决定梯度准不准。标签噪声、特征缺失、损失函数选错，换优化器补不回来，要回到[[#损失函数]]里改。

# 学习率调度

固定的学习率隐含一个前提：训练的每一步都需要同样大的步长。这个前提并不成立。参数随机初始化时地形未知，梯度大且方向不稳，步子过大会直接冲出去；训练进入稳定下降阶段后方向相对可靠，可以放大步子加速；接近极小点时又需要小步精修，否则总在极小点附近来回跨。

一个受控实验把这种敏感性量化。同一个两层 MLP（多层感知机，784 到 128 再到 10），同一个优化器，只训练 5 个 epoch，只改学习率一个量：

| 学习率 | 现象 | 5 个 epoch 后的损失 | 5 个 epoch 后的准确率 |
| --- | --- | --- | --- |
| 10 | 第一步损失冲到三位数，此后长期停在 2.33 附近，未能收敛 | 2.34 | 约 10% |
| 1.0 | 能下降，但每步都跨过谷底，损失明显震荡 | 1.07 | 约 59% |
| 0.1 | 平稳下降，是这一组里的最优区间 | 0.07 | 约 97% |
| $10^{-5}$ | 每步更新都极小，5 个 epoch 后几乎原地不动 | 2.31 | 约 15% |

四行对应的是发散、震荡、收敛、停滞四种状态，差别不在小数点后。所以 $\eta$ 通常不写成常数，而写成训练步数的函数，也就是学习率调度（learning rate schedule）。

调度只影响全局步长。Adam、AdamW 这类自适应优化器调整的是参数之间的相对步长，全局 $\eta_t$ 仍是更新式里的常数因子，全局学习率的变化由调度策略给出：

$$\theta_{t+1} = \theta_t - \eta_t \cdot \frac{\hat{\boldsymbol{m}}_t}{\sqrt{\hat{\boldsymbol{s}}_t} + \delta}$$

## 线性预热

训练最开始若干步里，动量和二阶矩的估计都还不可靠。预热（warmup）让学习率从一个小值线性上升到目标值，等这些估计稳定下来再开始正常步长。

线性预热把第 $t$ 步的学习率写成目标值的等比例缩放：

$$\eta_t = \eta_{\max} \cdot \frac{t}{T_{\text{warmup}}}$$

$t$ 是当前步数，$T_{\text{warmup}}$ 是预热总步数。预热长度有两种给法：按步数（例如前 1000 步）与批大小、数据集规模解耦，搬到别的任务上不用重算；按比例（例如总步数的 5% 到 10%）则随总训练量自动伸缩。

PyTorch 里对应 `LinearLR`，`start_factor` 是起步值占目标值的比例（因此起点一般不为零），`total_iters` 是预热总步数。上面的公式从 0 起算，是便于说明的简化；实际调度从 `start_factor` 给出的非零起点线性升到目标值。

```python
from torch.optim.lr_scheduler import LinearLR

optimizer = torch.optim.SGD(model.parameters(), lr=0.1)
# 从目标学习率的千分之一起步，上升到 1.0 倍（即 lr=0.1），共 warmup_steps 步
warmup_scheduler = LinearLR(
    optimizer, start_factor=1e-3, end_factor=1.0, total_iters=warmup_steps
)
```

预热的价值在 Adam 上最明显。Adam 的分母是二阶矩滑动平均开根号，前几步样本太少，这个估计极不稳定；分母偏小会把更新幅度放大，损失瞬间急剧放大。同一任务（四层 MLP，Adam，学习率 0.02，MNIST 手写数字数据集，5 个 epoch）加 200 步线性预热前后：

| 设置 | 前 50 个 batch 的最大损失 | 5 个 epoch 后的测试准确率 |
| --- | --- | --- |
| 直接使用 0.02 | 17.73 | 91.75% |
| 200 步线性预热 | 2.30 | 94.74% |

前 200 步学习率仍然很小，等于把累积二阶矩的时间让给了优化器；等学习率爬到 0.02，网络早已远离初始化时的不稳定区。

## 人工设定衰减节点

预热之后进入主体段。最直觉的调度由人指定何时降、降多少。

Step decay 每过一个固定间隔 $T_{\text{step}}$ 把学习率乘一次 $\gamma$，曲线呈阶梯状：

$$\eta_t = \eta_0 \cdot \gamma^{\lfloor t / T_{\text{step}} \rfloor}$$

MultiStep decay 不再等间隔，而是手动给出一组节点 $m_1, m_2, \dots$，每越过一个节点乘一次 $\gamma$：

$$\eta_t = \eta_0 \cdot \gamma^{k}, \qquad k = \sum_{i=1}^{N} \mathbf{1}_{\{t \ge m_i\}}$$

两种做法的好处是简单，几乎所有框架都原生支持。共同的问题是衰减节奏固定：到点就降，不管模型当前收敛得好不好。换任务、换数据集，原来的节点与间隔往往就不适用。

## 指数衰减

指数衰减把阶梯磨平，每一步都乘一个固定系数：

$$\eta_t = \eta_0 \cdot \gamma^{t}$$

对 $\gamma$ 的敏感度远超直觉。$\gamma = 0.99$ 与 $\gamma = 0.999$ 看上去只差一点，1000 步之后分别是 $0.99^{1000} \approx 0.00004$ 与 $0.999^{1000} \approx 0.37$，差四个数量级。

指数衰减在前期下降较快，后期变化趋缓。若任务需要前期较长时间保持较高学习率，可以考虑其他调度形状；具体效果仍需验证。

## 余弦退火

余弦退火借余弦函数的半个周期控制学习率，从起点到总步数 $T$ 平滑下降：

$$\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})\left(1 + \cos\left(\frac{t\pi}{T}\right)\right)$$

曲线两端平、中间陡。前期学习率较大、变化缓慢；中期下降较快；后期接近 $\eta_{\min}$，变化再次减缓。与阶梯式衰减相比，余弦曲线连续，可减少学习率突变。PyTorch 的 CosineAnnealingLR 在超过 `T_max` 后仍会按余弦变化，学习率可能再次上升；应按计划结束调度，不能假定自动保持最小值。

PyTorch 里有两个参数要设对：`T_max` 是调度器被调用的总次数，`eta_min` 是最小学习率；示例按 epoch 调用，因此写成 epoch 数。`scheduler.step()` 放在 epoch 循环内、`train(...)` 之后：

```python
from torch.optim.lr_scheduler import CosineAnnealingLR

optimizer = torch.optim.Adam(model.parameters(), lr=0.1)
scheduler = CosineAnnealingLR(optimizer, T_max=100, eta_min=1e-6)

for epoch in range(100):
    train(...)
    scheduler.step()          # 每个 epoch 降一次，形状由 T_max 决定
```

## 带重启的余弦退火

标准余弦退火把一个周期内降到最小值，此后若继续调用 step，学习率会按余弦继续变化；需要长训练时通常用带重启的版本。带重启的版本让学习率降到底后跳回较大值，重新开始一轮退火，循环往复：

$$\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})\left(1 + \cos\left(\pi \cdot \frac{T_{\text{cur}}}{T_i}\right)\right)$$

每次重启都是一次扰动，把模型从当前局部极小里推出去重新探索。周期还可以逐渐拉长，常见做法是 $T_{\text{mult}} = 2$，每轮周期翻倍，前期快速探索、后期充分收敛。

提高学习率可能帮助模型探索其他区域，但不保证离开局部极小。预热加衰减是大模型训练中的常见方案，是否加入重启还需考虑以下成本。

- 重启把学习率从极小值瞬间拉回高位，而动量与二阶矩还停在按小步长累积的状态，两者不匹配，损失可能剧烈震荡甚至发散。大规模训练一次失败就是数天算力。
- 调参成本高：$T_0$、$T_{\text{mult}}$、每轮之后的衰减系数都要重新搜索，换任务通常要重来；普通余弦不需要设置多个衰减节点，但仍需选择总步数与最小学习率（常取初始值的十分之一或 0）。
- 重启通过提高学习率改变更新幅度，可能帮助探索其他区域。数据增强、小批量抽样和权重衰减也会影响训练轨迹，但不能保证替代重启，也不能保证离开某个局部极小。

这类调度仍有适用场景：竞赛中需要反复试错时、可反复试错的训练。

## 按验证集效果调度

上面所有调度都是提前写好计划，不看模型实际的收敛状态。训练长度不确定时，预设计划可能无法适应实际收敛状态。

ReduceLROnPlateau 换一个依据：监控验证集指标，连续若干轮没有改善才降学习率。

```python
from torch.optim.lr_scheduler import ReduceLROnPlateau

scheduler = ReduceLROnPlateau(
    optimizer,
    mode='min',        # 监控指标越小越好，例如验证损失
    factor=0.5,        # 触发时 lr 乘以 0.5
    patience=5,        # 连续 5 轮无改善才触发
    min_lr=1e-6,       # 下限，防止学习率被降到 0
)

for epoch in range(100):
    train(...)
    val_loss = evaluate(...)
    scheduler.step(val_loss)   # 与其他调度器的关键差异：必须传入监控指标
```

机制背后是一句朴素判断：模型几轮没有进步，说明当前步长下的优化遇到了瓶颈，要么在局部极小附近震荡，要么步长太大跨过了更优位置；把学习率降下来重新探索，往往能打破僵局。

代价有两条与机制绑定。必须定期在验证集上评估，大规模训练下这笔开销不可忽略；`patience` 机制让降低总是滞后于模型的实际状态。还有一条更根本：学习率只降不升，一旦降下去就不会回头，无法像带重启的余弦那样跳出局部极小。

选择因此落到训练规模上。预训练场景（大语言模型、大规模视觉模型）成本极高，验证评估的开销很高，且训练流程需要高度可复现，任意时刻的学习率都应能由步数直接算出，这时选余弦退火。中小规模任务、快速实验或微调，验证集开销可以接受，对任务的收敛特征又不熟，就选 ReduceLROnPlateau。

## 单周期调度

前面所有调度都遵循同一个假设：学习率总体上向下降，预热只是开头很短一段。

OneCycleLR 把调度粒度从 epoch 改到 batch，让学习率先升到较高、再降到很低，常用于训练预算固定的从头训练。批量大小与优化器没有硬性限制，SGD 与 Adam 都可以搭配；微调时是否使用取决于任务与预算。两点设计与前面的调度不同：学习率与动量反向联动，学习率升时动量降、学习率降时动量升；`scheduler.step()` 必须放在 batch 循环里，而不是 epoch 循环里。

```python
from torch.optim.lr_scheduler import OneCycleLR

optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
total_steps = 100 * len(train_loader)      # 总 step 数 = epoch 数 × 每个 epoch 的 batch 数

scheduler = OneCycleLR(
    optimizer,
    max_lr=0.1,               # 学习率峰值
    total_steps=total_steps,
    pct_start=0.3,            # 前 30% 的 step 用于上升
    anneal_strategy='cos',    # 下降阶段用余弦形状
)

for epoch in range(100):
    for batch in train_loader:
        loss = train_step(batch)
        optimizer.step()
        scheduler.step()      # 每个 batch 后更新，这是与前面的调度最容易被忽略的差别
```

OneCycleLR 需要预先确定总训练步数，或通过 epoch 数与每轮步数推算。批大小没有固定下限；微调时也可使用，但需谨慎选择学习率峰值，避免破坏预训练参数。

按本例配置，OneCycleLR 与预热加余弦有三处差异：起始学习率为峰值的约二十五分之一，峰值为 0.1 时从 0.004 开始。前 30% 的 step 用于上升，而对照预热设置为 10%。末段下降到 $10^{-6}$ 量级。这些形状由参数决定，不是两类调度的固定差异。在所比较的 MNIST 设置中，两者结果接近。CIFAR-100 配 ResNet 的比较中，末段较低学习率对应约 0.3 到 0.8 个百分点的提升，不能推断其他任务也有同样收益。

## 怎么选调度策略

七种策略按调度依据与典型场景归位：

| 策略 | 调度依据 | 典型场景 |
| --- | --- | --- |
| Step decay | 固定间隔，人工设定 | 小规模任务，已有调参经验 |
| MultiStep decay | 指定节点，人工设定 | 对任务收敛特征熟悉 |
| Exponential decay | 每步乘固定系数 | 需要持续按比例降低学习率时 |
| Cosine Annealing | 余弦曲线，按步数执行 | 大规模预训练中的常见选择 |
| Cosine + Warm Restarts | 余弦曲线加周期重启 | 竞赛中需要反复试错时，需要跳出局部极小 |
| ReduceLROnPlateau | 验证集指标停滞时触发 | 中小规模任务，快速实验 |
| OneCycleLR | 单周期先升后降，按 step 执行 | 从头训练且计算预算有限 |

三条经验：大规模预训练常配合预热与余弦；中小规模训练或微调可以先用 ReduceLROnPlateau 观察验证集；从头训练且预算有限，用 OneCycleLR 配大 batch 与 SGD。
