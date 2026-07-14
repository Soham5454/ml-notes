# 13 — GPU/CUDA Training & Mixed Precision

Recap from file 01: neural networks are enormous amounts of matrix multiplication, and that's genuinely impractical on CPU alone beyond toy examples. This file is about actually using the GPU properly, not just knowing it exists.

## The device pattern — the one to use in literally every script from now on

```python
import torch

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

model = model.to(device)   # move model's parameters to GPU
```

Every batch of data also has to move to the same device inside the training loop:

```python
for images, labels in train_loader:
    images, labels = images.to(device), labels.to(device)
    outputs = model(images)
    loss = criterion(outputs, labels)
    loss.backward()
    optimizer.step()
```

The rule flagged back in file 02 matters constantly here: **all tensors in an operation must be on the same device.** Forgetting to move either the model or the data (or moving only one of them) throws a genuinely common runtime error — `Expected all tensors to be on the same device`.

## Why GPUs are so much faster for this specific workload

CPUs have a handful of powerful cores good at complex sequential logic. GPUs have thousands of simpler cores built specifically for doing the same operation on lots of data simultaneously — which is exactly what matrix multiplication is: the same multiply-and-add pattern repeated across every element. Recap from NumPy notes on the `@` operator — the actual math didn't change from CPU to GPU, just how many of those multiply-adds happen in parallel at once.

## Checking GPU memory usage — a genuinely practical habit

```python
print(torch.cuda.memory_allocated() / 1e9, "GB allocated")
print(torch.cuda.memory_reserved() / 1e9, "GB reserved")
```

"CUDA out of memory" errors are genuinely one of the most common real-world PyTorch problems once working with bigger models/batches. Standard fixes, in order of what to try first:
1. Reduce `batch_size` in the DataLoader.
2. Use a smaller model, or the feature-extraction transfer learning approach (file 12) instead of full fine-tuning.
3. Use mixed precision (below) — genuinely cuts memory usage significantly.
4. Clear unused cached memory: `torch.cuda.empty_cache()` (helps in some cases, not a magic fix for a model that's genuinely too big).

## Mixed precision training — the memory/speed upgrade

By default, PyTorch tensors use 32-bit floating point (`float32`). Mixed precision training uses 16-bit floats (`float16`) for most operations, keeping 32-bit precision only where numerically necessary (like gradient accumulation) — genuinely close to half the memory usage and often a real speedup on modern GPUs with dedicated hardware for fp16 math (Tensor Cores).

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for images, labels in train_loader:
    images, labels = images.to(device), labels.to(device)
    optimizer.zero_grad()

    with autocast():                      # forward pass runs in mixed precision
        outputs = model(images)
        loss = criterion(outputs, labels)

    scaler.scale(loss).backward()          # scales loss to prevent tiny gradients underflowing in fp16
    scaler.step(optimizer)                  # unscales and calls optimizer.step()
    scaler.update()                          # adjusts the scale factor for next iteration
```

`GradScaler` exists because fp16 has a much smaller numeric range than fp32 — very small gradient values can literally round down to zero in fp16 ("underflow"), silently breaking training. Scaling the loss up before `.backward()`, then unscaling gradients before the optimizer step, avoids this without changing the actual math result. Genuinely worth using this pattern by default on any real GPU training run rather than hand-rolling fp16 manually.

## Multi-GPU — just aware of it, not deep-diving

```python
if torch.cuda.device_count() > 1:
    model = torch.nn.DataParallel(model)   # simplest multi-GPU option, splits batches across GPUs
```

`DataParallel` is genuinely the simplest entry point but has real overhead and is considered somewhat outdated; `DistributedDataParallel` is the modern, more efficient approach for serious multi-GPU/multi-node training. Filing this away as a "look deeper when actually needed" topic rather than something to master right now — single-GPU training covers the vast majority of learning/personal-project needs.

## Benchmarking CPU vs GPU — worth actually seeing once

```python
import time

x_cpu = torch.randn(5000, 5000)
x_gpu = x_cpu.to(device)

start = time.time()
_ = x_cpu @ x_cpu
print(f"CPU: {time.time() - start:.4f}s")

torch.cuda.synchronize()   # important! GPU ops are async, must sync before timing
start = time.time()
_ = x_gpu @ x_gpu
torch.cuda.synchronize()
print(f"GPU: {time.time() - start:.4f}s")
```

`torch.cuda.synchronize()` matters genuinely — GPU operations are launched asynchronously (the CPU code continues immediately without waiting), so timing GPU code without syncing first gives a falsely fast, meaningless number.

## Try this

```python
# Time a matrix multiply of size 8000x8000 on CPU vs GPU (if available)
# Then wrap a small training loop in autocast()+GradScaler and compare memory_allocated() before/after
```

Actually seeing the CPU vs GPU timing gap firsthand (often 20-50x+ for large matrix multiplies) makes file 01's "impractical without GPU" claim concrete instead of just something read and accepted on faith.

## What's next
File 14 — saving and loading models properly, so all this training effort doesn't evaporate the moment the Python session closes.
