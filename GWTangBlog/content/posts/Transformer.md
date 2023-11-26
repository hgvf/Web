+++
title = 'Attention is All You Need'
date = 2023-11-26T00:52:32+08:00
category = ["Papers"]
tags = ["Deep-learning", "Machine Translation"]
+++


* https://arxiv.org/pdf/1706.03762.pdf

*    **Model architecture**:

![Transformer network](https://i.imgur.com/ktilrsi.png)

    (1) 每個 sub-layer 過後都會經過 **layer normalization**，讓 input 的 mean=0, variance=1，讓資料都投影到差不多的 range，可以提升 training 效率。
    (2) 任兩個 sub-layers 間都使用 **residual connection**，在做 backpropagation 時可以使梯度不要爆炸或消失，也不會使 information 完全由 layer 決定，而是多參考初始狀態。
    (3) 經過完每個 sub-layer 之後，在 residual connection & layer normalization 之前，會先 dropout(0.1)，甚至是 input embedding 也會經過 dropout。
    (4) 使用 37k 大小的字典做 byte-pair encoding。
    (5) 最終結果是用倒數 10 個 checkpoints 的 model weights 作 average。
    (6) Label smoothing=0.1。
---
*    **Attention**: 

![Attention](https://i.imgur.com/oMONStd.png)

    (1) Scaled Dot-Production attention:
    $Attention(Q, K, V)=softmax(\frac{QK^T}{\sqrt[]{d_k}})V$
    (2) Multi-head attention:
    $MultiHead(Q, K, V)=Concat(head_1, ..., head_h)W^O$
    $head_i=Attention(QW_i^Q, KW_i^K, VW_i^V)$
    
    - 其中 key, value 都是 embedding size / nhead。
    - scale 的目的則是因為若 dmodel 很大，則 query, key 內積之 variance 會很大。
    (因為若原本 query & key 都是 random，則初始 mean=0, variance=0，內積過後 mean=0, variance=dmodel)

---
*    **Positional Encoding**:

    利用 sine, cosine 算出不需要訓練的數值:
    $PE_(pos, 2i) = sin(\frac{pos}{10000^\frac{2i}{d_{model}}})$
    $PE_(pos, 2i+1) = cos(\frac{pos}{10000^\frac{2i}{d_{model}}})$
    
    - 希望可以應付很長的 sequence。
    - 也確保相同距離的 tokens，但整體 sequence 長度不同，一樣有相同的 position embedding 差異。
    (ex. 兩個 tokens 在 Seq1(length=5) 和 Seq2(length=512)都差距 3 個 tokens，則 position information 也保證一樣。)
    - 也就是偶數維度使用 sin 計算出的數值，奇數維度使用 cos 計算的數值。
    - 同樣也會經過 dropout。

---
*    **Noam learning rate scheduler**:

    $lrate=d^{-0.5}_{model}  \cdot  min(step\_num^{-0.5}, step\_num \cdot warmup\_step^{-1.5})$
    
    - 也就是在 warmup 期間使用極小的 learning rate，之後才用較大的 learning rate 更新梯度。