# 05 — Activation Functions

## Why they exist at all — the one thing to never forget

Without activation functions, stacking linear layers is pointless — `Linear(Linear(x))` is still just one big linear function mathematically (a matrix multiply of matrix multiplies collapses into a single matrix multiply). Activation functions inject **non-linearity**, which is what lets a network learn curved decision boundaries, images, language — anything that isn't just a straight line/plane.

Genuinely the one-sentence version: no activation functions = no matter how many layers, the whole network is mathematically equivalent to a single linear regression.

## Sigmoid

```python
import torch
import torch.nn as nn

sigmoid = nn.Sigmoid()
x = torch.tensor([-2., -1., 0., 1., 2.])
print(sigmoid(x))   # squashes everything into (0, 1)
```

Formula: `1 / (1 + e^-x)`. Output always between 0 and 1 — historically used for binary classification output layers, and still used there today. Rarely used in hidden layers anymore because of the **vanishing gradient problem**: for large positive or negative inputs, the curve flattens out, gradient becomes near-zero, and deep networks stop learning in early layers.

## Tanh

```python
tanh = nn.Tanh()
print(tanh(x))   # squashes into (-1, 1)
```

Same vanishing-gradient issue as sigmoid, just centered at zero instead of 0.5. Slightly better than sigmoid for hidden layers historically, but both have mostly been replaced by ReLU for hidden layers in modern architectures.

## ReLU — the default choice, and why

```python
relu = nn.ReLU()
print(relu(x))   # tensor([0., 0., 0., 1., 2.]) — negative values become 0, positive pass through unchanged
```

Formula: `max(0, x)`. Genuinely simple, and that simplicity is exactly why it works so well:
- No vanishing gradient for positive inputs (gradient is exactly 1, not squashed).
- Computationally cheap (just a comparison, no exponential like sigmoid/tanh).
- Empirically, networks with ReLU train faster and reach better accuracy than sigmoid/tanh-based ones for most architectures.

The one real downside: **dying ReLU problem** — if a neuron's output goes negative and stays negative across training, its gradient is permanently 0 and it never updates again ("dead neuron"). Variants exist specifically to fix this:

```python
leaky_relu = nn.LeakyReLU(negative_slope=0.01)
# allows a small negative slope instead of a hard 0, so dead neurons can still get gradient

elu = nn.ELU()
# smooth exponential curve for negative values instead of a hard cutoff
```

## Softmax — for multi-class output layers

```python
softmax = nn.Softmax(dim=1)
logits = torch.tensor([[2.0, 1.0, 0.1]])
print(softmax(logits))   # converts to probabilities that sum to 1 across dim=1
```

Used almost exclusively in the **output layer** for multi-class classification — converts raw scores ("logits") into a probability distribution across classes. Important practical note that trips people up: `nn.CrossEntropyLoss` (file 06) actually applies softmax internally, so the model's final layer should output raw logits, NOT pass them through softmax manually first — doing both is a genuinely common and subtle bug.

## GELU — worth knowing since it shows up in transformers later

```python
gelu = nn.GELU()
```

Smoother version of ReLU, used in BERT/GPT-style architectures. Not critical to deeply understand the math right now — just filing this away because file 15 (transformers) will reference it, and I don't want to be surprised by an unfamiliar activation function showing up there.

## Quick decision table (the one I'll actually reference later)

| Situation | Activation |
|---|---|
| Hidden layers, general default | ReLU |
| Hidden layers, worried about dead neurons | LeakyReLU |
| Binary classification output | Sigmoid |
| Multi-class classification output | Softmax (or raw logits + CrossEntropyLoss) |
| Transformer architectures | GELU |
| Output needs to be in (-1, 1) | Tanh (e.g. some GAN generators, file 17) |

## Try this — visualize it in your head before running

```python
x = torch.linspace(-5, 5, 11)
for name, fn in [("ReLU", nn.ReLU()), ("Sigmoid", nn.Sigmoid()), ("Tanh", nn.Tanh())]:
    print(name, fn(x))
```

Genuinely useful exercise: predict roughly what each output looks like before running (ReLU clips negatives to 0, sigmoid squashes to 0-1, tanh squashes to -1 to 1) — if the prediction matches, the mental model is solid.

## What's next
File 06 — loss functions, which is how the network actually knows whether ReLU-activated output is any good or garbage.
