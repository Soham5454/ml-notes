# 14 — Saving & Loading Models

Referenced back in file 09's early stopping example (`torch.save(model.state_dict(), 'best_model.pt')`) without fully explaining it. This file covers it properly.

## `state_dict` — the thing that actually gets saved

```python
model = MLP(10, 32, 2)   # from file 04
print(model.state_dict().keys())
# odict_keys(['layer1.weight', 'layer1.bias', 'layer2.weight', 'layer2.bias'])
```

A `state_dict` is genuinely just a Python dictionary mapping each layer's name to its current weight/bias tensors. It does NOT contain the model's architecture/code — just the numbers. This is the recommended, official way to save PyTorch models, as opposed to saving the whole model object directly.

## Saving and loading — the standard, recommended pattern

```python
# Saving
torch.save(model.state_dict(), 'model_weights.pt')

# Loading — requires re-creating the model architecture FIRST
model = MLP(10, 32, 2)                              # must match the original architecture exactly
model.load_state_dict(torch.load('model_weights.pt'))
model.eval()                                          # set to eval mode before inference (recap file 04/09)
```

The genuinely important gotcha here: loading a `state_dict` requires the model class/architecture to already exist in code and match exactly (same layer names, same shapes) — the file only has numbers, not the blueprint. This is different from saving the entire model object:

```python
# Saving the whole model (architecture + weights) — works, but less recommended
torch.save(model, 'full_model.pt')
model = torch.load('full_model.pt')   # no need to redefine the class... in theory
```

Official PyTorch recommendation is genuinely to prefer `state_dict` over saving the whole object — saving the whole object ties the file to the exact class definition and Python environment at save time, which can break if the code changes later or moves to a different project structure. `state_dict` is more portable and is the pattern used almost everywhere in real projects.

## Saving full training checkpoints — not just the model

For resuming interrupted training (not just doing inference later), save the optimizer state and epoch number too:

```python
checkpoint = {
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss.item(),
}
torch.save(checkpoint, 'checkpoint.pt')

# Resuming later
checkpoint = torch.load('checkpoint.pt')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
start_epoch = checkpoint['epoch'] + 1
```

Saving the optimizer state matters genuinely more than it might seem — Adam (file 07) keeps per-parameter momentum/adaptive learning rate history internally. Resuming training with just the model weights but a freshly-initialized optimizer loses all that accumulated history, and training can visibly stumble for a few epochs while the optimizer "re-learns" it.

## Loading onto a different device than it was saved from

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model.load_state_dict(torch.load('model_weights.pt', map_location=device))
```

`map_location` matters genuinely when a model was trained on GPU but is being loaded on a CPU-only machine (or vice versa) — without it, loading a GPU-saved checkpoint on a CPU-only machine throws an error trying to find a CUDA device that doesn't exist there.

## Saving the best model during training — completing file 09's early stopping example

```python
best_val_loss = float('inf')

for epoch in range(num_epochs):
    # ... training and validation steps ...

    if val_loss < best_val_loss:
        best_val_loss = val_loss
        torch.save(model.state_dict(), 'best_model.pt')
        print(f"Saved new best model at epoch {epoch+1}, val_loss={val_loss:.4f}")

# after training completes, load the best version for actual use, not just whatever the last epoch produced
model.load_state_dict(torch.load('best_model.pt'))
model.eval()
```

Genuinely important habit — the LAST epoch's weights are not necessarily the BEST epoch's weights, especially once overfitting starts (recap file 09's train/val divergence pattern). Always evaluate/deploy the checkpoint with the best validation performance, not just whatever happened to finish last.

## Exporting just for inference — a lighter-weight option

```python
model.eval()
example_input = torch.rand(1, 10)
traced_model = torch.jit.trace(model, example_input)
traced_model.save('model_traced.pt')
```

`torch.jit.trace` converts the model into TorchScript — a serialized, self-contained format that doesn't need the original Python class definition to load, and can even run outside Python entirely (C++ runtime, mobile). Genuinely more relevant for deployment (file 18 covers this properly) but flagging it here since it's conceptually still "saving a model," just for a different purpose than resuming training.

## Try this

```python
# Train the MLP from file 07 for a few epochs, save a full checkpoint (model + optimizer + epoch)
# Then write code that loads the checkpoint and resumes training from that exact epoch
# Verify the loss picks up smoothly rather than spiking back up
```

Actually watching training resume smoothly (loss continuing to decrease from where it left off, not jumping back up) is the real confirmation that the optimizer state was restored correctly, not just the model weights.

## What's next
File 15 — Transformers & Attention, the architecture that's replaced RNNs for most modern NLP and is the direct bridge into Hugging Face.
