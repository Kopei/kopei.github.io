---
title: Transformer Intepretion and Code
comment: true
date: 2023-04-02 17:12:16
tags: transformer, AI
updated: 2023-04-11 18:12:00
---
## Attention is all your need
`Attention is all your need`这篇谷歌的`transformer`开山之作奠定了如今大热的GPT和机器视觉领域神经网络的基础架构。本文将在理解论文的基础上，结合其它材料，进一步深入了解具体代码实现（pytorch），并给出一个`fine tune`的实际应用例子。

### Paper Introduction
`Transformer`是第一个提出只使用`attention`+`residual connection`+`MLP`架构的神经网络, 起初论文用这种架构去做序列到序列的文本翻译工作，相比`RNN`和`CNN`, `transformer`在大规模训练集上的表现更好，同时这个架构也提高了计算并行性和计算效率。那么为什么它会表现地更好? 初步的研究发现，原因有如下几点：
- `transformer`在网络中引入了更少的`inductive bias`归纳偏置，所以有更好的泛化性，对于没有训练过的样本就有更好的表现; 
- 同时attention机制可以全局地计算出序列之间的相关性，相对CNN可以更好地理解上下文，而不是局限卷积窗口特征;
- 更少的网络深度减少了长距离传输梯度消失的问题，相对RNN就有更好的长输入表现;
- 当然`transformer`引入残差连接也是性能提升的另一个因素，具体还有其他原因分析还待进一步研究。

### Model Architecture
`transformer`使用了`encoder-decoder`编码器-解码器架构, 整体架构论文完美地画了出来。实现的代码如下:
![arch](/images/screenshots/transformer_arch.png)
```python
import torch.nn as nn
from torch.nn.functional import log_softmax

class EncoderDecoder(nn.Module):
    """
    Standard encoder decoder class.
    """
    def __init__(self, encoder, decoder, src_embed, tgt_embed, generator):
        super(EncoderDecoder, self).__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.src_embed = src_embed
        self.tgt_embed = tgt_embed
        self.generator = generator

    def encode(self, src, src_mask):
        return self.encoder(self.src_embed(src), src_mask)

    def decode(self, memory, src_mask, tgt, tgt_mask):
        """memory is feed by encoder output as interim input for decoder"""
        return self.decoder(self.tgt_embed(tgt), memory, src_mask, tgt_mask)

    def forward(self, src, tgt, src_mask, tgt_mask):
        return self.decode(self.encode(src, src_mask), src_mask, tgt, tgt_mask)

class Generator(nn.Module):
    """standard linear + softmax generator"""
    def __init__(self, d_model, vocab):
        super(Generator, self).__init__()
        self.projection = nn.Linear(d_model, vocab)

    def forward(self, x):
        return log_softmax(self.projection(x), dim=-1)
```
上图中编码器和解码器实际上是由6个一样的网络串行堆叠而成，所以如果只看编解码器，它的大致架构如下图:
![stacked_transformer](/images/screenshots/stacked_encoder_decoder.webp)

#### 编码器输入
对于翻译序列的例子，在数据输入阶段经过了如下流程：
1. 当输入句子X(x1,x2...xn), 模型先对其做`embedding`,生成每个词对应的向量Z(z1,z2...zn). 
2. 然后在Z后加入位置编码信息，位置信息可以用余弦来表示，随后加入编码器输入。
```python
import math

class Embeddings(nn.Module):
    def __init__(self, d_model, vocab):
        super(Embeddings, self).__init__()
        self.lut = nn.Embedding(vocab, d_model)  # look up table
        self.d_model = d_model

    def forward(self, x):
        return self.lut(x) * math.sqrt(self.d_model)
    
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, dropout, max_len=5000):
        """dropout is random set to zero in some points to prevent overfitting"""
        super(PositionalEncoding, self).__init__()
        self.dropout = nn.Dropout(p=dropout)

        pe = torch.zeros(max_len, d_model) # 5000*512 tensor
        position = torch.arange(0, max_len).unsqueeze(1) # 5000*1
        div_term = torch.exp(torch.arange(0, d_model, 2) * -(math.log(10000.0)/d_model))
        # 用指数转变把除法改成乘法, -1幂
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
编码器由6个一样的网络串行组成，每个网络由一个多头注意力层和`Feed forward`前向反馈网络层组成。
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

#### 多头注意层
多头注意层是输入（K,V,Q）经过8个可学习的线性投影，再经过并行的自注意机制（点积计算），最后把结果相加并且投影缩放回来。这篇论文用的自注意力机制是`scaled dot product`即阶化的点积。整体自注意力层和多头层论文也很好地画了出来。
![attention](/images/screenshots/multihead.png)
自注意计算的公式是: {% mathjax %} \mathrm{Attention}(Q, K, V) = \mathrm{softmax}(\frac{QK^T}{\sqrt{d_k}})V {% endmathjax %}
```python
def attention(query, key, value, mask=None, dropout=None):
    d_k = query.size(-1) # dim number
    scores = torch.matmul(query, key.transpose(-2, -1)) / math.sqrt(d_k)
    if mask is not None:
        scores = scores.masked_fill(mask==0, -1e9) # 掩码是0的位置赋值很小的数
    p_attn = scores.softmax(dim=-1)
    if dropout is not None:
        p_attn = dropout(p_attn) # set p_attn% to 0
    return torch.matmul(p_attn, value), p_attn
```

多头注意力机制的公式是: 
{% mathjax %} \mathrm{MultiHead}(Q, K, V) =
    \mathrm{Concat}(\mathrm{head_1}, ..., \mathrm{head_h})W^O \\
    \text{ where }~\mathrm{head_i} = \mathrm{Attention}(QW^Q_i, KW^K_i, VW^V_i) {%endmathjax%}
**Where** the projections are parameter matrices:
{% mathjax %} W^Q_i \in
\mathbb{R}^{d_{\text{model}} \times d_k}, W^K_i \in
\mathbb{R}^{d_{\text{model}} \times d_k}, W^V_i \in
\mathbb{R}^{d_{\text{model}} \times d_v}  and  W^O \in
\mathbb{R}^{hd_v \times d_{\text{model}}}. {% endmathjax %}

多头注意力代码如下:
```python
import copy
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, h=8, dropout=0.1):
        super(MultiHeadAttention, self).__init__()
        self.d_k = d_model // h
        self.h = h
        self.linears = clones(nn.Linear(d_model, d_model), 4)
        self.attn = None
        self.dropout = nn.Dropout(p=dropout)

    def forward(self, query, key, value, mask=None):
        if mask is not None:
            mask = mask.unsqueeze(1)
        nbatches = query.size(0)
        # projection to h * d_k, split by adding dimension
        query, key, value = [
            lin(x).view(nbatches, -1, self.h, self.d_k).transpose(1,2)
            for lin, x in zip(self.linears, (query, key, value))
        ]
        # self attention
        x, self.attn = attention(query, key, value, mask=mask, dropout=self.dropout)
        # concate and linear again
        x = (x.transpose(1,2).contiguous().view(nbatches, -1, self.h * self.d_k))
        del query
        del key
        del value
        return self.linears[-1](x)
```

#### 前馈网络
前向反馈网络就是一个2层感知机MLP，经过一层ReLU然后再经过一层线性层, 公式如下: 
{% mathjax %}\mathrm{FFN}(x)=\max(0, xW_1 + b_1) W_2 + b_2{% endmathjax %}
```python
class PositionwiseFeedForward(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super(PositionwiseFeedForward, self).__init__()
        self.w1 = nn.Linear(d_model, d_ff)
        self.w2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        return self.w2(self.dropout(self.w1(x).relu()))
```

#### LayerNorm
在每个子层输出还需要做layerNorm和残差连接. LayerNorm的大致公式如下图, LayerNorm主要是将样本值在特征维度上做归一化处理:
![layerNorm](/images/screenshots/layernorm.png)

```python
class LayerNorm(nn.Module):
    def __init__(self, features, eps=1e-6):
        super(LayerNorm, self).__init__()
        self.gamma = nn.Parameter(torch.ones(features))
        self.beta = nn.Parameter(torch.zeros(features))
        self.eps = eps

    def forward(self, x):
        # 计算均值和方差
        mean = x.mean(dim=-1, keepdim=True)
        var = x.var(dim=-1, keepdim=True)
        # 归一化
        x = (x - mean) / torch.sqrt(var + self.eps)
        # 线性化
        x = self.gamma.unsqueeze(-1) * x + self.beta.unsqueeze(-1)
        return x
```

#### 残差连接
每个子层的输出将通过残差连接，然后再通过`LayerNorm`, 这里代码实现将先进行归一化然后再做dropout[(cite)](http://jmlr.org/papers/v15/srivastava14a.html)最后再进行残差连接, 目的是为了代码简单。
子层输出经过公式: {% mathjax %} $\mathrm{LayerNorm}(x + \mathrm{Sublayer}(x))$, where $\mathrm{Sublayer}(x)$ {% endmathjax %} 
```python
class SublayerConnection(nn.Module):
    """
    A residual connection followed by a layer norm.
    Note for code simplicity the norm is first as opposed to last.
    """
    def __init__(self, size, dropout):
        super(SublayerConnection, self).__init__()
        self.norm = LayerNorm(size)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, sublayer):
        "Apply residual connection to any sublayer with the same size."
        return x + self.dropout(sublayer(self.norm(x)))
```

整个编码层代码如下：
```python
class EncoderLayer(nn.Module):
    def __init__(self, size, self_attn, feed_forward, dropout):
        super().__init__()
        self.self_attn = self_attn
        self.feed_forward = feed_forward
        self.sublayer = clones(SublayerConnection(size, dropout), 2)
        self.size = size

    def forward(self, x, mask):
        x = self.sublayer[0](x, lambda x: self.self_attn(x, x, x, mask))
        return self.sublayer[1](x, self.feed_forward)
```

#### 解码器层
解码器层比编码器多了一个带`mask`的多头注意力层，同时编码器的输出也会是解码器的输入，另一个输入是编码器按位之前已经生成结果。
```python
class Decoder(nn.Module):
    def __init__(self, layer, N):
        super().__init__()
        self.layers = clones(layer, N)
        self.norm = LayerNorm(layer.size)
    
    def forward(self, x, memory, src_mask, tgt_mask):
        for layer in self.layers:
            x = layer(x, memory, src_mask, tgt_mask)
        return self.norm(x)

class DecoderLayer(nn.Module):
    def __init__(self, size, self_attn, src_attn, feed_forward, dropout):
        """(k,v) from sources called src_attn"""
        super().__init__()
        self.size = size
        self.self_attn = self_attn
        self.src_attn = src_attn
        self.feed_forward = feed_forward
        self.sublayer = clones(SublayerConnection(size, dropout), 3)

    def forward(self, x, memory, src_mask, tgt_mask):
        m = memory
        x = self.sublayer[0](x, lambda x: self.self_attn(x, x, x, tgt_mask))
        x = self.sublayer[1](x, lambda x: self.src_attn(x, m, m, src_mask))
        return self.sublayer[2](x, self.feed_forward)

def subsequent_mask(size):
    "Mask out subsequent positions."
    attn_shape = (1, size, size)
    subsequent_mask = torch.triu(torch.ones(attn_shape), diagonal=1).type(
        torch.uint8
    )
    return subsequent_mask == 0
```

### 整体模型
```python
def make_model(
    src_vocab, tgt_vocab, N=6, d_model=512, d_ff=2048, h=8, dropout=0.1
):
    "Helper: Construct a model from hyperparameters."
    c = copy.deepcopy
    attn = MultiHeadedAttention(h=h, d_model=model)
    ff = PositionwiseFeedForward(d_model=model, d_ff=d_ff, dropout=dropout)
    position = PositionalEncoding(d_model, dropout)
    model = EncoderDecoder(
        Encoder(EncoderLayer(d_model, c(attn), c(ff), dropout), N),
        Decoder(DecoderLayer(d_model, c(attn), c(attn), c(ff), dropout), N),
        nn.Sequential(Embeddings(d_model, src_vocab), c(position)),
        nn.Sequential(Embeddings(d_model, tgt_vocab), c(position)),
        Generator(d_model, tgt_vocab),
    )

    # This was important from their code.
    # Initialize parameters with Glorot / fan_avg.
    for p in model.parameters():
        if p.dim() > 1:
            nn.init.xavier_uniform_(p)
    return model

# 没有训练的模型测试记录能力，结果是不能记忆输入。
def inference_test():
    test_model = make_model(11, 11, 2)
    test_model.eval()
    src = torch.LongTensor([[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]])
    src_mask = torch.ones(1, 1, 10)

    memory = test_model.encode(src, src_mask)
    ys = torch.zeros(1, 1).type_as(src)

    for i in range(9):
        out = test_model.decode(
            memory, src_mask, ys, subsequent_mask(ys.size(1)).type_as(src.data)
        )
        prob = test_model.generator(out[:, -1])
        _, next_word = torch.max(prob, dim=1)
        next_word = next_word.data[0]
        ys = torch.cat(
            [ys, torch.empty(1, 1).type_as(src.data).fill_(next_word)], dim=1
        )

    print("Example Untrained Model Prediction:", ys)

def run_tests():
    for _ in range(10):
        inference_test()

def show_example(fn, args=[]):
    if __name__ == "__main__":
        return fn(*args)

show_example(run_tests)
```

