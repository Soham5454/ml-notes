# 08 — Optimization & Gradient Descent Math

Files 05-07 built up to computing gradients correctly. This file is about what happens AFTER the gradient is known — how different optimizers (recap PyTorch file 07: SGD vs Adam) actually use it, and why some converge faster or more reliably than others.

## Plain gradient descent — the baseline, fully derived

```python
def gradient_descent(f, grad_f, x0, lr=0.1, steps=50):
    x = x0
    history = [x]
    for _ in range(steps):
        g = grad_f(x)
        x = x - lr * g      # recap file 06's exact update rule
        history.append(x)
    return x, history

# minimize f(x) = (x - 5)^2, true minimum at x=5
f = lambda x: (x - 5)**2
grad_f = lambda x: 2 * (x - 5)

final_x, history = gradient_descent(f, grad_f, x0=0.0, lr=0.1)
print(final_x)   # should converge close to 5.0
```

## The learning rate problem, mathematically demonstrated

```python
# Too high a learning rate genuinely causes overshooting/divergence
final_x_bad, history_bad = gradient_descent(f, grad_f, x0=0.0, lr=1.1)
print(history_bad[:5])   # values will oscillate and grow instead of converging
```

Recap the XGBoost bridge file's original callout and PyTorch file 07's `lr` discussion — this is genuinely the mathematical proof of why: with `lr=1.1` on this particular bowl-shaped function, each step overshoots the minimum by MORE than the previous step, and the oscillation amplifies rather than dampens. There's an actual mathematical stability boundary (related to the function's curvature/second derivative) beyond which a fixed learning rate guarantees divergence — this is precisely the theoretical justification for why adaptive methods like Adam exist.

## Convexity — why some loss surfaces are "easy" and others aren't

A function is **convex** if a straight line between any two points on its curve never dips below the curve itself — genuinely means it has exactly ONE minimum, no false valleys to get stuck in.

```python
import numpy as np

# Convex: f(x) = x^2  -- single bowl, one global minimum
x = np.linspace(-5, 5, 100)
convex_example = x**2

# Non-convex: f(x) = x^4 - 4x^2  -- has TWO local minima and one local maximum (recap file 05's "try this")
nonconvex_example = x**4 - 4*x**2
```

Linear/logistic regression's loss functions (recap scikit-learn notes) are genuinely convex — gradient descent is GUARANTEED to find the global minimum given enough steps and a reasonable learning rate. Neural network loss surfaces (recap PyTorch training loops) are NOT convex — full of local minima and saddle points, which is precisely why training deep networks is empirically harder and less theoretically guaranteed than fitting a linear model, and why techniques like momentum matter.

## The second derivative test — distinguishing minima, maxima, saddle points

```python
# f(x) = x^3 - 3x
# f'(x) = 3x^2 - 3    -- critical points where this = 0: x = 1, x = -1
# f''(x) = 6x

def f_double_prime(x):
    return 6 * x

print(f_double_prime(1))    # 6, POSITIVE -> this is a local MINIMUM (curves upward)
print(f_double_prime(-1))   # -6, NEGATIVE -> this is a local MAXIMUM (curves downward)
```

Rule (recap file 05's preview): at a critical point (first derivative = 0), a POSITIVE second derivative means the function curves upward (minimum), NEGATIVE means it curves downward (maximum). A second derivative of exactly zero is inconclusive on its own — could be a saddle point, needs deeper analysis, genuinely common in high-dimensional neural network loss surfaces.

## Momentum — using past gradients, not just the current one

```python
def gradient_descent_momentum(grad_f, x0, lr=0.1, momentum=0.9, steps=50):
    x = x0
    velocity = 0
    history = [x]
    for _ in range(steps):
        g = grad_f(x)
        velocity = momentum * velocity - lr * g    # accumulate a "rolling" direction
        x = x + velocity
        history.append(x)
    return x, history
```

Momentum keeps a running average of past gradients (like a ball rolling downhill picking up speed) — genuinely helps in two real ways: it smooths out noisy gradient estimates (from mini-batches, recap PyTorch file 08), and it can carry through small local bumps/saddle points that would otherwise stall plain gradient descent.

## Adam — the actual math behind PyTorch's default optimizer

Recap PyTorch file 07's Adam usage — here's what it's actually computing, per parameter, at each step:

```python
def adam_step(grad, m, v, t, lr=0.001, beta1=0.9, beta2=0.999, eps=1e-8):
    m = beta1 * m + (1 - beta1) * grad          # exponential moving average of the gradient (like momentum)
    v = beta2 * v + (1 - beta2) * grad**2         # exponential moving average of the SQUARED gradient

    m_hat = m / (1 - beta1**t)                     # bias correction (early steps are biased toward 0)
    v_hat = v / (1 - beta2**t)

    update = lr * m_hat / (np.sqrt(v_hat) + eps)    # adaptive step size, larger where v_hat is small
    return update, m, v
```

Genuinely the key insight: `v` (the moving average of SQUARED gradients) tracks how much a parameter's gradient has been fluctuating. Dividing by `sqrt(v_hat)` means parameters with consistently small gradients get a RELATIVELY BIGGER effective step, and parameters with wildly fluctuating gradients get a relatively SMALLER step — this is precisely why Adam is described as "adapting the learning rate per-parameter" (recap PyTorch file 07), and why it's more forgiving of a not-perfectly-tuned global `lr` than plain SGD.

## Try this

```python
# Using f(x) = x^4 - 4x^2 (has 2 minima, 1 maximum -- from file 05's "try this")
# Run plain gradient descent from THREE different starting points: x0 = -3, x0 = 0.1, x0 = 3
# Observe that depending on starting point, gradient descent converges to DIFFERENT local minima
# This is genuinely a direct, hands-on demonstration of why neural network training is sensitive
# to weight initialization -- same optimization process, different starting point, different outcome
```

Seeing gradient descent land in different minima purely based on starting point is genuinely the clearest, most concrete demonstration in this whole folder of why "random weight initialization" (something used without a second thought every time `nn.Linear()` gets called in PyTorch) is a real, consequential design decision, not just a technical formality.

## What's next — moving into Probability
File 09 — Probability Fundamentals. Linear algebra explained the "what," calculus explained the "how it learns" — probability explains how to reason about UNCERTAINTY, which underlies every loss function measuring "how wrong" a prediction is, every Bayesian technique, and every generative model (recap PyTorch files 16/17's autoencoders and GANs).
