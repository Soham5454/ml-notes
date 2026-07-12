# 01 — Introduction & Array Basics

## What is NumPy and why does it exist
NumPy (Numerical Python) gives Python fast array operations. Regular Python lists are flexible but slow for numeric work because each element is a full Python object with its own type info and memory overhead. NumPy arrays store data in one contiguous block of memory, all the same type, which is why operations on them are dramatically faster.

This matters a lot once you start dealing with datasets with thousands/millions of rows — a Python list loop would crawl, a NumPy vectorized operation handles it near-instantly.

## The core object: ndarray
Everything in NumPy revolves around the `ndarray` (n-dimensional array).

```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])
print(arr)             # [1 2 3 4 5]
print(type(arr))         # <class 'numpy.ndarray'>
```

## Lists vs NumPy arrays — the actual difference
```python
py_list = [1, 2, 3]
np_arr = np.array([1, 2, 3])

py_list * 2     # [1, 2, 3, 1, 2, 3]   -> repeats the list (not what you'd want for math)
np_arr * 2      # [2, 4, 6]              -> element-wise multiplication (this is the whole point)
```
This single difference is why NumPy exists — it lets you write math the way you'd write it on paper, without manual loops.

## Array attributes
```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.shape      # (2, 3)   -> 2 rows, 3 columns
arr.ndim       # 2         -> number of dimensions
arr.size       # 6          -> total number of elements
arr.dtype      # dtype('int64')  -> data type of elements
arr.itemsize   # 8            -> bytes per element
arr.nbytes     # 48             -> total bytes used (size * itemsize)
```

## Dimensions — 1D, 2D, 3D
```python
d1 = np.array([1, 2, 3])                     # 1D -> a vector
d2 = np.array([[1, 2], [3, 4]])                # 2D -> a matrix
d3 = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])  # 3D -> a "stack" of matrices, common in image/tensor data
```
Note to self: this is the exact structure PyTorch tensors build on later — a batch of images is basically a 4D array (batch, height, width, channels). Understanding NumPy dimensions properly now saves confusion later with tensors.

## Data types (dtype)
NumPy arrays are homogeneous — every element MUST be the same type, unlike Python lists which can mix types freely.
```python
np.array([1, 2, 3]).dtype           # int64  (or int32 on some systems)
np.array([1.0, 2.0]).dtype            # float64
np.array([1, 2.5]).dtype                # float64 -> NumPy upcasts everything to float if mixed
np.array(['a', 'b']).dtype                # <U1 (unicode string)

# explicitly setting dtype
arr = np.array([1, 2, 3], dtype=np.float32)   # useful for controlling memory usage
```
Why this matters for ML: float32 vs float64 makes a real difference in memory and speed when training models — this is the same reason mixed precision training exists (I looked into this a bit given my RTX 3050's 4GB VRAM limit).

## Why NumPy is fast — vectorization in one line
```python
# Slow way (pure Python loop)
result = []
for x in range(1000000):
    result.append(x * 2)

# Fast way (vectorized NumPy)
arr = np.arange(1000000)
result = arr * 2
```
The vectorized version runs in optimized C code under the hood instead of Python's interpreter loop — this is usually a 10-100x speedup depending on the operation.
