# 04 — Building Blocks: nn.Module

File 03 ended with `y = w*x + b` done manually with raw tensors. Nobody builds real networks that way — `nn.Module` is the wrapper that manages weights, biases, and forward-pass logic for you.

## The simplest possible network

```python
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(in_features=3, out_features=1)

    def forward(self, x):
        return self.linear(x)

model = SimpleNet()
x = torch.rand(5, 3)     # batch of 5 samples, 3 features each
output = model(x)
print(output.shape)       # torch.Size([5, 1])
```

Two rules that are non-negotiable for every custom model:
1. Always call `super().__init__()` first inside `__init__` — without it, PyTorch can't register the layers properly and things silently break.
2. Define layers in `__init__`, define the actual data flow in `forward()`. Never call `model.forward(x)` directly — call `model(x)`, which internally calls `forward` plus some hooks PyTorch needs.

## `nn.Linear` — what it actually is

`nn.Linear(in_features, out_features)` is exactly the `y = xW^T + b` operation, except PyTorch initializes `W` and `b` for you (with sensible random initialization) and marks them `requires_grad=True` automatically.

```python
layer = nn.Linear(3, 1)
print(layer.weight.shape)   # torch.Size([1, 3])
print(layer.bias.shape)     # torch.Size([1])
print(layer.weight.requires_grad)  # True
```

Genuinely this one line replaces the manual `w = torch.tensor(..., requires_grad=True)` from file 03 — same underlying math, but PyTorch manages the parameters as a proper object now.

## A real multi-layer network

```python
class MLP(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.layer1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.layer2 = nn.Linear(hidden_size, output_size)

    def forward(self, x):
        x = self.layer1(x)
        x = self.relu(x)
        x = self.layer2(x)
        return x

model = MLP(input_size=10, hidden_size=32, output_size=2)
```

This is a genuinely standard 2-layer MLP (multi-layer perceptron) — the "hello world" architecture of deep learning. Input → linear → activation → linear → output.

## `nn.Sequential` — shortcut for simple stacks

```python
model = nn.Sequential(
    nn.Linear(10, 32),
    nn.ReLU(),
    nn.Linear(32, 2)
)
```

Same network as the MLP class above, less code. Use `nn.Sequential` for straightforward stacks; use the class-based approach the moment there's branching logic, skip connections, or conditional forward passes (CNNs and beyond will need the class approach — noting this now).

## Inspecting and counting parameters

```python
model = MLP(10, 32, 2)

for name, param in model.named_parameters():
    print(name, param.shape)

total_params = sum(p.numel() for p in model.parameters())
print(f"Total trainable parameters: {total_params}")
```

Genuinely useful habit — checking parameter count is a quick sanity check that the architecture is what I think it is, and it's the number people quote when comparing model sizes ("this model has 7B parameters" etc — same `.numel()` idea just at a much bigger scale).

## `model.parameters()` — why this matters for file 07

The optimizer (file 07) needs to know exactly which tensors to update. `model.parameters()` returns every weight/bias in the model in one iterable — this is literally what gets passed to the optimizer:

```python
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
```

This is the connective thread between file 04 (defining the model) and file 07 (training it) — the optimizer is blind to architecture, it just updates whatever tensors `model.parameters()` hands it.

## Train mode vs eval mode

```python
model.train()   # enables dropout, batchnorm update behavior (file 09)
model.eval()    # disables them, needed for correct inference results
```

Small thing but genuinely a common bug source — forgetting `model.eval()` before checking validation accuracy gives inconsistent results because dropout is still randomly killing neurons during "evaluation."

## Try this

```python
# Build a 3-layer network: 20 -> 64 -> 32 -> 1, ReLU activations between hidden layers
# Then print total parameter count and verify output shape for a batch of 8 samples
```

Building this from scratch without peeking back at the MLP example above is the real test of whether `nn.Module` mechanics have actually sunk in.

## What's next
File 05 — activation functions in depth (why ReLU won over sigmoid, what each one is actually for).
