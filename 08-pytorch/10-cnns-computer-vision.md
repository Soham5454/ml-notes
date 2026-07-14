# 10 — Convolutional Neural Networks (CNNs)

Recap from the XGBoost bridge file: this is the "unstructured data, learned features" case I flagged there. Images don't come as clean pre-defined columns like tabular data — a CNN's whole job is to learn useful features (edges, textures, shapes) directly from raw pixels.

## Why not just flatten an image and feed it into an MLP?

A 224x224 RGB image has 224*224*3 = 150,528 values. Feeding that flat into `nn.Linear` would need a weight for every single pixel independently — ignores the fact that nearby pixels are spatially related, and the parameter count would be enormous. A CNN instead slides small filters across the image, reusing the same small set of weights at every position — this is called **parameter sharing**, and it's the core reason CNNs work well on images while being far smaller than an equivalent MLP would need to be.

## The convolution operation itself

```python
import torch
import torch.nn as nn

conv = nn.Conv2d(in_channels=3, out_channels=16, kernel_size=3, stride=1, padding=1)

x = torch.rand(8, 3, 32, 32)   # batch of 8, RGB (3 channels), 32x32 images
out = conv(x)
print(out.shape)   # torch.Size([8, 16, 32, 32])
```

Breaking down what each argument means, genuinely worth internalizing:
- `in_channels=3` — RGB has 3 channels. Grayscale would be 1.
- `out_channels=16` — how many different filters/feature detectors this layer learns. Each one might learn to detect something different (an edge, a color blob, a texture) — this is learned automatically during training, never hand-specified.
- `kernel_size=3` — each filter is a 3x3 sliding window.
- `padding=1` — pads the image border with zeros so the output spatial size matches the input (without padding, convolution shrinks the image slightly each layer).

## Pooling — downsampling to reduce spatial size

```python
pool = nn.MaxPool2d(kernel_size=2, stride=2)
x = torch.rand(8, 16, 32, 32)
out = pool(x)
print(out.shape)   # torch.Size([8, 16, 16, 16]) — spatial dims halved
```

`MaxPool2d` takes the maximum value in each 2x2 window, halving height and width. Purpose: reduce computation for later layers, and add a bit of translation invariance (small shifts in the input don't change the pooled output much).

## A full, simple CNN — the pattern to actually memorize

```python
class SimpleCNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.conv1 = nn.Conv2d(3, 16, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(16, 32, kernel_size=3, padding=1)
        self.pool = nn.MaxPool2d(2, 2)
        self.relu = nn.ReLU()
        # after 2 pooling operations, a 32x32 image becomes 8x8
        self.fc1 = nn.Linear(32 * 8 * 8, 128)
        self.fc2 = nn.Linear(128, num_classes)

    def forward(self, x):
        x = self.pool(self.relu(self.conv1(x)))   # 32x32 -> 16x16
        x = self.pool(self.relu(self.conv2(x)))   # 16x16 -> 8x8
        x = x.view(x.size(0), -1)                  # flatten, keeping batch dim
        x = self.relu(self.fc1(x))
        x = self.fc2(x)                             # raw logits, feeds CrossEntropyLoss
        return x
```

Genuinely the standard shape of every CNN, from this toy example up to ResNet: **stack of (conv → activation → pool) blocks, then flatten, then a small MLP head for the final classification.** The conv blocks learn spatial features; the MLP head at the end just does the final decision-making, same as any classifier from file 04.

`x.view(x.size(0), -1)` is the exact `unsqueeze`/reshape concept flagged back in file 02 — flattening the spatial+channel dimensions into one long vector per sample while keeping the batch dimension intact, since the FC layers expect 2D input `[batch, features]`.

## Full training loop on MNIST — tying files 06, 07, 08, 10 together

```python
from torchvision import datasets, transforms
from torch.utils.data import DataLoader
import torch.optim as optim

transform = transforms.Compose([transforms.ToTensor()])
train_data = datasets.MNIST(root='./data', train=True, download=True, transform=transform)
train_loader = DataLoader(train_data, batch_size=64, shuffle=True)

model = SimpleCNN(num_classes=10)   # note: MNIST is 1-channel, adjust conv1 in_channels=1 for real use
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

for epoch in range(5):
    for images, labels in train_loader:
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
    print(f"Epoch {epoch+1} done, last batch loss: {loss.item():.4f}")
```

This is genuinely the entire roadmap so far in one runnable block — Dataset/DataLoader (file 08) feeding batches into a CNN (file 10) whose output goes through CrossEntropyLoss (file 06) and gets optimized via the standard five-step loop (file 07).

## Feature importance / interpretability — the honest gap flagged in the XGBoost bridge

Recap from file 11 in the XGBoost folder: there's no clean equivalent of "which feature did this split use" for a CNN. What exists instead:
- **Grad-CAM** — highlights which regions of an input image most influenced the prediction, using gradients flowing back to the last conv layer.
- **Filter visualization** — literally visualizing what pattern each learned filter responds to.

Not implementing these in code here since it's a genuinely deep sub-topic on its own, but noting it exists so I know what to look up specifically when a real project needs image-model interpretability.

## Try this

```python
# Modify SimpleCNN to work on MNIST directly (in_channels=1 instead of 3, adjust image size math for 28x28 input)
# Train for 3 epochs, then manually predict on 5 test images and compare against true labels
```

Seeing the model correctly guess handwritten digits it's never seen before is genuinely the first "this feels like real AI" moment in the whole roadmap — worth actually running this rather than just reading it.

## What's next (batch 2, after these get pushed to GitHub)
File 11 — RNNs/LSTMs for sequence data, then transfer learning, GPU/mixed precision, saving/loading, transformers, autoencoders, GANs, deployment, and the final PyTorch→Hugging Face bridge file.
