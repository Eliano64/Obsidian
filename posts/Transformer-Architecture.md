---
title: Transformer Architecture
tags:
  - Transformer
categories:
  - Artificial Intelligence
date: 2026-11-10 13:35:00
katex: true
---

**Transformer** 是彻底舍弃循环递推、完全基于自注意力构建序列全局交互的模型架构，同时支持编码阶段的并行计算与解码阶段的自回归生成。

[html-card height=800](../assets/transformer-architecture.html)

原始 Encoder-Decoder 架构自 2017 年提出以来，现代主流大语言模型（LLaMA、Qwen、Mistral、DeepSeek 等）在五个核心部件上完成了系统升级：

| 部件 | 原始 Transformer（2017） | 现代大语言模型主流配置 |
| --- | --- | --- |
| 宏观拓扑 | Encoder-Decoder | [[Transformer-Variants#Decoder-Only 路线的架构优势]] |
| 归一化位置与算子 | Post-Norm + LayerNorm | [[Encoder-Workflow#Pre-Norm]] + [[Norm#RMSNorm]] |
| 位置编码 | 输入端相加正弦编码 | [[Position-Encoding#旋转位置编码（RoPE）]] |
| 前馈网络激活 | 两矩阵 FFN + ReLU | [[Activation-Function-Comparison#带小斜率与门控的变体]] |
| 注意力缓存与内核 | 标准 MHA + 朴素显存读写 | [[Multi-Head-Attention#MQA、GQA 与 MLA（KV Cache 的结构优化）]]、[[Attention-In-Transformer#FlashAttention：硬件感知精确注意力]] |
