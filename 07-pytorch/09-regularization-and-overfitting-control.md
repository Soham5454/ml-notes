# 09 — Regularization & Overfitting Control

Recap from the XGBoost bridge file (file 11 there): I already flagged that `reg_alpha`/`reg_lambda` (L1/L2) map onto "weight decay" in DL, and dropout is a genuinely DL-specific mechanism. This file is where that gets properly explained instead of just mentioned.

## Weight decay (L2 regularization) — the direct XGBoost carryover

```python
optimizer = torch.optim.Adam(model.parameters(), lr=0.001, weight_decay=1e-4)
```

Mechanically identical purpose to `reg_lambda` in XGBoost — adds a penalty proportional to the squared magnitude of the weights into the loss, discouraging any single weight from growing huge and over-relying on one feature/pathway. `weight_decay` here is just the PyTorch-side name for the same L2 penalty coefficient.

## Dropout — the genuinely DL-specific one

```python
import torch.nn as nn

class MLPWithDropout(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Linear(10, 64)
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(p=0.3)     # 30% of neurons randomly zeroed each forward pass
        self.layer2 = nn.Linear(64, 1)

    def forward(self, x):
        x = self.relu(self.layer1(x))
        x = self.dropout(x)
        x = self.layer2(x)
        return x
```

During training, dropout randomly zeroes out 30% of the neurons in that layer on every forward pass — forces the network to not depend too heavily on any single neuron/pathway, since that neuron might not be there next batch. It's a genuinely different mechanism from L1/L2 (which just shrinks weight magnitudes) but the underlying goal is the same: reduce over-reliance on any single feature/pathway.

Critical practical detail, connects straight back to file 04's `train()`/`eval()` note: dropout is ONLY active during `model.train()`. Calling `model.eval()` automatically disables it (all neurons active, no randomness) — which is exactly what's wanted at inference/validation time, since randomly dropping neurons during evaluation would make results non-reproducible and worse.

## Batch Normalization — stabilizes training, has a regularizing side-effect

```python
class MLPWithBatchNorm(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Linear(10, 64)
        self.bn1 = nn.BatchNorm1d(64)
        self.relu = nn.ReLU()
        self.layer2 = nn.Linear(64, 1)

    def forward(self, x):
        x = self.layer1(x)
        x = self.bn1(x)
        x = self.relu(x)
        x = self.layer2(x)
        return x
```

BatchNorm normalizes activations within each mini-batch (mean 0, variance 1) before the activation function, then applies its own learnable scale/shift. Main purpose is actually training stability/speed (lets you use higher learning rates, reduces sensitivity to weight initialization), but it has a genuine mild regularizing side-effect too since the batch statistics introduce a bit of noise, similar in spirit to dropout.

Same `train()`/`eval()` sensitivity as dropout — BatchNorm behaves differently in each mode (uses batch statistics during training, running averages during eval), so forgetting `model.eval()` before validation/inference is a bug here too.

## Early stopping — identical pattern from XGBoost, just monitoring a different axis

```python
best_val_loss = float('inf')
patience = 5
patience_counter = 0

for epoch in range(num_epochs):
    # ... training step ...
    # ... validation step, get val_loss ...

    if val_loss < best_val_loss:
        best_val_loss = val_loss
        patience_counter = 0
        torch.save(model.state_dict(), 'best_model.pt')   # save the best version (file 14 covers this properly)
    else:
        patience_counter += 1
        if patience_counter >= patience:
            print(f"Early stopping at epoch {epoch+1}")
            break
```

Genuinely the exact same diagnostic logic flagged in the XGBoost bridge file — watch validation performance, stop once it stops improving for `patience` rounds. Only difference: "rounds" here means "epochs" instead of "boosting rounds." The train/val divergence plot pattern from XGBoost file 05/06 is the identical shape to watch for here.

## Data augmentation — a regularization technique unique to unstructured data

```python
from torchvision import transforms

train_transform = transforms.Compose([
    transforms.RandomHorizontalFlip(),
    transforms.RandomRotation(10),
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),
])
```

No equivalent in XGBoost/tabular world — this artificially creates variations of training images (flipped, rotated, color-jittered) so the model sees more diversity without needing more real data. This becomes genuinely essential once CNNs (file 10) get trained on small image datasets.

## Quick summary table — the one to actually reference

| Technique | What it targets | Analogous XGBoost concept |
|---|---|---|
| Weight decay (L2) | Large weight magnitudes | `reg_lambda` |
| Dropout | Over-reliance on specific neurons | No direct equivalent (closest: random feature subsampling `colsample_bytree`) |
| BatchNorm | Training instability + mild regularization | No equivalent |
| Early stopping | Training too long past best validation point | Early stopping (identical concept) |
| Data augmentation | Insufficient training data diversity (images/text/audio) | No equivalent (tabular data can't be "flipped") |

## Try this

```python
# Take the MLP from file 07, add Dropout(0.3) and BatchNorm1d after the first hidden layer
# Train it on the same toy data with and without dropout/batchnorm, compare final val_loss
```

Seeing whether validation loss changes (usually improves, or at least stabilizes, with a bit of dropout on toy data) is a genuinely good gut-check that these techniques are being applied correctly and not just added as decoration.

## What's next
File 10 — Convolutional Neural Networks, the first genuinely new architecture family (as opposed to just stacked linear layers) — this is where "unstructured data, learned features" from the XGBoost bridge file starts becoming concrete.
