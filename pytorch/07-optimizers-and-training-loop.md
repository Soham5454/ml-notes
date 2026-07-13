# 07 — Optimizers & The Training Loop

This is the file everything since 01 has been building toward. Tensors, autograd, nn.Module, activations, loss functions — they all snap together here.

## What an optimizer actually does

Once `loss.backward()` computes gradients for every parameter, something has to actually use those gradients to update the weights. That's the optimizer's entire job: `new_weight = old_weight - learning_rate * gradient`, in the simplest case (plain SGD).

```python
import torch.optim as optim

optimizer = optim.SGD(model.parameters(), lr=0.01)
```

## SGD vs Adam — the two to actually know well

**SGD (Stochastic Gradient Descent):** the basic update rule above. Simple, predictable, but can be slow to converge and sensitive to learning rate choice.

**Adam:** adapts the learning rate per-parameter automatically based on recent gradient history (combines ideas called momentum and RMSProp). Genuinely the default choice for most modern deep learning — converges faster and is more forgiving of a not-perfectly-tuned learning rate.

```python
optimizer = optim.Adam(model.parameters(), lr=0.001)
```

Recap link to file 11 (XGBoost bridge): `lr` here is the exact same concept as XGBoost's `learning_rate` — too high overshoots and destabilizes, too low takes forever to converge. Literally the same tradeoff, same name even.

## The training loop — the pattern that repeats forever

This exact structure shows up in every PyTorch project ever written, CNNs, RNNs, transformers, all of it:

```python
import torch
import torch.nn as nn
import torch.optim as optim

model = MLP(10, 32, 2)               # from file 04
criterion = nn.CrossEntropyLoss()     # from file 06
optimizer = optim.Adam(model.parameters(), lr=0.001)

num_epochs = 20

for epoch in range(num_epochs):
    optimizer.zero_grad()             # STEP 1: clear old gradients

    outputs = model(X_train)           # STEP 2: forward pass
    loss = criterion(outputs, y_train)  # STEP 3: compute loss

    loss.backward()                    # STEP 4: backward pass (autograd magic)
    optimizer.step()                   # STEP 5: update weights

    print(f"Epoch {epoch+1}/{num_epochs}, Loss: {loss.item():.4f}")
```

Five steps, always in this order. Genuinely worth memorizing as a rhythm rather than reading it line by line every time: **zero_grad → forward → loss → backward → step**.

## Why `zero_grad()` has to come first — the bug that will happen eventually

Recap from file 03: gradients accumulate by default, they don't reset automatically. Skip `zero_grad()` and every epoch's gradients pile on top of the last one, and the model updates using garbage combined gradients. This is genuinely one of the most common real bugs in PyTorch code — a loss that goes haywire or refuses to decrease is often just a missing `zero_grad()`.

## Adding a validation loop (the train/val split discipline carrying forward)

Recap from scikit-learn file 03 and XGBoost file 06 — train/validation split discipline doesn't change here, it's just now inside the epoch loop:

```python
for epoch in range(num_epochs):
    model.train()                      # enable dropout/batchnorm training behavior
    optimizer.zero_grad()
    outputs = model(X_train)
    loss = criterion(outputs, y_train)
    loss.backward()
    optimizer.step()

    model.eval()                       # disable dropout/batchnorm training behavior
    with torch.no_grad():              # no need to track gradients during validation
        val_outputs = model(X_val)
        val_loss = criterion(val_outputs, y_val)

    print(f"Epoch {epoch+1}: train_loss={loss.item():.4f}, val_loss={val_loss.item():.4f}")
```

If `train_loss` keeps dropping but `val_loss` starts climbing back up — that's overfitting, the exact same train/val divergence pattern from the XGBoost early-stopping notes (file 06 there), just plotted against epochs instead of boosting rounds.

## Mini-batches — why the loop above isn't quite the full picture yet

The loop above assumes the entire dataset fits in one forward pass, which is unrealistic for real datasets. Real training loops iterate over batches using a `DataLoader` (file 08 is entirely about this):

```python
for epoch in range(num_epochs):
    for batch_X, batch_y in train_loader:     # DataLoader handles the batching
        optimizer.zero_grad()
        outputs = model(batch_X)
        loss = criterion(outputs, batch_y)
        loss.backward()
        optimizer.step()
```

Same five-step rhythm, just nested one level deeper inside a batch loop. This is genuinely the real, final shape training loops take from here on — every file from 08 onward builds on exactly this structure.

## Try this

```python
# Using the MLP from file 04, train it on random toy data (X = torch.randn(100,10), y = torch.randint(0,2,(100,)))
# for 50 epochs, print loss every 10 epochs, and confirm loss trends downward
```

Watching the loss number actually go down, epoch by epoch, on made-up random-ish data is genuinely the single most satisfying "click" moment in this whole roadmap — it's the first time the abstract theory turns into a number visibly improving on screen.

## What's next
File 08 — Dataset and DataLoader, replacing toy in-memory tensors with a proper, reusable way to feed real data into this loop in batches.
