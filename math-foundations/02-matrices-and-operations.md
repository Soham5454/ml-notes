# 02 — Matrices & Matrix Operations

File 01 covered vectors. A matrix is genuinely just a rectangular grid of numbers — but more usefully, it can be thought of as either a collection of vectors OR a function that transforms vectors. Both views matter for ML.

## Creating and understanding matrix shape

```python
import numpy as np

A = np.array([[1, 2, 3],
              [4, 5, 6]])
print(A.shape)   # (2, 3) — 2 rows, 3 columns
```

Recap PyTorch file 02 — `A.shape` here is the exact same concept as `tensor.shape`. Every weight matrix in every `nn.Linear` layer is precisely this kind of object.

## Matrix addition and scalar multiplication — element-wise, straightforward

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

print(A + B)     # element-wise addition
print(2 * A)      # element-wise scaling
```

## Matrix multiplication — the operation that actually matters

```python
A = np.array([[1, 2], [3, 4]])      # shape (2,2)
B = np.array([[5, 6], [7, 8]])      # shape (2,2)

C = A @ B    # matrix multiplication
print(C)
```

Rule that has to be memorized cold: for `A @ B` to work, A's number of COLUMNS must equal B's number of ROWS. Result shape is `(A's rows, B's columns)`. Recap PyTorch file 02's shape-mismatch warning — this exact rule is the source of probably 50% of all shape-error bugs in deep learning code.

**What matrix multiplication actually computes:** each entry in the result is a DOT PRODUCT (recap file 01) between a row of A and a column of B. This is genuinely the connective insight — matrix multiplication is just "compute every possible dot product between A's rows and B's columns, arrange the results in a grid."

```python
# Manually verifying one entry
row0_A = A[0, :]      # [1, 2]
col0_B = B[:, 0]       # [5, 7]
manual = np.dot(row0_A, col0_B)   # 1*5 + 2*7 = 19
print(manual, C[0, 0])   # should match
```

## Why this is THE operation behind every neural network layer

Recap PyTorch file 04's `nn.Linear` — computing `y = xW^T + b` for a whole BATCH of inputs at once is exactly a matrix multiplication: each row of the input matrix `x` (one sample) gets dot-producted against each row of `W` (one neuron's weights), producing the batch of outputs all in one operation. This is genuinely why GPUs matter so much (recap PyTorch file 13) — a GPU is built specifically to do enormous numbers of these dot products in parallel.

## The identity matrix

```python
I = np.eye(3)
print(I)
# [[1,0,0],
#  [0,1,0],
#  [0,0,1]]

A = np.random.rand(3, 3)
print(np.allclose(A @ I, A))   # True — multiplying by I changes nothing
```

The identity matrix is the matrix-world equivalent of multiplying by 1 — genuinely useful as a sanity-check tool and appears constantly in more advanced linear algebra (eigen-decomposition, file 03).

## Matrix transpose

```python
A = np.array([[1, 2, 3], [4, 5, 6]])   # shape (2,3)
print(A.T)                               # shape (3,2), rows become columns
```

Recap PyTorch file 15's attention formula: `Q @ K.transpose(-2,-1)` — this transpose is precisely what makes the shapes align correctly for the Query-Key dot products to be computed as a matrix multiplication across all positions simultaneously.

## Matrix inverse — "undoing" a matrix

```python
A = np.array([[2, 0], [0, 3]])
A_inv = np.linalg.inv(A)
print(A_inv)                   # [[0.5, 0], [0, 0.333]]
print(np.allclose(A @ A_inv, np.eye(2)))   # True
```

`A @ A_inv = I` — genuinely the matrix equivalent of `x * (1/x) = 1`. Not every matrix has an inverse (only square matrices, and only if they're "non-singular" — covered properly with determinants below). Matrix inverses show up in the closed-form solution to linear regression (`(X^T X)^-1 X^T y`) — connects directly to file 04's linear transformations and the statistics phase later (file 16).

## Determinant — a single number summarizing a matrix

```python
A = np.array([[1, 2], [3, 4]])
det = np.linalg.det(A)
print(det)   # 1*4 - 2*3 = -2.0
```

Geometric meaning genuinely worth knowing: the determinant measures how much a matrix scales AREA (2D) or VOLUME (3D+) when used as a transformation (file 04 covers this properly). A determinant of 0 means the matrix "squashes" space into a lower dimension — and exactly means the matrix has NO inverse (can't undo an operation that destroyed information).

## Try this

```python
# Build a 3x4 matrix A and a 4x2 matrix B, compute A @ B, and verify the output shape
# matches the rule: (A's rows, B's columns)
# Then manually compute ONE entry of the result via np.dot on the corresponding row/column
# and confirm it matches the matrix multiplication result exactly
```

Doing this by hand once, matching manual dot products to matrix multiply output, is genuinely the exercise that makes matrix multiplication stop feeling like a black-box `@` symbol.

## What's next
File 03 — Eigenvalues, Eigenvectors & SVD — the tools behind PCA, dimensionality reduction, and understanding what a matrix "does" to space at a deeper level than just shapes.
