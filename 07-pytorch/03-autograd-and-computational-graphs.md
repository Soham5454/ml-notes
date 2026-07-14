# 03 — Autograd & Computational Graphs

This is genuinely the single most important concept in PyTorch. Once this clicks, the training loop in file 07 stops feeling like magic.

## The core idea

Every neural network training step needs gradients — "if I nudge this weight slightly, how much does the loss change?" Computing that by hand (calculus, chain rule, layer by layer) is exactly what backpropagation is. Autograd does this automatically by **recording every operation** performed on tensors that have `requires_grad=True`, building a graph of those operations, then walking that graph backward to compute all gradients in one call.

## A minimal example — the one to actually understand deeply

```python
import torch

x = torch.tensor(3.0, requires_grad=True)
y = x ** 2 + 2 * x + 1     # y = x^2 + 2x + 1

y.backward()                # compute dy/dx
print(x.grad)                # tensor(8.)
```

Manual check: dy/dx = 2x + 2. At x=3, that's 2(3)+2 = 8. Matches exactly. This is genuinely worth verifying by hand once — it removes the "black box" feeling completely.

## What `requires_grad=True` actually does

It tells PyTorch: "track every operation done to this tensor, because I'll want gradients w.r.t. it later." Model weights always have this set to `True` (done automatically when you use `nn.Linear` etc. — see file 04). Input data normally does NOT need `requires_grad=True` unless you're doing something unusual like adversarial examples.

## The computational graph, visualized in words

```
x (leaf, requires_grad=True)
  |
  v
x**2  ---> intermediate node
  |
  v
x**2 + 2*x  ---> intermediate node
  |
  v
y = x**2 + 2*x + 1  ---> final output (root)
```

`.backward()` walks this graph from `y` back to `x`, applying the chain rule at every node, and accumulates the result into `x.grad`.

Important word there: **accumulates**. Gradients don't get overwritten automatically — they add up across multiple `.backward()` calls. This is why the training loop (file 07) always has a `.zero_grad()` call before computing new gradients. If you forget it, gradients from the previous batch silently add onto the current batch's gradients and training goes haywire. Noting this now because it's a genuinely common beginner bug.

## Multi-variable example (closer to real NN math)

```python
x = torch.tensor(2.0, requires_grad=True)
w = torch.tensor(3.0, requires_grad=True)
b = torch.tensor(1.0, requires_grad=True)

y = w * x + b     # y = 3(2) + 1 = 7
y.backward()

print(w.grad)   # dy/dw = x = 2.0
print(x.grad)   # dy/dx = w = 3.0
print(b.grad)   # dy/db = 1.0
```

This `y = w*x + b` line is literally the equation for a single neuron with no activation function. Genuinely the entire foundation of file 04 is this one line, repeated across thousands of weights.

## Stopping gradient tracking

Sometimes you don't want autograd running — during evaluation/inference, or when doing plain data manipulation. Two ways:

```python
with torch.no_grad():
    y = x * 2     # no graph built, faster, less memory

x_detached = x.detach()   # returns a tensor that shares data but has no grad history
```

`torch.no_grad()` gets used constantly during model evaluation/validation loops (file 07) — since you're only doing a forward pass to check accuracy, not training, there's no reason to waste memory building a graph you'll never call `.backward()` on.

## `.grad` only exists on leaf tensors by default

```python
x = torch.tensor(2.0, requires_grad=True)
y = x * 3
z = y * 2
z.backward()

print(x.grad)   # works fine
print(y.grad)   # None + a warning, because y is an intermediate (non-leaf) tensor
```

If gradients on an intermediate tensor are genuinely needed for debugging, `y.retain_grad()` before calling backward keeps it around. Rare need, but good to know it exists rather than being confused by the `None`.

## Try this (builds real intuition)

```python
# Predict what w.grad and b.grad will be BEFORE running, then check
x = torch.tensor(5.0)
w = torch.tensor(2.0, requires_grad=True)
b = torch.tensor(1.0, requires_grad=True)

y_pred = w * x + b
target = torch.tensor(15.0)
loss = (y_pred - target) ** 2   # squared error

loss.backward()
print(w.grad, b.grad)
```

Working this out by hand (chain rule through the square) before running it is genuinely the single best exercise for actually internalizing autograd — this exact pattern (linear function → loss → backward) is the entire training loop in miniature.

## What's next
File 04 — using `nn.Module` to build actual neural network layers instead of writing raw `w*x+b` by hand every time.
