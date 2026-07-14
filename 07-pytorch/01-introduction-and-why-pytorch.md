# 01 — Introduction & Why PyTorch

Starting the DL phase of the roadmap now. Coming straight off XGBoost, so I keep comparing everything back to that mentally — that's actually helping, not confusing me.

## What PyTorch actually is

PyTorch is a deep learning framework built around one core object — the **tensor** — plus an automatic differentiation engine (autograd) that tracks every operation done on tensors so it can compute gradients for you later. That's it. Everything else (nn.Module, optimizers, DataLoader) is convenience built on top of these two ideas.

Genuinely the whole point of a DL framework: you never have to hand-derive backpropagation math. You define the forward pass, PyTorch figures out the backward pass by itself.

## Why not just use NumPy for neural networks?

I asked myself this honestly before starting. Three real reasons:

1. **No autograd in NumPy.** You'd have to manually write the gradient/backprop math for every layer. Doable for a toy 2-layer net, genuinely painful for anything real.
2. **No native GPU support in NumPy.** NumPy arrays live on CPU only. PyTorch tensors move to GPU with one line (`.to('cuda')`) and every matrix multiply then runs on GPU cores.
3. **No training-loop ecosystem.** DataLoader, loss functions, optimizers, pretrained models — none of that exists in raw NumPy.

Recap from the XGBoost bridge file: this is exactly the "unstructured data needs learned features + heavy matrix math needs GPU" gap I noted. PyTorch is the tool built to fill it.

## PyTorch vs TensorFlow (just so I know the landscape)

Not going deep here since I'm committing to PyTorch for this roadmap, but worth knowing:
- PyTorch uses a **dynamic computation graph** (built on the fly as code runs) — feels like normal Python, easy to debug.
- TensorFlow (older versions) used a **static graph** (define first, run later) — faster in production historically but harder to debug.
- Research world runs mostly on PyTorch now. Industry deployment sometimes still TensorFlow/JAX. Good enough reason to learn PyTorch first.

## Installation

```bash
# CPU only
pip install torch torchvision torchaudio

# With CUDA (check pytorch.org for the exact command matching your CUDA version)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Sanity check after install:

```python
import torch

print(torch.__version__)
print(torch.cuda.is_available())   # True if you have a working GPU + CUDA setup
```

If `cuda.is_available()` is `False` on a machine that does have an NVIDIA GPU, it's almost always a CUDA/driver version mismatch, not a PyTorch bug — note to self for later debugging.

## The mental model I'm carrying forward

Old world (scikit-learn/XGBoost): `.fit(X, y)` → `.predict(X)`. One call does everything internally.

New world (PyTorch): I write the training loop myself — forward pass, compute loss, backward pass, optimizer step — in explicit Python code. More manual, but genuinely gives full control, which matters once architectures get custom (CNNs, RNNs, custom losses).

## Try this (5 min sanity exercise)

```python
import torch
x = torch.tensor([1.0, 2.0, 3.0])
y = x * 2
print(y)          # tensor([2., 4., 6.])
print(y.dtype)     # torch.float32
```

If this runs without error, the environment is good and I can move to actual tensor operations in file 02.

## What's next
File 02 — tensors in depth (creation, indexing, reshaping, broadcasting, GPU tensors) since literally everything in PyTorch is built on this one data structure.
