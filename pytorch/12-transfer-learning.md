# 12 — Transfer Learning

Genuinely one of the most practically important techniques in this entire roadmap. Training a strong CNN from scratch needs millions of images and days of GPU time. Transfer learning skips almost all of that.

## The core idea

A model trained on a huge, general dataset (like ImageNet — 1.2 million images, 1000 classes) learns general-purpose visual features in its early layers — edges, textures, shapes, colors. Those early-layer features are genuinely useful for almost ANY image task, not just the original 1000 classes. Transfer learning reuses those learned features and only retrains the final layers for a new, specific task.

Recap link to the XGBoost bridge file's honest framework: this is precisely the case where deep learning's advantage over XGBoost is real — but transfer learning is what makes that advantage accessible without needing ImageNet-scale data or compute of your own.

## Loading a pretrained model

```python
import torch
import torchvision.models as models

model = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)
print(model)   # inspect the architecture — note the final fc layer outputs 1000 classes (ImageNet)
```

`ResNet18` (residual network, 18 layers) is a genuinely solid default choice for transfer learning — good accuracy, not too heavy computationally. Bigger options (ResNet50, EfficientNet, ViT) trade more compute for better accuracy.

## Two approaches: feature extraction vs fine-tuning

**Approach 1 — Feature extraction (freeze everything except the final layer)**

```python
model = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)

for param in model.parameters():
    param.requires_grad = False    # freeze ALL existing weights

num_classes = 10   # my new task, e.g. 10 custom categories
model.fc = torch.nn.Linear(model.fc.in_features, num_classes)   # replace final layer, unfrozen by default

# only model.fc's parameters will actually get gradients/updates now
optimizer = torch.optim.Adam(model.fc.parameters(), lr=0.001)
```

Genuinely the fastest and most data-efficient option — good when the new dataset is small (hundreds to low-thousands of images) and reasonably similar to natural images.

**Approach 2 — Fine-tuning (unfreeze some/all layers, train with a small learning rate)**

```python
model = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)
model.fc = torch.nn.Linear(model.fc.in_features, num_classes)

# unfreeze everything, but use a SMALL learning rate so pretrained weights aren't destroyed
optimizer = torch.optim.Adam(model.parameters(), lr=0.0001)   # 10x smaller than typical from-scratch lr
```

Fine-tuning genuinely needs more data than pure feature extraction (thousands+ images) since more parameters are being updated, but usually reaches higher accuracy if the new task's images differ meaningfully from ImageNet's natural photos (e.g. medical scans, satellite imagery).

## Why the learning rate has to be small during fine-tuning

Recap from file 07 — a learning rate too high destabilizes training. This matters MORE here specifically because the pretrained weights are already good; a large learning rate risks a big destructive update in the first few steps that wipes out useful pretrained features before the model has a chance to adapt gradually. This is genuinely a distinct, important reason beyond the general "too high = unstable" rule from file 07.

## Preprocessing must match what the pretrained model expects

```python
from torchvision import transforms

preprocess = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                          std=[0.229, 0.224, 0.225])   # exact ImageNet stats, must match training
])
```

This is genuinely non-negotiable — if input images aren't normalized with the SAME mean/std the pretrained model originally saw during its own training, the model's learned features get fed distributionally different data than what they were tuned on, and performance quietly degrades without throwing any error. A silent bug worth remembering explicitly.

## A full mini fine-tuning loop

```python
import torch.nn as nn
import torch.optim as optim

model = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)
model.fc = nn.Linear(model.fc.in_features, num_classes=5)

criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.fc.parameters(), lr=0.001)   # feature-extraction mode

for epoch in range(10):
    for images, labels in train_loader:      # from file 08's DataLoader pattern
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
    print(f"Epoch {epoch+1}, loss: {loss.item():.4f}")
```

Same exact five-step training rhythm from file 07 — transfer learning doesn't change the training loop mechanics at all, it only changes which parameters are frozen and what the model starts out already knowing.

## Beyond images — transfer learning in NLP (bridge forward to file 19)

The same principle applies to text: pretrained language models (BERT, GPT-family) learned general language understanding from huge text corpora, and get fine-tuned for specific tasks (sentiment analysis, question answering) the same way ResNet gets fine-tuned for custom image classes. Hugging Face (the final destination of this whole roadmap) is essentially a library built entirely around making this NLP transfer-learning workflow easy — noting this connection now, file 19 covers it properly.

## Try this

```python
# Load resnet18, freeze all layers except the final fc layer
# Replace fc for a 3-class custom problem, and count how many parameters are actually trainable
# vs the total parameter count of the full model (should be a tiny fraction)
```

Seeing just how few parameters actually need training in feature-extraction mode (often <1% of the full model) makes it genuinely obvious why transfer learning is so much faster and more data-efficient than training from scratch.

## What's next
File 13 — GPU/CUDA training and mixed precision, so these bigger pretrained models actually train fast instead of taking forever on CPU.
