# 13 — Deep Learning Essentials (Einsum, Stacking, Memory Layout, Views vs Copies)

Adding this as a separate file instead of editing the earlier ones since those are already pushed — this one's specifically for the NumPy concepts that matter more once you're doing DL (PyTorch tensors) rather than just classic ML.

## np.einsum — Einstein summation notation
This is the one I was most tempted to skip, but it's genuinely everywhere in DL code once you start reading transformer/attention implementations, so worth actually sitting with it.

### The basic idea
`einsum` lets you describe exactly how you want dimensions to be summed/multiplied using a string notation, instead of chaining `.reshape()`, `.transpose()`, and `@` together. It looks intimidating at first but it's really just a compact way to say "match these indices, sum over that one."

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.einsum('i,i->i', a, b)     # [4 10 18]   -> element-wise multiply (same as a * b)
np.einsum('i,i->', a, b)        # 32           -> dot product (same as np.dot(a, b))
```

### Matrix multiplication via einsum
```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

np.einsum('ij,jk->ik', A, B)     # same result as A @ B
```
Reading the notation: `ij,jk->ik` means "take a matrix indexed by i,j and one indexed by j,k, sum over the shared j, and give me back something indexed by i,k." Once this clicked, matrix multiplication finally felt like something I understood mechanically, not just something `@` magically does.

### Batch matrix multiplication (the actual DL use case)
```python
batch_A = np.random.rand(32, 4, 5)     # 32 matrices, each 4x5   (batch_size, m, n)
batch_B = np.random.rand(32, 5, 6)       # 32 matrices, each 5x6   (batch_size, n, p)

result = np.einsum('bij,bjk->bik', batch_A, batch_B)     # shape (32, 4, 6)
```
This is exactly the shape of operation that happens constantly in transformer attention layers — batched matrix multiplication across many samples at once. `torch.einsum` uses the identical syntax, so this transfers directly.

### Transpose via einsum
```python
A = np.array([[1, 2, 3], [4, 5, 6]])
np.einsum('ij->ji', A)     # same as A.T
```

## Stacking for batches — np.stack vs np.concatenate
This distinction matters a lot in DL specifically, since building a "batch dimension" is one of the first things that happens before any forward pass.

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.stack([a, b])          # shape (2, 3)  -> creates a NEW dimension, stacks them as separate rows
np.concatenate([a, b])      # shape (6,)     -> just joins them end to end, no new dimension
```
`stack` is what you use when you have a list of individual samples (each shape `(features,)`) and want to combine them into a proper batch of shape `(batch_size, features)`. `concatenate` assumes the dimension you're joining along already exists.

```python
images = [np.random.rand(28, 28) for _ in range(32)]     # 32 individual 28x28 images
batch = np.stack(images)                                     # shape (32, 28, 28) -> ready to feed a model
```

### axis parameter with stack
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
np.stack([a, b], axis=0)     # shape (2, 3)  -> stack as rows (default)
np.stack([a, b], axis=1)       # shape (3, 2)  -> stack as columns instead
```

## Memory layout — C-order vs Fortran-order, and why it matters
NumPy arrays are stored as one flat block of memory. How that flat block maps to the array's rows/columns is the "memory layout," and it affects speed.

- **C-order (row-major)** — default in NumPy. Elements of the SAME ROW are stored next to each other in memory.
- **Fortran-order (column-major)** — elements of the SAME COLUMN are stored next to each other instead.

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
a.flags['C_CONTIGUOUS']     # True  -> stored in row-major order by default
a.flags['F_CONTIGUOUS']       # False

a_fortran = np.asfortranarray(a)
a_fortran.flags['F_CONTIGUOUS']     # True
```

### Why this matters for DL specifically
Operations that access memory in the "natural" order for its layout are faster, since the CPU/GPU can read contiguous memory efficiently instead of jumping around. When you do `.transpose()` or certain slicing operations, the resulting array is often NO LONGER contiguous — it's just a different VIEW over the same underlying memory with different strides.
```python
a = np.arange(12).reshape(3, 4)
a.strides         # (32, 8)  -> bytes to step to get to the next row / next column

a_t = a.T
a_t.strides         # (8, 32)  -> transposed view, same memory, different stride pattern, NOT contiguous anymore
a_t.flags['C_CONTIGUOUS']     # False
```
This is EXACTLY why `.contiguous()` exists in PyTorch — after operations like `.transpose()` or `.permute()`, a tensor's memory layout becomes non-contiguous, and some GPU operations require contiguous memory to run efficiently (or at all). You'll hit errors like "view size is not compatible with input tensor's size and stride" in PyTorch specifically because of this exact concept, so understanding it here at the NumPy level means that error will make sense instead of feeling random when it eventually happens.

## Views vs Copies — the deep dive
Touched on this earlier in the indexing file, but it deserves proper treatment here since in-place operations are a genuinely big deal for memory efficiency on a 4GB VRAM card.

### Checking if something is a view
```python
a = np.arange(10)
b = a[2:5]              # slice -> this is a VIEW
b.base is a                # True -> confirms b shares memory with a

c = a[[2, 3, 4]]           # fancy indexing -> this is a COPY
c.base is None                # True -> confirms c is independent, owns its own memory
```
Rule of thumb: **basic slicing (`a[1:5]`) → view. Fancy indexing (`a[[1,3,5]]`) or boolean masking (`a[a>3]`) → copy.**

### np.shares_memory — checking directly
```python
a = np.arange(10)
b = a[2:5]
np.shares_memory(a, b)     # True
```

### Why in-place operations matter for DL memory management
```python
a = np.random.rand(1000, 1000)

b = a + 1          # creates a NEW array, uses extra memory temporarily
a += 1                # modifies IN PLACE, no extra memory allocated
```
On a memory-constrained GPU, this exact distinction (`x = x + 1` vs `x += 1`, or in PyTorch, `x.add_()` with the trailing underscore convention for in-place ops) is a real technique for fitting bigger batches/models into limited VRAM — directly connects to the mixed precision / gradient checkpointing stuff I looked into earlier for the RTX 3050.

### A gotcha combining views and in-place ops
```python
a = np.arange(10)
b = a[2:5]        # view
b += 100             # modifies the VIEW in place...
print(a)               # [0 1 102 103 104 5 6 7 8 9]  -> original array changed too, since b shares memory with a
```
This is worth remembering precisely because it's the kind of bug that's silent and confusing — code runs fine, no error, just wrong values somewhere downstream because you modified a view without realizing it was connected to the original array.

## Quick summary — what to actually remember
- `einsum` = flexible notation for exact multiply-and-sum patterns, essential for reading/writing attention/transformer code later
- `np.stack` = adds a new batch dimension; `np.concatenate` = joins along an existing one
- Non-contiguous memory (after transpose/permute) is why PyTorch sometimes demands `.contiguous()` before certain ops
- Basic slicing = view (shares memory, fast but risky); fancy indexing/masking = copy (safe but uses more memory)
