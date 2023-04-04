---
title: Transformer Architecture Code
comment: true
date: 2023-04-02 17:12:16
tags: transformer, AI
---
## Attention is all your need
`Attention is all your need`这篇谷歌的transformer开山之作奠定了如今大热的GPT和机器视觉领域神经网络架构的基础。本文将在理解论文的基础上，结合其它材料，进一步深入了解具体代码实现（pytorch），并给出一个`fine tune`的实际应用例子。

### Paper Introduction
`Transformer`是第一个提出只使用`attention`+`residual connection`+`MLP`架构的神经网络, 起初论文用这种架构去做序列到序列的文本翻译工作，相比`RNN`和`CNN`, `transformer`在大规模训练集上的表现更好，同时这个架构也提高了计算并行性和计算效率。那么为什么会表现地更好? 初步的研究发现，原因有如下几点：
- `transformer`在网络中引入了更少的`inductive bias`归纳偏置，所以有更好的泛化性，对于没有训练过的样本就有更好的表现; 
- 同时attention机制可以全局地计算出输入序列之间的相关性，相对CNN可以更好地理解上下文，而不是局限窗口特征;
- 更少的网络深度减少了长距离传输梯度消失的问题，相对RNN就有更好的长输入表现;
- 当然`transformer`引入残差连接也是性能提升的另一个因素，具体还有其他原因分析还待进一步研究。

### Model Architecture
`transformer`使用了`encoder-decoder`编码器-解码器架构, 整体架构论文很好地画出了。代码如下:
![arch](/images/screenshots/transformer_arch.png)
```python
import torch.nn as nn
from torch.nn.functional import log_softmax

class EncoderDecoder(nn.Moudle):
    """
    Standard encoder decoder class.
    """
    def __init__(self, encoder, decoder, src_embed, tgt_embed, generator):
        super(EncoderDecoder, self).__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.src_embed = embed
        self.tgt_embed = tgt_embed
        self.generator = generator

    def encode(self, src, src_mask):
        return self.encoder(self.src_embed(src), src_mask)

    def decode(self, memory, src_mask, tgt, tgt_mask):
        """memory is the input from encoder"""
        return self.decoder(self.tgt_embed(tgt), memory, src_mask, tgt_mask)

    def forward(self, src, tgt, src_mask, tgt_mask):
        return self.decode(self.encode(src, src_mask), src_mask, tgt, tgt_mask)

class Generator(nn.Module):
    """standard linear + softmax generator"""
    def __init__(self, d_model, vocab):
        super(self, Generator, self).__init__()
        self.projection = nn.Linear(d_model, vocab)

    def forward(self, x):
        return log_softmax(self.projection(x), dim=-1)

```

#### 编码器输入
对于翻译序列的例子，在数据输入阶段经过了如下流程：
1. 当输入句子X(x1,x2...xn), 模型先对其做`embedding`,生成每个词对应的向量Z(z1,z2...zn). 
2. 然后在Z后加入位置编码信息，位置信息可以用余弦来表示，随后进入编码器输入。
```python
import math
class Embedding(nn.Module):
    def __init__(self, d_model, vocab):
        super(Embedding, self).__init__()
        self.lut = nn.Embedding(vocab, d_model)
        self.d_model = d_model

    def forward(self, x):
        """ 
        same weight matrix between the two embedding layers and the pre-softmax linear transformation. this is multiply sqrts(d_modle) """
        return self.lut(x) * math.sqrt(self.d_model)
    
class PositionalEmbedding(nn.Module):
    def __init__(self, d_model, dropout, max_len=5000):
        """dropout is random set to zero in some points to prevent overfitting"""
        super(PositionalEmbedding, self).__init__()
        self.dropout = nn.Dropout(p=dropout)

        # 5000*512 tensor
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2) * -(math.log(10000.0)/d_model))
        # 用指数转变把除法改成乘法
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        # 保存状态
        self.register_buffer("pe", pe)

    def forward(self, x):
        # todo why requires gradients?
        return self.dropout(x + self.pe[:, : x.size(1)].requires_grad_(False))
```

#### 编码器层
编码器由6个一样的网络组成，每个网络由一个多头注意力层和`Feed forward`前向反馈层组成。
```python
def clones(module, N):
    return nn.ModuleList([copy.deepcopy(module) for _ in range(N)])

class Encoder(nn.Module):
    def __init__(self, layer, N):
        super(Encoder, self).__init__()
        self.layers = clones(layer, N)
        self.norm = LayerNorm(layer.size)

    def forward(self, x, mask):
        for layer in self.layers:
            x = layer(x, mask)
        return self.norm(x)
```

多头注意层是6个经过线性投影(可学习的)的K,V,Q，再做并行做自注意机制，最后把结果相加并且投影回来的输出。这篇论文用的自注意力机制是`scaled dot product`即阶化的点乘。整体自注意力层和多头层论文也很好地画了出来。
![attention](/images/screenshots/multihead.png)
自注意计算的公式是: {% mathjax %} \mathrm{Attention}(Q, K, V) = \mathrm{softmax}(\frac{QK^T}{\sqrt{d_k}})V {% endmathjax %}
```python
def attention(query, key, value, mask=None, dropout=None):
    d_k = query.size(-1) # dim number
    scores = torch.matmul(query, key.transpose(-2, -1)) / math.sqrt(d_k)
    if mask i not None:
        scores = scores.masked_fill(mask==0, -1e9) # 掩码是0的位置赋值很小的数
    p_attn = scores.softmax(dim=-1)
    if dropout is not None:
        p_attn = dropout(p_attn) # set p_attn% to 0
    return torch.matmul(p_attn, value), p_attn
```
多头注意力机制的公式是: 
{% mathjax %} \mathrm{MultiHead}(Q, K, V) =
    \mathrm{Concat}(\mathrm{head_1}, ..., \mathrm{head_h})W^O \\
    \text{ where}~\mathrm{head_i} = \mathrm{Attention}(QW^Q_i, KW^K_i, VW^V_i) {%endmathjax%}
Where the projections are parameter matrices:
{% mathjax %} W^Q_i \in
\mathbb{R}^{d_{\text{model}} \times d_k}, W^K_i \in
\mathbb{R}^{d_{\text{model}} \times d_k}, W^V_i \in
\mathbb{R}^{d_{\text{model}} \times d_v} and W^O \in
\mathbb{R}^{hd_v \times d_{\text{model}}}. {% endmathjax %}

前向反馈网络就是一个多层感知机MLP，经过一层ReLU然后再经过一层线性层, 公式如下: {% mathjax %}\mathrm{FFN}(x)=\max(0, xW_1 + b_1) W_2 + b_2{% endmathjax %}


在每个子层输出还需要做layerNorm和残差连接. LayerNorm的大致公式如下图:
![layerNorm](/images/screenshots/layernorm.png)

```python
class LayerNorm(nn.Module):
    def __init__(self, features, eps=1e-6):
        super(LayerNorm, self).__init__()
        self.gamma

```