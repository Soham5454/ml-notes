# 08 — Model Evaluation & Metrics

`.score()` alone genuinely isn't enough to properly evaluate a model — this file covers the actual metrics that matter, and more importantly, WHEN each one is appropriate (a genuinely common source of mistakes if picked carelessly).

## Classification Metrics

### Accuracy — the simplest, but genuinely often misleading
```python
from sklearn.metrics import accuracy_score

accuracy = accuracy_score(y_test, predictions)     # (correct predictions) / (total predictions)
```
The problem with accuracy: on an IMBALANCED dataset (recap from the Seaborn EDA class imbalance notes), a model that just predicts the majority class every single time can still score high accuracy while being completely useless. If 95% of emails aren't spam, a model that predicts "not spam" always gets 95% accuracy without learning anything.

### Confusion Matrix — the foundation everything else builds on
```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, predictions)
#              Predicted No    Predicted Yes
# Actual No         TN               FP
# Actual Yes        FN               TP
```
- **True Positive (TP)** — correctly predicted positive
- **True Negative (TN)** — correctly predicted negative
- **False Positive (FP)** — incorrectly predicted positive (a "false alarm")
- **False Negative (FN)** — incorrectly predicted negative (a "missed case")

Visualizing this connects directly to the Seaborn heatmap confusion matrix pattern from the Seaborn ML/DL file — same matrix, now actually computed from a real trained model instead of example data.

### Precision — "of everything I predicted positive, how much was actually positive?"
```python
from sklearn.metrics import precision_score
precision = precision_score(y_test, predictions)     # TP / (TP + FP)
```
Matters most when FALSE POSITIVES are costly — e.g. a spam filter that wrongly flags an important email as spam (a false positive) is a real problem, so you'd want HIGH precision for that use case.

### Recall — "of everything that was ACTUALLY positive, how much did I correctly catch?"
```python
from sklearn.metrics import recall_score
recall = recall_score(y_test, predictions)     # TP / (TP + FN)
```
Matters most when FALSE NEGATIVES are costly — e.g. a disease detection model that misses an actual sick patient (a false negative) is a serious problem, so you'd want HIGH recall for that use case, even at the cost of more false alarms.

### The precision-recall tradeoff (a genuinely important real tension)
Improving precision often hurts recall and vice versa — being MORE cautious about predicting "positive" (raising precision) naturally means missing more genuine positive cases (lowering recall). There's rarely a way to maximize both simultaneously; the right balance depends entirely on which type of mistake (false positive vs false negative) is more costly for the specific real-world problem.

### F1 Score — the harmonic mean of precision and recall
```python
from sklearn.metrics import f1_score
f1 = f1_score(y_test, predictions)
```
A single number balancing both precision and recall — genuinely useful when you want ONE metric to optimize/compare models by, rather than juggling two separate numbers. Using the HARMONIC mean (not a simple average) specifically punishes cases where one of the two is very low, even if the other is very high.

### classification_report — getting everything at once
```python
from sklearn.metrics import classification_report
print(classification_report(y_test, predictions))
```
Gives precision, recall, F1, and support (sample count) for EVERY class at once — genuinely the first thing I'd run for a full classification evaluation instead of computing each metric separately.

### ROC Curve and AUC
```python
from sklearn.metrics import roc_curve, roc_auc_score

probabilities = model.predict_proba(X_test)[:, 1]     # probability of the POSITIVE class
fpr, tpr, thresholds = roc_curve(y_test, probabilities)
auc = roc_auc_score(y_test, probabilities)

plt.plot(fpr, tpr, label=f"AUC = {auc:.2f}")
plt.plot([0, 1], [0, 1], "k--")     # diagonal reference line = random guessing
plt.xlabel("False Positive Rate")
plt.ylabel("True Positive Rate")
plt.legend()
plt.show()
```
ROC curve shows the tradeoff between true positive rate and false positive rate ACROSS every possible classification threshold (not just the default 0.5) — AUC (Area Under the Curve) condenses this into one number, where 1.0 is a perfect classifier and 0.5 is equivalent to random guessing. Genuinely useful for comparing models independent of any specific threshold choice.

## Regression Metrics

### Mean Absolute Error (MAE)
```python
from sklearn.metrics import mean_absolute_error
mae = mean_absolute_error(y_test, predictions)     # average absolute difference between predicted and actual
```
Genuinely the most intuitive regression metric — directly interpretable in the same units as the target (e.g. "on average, predictions are off by $500").

### Mean Squared Error (MSE) and Root Mean Squared Error (RMSE)
```python
from sklearn.metrics import mean_squared_error
import numpy as np

mse = mean_squared_error(y_test, predictions)
rmse = np.sqrt(mse)     # RMSE brings it back to the same units as the target, since MSE squares the units
```
Squaring the errors before averaging means MSE (and RMSE) penalizes LARGE errors disproportionately more than small ones — a single big miss hurts MSE more than several small misses would. Whether that's desirable depends on the problem: if occasional large errors are genuinely worse (not just proportionally worse) than many small ones, MSE/RMSE reflects that better than MAE does.

### R² Score (coefficient of determination)
```python
from sklearn.metrics import r2_score
r2 = r2_score(y_test, predictions)     # also directly available via model.score() for regression models
```
Represents the proportion of variance in the target that the model successfully explains — ranges up to 1.0 (perfect prediction), with 0 meaning the model performs no better than just always predicting the average value. Can even go NEGATIVE if the model performs WORSE than that naive average-baseline.

## Choosing the right metric — a genuinely important judgment call
There's no single "best" metric — the right choice depends entirely on the actual problem:
- Imbalanced classification → precision/recall/F1, NOT plain accuracy
- Cost of false positives vs false negatives differs → weight precision vs recall accordingly
- Regression with outliers that matter a lot → RMSE (penalizes big misses more)
- Regression where all errors matter roughly equally → MAE (more robust to outliers, more directly interpretable)
- Comparing classifiers across all possible thresholds → ROC-AUC

## Cross-validated metrics (recap from file 03, applying real metrics instead of just default score)
```python
from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X, y, cv=5, scoring="f1")     # explicitly specify which metric to use in cross-validation
```
