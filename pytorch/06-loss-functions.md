# 06 — Loss Functions

A loss function is the single number that tells the network "how wrong are you right now." Everything else in training (autograd, optimizer) exists purely to make this number smaller.

## Regression: Mean Squared Error (MSE)

```python
import torch
import torch.nn as nn

criterion = nn.MSELoss()

predictions = torch.tensor([2.5, 0.0, 2.0])
targets = torch.tensor([3.0, -0.5, 2.0])

loss = criterion(predictions, targets)
print(loss)   # average of (pred - target)^2
```

Same MSE from scikit-learn regression metrics, genuinely identical formula, now used as the *training signal* instead of just an evaluation metric. Squaring the error means big mistakes get punished disproportionately more than small ones — a prediction off by 10 contributes 100x more loss than one off by 1, not 10x.

## Binary Classification: Binary Cross-Entropy (BCE)

```python
criterion = nn.BCELoss()

# model output must already be a probability (0 to 1), e.g. after Sigmoid
predictions = torch.tensor([0.9, 0.1, 0.8])
targets = torch.tensor([1.0, 0.0, 1.0])

loss = criterion(predictions, targets)
```

BCE punishes confident-and-wrong predictions much harder than MSE would. If the true label is 1 and the model confidently predicted 0.01, the loss shoots up toward infinity — this is intentional, it's what pushes the model hard to fix confidently wrong predictions fast.

Practical gotcha: `nn.BCELoss` expects the input already passed through sigmoid. There's a combined version, `nn.BCEWithLogitsLoss`, that takes raw logits and applies sigmoid internally — more numerically stable, and the one actually recommended for real use:

```python
criterion = nn.BCEWithLogitsLoss()
# pass in raw model output (no sigmoid applied), targets stay as 0/1
```

## Multi-class Classification: Cross-Entropy Loss

```python
criterion = nn.CrossEntropyLoss()

# raw logits, shape [batch_size, num_classes] — NOT softmaxed
logits = torch.tensor([[2.0, 1.0, 0.1], [0.1, 2.5, 0.3]])
# class indices, NOT one-hot encoded
targets = torch.tensor([0, 1])

loss = criterion(logits, targets)
```

Two things that genuinely trip people up here, worth repeating from file 05:
1. Pass **raw logits**, not softmax output — `CrossEntropyLoss` applies `log_softmax` internally.
2. Targets are **class index integers** (e.g. `0`, `1`, `2`), not one-hot vectors. Coming from a scikit-learn/pandas background where labels were often one-hot encoded for some frameworks, this is a real habit to unlearn.

## Building the mental link back to XGBoost/scikit-learn

Recap: in scikit-learn, `log_loss` for classification is mathematically the same idea as cross-entropy here — penalize confident wrong predictions heavily, reward confident correct ones. The formula genuinely didn't change moving from classical ML to DL, just the packaging.

## Custom loss functions

Sometimes the built-in losses don't fit — e.g. a business problem where under-predicting costs more than over-predicting. Writing a custom loss is just a normal Python function using tensor ops (so autograd can differentiate through it):

```python
def custom_weighted_mse(predictions, targets, under_penalty=2.0):
    diff = predictions - targets
    loss = torch.where(diff < 0, under_penalty * diff**2, diff**2)
    return loss.mean()
```

As long as every operation inside uses tensor math (not plain Python floats/loops that break the graph), autograd can still compute gradients through it automatically — genuinely one of the most powerful things about PyTorch, that loss functions are just differentiable Python code, no special framework magic needed.

## Quick reference table

| Task | Loss | Final layer output needed |
|---|---|---|
| Regression | `nn.MSELoss` | raw number |
| Binary classification | `nn.BCEWithLogitsLoss` | raw logit (no sigmoid) |
| Multi-class classification | `nn.CrossEntropyLoss` | raw logits (no softmax) |
| Multi-label classification (multiple true labels possible) | `nn.BCEWithLogitsLoss` (per label) | raw logits |

## Try this

```python
# Manually verify: for a single sample, true class = 1, logits = [1.0, 3.0, 0.5]
# compute CrossEntropyLoss by hand: softmax the logits, take -log(prob of correct class)
# then check against nn.CrossEntropyLoss output
```

Doing the softmax + negative-log-likelihood by hand once, matching it to the PyTorch output, is genuinely the best way to stop treating cross-entropy as a black box formula.

## What's next
File 07 — putting autograd + nn.Module + loss functions together into the actual training loop, which is where all of this finally becomes "the network is learning."
