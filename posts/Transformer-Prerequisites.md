---
title: Transformer Prerequisites
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-10-25 19:52:26
katex: true
---

# 文本的表示与基本运算

Transformer 先把文本变成向量，再根据上下文更新各位置的表示。理解这一过程，需要明确数据怎样排列、基本运算怎样改变表示，以及序列中的信息如何汇总。

公式统一采用行向量：一个 Token 对应一行，多个位置按行排列。

## 1. 标量、向量、矩阵与张量

标量是一个数，向量是一组有序数值，矩阵按行和列组织数值。在深度学习中，张量通常指多维数组，标量、向量和矩阵分别是零阶、一阶和二阶张量。

模型处理的离散文本单位称为 Token（词元），可以是词、子词或字符。分词（tokenization）按照分词器的规则切分文本，再为每个 Token 分配词表编号。一个自然语言词可能拆成多个 Token，因此模型的序列长度按 Token 数量计算。

编号还需转换为向量，才能参与后续运算。一个 Token 的向量包含 $d$ 个数，长度为 $n$ 的序列因此组成 $n\times d$ 的矩阵。这里的 $d$ 是表示维数，也常记为 $d_{\text{model}}$。

把 $B$ 条序列组成一个批次，并用补位符号 `<pad>` 补齐到相同长度 $n$，得到形状为 $B\times n\times d$ 的三阶张量。三个方向分别对应样本、位置和特征。

[html-card height=580](../assets/transformer-embedding-batch.html)


补齐使序列能够统一计算，但补位不属于真实文本。注意力不应从这些位置汇总内容，训练损失也不应把补位当成预测目标。具体的屏蔽方式在 [[Transformer-Prerequisites#9. Softmax|Softmax]] 一节说明。

> [!note] 阶数与维数
> 三阶张量表示有三个索引方向；一个 Token 的表示有 512 维，则表示该向量有 512 个分量。两种说法中的“维”含义不同。PyTorch 用 `Tensor` 统一表示这些数组。

## 2. 加法、点积与矩阵乘法

同形状的向量可以逐元素相加，例如 $[1,2]+[3,1]=[4,3]$。结果保留原来的形状，因此可以用于合并同一位置的两份表示，后面的残差连接就采用这种运算。

点积把两个等长向量变成一个数。例如 $[1,2]\cdot[3,1]=1\times3+2\times1=5$。向量的方向与长度都会影响点积；注意力机制用学习得到的向量计算匹配分数，再把这些分数转换成权重。

矩阵乘法同时计算许多点积。设 $\mathbf{A}\in\mathbb{R}^{m\times p}$、$\mathbf{B}\in\mathbb{R}^{p\times n}$，乘积 $\mathbf{C}=\mathbf{AB}$ 的形状为 $m\times n$，其中：

$$
C_{ij}=\sum_{k=1}^{p}A_{ik}B_{kj}
$$

每个 $C_{ij}$ 都是 $\mathbf{A}$ 的第 $i$ 行与 $\mathbf{B}$ 的第 $j$ 列的点积。前一个矩阵的列数与后一个矩阵的行数必须相同。

如果两组向量都按行排列，形状分别为 $m\times d$ 和 $n\times d$，第二组需要先转置成 $d\times n$，才能一次得到全部两两点积。结果是 $m\times n$ 的分数矩阵，注意力计算会用到这种两两打分。

[html-card height=820](../assets/transformer-matmul-cell.html)

## 3. Embedding

Token 是离散符号，编号只表示词表中的位置。编号 8 与编号 9 相邻，不代表对应的 Token 在意义上接近，因此不能把编号大小直接当作语义特征。

Embedding 为每个 Token 保存一个可学习向量。设词表大小为 $M$，表示维数为 $d$，嵌入矩阵为 $\mathbf{E}\in\mathbb{R}^{M\times d}$。编号为 $i$ 的 Token 取出第 $i$ 行，作为输入向量 $\boldsymbol{x}_i$。

例如词表有 50,000 个 Token、$d=512$，单个 Token 的 one-hot 表示需要 50,000 个分量，Embedding 则输出 512 个分量。查表与 one-hot 向量乘嵌入矩阵等价，但实现时无需构造整条 one-hot 向量。

训练损失会更新嵌入矩阵，使向量适应预测任务。语义关系可能表现为相近方向或相似偏移，但不保证每种关系都能用向量加减表达。编号、one-hot 与词向量的区别见 [[Word-Embeddings#词的表示问题|词的表示问题]]。

Embedding 提供初始表示，后续序列层再结合上下文更新表示。同一个 Token 在不同上下文中的最终表示因此可以不同。

查表得到的向量只取决于 Token 编号，不包含出现位置。模型还需要位置信息来区分序列顺序。一种常见做法是为每个位置构造一个向量，再与该位置的 Token 嵌入相加；两个向量必须同形状。顺序信息为何需要单独提供，见 [[Attention#位置信息|位置信息]]。

# 单个位置的特征变换

Embedding 把符号变成向量，但输入向量还需要根据任务重新组合特征。线性层改变表示，激活函数引入非线性，两者组成前馈网络。

## 4. 线性变换

对行向量 $\boldsymbol{x}\in\mathbb{R}^{1\times d_{\text{in}}}$，线性层的计算为：

$$
\boldsymbol{y}=\boldsymbol{x}\mathbf{W}+\boldsymbol{b}$$

$\mathbf{W}\in\mathbb{R}^{d_{\text{in}}\times d_{\text{out}}}$ 是可学习权重，$\boldsymbol{b}$ 有 $d_{\text{out}}$ 个分量，输出 $\boldsymbol{y}$ 的形状为 $1\times d_{\text{out}}$。严格来说，含偏置的 $\boldsymbol{x}\mathbf{W}+\boldsymbol{b}$ 是仿射变换；不含偏置的 $\boldsymbol{x}\mathbf{W}$ 才是线性变换。

权重决定输入特征如何组合，偏置改变输出的基准值。因此，即使输入与输出维数相同，线性层也能改变表示；改变维数只是线性层的用途之一。

对序列矩阵 $\mathbf{X}$，同一个 $\mathbf{W}$ 作用于每一行，偏置也加到每一行。不同位置共享参数，计算本身不混合位置。

同一份输入经过不同权重，还可以得到不同表示，后面的注意力计算会用到这一点。

## 5. 激活函数

只堆叠仿射层，整个网络仍能合并成一次仿射变换。以两层、暂不考虑偏置为例，$\boldsymbol{y}=(\boldsymbol{x}\mathbf{W}_1)\mathbf{W}_2=\boldsymbol{x}(\mathbf{W}_1\mathbf{W}_2)$，增加层数没有引入非线性。

在两次变换之间加入非线性激活函数，才能表达更复杂的关系。例如 ReLU 定义为：

$$

\operatorname{ReLU}(u)=\max(0,u)
$$
ReLU 逐元素把负值置为 0，保留正值。前一层先组合输入，激活函数逐元素改变这些组合的响应，后一层就能利用这种非线性结果。

不同激活函数的导数会影响梯度传播。这里采用 ReLU 说明结构；GELU 等选择及其差别见 [[Activation-Function-Comparison#带小斜率与门控的变体|带小斜率与门控的变体]]。

## 6. FFN

FFN 是前馈网络（Feed-Forward Network）。Transformer 中常见的逐位置 FFN 由两层仿射变换和中间的激活函数组成：

$$


\operatorname{FFN}(\mathbf{X})=\sigma(\mathbf{X}\mathbf{W}_1+\boldsymbol{b}_1)\mathbf{W}_2+\boldsymbol{b}_2$$

这里 $\sigma$ 表示激活函数。第一层把每个位置从 $d_{\text{model}}$ 维变换到 $d_{\text{ff}}$ 维，第二层恢复到 $d_{\text{model}}$ 维；例如 $512\rightarrow2048\rightarrow512$。中间层提供更多特征组合，非线性激活使整个过程无法合并成单层仿射变换。

同一层 FFN 对所有位置使用同一组参数，但每一行独立计算。不同位置的输出可以不同，因为输入向量不同；这个子层本身不进行位置之间的信息汇总。

[html-card height=490](../assets/transformer-ffn.html)


FFN 的输出形状与后面的 [[Transformer-Prerequisites#11. 残差连接|残差连接]] 有关。

# 序列的输入、输出与信息汇总

逐位置变换能处理每个 Token 的特征，但翻译等任务还需要组织输入与输出序列，并决定输出当前位置应利用哪些输入信息。

## 7. Encoder 与 Decoder

Encoder（编码器）把输入变成适合后续任务使用的表示；Decoder（解码器）根据这些表示和已有的输出，产生目标结果。两者描述的是计算分工，不限定必须使用循环网络或注意力。

在带注意力的序列模型中，编码器可以保留每个输入位置的表示，组成矩阵 $\mathbf{H}$。解码器生成目标序列时，按当前需求利用这些表示，不必只接收一个固定的汇总向量。

例如翻译时，编码器处理源句，解码器根据源句表示和已经生成的目标前缀预测后续 Token。逐步生成的过程见 [[Encoder-Decoder-And-Seq2Seq#生成：从 BOS 到 EOS|从 BOS 到 EOS 的生成过程]]。

## 8. Seq2Seq

Seq2Seq 是序列到序列（Sequence-to-Sequence）：输入与输出都是序列，但长度和顺序不必对应。Encoder-Decoder 描述一种结构分工，Seq2Seq 描述任务形式，两者不能直接等同。

[html-card height=820](../assets/seq2seq-topology.html)


例如，输入“桌子 底下 有 一只 猫”包含 5 个 Token，输出“There is a cat under the table”按示例划分包含 7 个 Token。“桌子底下”对应的内容还出现在输出后部，因此翻译需要重新组织信息，不能逐个替换输入 Token。

自回归生成以起始符号 `<bos>` 为条件预测第一个 Token，再把已经生成的前缀用于下一次预测，直到生成 `<eos>` 或达到长度上限。训练时目标序列已知，可以用真实前缀作为输入；这与推理时使用模型生成的前缀不同。

经典循环 Seq2Seq 把整条输入压缩为固定维度的上下文向量 $\boldsymbol{c}$，长序列的信息容易受到压缩限制。注意力保留各输入位置，再为不同输出位置计算不同的汇总结果，从而减轻对单个固定向量的依赖。

## 9. Softmax

注意力先为候选位置计算分数，但任意实数分数不能直接作为分配比例。Softmax 将一组分数变成非负且总和为 1 的权重：

$$
p_i=\frac{\exp(z_i)}{\sum_{j=1}^{r}\exp(z_j)}
$$

$z_i$ 是第 $i$ 个候选的分数，$r$ 是候选数量，分母对同一组候选求和。分数越大，权重越大；权重取决于整组分数，而非单个分数的绝对大小。

例如，分数 $[-1,-0.3,1.7,-0.3]$ 得到的权重约为 $[0.0502,0.1012,0.7475,0.1012]$。第三项占据约 74.75% 的权重，表示这次汇总更重视第三个候选。

实际计算常先减去最大分数，再取指数。整组分数减去同一个常数不改变 Softmax 结果，却能减少指数溢出的风险。

[html-card height=820](../assets/transformer-softmax-weights.html)

> [!note] 掩码与负无穷
> 掩码（mask）标记哪些位置允许参与计算。注意力中，补位位置的分数在 Softmax 前设为 $-\infty$。由于 $\exp(-\infty)=0$，这些位置的权重为 0，便不会参与后续加权汇总。每一行至少要有一个有效位置，才能正常归一化。
>
> 训练损失只对非 `<pad>` 目标求和或取平均，注意力掩码不会自动完成这一步。生成任务还会限制未来位置的可见性，见 [[Attention#并行的边界|解码器训练中的可见范围]]。

> [!note] 注意力权重与词表概率
> 注意力中的候选是序列位置，权重用于汇总这些位置的表示。输出层对词表 Token 给出的原始分数称为 logits，Softmax 将这些分数转换成下一 Token 的预测分布。两处使用同一运算，解释对象不同；注意力权重高不等于对应词即将被生成。

## 10. Attention

Attention 根据当前查询，为候选位置分配权重，再按权重汇总内容。查询 $\mathbf{Q}$ 表示当前需要匹配的信息，键 $\mathbf{K}$ 参与匹配，值 $\mathbf{V}$ 是实际汇总的内容。

自注意力通常用三组可学习权重，分别从同一序列的输入矩阵 $\mathbf{X}$ 产生查询、键和值：

$$
\mathbf{Q}=\mathbf{X}\mathbf{W}_Q,\qquad \mathbf{K}=\mathbf{X}\mathbf{W}_K,\qquad \mathbf{V}=\mathbf{X}\mathbf{W}_V$$

三组权重分别学习，因此即使来源相同，查询、键和值也不必相同。

设有 $m$ 个查询、$n$ 个候选位置，查询与键的维数都是 $d_k$。$\mathbf{Q}$ 的形状为 $m\times d_k$，$\mathbf{K}$ 的形状为 $n\times d_k$，因此 $\mathbf{QK^{\top}}$ 得到 $m\times n$ 的分数矩阵。每一行对应一个查询，每一列对应一个候选。

以未缩放的点积注意力为例，先计算匹配分数，再归一化并汇总值：

$$
\mathbf{A}=\operatorname{softmax}(\mathbf{QK^{\top}}),\qquad \mathbf{O}=\mathbf{AV}$$

Softmax 对每一行的 $n$ 个分数分别归一化。若 $\mathbf{V}$ 的形状为 $n\times d_v$，输出 $\mathbf{O}$ 的形状就是 $m\times d_v$。第 $i$ 个查询的输出为：

$$

\mathbf{O}_{i,:}=\sum_{j=1}^{n}A_{ij}\mathbf{V}_{j,:}$$

因而，不同查询即使使用同一组键和值，也可以得到不同的上下文表示。打分、归一化与加权求和的逐步解释见 [[Attention#打分、归一化与加权|一次注意力计算]]。

这里以一组 $\mathbf{Q},\mathbf{K},\mathbf{V}$ 说明输出投影。汇总结果 $\mathbf{O}$ 未必与输入同形状。若输入为 $m\times d_{\text{model}}$，可再用可学习的输出投影 $\mathbf{W}_O\in\mathbb{R}^{d_v\times d_{\text{model}}}$ 做线性映射。结果 $\mathbf{OW}_O$ 的形状为 $m\times d_{\text{model}}$，因此可以与输入残差相加。

查询来自解码器、键和值来自编码器时，汇总的是另一条序列的信息。三者都来自同一条序列时，这种机制称为自注意力。两种形式都在位置之间汇总信息，与逐位置独立计算的 FFN 分工不同。

> [!note] 从点积注意力到 Transformer
> Transformer 还会在 Softmax 前缩放点积分数，以控制数值幅度。缩放的理由与公式见 [[Attention#缩放|点积分数的缩放]] 。

[html-card height=820](../assets/attention-mechanism-slides.html)


# 深层网络中的残差与归一化

Attention 汇总位置之间的信息，FFN 变换各位置的特征。反复堆叠这两类子层时，还需要让原输入能够直接参与后续计算，并控制表示的数值尺度。

## 11. 残差连接

设子层输入为 $\mathbf{X}$，子层计算为 $F(\mathbf{X})$，残差连接按元素相加：

$$\mathbf{Y}=\mathbf{X}+F(\mathbf{X})$$

输入直接参与输出，子层只需学习相对于输入的修正。例如 $F(\mathbf{X})$ 接近 0 时，整个相加结果接近 $\mathbf{X}$，不要求子层重新构造完整输入。

$\mathbf{X}$ 与 $F(\mathbf{X})$ 必须同形状。这也解释了 FFN 为什么在中间扩展维数后，仍要恢复到 $d_{\text{model}}$ 维。

相加还为梯度提供一条不必经过 $F$ 内部全部变换的直接路径，有助于深层网络的优化。见 [[Residual-Networks#残差写法与直接路径|残差写法与直接路径]]。

## 12. LayerNorm

残差相加保留输入，但不会主动控制数值尺度。LayerNorm（层归一化）对每个 Token 的特征分别计算均值与方差，再标准化，使后续计算不必完全依赖输入的原始尺度。

对单个 Token 的向量 $\boldsymbol{x}=(x_1,\ldots,x_d)$，均值和方差为：

$$
\mu=\frac{1}{d}\sum_{j=1}^{d}x_j,\qquad
v=\frac{1}{d}\sum_{j=1}^{d}(x_j-\mu)^2
$$

随后逐分量计算：

$$
\hat{x}_j=\frac{x_j-\mu}{\sqrt{v+\epsilon}},\qquad
y_j=\gamma_j\hat{x}_j+\beta_j
$$

$\epsilon$ 是防止除零的小常数，$\gamma_j$ 与 $\beta_j$ 是可学习的缩放和平移参数。标准化后的均值为 0，方差在 $v$ 远大于 $\epsilon$ 时近似为 1；再经过缩放和平移，最终输出不必保持这两个数值。

对于 $B\times n\times d$ 的输入，每个样本、每个位置都用各自的 $d$ 个特征计算一组均值和方差，不跨 Token，也不跨批次样本。可学习参数则在位置与样本之间共享。与 BatchNorm 的区别见 [[Norm#两种归一化的对比|两种归一化的对比]]。

[html-card height=620](../assets/transformer-residual-layernorm.html)

