# 06 — Partial Derivatives & Gradients

File 05 covered functions of ONE variable. A real loss function depends on potentially billions of weights simultaneously. This file extends derivatives to that many-variable world — genuinely the exact math that makes `loss.backward()` (PyTorch file 03) work for a whole network at once.

## Partial derivatives — the core idea

A partial derivative measures how a function changes with respect to ONE variable, while treating all other variables as constant/frozen.

```python
# f(x, y) = x^2 + 3xy + y^2
#
# partial derivative with respect to x (treat y as constant):
# ∂f/∂x = 2x + 3y
#
# partial derivative with respect to y (treat x as constant):
# ∂f/∂y = 3x + 2y

def f(x, y):
    return x**2 + 3*x*y + y**2

def df_dx(x, y):
    return 2*x + 3*y

def df_dy(x, y):
    return 3*x + 2*y

# numerical verification
x, y = 2.0, 1.0
h = 1e-6
numerical_dx = (f(x+h, y) - f(x, y)) / h
print(numerical_dx, df_dx(x, y))   # should closely match
```

The `∂` symbol (instead of regular `d`) specifically signals "this is a partial derivative — there are other variables being held constant." Genuinely just notation, the actual computation rules (power rule, sum rule, etc from file 05) work identically, just applied one variable at a time.

## The gradient — collecting all partial derivatives into a vector

```python
import numpy as np

def gradient(x, y):
    return np.array([df_dx(x, y), df_dy(x, y)])

grad = gradient(2.0, 1.0)
print(grad)   # [7. 8.]
```

The gradient is a VECTOR (recap file 01) containing every partial derivative — one per input variable. Genuinely the single most important object in this entire Math Foundations section: **the gradient points in the direction of steepest INCREASE of the function.** Gradient descent moves in the OPPOSITE direction (steepest decrease) to minimize loss.

## This is EXACTLY what `x.grad` contains in PyTorch

Recap PyTorch file 03's autograd example directly:

```python
# PyTorch version (from PyTorch file 03)
# x = torch.tensor(2.0, requires_grad=True)
# w = torch.tensor(3.0, requires_grad=True)
# y = w * x
# y.backward()
# w.grad contains ∂y/∂w, x.grad contains ∂y/∂x

# The pure math equivalent:
def y(w, x):
    return w * x

def dy_dw(w, x):
    return x        # treating x as constant

def dy_dx(w, x):
    return w        # treating w as constant

w, x = 3.0, 2.0
print(dy_dw(w, x), dy_dx(w, x))   # 2.0 3.0 — matches x.grad=3.0, w.grad=2.0 from the PyTorch example exactly
```

This is genuinely the moment autograd stops being magic — `loss.backward()` is computing partial derivatives with respect to EVERY parameter in the network simultaneously, using exactly this math, just automated and scaled to millions/billions of parameters via the chain rule (file 07).

## Gradient descent — the update rule, now with full mathematical grounding

Recap PyTorch file 07's optimizer — the plain SGD update rule can now be written precisely:

```python
def gradient_descent_step(w, x, y_true, lr=0.01):
    y_pred = w * x
    loss = (y_pred - y_true) ** 2         # squared error

    # dloss/dw via chain rule (full treatment in file 07 of THIS folder)
    grad = 2 * (y_pred - y_true) * x

    w_new = w - lr * grad                   # move OPPOSITE the gradient direction
    return w_new
```

`w_new = w - lr * grad` — this exact line is what `optimizer.step()` does internally for every single parameter in a PyTorch model. `lr` (recap PyTorch file 07's learning_rate discussion, and the XGBoost bridge file's original callout) controls how big a step is taken along the gradient direction.

## The gradient as a "steepest ascent" direction — a genuinely important geometric fact

```python
# For f(x, y) = x^2 + y^2 (a bowl shape, minimum at origin)
def f(x, y):
    return x**2 + y**2

def grad_f(x, y):
    return np.array([2*x, 2*y])

point = np.array([3.0, 4.0])
g = grad_f(*point)
print(g)   # [6. 8.] — points AWAY from the origin, the direction of steepest increase
```

At any point on a surface, the gradient vector points in the direction where the function increases FASTEST. Standing at (3,4) on this bowl-shaped surface, moving in direction `[6,8]` increases `f` as quickly as possible; moving in direction `[-6,-8]` decreases it as quickly as possible — precisely the direction gradient descent steps toward.

## Visualizing a loss surface — connects directly to file 08's optimization landscape

```python
import numpy as np

x_vals = np.linspace(-5, 5, 50)
y_vals = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x_vals, y_vals)
Z = X**2 + Y**2    # bowl-shaped loss surface

# the gradient at any (X,Y) point is [2X, 2Y] — always pointing away from the minimum at (0,0)
```

This bowl shape is genuinely the simplest possible "loss landscape" — real neural network loss landscapes have thousands/millions of dimensions and far more complex shapes (valleys, saddle points, local minima), but the core mechanic never changes: compute the gradient, step opposite to it, repeat.

## Try this

```python
# For f(x, y) = (x-3)^2 + (y+2)^2 (a bowl centered at (3, -2), i.e. the true minimum)
# 1. Derive the gradient by hand
# 2. Starting from (0, 0), manually perform 10 gradient descent steps with lr=0.1
# 3. Confirm the (x, y) point converges toward (3, -2)
```

Watching (x,y) actually converge toward the true minimum through manual gradient descent steps is genuinely the clearest bridge between this file's pure math and the PyTorch training loops already being run — same mechanism, just done by hand instead of `optimizer.step()`.

## What's next
File 07 — The Chain Rule & Backpropagation Math, extending this to functions COMPOSED of other functions (exactly what a multi-layer network is) — this is the file that fully explains how `loss.backward()` computes gradients through an entire deep network in one call.
