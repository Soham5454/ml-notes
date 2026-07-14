# 11 — RNNs, LSTMs & Sequence Models

CNNs (file 10) exploit spatial structure in images. Sequence data — text, time series, audio — has a different kind of structure: **order matters, and earlier elements affect later ones.** RNNs are the architecture family built specifically for that.

## Why a normal MLP/CNN doesn't work for sequences

An MLP treats every input feature as independent and fixed-position. But sentences and time series have variable length, and the meaning of a word/value genuinely depends on what came before it ("bank" means something different after "river" vs after "money"). RNNs solve this by keeping a **hidden state** that gets updated at every step and carries information forward.

## The basic RNN cell — the core idea in one formula

At each time step `t`: `hidden_t = tanh(W_input * x_t + W_hidden * hidden_{t-1} + b)`

The same weights (`W_input`, `W_hidden`) get reused at every time step — genuinely another case of parameter sharing, same principle as CNN filters sliding across an image, just sliding across time instead of space here.

```python
import torch
import torch.nn as nn

rnn = nn.RNN(input_size=10, hidden_size=20, batch_first=True)

x = torch.randn(8, 5, 10)   # batch=8, sequence_length=5, features=10
output, hidden = rnn(x)
print(output.shape)   # [8, 5, 20] — hidden state at every time step
print(hidden.shape)   # [1, 8, 20] — final hidden state only
```

`batch_first=True` matters — without it, PyTorch defaults to `[seq_len, batch, features]` ordering, which trips people up constantly since every other layer expects batch-first. Setting it explicitly is a genuinely good habit.

## The vanishing gradient problem in RNNs — why plain RNNs are rarely used now

Backpropagation through a long sequence means multiplying gradients through many time steps. If those gradients are consistently less than 1 (common with tanh), they shrink toward zero over long sequences — the network effectively "forgets" anything from many steps ago. This is genuinely the same vanishing gradient concept from file 05 (sigmoid/tanh saturating), just compounding across time instead of across layers.

## LSTM — the fix, with actual memory management

```python
lstm = nn.LSTM(input_size=10, hidden_size=20, batch_first=True)

x = torch.randn(8, 5, 10)
output, (hidden, cell) = lstm(x)
print(output.shape)   # [8, 5, 20]
print(cell.shape)      # [1, 8, 20] — the "cell state," genuinely the key addition over plain RNN
```

LSTM introduces a separate **cell state** (long-term memory) alongside the hidden state (short-term/output), controlled by three learned "gates":
- **Forget gate** — decides what to throw away from the cell state
- **Input gate** — decides what new information to add
- **Output gate** — decides what to output based on the cell state

Don't need to memorize the gate equations by heart, but the concept genuinely matters: LSTM can choose to preserve information across long sequences instead of it decaying automatically, which is exactly what fixes the vanishing gradient issue for long-range dependencies.

## GRU — a simpler, faster alternative to LSTM

```python
gru = nn.GRU(input_size=10, hidden_size=20, batch_first=True)
```

GRU merges the forget/input gates into a single "update gate" and drops the separate cell state — fewer parameters, trains faster, and in practice performs comparably to LSTM on many tasks. Genuinely a reasonable default to try first; LSTM if GRU underperforms.

## A full example — sentiment-style sequence classification

```python
class SentimentRNN(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_size, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_size, batch_first=True)
        self.fc = nn.Linear(hidden_size, num_classes)

    def forward(self, x):
        embedded = self.embedding(x)              # [batch, seq_len] -> [batch, seq_len, embed_dim]
        output, (hidden, cell) = self.lstm(embedded)
        last_hidden = hidden[-1]                    # final hidden state, [batch, hidden_size]
        logits = self.fc(last_hidden)
        return logits

model = SentimentRNN(vocab_size=5000, embed_dim=64, hidden_size=128, num_classes=2)
```

`nn.Embedding` is genuinely new here — text can't go directly into a network as raw word strings, so each word gets mapped to an integer index (vocabulary lookup), then `nn.Embedding` converts that index into a learned dense vector. This embedding is trained jointly with the rest of the network — the model literally learns which words are semantically similar based on the task, without being told any dictionary/thesaurus ahead of time.

## Bidirectional RNNs — reading both directions

```python
bilstm = nn.LSTM(input_size=10, hidden_size=20, batch_first=True, bidirectional=True)
x = torch.randn(8, 5, 10)
output, _ = bilstm(x)
print(output.shape)   # [8, 5, 40] — 20 forward + 20 backward, concatenated
```

Useful when the whole sequence is available upfront (not real-time streaming) — e.g. for classifying a full sentence, knowing what comes *after* a word can help interpret it, same reason humans sometimes need to read a whole sentence before a word's meaning clicks.

## Where RNNs fit vs where transformers took over

Honest note carrying forward to file 15: RNNs process sequences one step at a time, which is inherently slow to train (can't parallelize across time steps) and still struggles with very long-range dependencies despite LSTM's fixes. This is genuinely the core reason transformers (which process the whole sequence in parallel using attention) have replaced RNNs for most large-scale NLP work. RNNs/LSTMs are still genuinely useful for smaller sequence tasks, time-series forecasting, and cases where sequential/streaming processing is a hard requirement.

## Try this

```python
# Build a simple LSTM that takes a sequence of 10 random numbers and predicts the next number
# (sequence-to-one regression) — train it on a simple pattern like an arithmetic sequence
# and check if the LSTM learns to extrapolate the pattern
```

Watching an LSTM actually learn to predict "the next number in a simple pattern" is genuinely the clearest way to feel why the hidden state carries information forward — much more concrete than reading the gate equations.

## What's next
File 12 — Transfer Learning: using models someone else already spent thousands of GPU-hours training, instead of training everything from scratch.
