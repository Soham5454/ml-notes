# 09 — ML/DL Visualization

This file exists specifically because plotting needs shift once you move from general data viz into actually training models — same matplotlib tools from files 01-08, but applied to a specific recurring set of DL-relevant patterns.

## 1. Training/validation loss curves — the single most common DL plot
```python
epochs = np.arange(1, 21)
train_loss = 1 / epochs + np.random.normal(0, 0.02, 20)
val_loss = 1 / epochs + 0.05 + np.random.normal(0, 0.03, 20)

fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(epochs, train_loss, label="Training Loss", marker="o")
ax.plot(epochs, val_loss, label="Validation Loss", marker="s")
ax.set_xlabel("Epoch")
ax.set_ylabel("Loss")
ax.set_title("Training vs Validation Loss")
ax.legend()
plt.show()
```
This exact plot is how you spot **overfitting** — if training loss keeps dropping but validation loss starts climbing back up, that gap is the model memorizing training data instead of generalizing. I've read about overfitting conceptually before, but seeing it as "the point where these two lines diverge" makes it a concrete, visually recognizable pattern instead of just an abstract definition.

## 2. Accuracy curves (same idea, different metric)
```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(epochs, train_acc, label="Training Accuracy")
ax.plot(epochs, val_acc, label="Validation Accuracy")
ax.set_xlabel("Epoch")
ax.set_ylabel("Accuracy")
ax.legend()
plt.show()
```

## 3. Loss and accuracy side by side (a very standard "training report" layout)
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))

ax1.plot(epochs, train_loss, label="Train")
ax1.plot(epochs, val_loss, label="Val")
ax1.set_title("Loss")
ax1.legend()

ax2.plot(epochs, train_acc, label="Train")
ax2.plot(epochs, val_acc, label="Val")
ax2.set_title("Accuracy")
ax2.legend()

plt.tight_layout()
plt.show()
```
This two-panel layout is genuinely the standard way training results get reported in almost every ML/DL tutorial or paper I've come across — worth having this exact template ready to reuse.

## 4. Confusion matrix — visualizing classification performance
```python
confusion = np.array([[50, 3, 2], [4, 45, 1], [2, 3, 48]])     # example 3-class confusion matrix
labels = ["Class A", "Class B", "Class C"]

fig, ax = plt.subplots()
im = ax.imshow(confusion, cmap="Blues")

ax.set_xticks(range(len(labels)))
ax.set_yticks(range(len(labels)))
ax.set_xticklabels(labels)
ax.set_yticklabels(labels)
ax.set_xlabel("Predicted Label")
ax.set_ylabel("True Label")
ax.set_title("Confusion Matrix")

for i in range(len(labels)):
    for j in range(len(labels)):
        ax.text(j, i, confusion[i, j], ha="center", va="center",
                 color="white" if confusion[i, j] > confusion.max()/2 else "black")

plt.colorbar(im)
plt.show()
```
Directly builds on the heatmap + text-annotation combo from file 07 — the diagonal cells being darker/higher means the model is correctly classifying most examples; scattered high values OFF the diagonal show exactly which classes the model is confusing with each other, which is genuinely more actionable than a single "accuracy" number alone.

## 5. Visualizing sample images from a dataset (a standard first EDA step for image data)
```python
fig, axes = plt.subplots(2, 5, figsize=(12, 5))
for i, ax in enumerate(axes.flat):
    ax.imshow(images[i], cmap="gray")     # cmap='gray' for grayscale images like MNIST digits
    ax.set_title(f"Label: {labels[i]}")
    ax.axis("off")     # hides the x/y tick marks -- irrelevant clutter for an image display
plt.tight_layout()
plt.show()
```
`axes.flat` flattens a 2D grid of axes into a simple 1D iterable — genuinely useful shortcut when looping over a grid without needing to track row/column indices manually. `ax.axis("off")` is something I'll use constantly for image displays specifically, since tick marks/numbers on an image plot are just visual noise.

## 6. Visualizing model predictions vs actual (regression-specific)
```python
fig, ax = plt.subplots()
ax.scatter(y_actual, y_predicted, alpha=0.5)
ax.plot([y_actual.min(), y_actual.max()], [y_actual.min(), y_actual.max()], "r--")     # perfect-prediction reference line
ax.set_xlabel("Actual")
ax.set_ylabel("Predicted")
ax.set_title("Predicted vs Actual")
plt.show()
```
The diagonal dashed line represents "perfect prediction" — points sitting close to that line mean the model's predictions are accurate, points scattered far from it show where the model is struggling.

## 7. Feature/weight visualization (a genuine DL-specific preview)
```python
fig, axes = plt.subplots(4, 4, figsize=(8, 8))
for i, ax in enumerate(axes.flat):
    ax.imshow(filters[i], cmap="gray")     # visualize the actual learned filters/weights of a CNN's first layer
    ax.axis("off")
plt.tight_layout()
plt.show()
```
Haven't built an actual CNN yet, but this exact pattern (a grid of small image plots, no axes ticks) is how you'd inspect what patterns a convolutional layer has learned to detect — noting this here so it's a ready-made template for whenever I reach that point in PyTorch.

## Honest summary
Every single plot in this file uses tools already covered in files 01-08 — line plots, subplots, imshow, text annotation, colormaps. Nothing new to "learn" here conceptually, just recognizing WHICH combination of already-known tools maps to WHICH specific ML/DL reporting need. That recognition is really the actual point of this file.

## Next in the roadmap
Python Basics → NumPy → pandas → matplotlib (done) → **Seaborn** (next — already covered, will write up similarly) → scikit-learn → XGBoost → PyTorch → Hugging Face
