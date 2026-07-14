# 10 — ML/DL Use Cases

Tying Seaborn specifically back to the ML/DL roadmap — where these exact plot types show up once actual modeling starts, not just during initial data exploration.

## 1. Feature correlation before modeling (recap from file 06, worth repeating in this context)
```python
corr = df.corr()
sns.heatmap(corr, annot=True, cmap="coolwarm", center=0)
```
Directly informs feature selection — dropping heavily correlated redundant features can help simpler models generalize better and speeds up training, especially relevant given my hardware constraints (RTX 3050, 4GB VRAM) where efficient feature sets matter more than they would on a beefier setup.

## 2. Class imbalance checking (recap from file 09, critical before training any classifier)
```python
sns.countplot(data=df, x="target")
```
A heavily imbalanced target (say 95% one class, 5% another) genuinely changes how a model needs to be trained — plain accuracy becomes a misleading metric, and techniques like class weighting or resampling become necessary. This simple countplot is usually the FIRST place this problem becomes visually obvious.

## 3. Feature distributions by class — spotting genuinely separable features
```python
sns.kdeplot(data=df, x="feature", hue="target", fill=True)
```
If a feature's distribution looks CLEARLY different across target classes, that's a strong candidate for a genuinely useful, informative feature. If the distributions overlap almost entirely, that feature likely won't help the model distinguish between classes much on its own.

## 4. Pairplot for a quick "is this dataset even separable" gut check (recap from file 07)
```python
sns.pairplot(df, hue="target", vars=["feature1", "feature2", "feature3"])
```
Before committing to training any model, this gives an early visual sense of whether the classes are naturally separable using the available features, or genuinely overlapping/tangled — which sets expectations for how hard the actual modeling problem will be.

## 5. Model evaluation — confusion matrix (recap of matplotlib version, much simpler here)
```python
sns.heatmap(confusion_matrix, annot=True, fmt="d", cmap="Blues",
             xticklabels=class_names, yticklabels=class_names)
plt.xlabel("Predicted")
plt.ylabel("Actual")
```
Compare this to the matplotlib version from the matplotlib DL file — same concept, but genuinely less manual code needed here (no manual text-annotation loop, `annot=True` handles it directly).

## 6. Comparing model performance across multiple runs/configs
```python
results_df = pd.DataFrame({
    "model": ["Model A", "Model A", "Model B", "Model B"],
    "run": [1, 2, 1, 2],
    "accuracy": [0.85, 0.87, 0.82, 0.84]
})
sns.barplot(data=results_df, x="model", y="accuracy")     # bars with automatic error bars showing variability across runs
```
Genuinely useful for reporting "Model A vs Model B" comparisons where you've run each multiple times (different random seeds/splits) — the automatic error bar from `barplot` visually communicates whether one model's advantage is real or just noise between runs.

## 7. Residual plots for regression models (recap from file 05, applied post-modeling instead of just during EDA)
```python
residuals = y_actual - y_predicted
sns.scatterplot(x=y_predicted, y=residuals)
plt.axhline(y=0, color="red", linestyle="--")     # reference line at zero
```
After actually training a regression model (not just during EDA), plotting residuals this way checks whether the model's errors are randomly scattered (good) or show a clear pattern (bad — means the model is systematically missing something, like a nonlinear relationship it can't capture).

## 8. Visualizing training metrics over epochs (Seaborn version of the matplotlib DL loss curve pattern)
```python
history_df = pd.DataFrame({
    "epoch": list(range(1, 21)) * 2,
    "loss": train_losses + val_losses,
    "type": ["train"] * 20 + ["val"] * 20
})
sns.lineplot(data=history_df, x="epoch", y="loss", hue="type")
```
Same overfitting-detection concept from the matplotlib ML/DL file (train vs val loss diverging), just using Seaborn's long-form-data + hue approach instead of manually plotting two separate lines — worth knowing both versions since training loop code more commonly outputs data in a format suited to one or the other.

## Honest summary
Seaborn's actual contribution to the ML/DL pipeline is entirely on the EXPLORATION and EVALUATION sides — understanding data before modeling, and understanding results after modeling. It doesn't do any of the actual training itself (that's scikit-learn/PyTorch's job), but nearly every step of a real ML/DL project involves reaching for one of these plot types at some point, either to decide what to do next or to communicate what happened.

## Next in the roadmap
Python Basics → NumPy → pandas → matplotlib → Seaborn (done) → **scikit-learn** (next) → XGBoost → PyTorch → Hugging Face
