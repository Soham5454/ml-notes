# 08 — Datasets & DataLoaders

File 07's training loop assumed all data fit in one tensor in memory. Real datasets — thousands/millions of images, text files, CSVs too big for RAM — need a proper pipeline. That's what `Dataset` + `DataLoader` are for.

## The `Dataset` class — how PyTorch wants data organized

```python
from torch.utils.data import Dataset

class MyDataset(Dataset):
    def __init__(self, features, labels):
        self.features = features
        self.labels = labels

    def __len__(self):
        return len(self.labels)

    def __getitem__(self, idx):
        return self.features[idx], self.labels[idx]
```

Only two methods are required: `__len__` (how many samples exist) and `__getitem__` (return one sample given an index). That's genuinely the entire contract — PyTorch's DataLoader handles everything else (shuffling, batching, parallel loading) once these two methods exist.

## `DataLoader` — the batching engine

```python
from torch.utils.data import DataLoader

dataset = MyDataset(X, y)
train_loader = DataLoader(dataset, batch_size=32, shuffle=True)

for batch_X, batch_y in train_loader:
    print(batch_X.shape)   # torch.Size([32, num_features])
    break
```

Key arguments worth actually understanding:
- `batch_size` — how many samples per forward/backward pass. Bigger batches = more stable gradient estimates but more memory use and slightly less noisy (sometimes less generalizable) updates.
- `shuffle=True` — genuinely essential for training data, so the model doesn't learn spurious patterns from data ordering. Should be `False` for validation/test loaders — order doesn't matter there and shuffling wastes compute.
- `num_workers` — number of background processes for loading data in parallel, useful once loading from disk (images) becomes a bottleneck.

## Why batching matters beyond just "fits in memory"

Recap link forward to file 07's mini-batch note: training on mini-batches (instead of the whole dataset at once, called "batch gradient descent," or one sample at a time, called pure SGD) is a genuine sweet spot — noisy-enough gradients to escape shallow local minima, stable-enough to converge reliably. This is precisely why the technique is called **mini-batch stochastic gradient descent** even when using Adam.

## `torchvision.transforms` — preprocessing for image data

```python
from torchvision import transforms

transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),                      # converts PIL image to tensor, scales to [0,1]
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                          std=[0.229, 0.224, 0.225])   # ImageNet stats, standard for transfer learning (file 12)
])
```

Recap from scikit-learn feature scaling notes: normalization matters even MORE in DL, not less — the same point flagged in the XGBoost bridge file. Unscaled pixel values (0-255) genuinely slow convergence and can destabilize training badly.

## Built-in datasets — using them directly (no custom class needed)

```python
from torchvision import datasets

train_data = datasets.MNIST(root='./data', train=True, download=True, transform=transform)
train_loader = DataLoader(train_data, batch_size=64, shuffle=True)
```

MNIST (handwritten digit images) is the classic "hello world" dataset for CNNs (file 10) — using it here specifically because it's small, fast to train on, and lets the focus stay on the mechanics rather than data wrangling.

## A full custom Dataset example — CSV-based (closer to real tabular DL use)

```python
import pandas as pd
import torch
from torch.utils.data import Dataset

class CSVDataset(Dataset):
    def __init__(self, csv_path, feature_cols, label_col):
        df = pd.read_csv(csv_path)
        self.X = torch.tensor(df[feature_cols].values, dtype=torch.float32)
        self.y = torch.tensor(df[label_col].values, dtype=torch.long)

    def __len__(self):
        return len(self.y)

    def __getitem__(self, idx):
        return self.X[idx], self.y[idx]
```

Genuinely satisfying moment here — this is the pandas skillset (file references to earlier pandas notes) plugging directly into the PyTorch pipeline. The classical ML data-wrangling habits don't get thrown away moving into DL, they become the first stage of the pipeline.

## Train/val/test split with DataLoader

```python
from torch.utils.data import random_split

full_dataset = CSVDataset('data.csv', feature_cols=[...], label_col='target')
train_size = int(0.8 * len(full_dataset))
val_size = len(full_dataset) - train_size
train_ds, val_ds = random_split(full_dataset, [train_size, val_size])

train_loader = DataLoader(train_ds, batch_size=32, shuffle=True)
val_loader = DataLoader(val_ds, batch_size=32, shuffle=False)
```

Same train/val split discipline carried all the way from scikit-learn, just expressed through `random_split` instead of `train_test_split`.

## Try this

```python
# Build a custom Dataset from a small synthetic CSV (make one with pandas: 100 rows, 4 feature columns, 1 label column)
# Wrap it in a DataLoader with batch_size=16, shuffle=True
# Iterate through one epoch and print the shape of each batch
```

Building this end-to-end once — CSV to Dataset to DataLoader to iterating batches — is genuinely the last piece needed before real training loops (file 07's pattern) can run on any real dataset instead of just toy in-memory tensors.

## What's next
File 09 — regularization and overfitting control (dropout, batchnorm, weight decay) — now that real data pipelines exist, overfitting becomes a genuine practical concern, not just a theoretical one.
