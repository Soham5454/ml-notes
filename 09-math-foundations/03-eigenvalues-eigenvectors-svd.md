# 03 — Eigenvalues, Eigenvectors & SVD

This is genuinely the file that explains what PCA (recap scikit-learn notes) was actually doing under the hood — I used `PCA()` as a black box before, this closes that gap.

## The core idea — what an eigenvector actually is

For most vectors, multiplying by a matrix A changes BOTH the vector's length AND its direction. An eigenvector is special: multiplying it by A changes ONLY its length (scales it), never its direction.

```
A @ v = λ * v
```

`v` is the eigenvector, `λ` (lambda) is the eigenvalue — the scaling factor. Genuinely worth reading that equation slowly: "applying this whole matrix transformation to v does the exact same thing as just multiplying v by a single number."

## Computing eigenvalues and eigenvectors

```python
import numpy as np

A = np.array([[4, 2],
              [1, 3]])

eigenvalues, eigenvectors = np.linalg.eig(A)
print(eigenvalues)     # [5. 2.]
print(eigenvectors)    # columns are the eigenvectors
```

Verifying the core equation manually:

```python
v = eigenvectors[:, 0]        # first eigenvector
lam = eigenvalues[0]           # its eigenvalue

left = A @ v
right = lam * v
print(np.allclose(left, right))   # True — confirms A@v == λ*v
```

## Why this matters for ML — the genuinely important connection

A matrix's eigenvectors point in the directions along which the matrix's transformation is simplest (pure scaling, no rotation/shearing). For a **covariance matrix** specifically (which measures how features vary together — bridges directly to file 12's covariance and file 16's correlation), the eigenvectors point in the directions of MAXIMUM variance in the data, and the eigenvalues tell you HOW MUCH variance lies along each direction.

This is EXACTLY what PCA does:
1. Compute the covariance matrix of the data.
2. Find its eigenvectors and eigenvalues.
3. Keep the eigenvectors with the LARGEST eigenvalues (the directions capturing the most variance/information).
4. Project the data onto those top eigenvectors — that's the dimensionality reduction.

```python
# A tiny from-scratch PCA, to demystify scikit-learn's PCA()
data = np.random.randn(100, 3) @ np.array([[1,0,0],[0,1,0],[0,0,0.01]])  # mostly variance in first 2 dims

cov_matrix = np.cov(data.T)
eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

# sort by eigenvalue, descending
order = np.argsort(eigenvalues)[::-1]
eigenvalues = eigenvalues[order]
eigenvectors = eigenvectors[:, order]

# keep top 2 components
top_2 = eigenvectors[:, :2]
reduced_data = data @ top_2
print(reduced_data.shape)   # (100, 2) — dimensionality reduced from 3 to 2
```

Genuinely satisfying moment — this from-scratch block does exactly what `sklearn.decomposition.PCA(n_components=2).fit_transform(data)` does internally. The scikit-learn call was never magic, just this eigen-decomposition wrapped in a convenient API.

## Singular Value Decomposition (SVD) — the more general, more robust tool

Eigen-decomposition only works cleanly on square matrices. SVD works on ANY matrix (including rectangular ones, like most real datasets — rows of samples, columns of features), which is why it's used more often in practice.

```python
A = np.array([[3, 1], [1, 3], [2, 2]])   # 3x2, non-square

U, S, Vt = np.linalg.svd(A)
print(U.shape, S.shape, Vt.shape)   # (3,3) (2,) (2,2)
```

SVD decomposes any matrix A into `A = U @ Σ @ V^T`, where U and V are orthogonal (rotation-like) matrices and Σ (from `S`) contains the **singular values** — genuinely the generalized version of eigenvalues, always non-negative, and again ranked by "how much information/variance" each corresponding direction captures.

## Where SVD shows up in real ML tools

- **PCA** — scikit-learn's actual `PCA()` implementation uses SVD internally rather than raw eigen-decomposition of the covariance matrix, since it's more numerically stable, especially for wide datasets (more features than samples).
- **Recommendation systems** — classic matrix factorization techniques (user-item rating matrices) are built directly on SVD, decomposing a huge sparse matrix into smaller, denser factor matrices.
- **LoRA (Low-Rank Adaptation)** — flagging this forward since it's coming in the Frontier GenAI phase — LoRA fine-tunes huge language models by learning small LOW-RANK matrices instead of updating the full weight matrix. "Low-rank" is directly an SVD concept: approximating a big matrix using only its most important singular values/vectors, discarding the rest. Understanding SVD now means LoRA won't feel like an arbitrary trick later — it'll be recognizable as "keep only the top few singular directions."

## Rank of a matrix — a quick, important related concept

```python
A = np.array([[1, 2], [2, 4]])   # second row is just 2x the first — redundant information
print(np.linalg.matrix_rank(A))   # 1, not 2
```

Rank counts the number of genuinely INDEPENDENT directions/dimensions of information in a matrix. A "low-rank" matrix has redundant rows/columns — this is precisely the property LoRA exploits, and also why highly correlated features (file 16) can cause issues in linear models: correlated features push a data matrix toward lower effective rank than its raw shape suggests.

## Try this

```python
# Take a real-ish dataset (e.g. make one with 5 correlated features using make_regression-style noise)
# Manually run PCA via covariance + eigen-decomposition (like the block above)
# Then run sklearn's PCA() on the same data and compare the reduced output
# (values might differ in sign/scale slightly, but the captured variance ratio should match closely)
```

Confirming a from-scratch PCA roughly matches scikit-learn's black-box version is genuinely the strongest proof that this file's theory isn't disconnected from the tools already used for months.

## What's next
File 04 — Linear Transformations, tying vectors, matrices, and eigenvectors together into the single unifying idea: every layer of every neural network is fundamentally a linear transformation followed by a non-linearity.
