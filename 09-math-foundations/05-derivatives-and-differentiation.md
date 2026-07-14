# 05 — Derivatives & Differentiation Rules

Linear algebra (files 01-04) explained what a network computes. Calculus explains how it LEARNS — and this file is genuinely the foundation that makes PyTorch's autograd (file 03 in the PyTorch folder) stop being a black box.

## What a derivative actually means

A derivative measures how much a function's output changes when its input changes by a tiny amount — the instantaneous rate of change, or geometrically, the slope of the function's curve at a specific point.

```
f'(x) = lim(h→0) [f(x+h) - f(x)] / h
```

Genuinely don't need to compute limits by hand for ML purposes, but the intuition matters completely: **the derivative tells you which direction and how steeply a function is currently changing.** This is EXACTLY what gradient descent uses — "which direction should I nudge this weight to make the loss go down."

## Basic differentiation rules — the ones that actually get used constantly

```
Power rule:      d/dx (x^n) = n * x^(n-1)
Constant rule:    d/dx (c) = 0
Sum rule:         d/dx (f(x) + g(x)) = f'(x) + g'(x)
Product rule:     d/dx (f(x) * g(x)) = f'(x)g(x) + f(x)g'(x)
Chain rule:       d/dx f(g(x)) = f'(g(x)) * g'(x)     -- covered fully in file 07
```

Worked example, genuinely worth doing by hand:

```python
# f(x) = 3x^2 + 2x + 5
# f'(x) = 6x + 2   (power rule + sum rule + constant rule)

def f(x):
    return 3*x**2 + 2*x + 5

def f_prime(x):
    return 6*x + 2

# numerical check using a tiny h
x = 2.0
h = 1e-6
numerical_derivative = (f(x + h) - f(x)) / h
print(numerical_derivative, f_prime(x))   # should be very close: ~26.0
```

This numerical check (the actual limit definition, approximated with a small `h`) is genuinely how gradient-checking works in practice when debugging custom loss functions — comparing the analytical (formula-based) derivative against a numerical approximation to catch bugs.

## Derivatives of the functions that actually show up in ML

```python
import numpy as np

# Sigmoid: s(x) = 1/(1+e^-x)
# s'(x) = s(x) * (1 - s(x))    -- genuinely elegant, worth memorizing
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def sigmoid_derivative(x):
    s = sigmoid(x)
    return s * (1 - s)

# ReLU: f(x) = max(0, x)
# f'(x) = 1 if x > 0, else 0    -- recap PyTorch file 05's "gradient is exactly 1" claim, now proven
def relu_derivative(x):
    return np.where(x > 0, 1, 0)
```

Recap PyTorch file 05's vanishing gradient explanation — now it's provable, not just asserted: sigmoid's derivative `s(x)*(1-s(x))` has a MAXIMUM value of exactly 0.25 (at x=0), and shrinks toward 0 as x moves away from 0 in either direction. Multiply several of these small numbers together across many layers (chain rule, file 07) and the combined gradient shrinks toward zero fast — that's the vanishing gradient problem, derived directly from this formula rather than just described conceptually.

ReLU's derivative being exactly 1 for all positive inputs (no shrinking at all) is precisely why it doesn't suffer the same vanishing gradient issue — again, now provable rather than asserted.

## Critical points — where derivatives equal zero

```python
# f(x) = x^2 - 4x + 3
# f'(x) = 2x - 4
# f'(x) = 0  =>  x = 2   -- this is where f has its minimum

def f(x):
    return x**2 - 4*x + 3

xs = np.linspace(-2, 6, 100)
ys = f(xs)
min_x = xs[np.argmin(ys)]
print(min_x)   # should be close to 2.0
```

A derivative of zero means the function is momentarily flat — could be a minimum, maximum, or saddle point (file 08 covers distinguishing these properly using second derivatives). This is genuinely the exact condition gradient descent is searching for: keep moving in the direction that REDUCES the loss until the gradient (derivative) gets close to zero, meaning further improvement has stalled.

## Second derivatives — a quick preview for file 08

```python
# f(x) = x^3
# f'(x) = 3x^2       -- first derivative: rate of change
# f''(x) = 6x        -- second derivative: rate of change OF the rate of change (curvature)
```

Second derivatives measure curvature — genuinely useful for understanding WHY some optimizers (like Adam, recap PyTorch file 07) adapt their step size, and for distinguishing a true minimum (curves upward, second derivative positive) from a maximum (curves downward) or saddle point. File 08 builds on this properly.

## Try this

```python
# Take f(x) = x^4 - 4x^2 + 2
# 1. Derive f'(x) by hand using the power rule
# 2. Find all x where f'(x) = 0 by solving the equation
# 3. Verify each critical point numerically (small-h approximation, like the block above)
# 4. Plot f(x) and visually confirm which critical points are minima and which are maxima
```

This function genuinely has TWO minima and one maximum — a good exercise for seeing that "derivative equals zero" alone doesn't tell you which type of critical point you've found, setting up exactly why file 08's second-derivative test matters.

## What's next
File 06 — Partial Derivatives & Gradients, extending single-variable calculus to functions of many variables — which is what EVERY real loss function actually is, since a network has thousands to billions of weights, not just one.
