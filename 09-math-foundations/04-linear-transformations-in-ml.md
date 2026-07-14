# 04 — Linear Transformations & Their Role in ML

Files 01-03 covered the pieces (vectors, matrices, eigen-structure) somewhat separately. This file ties them together into the single big idea that underlies literally every neural network layer.

## A matrix IS a function

The most important mental shift in this whole Linear Algebra section: stop thinking of a matrix as just "a grid of numbers" and start thinking of it as a FUNCTION that takes a vector in and produces a (possibly different-sized) vector out.

```python
import numpy as np

def transform(A, v):
    return A @ v

A = np.array([[2, 0], [0, 1]])   # stretches x-direction by 2, leaves y unchanged
v = np.array([1, 1])
print(transform(A, v))   # [2, 1]
```

This is genuinely EXACTLY what `nn.Linear` (recap PyTorch file 04) does — it's a learned function that maps an input vector to an output vector, and the "learning" is entirely about finding the right matrix A (weights) that performs a useful transformation.

## What makes a transformation "linear"

A transformation is linear if it satisfies two properties:
1. `T(a + b) = T(a) + T(b)` — additivity
2. `T(c * a) = c * T(a)` — scaling

```python
A = np.array([[2, 1], [0, 3]])
a = np.array([1, 2])
b = np.array([3, 1])

left = A @ (a + b)
right = (A @ a) + (A @ b)
print(np.allclose(left, right))   # True — confirms additivity
```

Genuinely important consequence, direct recap of PyTorch file 05's activation function motivation: stacking multiple linear transformations is STILL just one linear transformation (`A2 @ (A1 @ x) = (A2 @ A1) @ x`, and `A2 @ A1` is itself just another matrix). This is the actual mathematical proof behind the claim in file 05 — no matter how many `nn.Linear` layers get stacked, without a non-linear activation between them, the whole network collapses to one single linear function.

## Common transformations, visualized through their matrices

```python
theta = np.pi / 4   # 45 degrees
rotation = np.array([[np.cos(theta), -np.sin(theta)],
                      [np.sin(theta),  np.cos(theta)]])

scaling = np.array([[2, 0], [0, 0.5]])   # stretch x, shrink y

shear = np.array([[1, 0.5], [0, 1]])      # slanting transformation

v = np.array([1, 0])
print(rotation @ v)   # rotates v by 45 degrees
print(scaling @ v)     # stretches v
```

Not something to memorize by name, but genuinely useful to have SEEN once — every weight matrix in a trained network is some learned combination of rotation, scaling, and shearing effects on the input space, layer after layer, until the data becomes separable enough for the final classification/regression to work.

## Determinant revisited — geometric meaning, now with context

Recap file 02's determinant — now it can be properly explained: the determinant measures how much a transformation scales area/volume.

```python
A = np.array([[2, 0], [0, 3]])
print(np.linalg.det(A))   # 6.0 — this transformation scales area by exactly 6x

singular = np.array([[1, 2], [2, 4]])   # recap file 03's rank example — redundant rows
print(np.linalg.det(singular))   # 0.0 — this transformation squashes 2D space onto a 1D line
```

A determinant of 0 means information is being destroyed — space is being flattened into a lower dimension, and that flattening can't be undone (no inverse, recap file 02). This connects directly to why highly correlated or redundant features can cause numerical instability in linear models — the transformation implied by the feature matrix is close to singular.

## The forward pass of a neural network, expressed purely as transformations

Recap PyTorch file 04's MLP example — here's the exact same network described in pure linear-algebra language:

```python
# x: input vector
# Layer 1: linear transformation into a new space, then non-linearity breaks the "pure linear" collapse
h1 = relu(W1 @ x + b1)

# Layer 2: another linear transformation, in the NEW space defined by h1
output = W2 @ h1 + b2
```

Every layer is genuinely: (1) a linear transformation (rotate/scale/shear the current representation into a new space), (2) a translation (the `+b` bias term, technically making it an "affine" transformation, not purely linear, but the term "linear layer" is used loosely in ML), (3) a non-linear activation that prevents the whole stack from collapsing into one matrix. Files 01-04 together are genuinely the complete mathematical description of what "the network learns a representation" actually means: it's learning a sequence of transformations that reshape the data until it becomes easy to classify/predict.

## Try this

```python
# Take 2D points forming a rough circle (e.g. np.array([[cos(t), sin(t)] for t in np.linspace(0, 2*pi, 20)]))
# Apply a scaling matrix, then a rotation matrix, then a shear matrix, one at a time
# Observe (by printing or plotting) how the circle deforms into an ellipse, then rotates, then slants
```

Watching a simple shape visibly deform through each transformation is genuinely the clearest way to internalize "a matrix is a function that reshapes space" before trusting it as the invisible mechanism inside every `nn.Linear` call.

## What's next — moving from Linear Algebra into Calculus
File 05 — Derivatives & Differentiation Rules. Linear algebra explained WHAT a network's layers compute; calculus explains HOW the network learns to find good weight matrices in the first place — this is where autograd (recap PyTorch file 03) finally gets its full mathematical grounding.
