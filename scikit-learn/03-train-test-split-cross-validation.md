# 03 — Train-Test Split & Cross-Validation

## Why splitting data matters — the core problem this solves
If you train a model AND evaluate it on the exact same data, you're not measuring how well it actually learned — you're measuring how well it MEMORIZED. A model can get 100% accuracy on training data while being completely useless on new, unseen data. Splitting data lets you honestly evaluate generalization.

## train_test_split — the basic tool
```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```
- `test_size=0.2` — 20% of data reserved for testing, 80% for training (80/20 is a common default split, though 70/30 also shows up often)
- `random_state=42` — recap from the NumPy random module notes, this ensures the SAME split every time the code runs, for reproducibility

## Stratified splitting — preserving class proportions (important for classification)
```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
```
Without `stratify=y`, a random split on an imbalanced dataset (recap from the Seaborn EDA notes on class imbalance) could accidentally put almost ALL of one rare class into the test set, leaving barely any in training — `stratify=y` ensures both train and test sets maintain the SAME class proportions as the original data. I consider this close to a "should basically always use it for classification" default now.

## Three-way split — train, validation, and test
```python
X_train, X_temp, y_train, y_temp = train_test_split(X, y, test_size=0.3, random_state=42)
X_val, X_test, y_val, y_test = train_test_split(X_temp, y_temp, test_size=0.5, random_state=42)
# Result: 70% train, 15% validation, 15% test
```
Why THREE sets instead of two: the **validation set** is used during development to tune hyperparameters and compare models, while the **test set** stays completely untouched until the very end, giving one final, honest, unbiased performance estimate. If you tune hyperparameters directly against the test set, you're subtly "leaking" test information into your model choices — the validation set exists specifically to prevent that. This exact train/val/test structure is also standard in DL, where the validation set is checked every epoch (recap from the matplotlib loss curve notes).

## Cross-validation — a more robust evaluation than a single split
A single train-test split gives ONE estimate of performance, which can vary depending on which specific rows happened to land in the test set (especially with smaller datasets). Cross-validation runs multiple splits and averages the results for a more reliable estimate.

### K-Fold Cross-Validation
```python
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
scores = cross_val_score(model, X, y, cv=5)     # cv=5 -> 5-fold cross-validation
print(scores)          # array of 5 scores, one per fold
print(scores.mean())      # average performance across all folds -- more reliable than a single split
print(scores.std())          # how much performance VARIES across folds -- a large std suggests the model's performance is unstable/sensitive to which data it sees
```

### How K-Fold actually works (the mechanism)
The data gets split into K equal parts ("folds"). The model trains on K-1 folds and tests on the remaining 1 fold, and this repeats K times, with a DIFFERENT fold used as the test set each time. Every single data point ends up being used for testing EXACTLY once across the K rounds — the model just isn't tested on the same fold it trained on.

### Stratified K-Fold (combining stratification with cross-validation)
```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=skf)     # each fold maintains class proportions, same reasoning as stratified train_test_split
```
Genuinely my default now for classification cross-validation — combines the reliability of cross-validation with the fairness of maintaining class balance in every fold.

## cross_validate — getting more than just a score
```python
from sklearn.model_selection import cross_validate

results = cross_validate(model, X, y, cv=5, scoring=["accuracy", "precision", "recall"], return_train_score=True)
```
`cross_val_score` only gives you ONE metric; `cross_validate` gives multiple metrics AND can return training scores alongside validation scores, which is genuinely useful for spotting overfitting (recap from the matplotlib DL notes) — if training scores are much higher than validation scores across all folds, that's the same divergence pattern, just seen through cross-validation instead of a loss curve.

## Leave-One-Out Cross-Validation (an extreme version, rarely practical but worth knowing)
```python
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()
scores = cross_val_score(model, X, y, cv=loo)     # trains on ALL but one sample, tests on that one, repeats for EVERY sample
```
Genuinely thorough but computationally expensive — with N samples, you train N separate models. Only practical for genuinely small datasets.

## A realistic honest workflow combining all of this
```python
# 1. Split off a final test set that stays untouched until the very end
X_train_full, X_test, y_train_full, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

# 2. Use cross-validation on the remaining data to compare models / tune hyperparameters
scores = cross_val_score(model, X_train_full, y_train_full, cv=5)

# 3. Once satisfied, train the FINAL model on ALL of X_train_full, then evaluate ONCE on X_test
model.fit(X_train_full, y_train_full)
final_score = model.score(X_test, y_test)
```
This is genuinely the realistic version of "the workflow" — cross-validation for the iterative development/comparison phase, a single held-out test set for the final honest report at the end.
