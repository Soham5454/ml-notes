# 09 — Linear Algebra

This is the part that actually connects NumPy to "the math behind ML" — neural networks are fundamentally chains of matrix multiplications, so this section matters more than it might seem at first glance.

## Matrix multiplication (the REAL matrix multiply, not element-wise)
```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

a @ b            # matrix multiplication (preferred modern syntax)
np.matmul(a, b)    # same thing, more explicit
np.dot(a, b)         # also same result for 2D arrays (dot and matmul differ slightly for higher dimensions, but equivalent here)
```
Reminder to self: `a * b` is element-wise (covered in file 06), `a @ b` is TRUE matrix multiplication. These give completely different results and mixing them up is a classic bug.

## Dot product of vectors
```python
v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])
np.dot(v1, v2)     # 32   -> 1*4 + 2*5 + 3*6
```

## Transpose (already touched in file 04, relevant here too)
```python
a = np.array([[1, 2, 3], [4, 5, 6]])
a.T     # transposes rows/columns -> needed constantly to make matrix multiplication shapes line up correctly
```

## Determinant
```python
from numpy.linalg import det
matrix = np.array([[1, 2], [3, 4]])
det(matrix)     # -2.0
```

## Inverse of a matrix
```python
from numpy.linalg import inv
matrix = np.array([[1, 2], [3, 4]])
inv(matrix)     # the inverse matrix, such that matrix @ inv(matrix) = identity matrix
```
Note: not every matrix has an inverse (singular matrices don't) — trying to invert one throws a `LinAlgError`.

## Identity matrix (recap from file 02, relevant here)
```python
np.eye(3)     # multiplying anything by the identity matrix returns the original matrix unchanged
```

## Solving a system of linear equations
```python
from numpy.linalg import solve
# Example: 2x + y = 5, x + 3y = 10
A = np.array([[2, 1], [1, 3]])
b = np.array([5, 10])
solve(A, b)     # [1. 3.]  -> gives x=1, y=3
```

## Eigenvalues and eigenvectors
```python
from numpy.linalg import eig
matrix = np.array([[2, 0], [0, 3]])
values, vectors = eig(matrix)
```
Haven't needed this practically yet, but it comes up in PCA (dimensionality reduction) later in the ML roadmap — noting it here so I remember where NumPy already covers the foundation.

## Norm (magnitude of a vector)
```python
from numpy.linalg import norm
v = np.array([3, 4])
norm(v)     # 5.0   -> sqrt(3^2 + 4^2), standard Euclidean distance from origin
```
This is literally the formula behind distance metrics used in things like k-NN — good to actually see it as a one-liner instead of just abstractly knowing "distance formula."

## Why this matters for the ML roadmap
Every neural network layer is essentially: `output = input @ weights + bias`, followed by an activation function. That `@` is doing exactly the matrix multiplication covered above, at a much larger scale, on the GPU instead of CPU. Understanding this NumPy-level operation is what makes the PyTorch tensor operations later feel familiar instead of like new magic.
