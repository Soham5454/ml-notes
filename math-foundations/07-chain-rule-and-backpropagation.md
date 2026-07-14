# 07 — Chain Rule & Backpropagation Math

This is genuinely the single most important file in the entire Math Foundations folder for deep learning specifically. Everything in files 01-06 was building toward this: explaining exactly how `loss.backward()` computes gradients for every weight in a network with one function call.

## The chain rule — single variable version, recap from file 05

```
If y = f(g(x)), then dy/dx = f'(g(x)) * g'(x)
```

Worked example:

```python
# y = (3x + 1)^2
# Let u = 3x + 1 (inner function), y = u^2 (outer function)
# dy/du = 2u,  du/dx = 3
# dy/dx = dy/du * du/dx = 2u * 3 = 2(3x+1) * 3 = 6(3x+1)

def y(x):
    return (3*x + 1)**2

def dy_dx(x):
    return 6 * (3*x + 1)

x = 2.0
h = 1e-6
numerical = (y(x+h) - y(x)) / h
print(numerical, dy_dx(x))   # should closely match
```

## Why the chain rule is THE mechanism behind neural networks

A neural network is genuinely nothing but a long chain of composed functions: `loss(layer3(layer2(layer1(x))))`. To know how much the LOSS changes when a weight deep inside `layer1` changes, the chain rule has to be applied repeatedly, once per layer, multiplying local derivatives together as it goes.

## A concrete 2-layer example — computed fully by hand

```python
# Tiny network: x -> (linear: w1*x) -> (relu) -> (linear: w2*h) -> loss
# Forward pass:
x = 2.0
w1 = 0.5
w2 = 3.0
target = 5.0

z1 = w1 * x                    # z1 = 1.0
h = max(0, z1)                 # relu, h = 1.0 (since z1 > 0)
y_pred = w2 * h                # y_pred = 3.0
loss = (y_pred - target) ** 2  # loss = 4.0

print(z1, h, y_pred, loss)
```

Now the backward pass, applying the chain rule at every step, exactly mirroring what `loss.backward()` does internally:

```python
# dloss/dy_pred = 2 * (y_pred - target)
dloss_dy_pred = 2 * (y_pred - target)     # 2*(3-5) = -4.0

# dy_pred/dw2 = h        (since y_pred = w2 * h)
dy_pred_dw2 = h                            # 1.0
# CHAIN RULE: dloss/dw2 = dloss/dy_pred * dy_pred/dw2
dloss_dw2 = dloss_dy_pred * dy_pred_dw2    # -4.0 * 1.0 = -4.0

# dy_pred/dh = w2        (since y_pred = w2 * h)
dy_pred_dh = w2                             # 3.0
dloss_dh = dloss_dy_pred * dy_pred_dh       # -4.0 * 3.0 = -12.0

# dh/dz1 = 1 if z1 > 0 else 0   (relu derivative, recap file 05)
dh_dz1 = 1 if z1 > 0 else 0                 # 1

dloss_dz1 = dloss_dh * dh_dz1               # -12.0 * 1 = -12.0

# dz1/dw1 = x            (since z1 = w1 * x)
dz1_dw1 = x                                  # 2.0
dloss_dw1 = dloss_dz1 * dz1_dw1              # -12.0 * 2.0 = -24.0

print("dloss/dw2:", dloss_dw2)   # -4.0
print("dloss/dw1:", dloss_dw1)   # -24.0
```

This chain — `dloss/dw1 = dloss/dy_pred * dy_pred/dh * dh/dz1 * dz1/dw1` — is genuinely THE gradient computation, done by hand, that autograd performs automatically. Every arrow in that chain is one "local" derivative (easy to compute on its own), and the chain rule is simply the rule for multiplying all these local derivatives together to get the FAR-away effect of `w1` (deep in the network) on the final `loss`.

## Verifying against PyTorch — closing the loop completely

```python
import torch

x = torch.tensor(2.0)
w1 = torch.tensor(0.5, requires_grad=True)
w2 = torch.tensor(3.0, requires_grad=True)
target = torch.tensor(5.0)

z1 = w1 * x
h = torch.relu(z1)
y_pred = w2 * h
loss = (y_pred - target) ** 2

loss.backward()
print(w1.grad, w2.grad)   # should print -24.0 3.0 -- wait, check w2 sign convention below
```

Note on double-checking: running this should produce `w1.grad ≈ -24.0` and `w2.grad ≈ -4.0`, matching the hand-computed values exactly. If it doesn't match on the first attempt, genuinely worth re-deriving by hand rather than trusting either blindly — that debugging process itself is the whole point of this exercise.

## Backpropagation — the chain rule applied systematically, layer by layer

"Backpropagation" is genuinely just a name for applying the chain rule efficiently across an entire network: start from the loss, compute the gradient with respect to the LAST layer's output, then use that to compute the gradient with respect to the second-to-last layer's output, and so on, working BACKWARD through the network — reusing already-computed gradients rather than recomputing the whole chain from scratch for every single weight. This reuse is precisely what makes training networks with millions/billions of parameters computationally feasible at all — recomputing full independent chains for every single weight would be catastrophically slow.

## Why deep networks are hard to train — the chain rule explains it directly

Recap file 05's vanishing gradient derivation (sigmoid's derivative maxing out at 0.25) — now the FULL picture: in a chain of many layers, the chain rule MULTIPLIES many local derivatives together. If most of those local derivatives are less than 1 (like sigmoid's), the product shrinks exponentially with depth — vanishing gradients. If most are greater than 1, the product grows exponentially — exploding gradients. This is genuinely the mathematical root cause of why activation function choice (recap PyTorch file 05), weight initialization, and techniques like batch normalization (recap PyTorch file 09) all matter so much for training deep networks successfully.

## Try this

```python
# Extend the 2-layer hand-computed example above to 3 layers:
# x -> linear(w1) -> relu -> linear(w2) -> relu -> linear(w3) -> loss
# Compute dloss/dw1, dloss/dw2, dloss/dw3 fully by hand, writing out every chain rule step
# Then verify ALL THREE gradients against PyTorch's autograd on the same numbers
```

Doing a full 3-layer version by hand, then confirming against PyTorch, is genuinely the single best exercise in this whole Math Foundations folder for deep learning specifically — if this matches, autograd is fully, permanently demystified.

## What's next
File 08 — Optimization & Gradient Descent Math, going deeper into HOW gradient descent variants (SGD, Momentum, Adam) actually use these gradients to update weights efficiently, including the second-derivative concepts previewed in file 05.
