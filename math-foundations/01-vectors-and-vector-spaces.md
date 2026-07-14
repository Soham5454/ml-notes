# 01 — Vectors & Vector Spaces

Starting the Math Foundations phase now — genuinely should've done this before NumPy, but doing it now still fixes the gap. Every `torch.tensor`, every `np.array` I've been using for months is, underneath, exactly the objects covered in this file.

## What a vector actually is

A vector is an ordered list of numbers representing both a **magnitude** (length) and a **direction** in space. `[3, 4]` isn't just "two numbers" — it's an arrow from the origin to the point (3,4).

```python
import numpy as np

v = np.array([3, 4])
print(v)   # [3 4]
```

Recap link: this is literally the same object as `torch.tensor([3, 4])` from PyTorch file 02 — I used vectors constantly without the geometric intuition behind them. Fixing that now.

## Vector magnitude (norm)

```python
magnitude = np.linalg.norm(v)   # sqrt(3^2 + 4^2) = 5.0
print(magnitude)
```

Formula: `||v|| = sqrt(v1^2 + v2^2 + ... + vn^2)`. This is genuinely just the Pythagorean theorem extended to any number of dimensions. This exact formula reappears as **L2 regularization** (recap XGBoost/PyTorch regularization notes) — penalizing large weight vectors is literally penalizing a large vector norm.

## Vector addition and scalar multiplication

```python
a = np.array([1, 2])
b = np.array([3, 1])

print(a + b)        # [4, 3] — geometrically, place b's tail at a's head
print(2 * a)         # [2, 4] — same direction, doubled length
print(-1 * a)        # [-1, -2] — same magnitude, opposite direction
```

Genuinely important geometric intuition: vector addition is "walk along a, then walk along b" — the result is where you end up. Scalar multiplication stretches/shrinks (and flips if negative) without changing direction otherwise.

## Dot product — the single most important operation in this whole file

```python
dot = np.dot(a, b)   # 1*3 + 2*1 = 5
# equivalently
dot = a @ b           # same @ operator from PyTorch/NumPy notes
```

Formula: `a · b = a1*b1 + a2*b2 + ... + an*bn`. Two genuinely different but equally important ways to understand this:

**Algebraic view:** sum of element-wise products. This is EXACTLY what happens inside every neuron — recap PyTorch file 04's `nn.Linear`, which computes `y = xW^T + b`. Every single entry of that output is a dot product between an input vector and a weight vector.

**Geometric view:** `a · b = ||a|| ||b|| cos(θ)`, where θ is the angle between the vectors. This means the dot product measures how much two vectors point in the same direction:
- Dot product > 0 → vectors point roughly the same way
- Dot product = 0 → vectors are perpendicular (orthogonal)
- Dot product < 0 → vectors point roughly opposite ways

This geometric view is genuinely why the attention mechanism (recap PyTorch file 15) uses `Q @ K^T` — it's measuring how aligned/similar a Query vector is to a Key vector. Attention scores are LITERALLY dot products measuring vector similarity. That "why does Q·K make sense as a similarity score" question from file 15 gets answered right here.

## Cosine similarity — the dot product's most common ML application

```python
def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

print(cosine_similarity(a, b))
```

This is genuinely everywhere in ML — comparing word embeddings (recap PyTorch file 11's `nn.Embedding`), recommendation systems, semantic search, RAG systems (upcoming in the frontier GenAI phase). "How similar are these two vectors, ignoring their magnitude" is precisely what cosine similarity answers.

## Vector spaces — the formal container

A vector space is a set of vectors that's closed under addition and scalar multiplication — add two vectors from the space, or scale one, and the result is still in the space. Not deep-diving the formal axioms here since the practical intuition matters more for ML: **feature vectors, embeddings, and gradients all live in vector spaces**, and operations like addition/scaling always make sense within them.

## Basis vectors and dimensionality — connects to PCA later

```python
# Standard basis in 2D
e1 = np.array([1, 0])
e2 = np.array([0, 1])

v = np.array([3, 4])
# v can always be written as a combination of basis vectors
reconstructed = 3 * e1 + 4 * e2
print(reconstructed)   # [3, 4]
```

A **basis** is a minimal set of vectors that can combine (via scaling and adding) to reconstruct any vector in the space. The number of basis vectors needed equals the space's **dimensionality**. This directly sets up file 03's eigenvectors and PCA — dimensionality reduction is genuinely about finding a SMALLER, smarter basis that still captures most of the important structure in the data.

## Try this

```python
# Given two embedding-like vectors representing "king" and "queen" (make up 4-5 dimensional arrays)
# compute their cosine similarity, then compute similarity against a deliberately unrelated vector
# Confirm the related pair scores higher than the unrelated pair
```

Manually confirming that cosine similarity behaves sensibly on made-up "concept vectors" is genuinely the clearest way to build real intuition before trusting it blindly inside embedding models later.

## What's next
File 02 — Matrices, which are genuinely just organized collections of vectors, and matrix operations, which are what every single neural network layer computation reduces to.
