# NumPy — A to Z

My notes on NumPy, written while going through it properly for the ML roadmap (after Python basics, before pandas). NumPy is basically the foundation everything else in the ML stack sits on top of — pandas, scikit-learn, PyTorch, all of it use NumPy arrays under the hood, so I wanted to actually understand this instead of skimming it.

## Index

| # | Topic | File |
|---|-------|------|
| 01 | Introduction & Arrays Basics | [01-introduction-arrays.md](01-introduction-arrays.md) |
| 02 | Array Creation | [02-array-creation.md](02-array-creation.md) |
| 03 | Indexing & Slicing | [03-indexing-slicing.md](03-indexing-slicing.md) |
| 04 | Array Manipulation & Reshaping | [04-array-manipulation-reshaping.md](04-array-manipulation-reshaping.md) |
| 05 | Broadcasting | [05-broadcasting.md](05-broadcasting.md) |
| 06 | Vectorized Operations & Math | [06-vectorized-operations-math.md](06-vectorized-operations-math.md) |
| 07 | Aggregation & Statistics | [07-aggregation-statistics.md](07-aggregation-statistics.md) |
| 08 | Random Module | [08-random-module.md](08-random-module.md) |
| 09 | Linear Algebra | [09-linear-algebra.md](09-linear-algebra.md) |
| 10 | Boolean Masking & Fancy Indexing | [10-boolean-masking-fancy-indexing.md](10-boolean-masking-fancy-indexing.md) |
| 11 | File I/O & Performance | [11-file-io-performance.md](11-file-io-performance.md) |
| 12 | ML Use Cases (why this all matters) | [12-ml-use-cases.md](12-ml-use-cases.md) |
| 13 | Deep Learning Essentials (einsum, stacking, memory layout) | [13-deep-learning-essentials.md](13-deep-learning-essentials.md) |

## Setup
```bash
pip install numpy
```
```python
import numpy as np   # 'np' is the universal convention, everyone uses it
```

## Roadmap position
Python Basics → **NumPy (you are here)** → pandas → matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Hugging Face
