---
title: "示例笔记：Attention Is All You Need (Transformer)"
date: 2026-08-06
draft: false
math: true
tags: ["Transformer", "注意力机制", "NLP", "深度学习"]
categories: ["论文笔记"]
summary: "一篇示例笔记，演示这个站点如何呈现公式、图片、标签和阅读结构。用你自己的第一篇笔记替换它即可。"
cover:
  image: ""          # 可选：填一张封面图 URL 或 static/ 下的路径
  alt: ""
  caption: ""
---

> **论文**：Vaswani et al., *Attention Is All You Need*, NeurIPS 2017.
> **链接**：[arXiv:1706.03762](https://arxiv.org/abs/1706.03762)
>
> 这是一篇**示例笔记**，用来展示排版能力（公式、图、代码、标签、目录）。
> 你可以把它删掉，或复制它作为自己第一篇笔记的模板。

## 一句话总结

用纯注意力机制替代循环与卷积，提出 Transformer 架构，在并行度和长程依赖建模上大幅超越 RNN/CNN 序列模型。

## 动机

RNN 必须按时间步串行计算，$h_t$ 依赖 $h_{t-1}$，无法并行，长序列训练慢且梯度难传。作者想要一个**完全可并行**、且能直接建模任意两个位置依赖关系的架构。

## 方法

核心是**缩放点积注意力（Scaled Dot-Product Attention）**：

$$
\mathrm{Attention}(Q, K, V) = \mathrm{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

其中 $Q, K, V$ 分别是 query、key、value 矩阵，$d_k$ 是 key 的维度，$\sqrt{d_k}$ 用于缩放以稳定梯度。行内公式如 $d_k = 64$ 也能正常渲染。

**多头注意力**把上式并行做 $h$ 次再拼接：

$$
\mathrm{MultiHead}(Q,K,V) = \mathrm{Concat}(\mathrm{head}_1, \dots, \mathrm{head}_h)\,W^O
$$

一段伪代码（演示代码高亮与复制按钮）：

```python
def scaled_dot_product_attention(q, k, v):
    d_k = q.size(-1)
    scores = q @ k.transpose(-2, -1) / d_k ** 0.5
    weights = scores.softmax(dim=-1)
    return weights @ v
```

如果要插入图片，Markdown 语法即可（把图片放到 `static/images/` 下，或直接引用外链）：

```markdown
![架构图](/images/transformer-arch.png)
```

## 实验与结论

- 在 WMT14 英德翻译上取得当时 SOTA（28.4 BLEU），训练成本远低于此前最好模型。
- 训练完全并行，8 张 GPU 12 小时即可得到有竞争力的模型。

## 我的思考

- **优点**：可并行、长程依赖建模强、结构简洁，后续几乎所有大模型的基座。
- **局限**：自注意力对序列长度是 $O(n^2)$ 复杂度，长文本开销大——催生了后续一大批高效注意力工作。
- **可延伸**：位置编码的设计、注意力的稀疏化、以及它为何如此可扩展，都值得单独开一篇笔记。
