# 15 — Transformers & Attention

Recap from file 11's honest closing note: RNNs process sequences one step at a time, which is slow and still struggles with long-range dependencies. Transformers fix both problems with one core mechanism — **attention** — and now power essentially every major modern language model.

## The core problem attention solves

In a sentence, understanding a word often depends on OTHER words that could be far away — "The trophy didn't fit in the suitcase because **it** was too big" — resolving what "it" refers to requires connecting back to "trophy," several words earlier. An RNN has to carry that information forward through every intermediate hidden state, and it can degrade over distance even with LSTM's gating. Attention lets the model look directly at ANY other word in the sequence, regardless of distance, in a single step.

## Self-attention — the mechanism itself, conceptually

For each word, self-attention computes three vectors: **Query**, **Key**, and **Value** (Q, K, V). The intuition, genuinely worth internalizing rather than just memorizing:
- **Query** — "what am I looking for?" (from the current word's perspective)
- **Key** — "what do I contain?" (from every other word's perspective)
- **Value** — "what information do I actually offer if picked?"

The attention score between two words is computed by comparing one word's Query against another word's Key (dot product) — high similarity means "pay more attention here." Those scores get normalized (softmax) into weights that sum to 1, then used to compute a weighted sum of all the Value vectors. Every word ends up with a new representation that's a blend of every other word's information, weighted by relevance.

## The actual formula

```
Attention(Q, K, V) = softmax(Q @ K^T / sqrt(d_k)) @ V
```

`d_k` is the dimension of the key vectors — dividing by `sqrt(d_k)` keeps the dot products from growing too large as dimensionality increases, which would otherwise push softmax into regions with vanishing gradients (recap file 05's sigmoid/tanh saturation point — same underlying numerical-stability concern, different mechanism).

## Implementing scaled dot-product attention in raw PyTorch — worth doing once

```python
import torch
import torch.nn.functional as F

def attention(Q, K, V):
    d_k = Q.size(-1)
    scores = Q @ K.transpose(-2, -1) / (d_k ** 0.5)
    weights = F.softmax(scores, dim=-1)
    return weights @ V, weights

seq_len, d_k = 5, 8
Q = torch.rand(1, seq_len, d_k)
K = torch.rand(1, seq_len, d_k)
V = torch.rand(1, seq_len, d_k)

output, attn_weights = attention(Q, K, V)
print(output.shape)         # [1, 5, 8]
print(attn_weights.shape)   # [1, 5, 5] — how much each word attends to every other word
```

Genuinely worth printing `attn_weights` and looking at it as a small grid — each row sums to 1 (softmax), and the values show exactly which positions are "attending to" which other positions. This grid is precisely what gets visualized in attention-heatmap plots seen in transformer papers/blogs.

## Multi-head attention — running several attention "perspectives" in parallel

```python
mha = torch.nn.MultiheadAttention(embed_dim=64, num_heads=8, batch_first=True)

x = torch.rand(1, 5, 64)   # batch=1, seq_len=5, embed_dim=64
output, attn_weights = mha(x, x, x)   # self-attention: query, key, value are all the same input
print(output.shape)   # [1, 5, 64]
```

Instead of one attention computation, multi-head attention splits the embedding into several smaller chunks ("heads") and runs attention independently on each — genuinely lets different heads specialize in different kinds of relationships (one head might learn to track grammatical subject-verb agreement, another might track coreference like "it"/"trophy" above). Results get concatenated back together at the end.

## Positional encoding — the piece that stops attention from being order-blind

Attention treats the sequence as an unordered set by default — nothing in the Q/K/V math above knows word order. Positional encodings inject that information back in:

```python
import torch
import math

def positional_encoding(seq_len, d_model):
    pe = torch.zeros(seq_len, d_model)
    position = torch.arange(0, seq_len).unsqueeze(1).float()
    div_term = torch.exp(torch.arange(0, d_model, 2).float() * -(math.log(10000.0) / d_model))
    pe[:, 0::2] = torch.sin(position * div_term)
    pe[:, 1::2] = torch.cos(position * div_term)
    return pe
```

The sine/cosine pattern isn't arbitrary — it gives every position a unique, smoothly-varying signature that the model can learn to interpret as "distance" between positions. In practice, added directly to the word embeddings before they enter the first attention layer.

## A minimal Transformer encoder block using PyTorch's built-in layer

```python
encoder_layer = torch.nn.TransformerEncoderLayer(d_model=64, nhead=8, batch_first=True)
transformer_encoder = torch.nn.TransformerEncoder(encoder_layer, num_layers=6)

x = torch.rand(2, 10, 64)   # batch=2, seq_len=10, embed_dim=64
output = transformer_encoder(x)
print(output.shape)   # [2, 10, 64]
```

`nn.TransformerEncoderLayer` bundles multi-head attention + a feedforward MLP + layer normalization + residual connections into one reusable block — this exact block, stacked `num_layers` times, is genuinely the core of BERT-style models. GPT-style models use a similar but "decoder-only" variant with masked attention (each position can only attend to earlier positions, not future ones — necessary for autoregressive text generation).

## Why transformers train faster than RNNs despite doing "more" math

Recap from file 11's closing note — RNNs process one time step at a time sequentially, can't parallelize across the sequence dimension. Attention computes relationships between ALL positions simultaneously via matrix multiplication (`Q @ K^T`) — genuinely fully parallelizable across the sequence, which is precisely why transformers can be trained on much larger datasets in reasonable time on GPUs, and is the practical reason they scaled to the huge models (GPT, BERT, etc.) that now dominate NLP.

## Try this

```python
# Take the attention() function above, feed it a Q/K/V where one specific position's Query
# is deliberately made very similar to only ONE other position's Key (rest random/dissimilar)
# Print the resulting attn_weights and confirm that position's row is heavily weighted toward
# the matching Key's position
```

Manually engineering a case where attention weights concentrate strongly on one specific position is genuinely the clearest way to confirm the mechanism is doing exactly what the "Query looks for matching Key" intuition describes, not just trusting the formula blindly.

## What's next
File 16 — Autoencoders, the first fully generative/representation-learning architecture in this roadmap, moving beyond pure classification/prediction tasks.
