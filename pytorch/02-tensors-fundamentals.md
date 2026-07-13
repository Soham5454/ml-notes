# 02 — Tensors Fundamentals

Everything in PyTorch is a tensor. A tensor is genuinely just a NumPy array with superpowers — it can live on GPU, and it remembers the operations done on it (for autograd, covered in file 03).

## Creating tensors

```python
import torch

a = torch.tensor([1, 2, 3])                # from a list
b = torch.zeros(3, 4)                      # 3x4 zeros
c = torch.ones(2, 2)                       # 2x2 ones
d = torch.rand(3, 3)                       # uniform random [0,1)
e = torch.randn(3, 3)                      # normal distribution, mean 0 std 1
f = torch.arange(0, 10, 2)                 # tensor([0,2,4,6,8])
g = torch.eye(3)                           # identity matrix

print(a.shape, a.dtype)   # torch.Size([3]) torch.int64
```

Note to self: `torch.rand` = uniform, `torch.randn` = normal(Gaussian). I keep mixing these up so writing it down explicitly.

## NumPy <-> Tensor conversion (this bridge matters a LOT)

```python
import numpy as np

np_arr = np.array([1, 2, 3])
t = torch.from_numpy(np_arr)     # numpy -> tensor (shares memory!)
back = t.numpy()                  # tensor -> numpy (also shares memory!)
```

Genuinely important gotcha: `from_numpy` and `.numpy()` **share the same underlying memory**. Changing one changes the other. This bit me once during a debugging session — modified a numpy array thinking it was a copy, and the tensor changed too.

## Tensor attributes

```python
x = torch.randn(3, 4)
x.shape       # torch.Size([3, 4])
x.dtype       # torch.float32
x.device      # device(type='cpu')
x.requires_grad  # False by default, covered properly in file 03
```

## Indexing & slicing — basically identical to NumPy

```python
x = torch.arange(12).reshape(3, 4)
x[0]          # first row
x[:, 1]       # second column
x[1:, 2:]     # sub-matrix from row 1, col 2 onward
x[x > 5]      # boolean masking — same as pandas/numpy filtering
```

Recap link to pandas notes: boolean masking behaves exactly the same mental pattern as `df[df['col'] > 5]`. Same idea, different container.

## Reshaping — the thing that trips up beginners (including me, initially)

```python
x = torch.arange(12)
x.reshape(3, 4)     # 3 rows, 4 cols
x.view(3, 4)        # same result, BUT requires the tensor to be memory-contiguous

x.reshape(-1, 4)    # -1 means "figure this dimension out automatically"
x.unsqueeze(0)      # adds a dimension at position 0 — shape becomes [1, 12]
x.squeeze()         # removes dimensions of size 1
```

`view` vs `reshape`: `view` fails if the tensor's memory layout isn't contiguous (e.g. after a `.transpose()`). `reshape` will silently make a copy if needed. Honestly, default to `reshape` unless there's a specific perf reason to use `view`.

`unsqueeze`/`squeeze` matter constantly in DL because a lot of layers expect a specific number of dimensions — e.g. a single image needs a batch dimension added (`[C,H,W]` → `[1,C,H,W]`) before it can go through a model that expects batches. This will come up again in file 10 (CNNs) — noting it now so it's not a surprise later.

## Broadcasting — same rule as NumPy

```python
a = torch.ones(3, 4)
b = torch.tensor([1, 2, 3, 4])   # shape [4]
c = a + b                        # b is broadcast across all 3 rows
```

Rule (same as NumPy): compare shapes from the right; dimensions must be equal, or one of them must be 1, or one of them must not exist. If none of these hold, it errors out.

## Basic operations

```python
x = torch.tensor([1., 2., 3.])
y = torch.tensor([4., 5., 6.])

x + y            # element-wise add
x * y            # element-wise multiply (NOT matrix multiply!)
torch.dot(x, y)  # dot product -> scalar

A = torch.rand(2, 3)
B = torch.rand(3, 4)
A @ B            # matrix multiplication, shape [2,4]
torch.matmul(A, B)  # same thing, @ is shorthand for this
```

The `*` vs `@` distinction is genuinely one of the most common bugs beginners hit. `*` = element-wise. `@` = matrix multiply. Recap from NumPy notes — same distinction existed there, carrying forward unchanged.

## GPU tensors

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

x = torch.rand(3, 3).to(device)   # move tensor to GPU (if available)
# or equivalently
x = x.cuda() if torch.cuda.is_available() else x
```

Important rule: **all tensors involved in an operation must be on the same device.** Mixing a CPU tensor with a GPU tensor throws a runtime error. This becomes a real habit to build — always check `.device` when debugging a mysterious crash.

## Try this (to make it stick)

```python
# Predict the output shape BEFORE running this, then check
a = torch.rand(4, 3)
b = torch.rand(3)
result = a @ b
print(result.shape)   # what do you expect? run it and confirm
```

If I can predict tensor shapes in my head before running code, that's the real sign this has clicked — shape mismatches are genuinely the #1 error type in all of PyTorch, so building shape-intuition early pays off constantly later.

## What's next
File 03 — autograd. This is where tensors stop being "just arrays" and start being the thing that makes neural network training possible.
