# 04 — Classification & Regression

## XGBClassifier — full binary classification example
```python
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)     # recap stratify from scikit-learn train-test split notes -- genuinely important here too

model = XGBClassifier(n_estimators=100, max_depth=4, learning_rate=0.1, random_state=42)
model.fit(X_train, y_train)

predictions = model.predict(X_test)
print(classification_report(y_test, predictions))     # recap from scikit-learn evaluation notes -- works identically here
```

## Multi-class classification
```python
model = XGBClassifier(objective="multi:softprob", num_class=3)     # num_class needed when objective is explicitly multi-class
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```
Genuinely, the scikit-learn wrapper often auto-detects this from `y_train` having more than 2 unique values, so explicitly setting `objective`/`num_class` isn't always strictly necessary — but knowing it exists matters for understanding what's happening, and for the native API where auto-detection doesn't apply.

## Getting prediction probabilities (recap from scikit-learn LogisticRegression notes, same pattern)
```python
probabilities = model.predict_proba(X_test)
# array([[0.1, 0.9], [0.7, 0.3], ...])   -- same shape/meaning as scikit-learn's predict_proba
```

## XGBRegressor — full regression example
```python
from xgboost import XGBRegressor
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

model = XGBRegressor(n_estimators=200, max_depth=4, learning_rate=0.05, random_state=42)
model.fit(X_train, y_train)

predictions = model.predict(X_test)
rmse = np.sqrt(mean_squared_error(y_test, predictions))     # recap from scikit-learn regression metrics notes
r2 = r2_score(y_test, predictions)
```

## Different regression objectives
```python
XGBRegressor(objective="reg:squarederror")     # standard regression, default
XGBRegressor(objective="reg:squaredlogerror")     # useful when the target spans a wide range and relative error matters more than absolute error
XGBRegressor(objective="reg:absoluteerror")          # optimizes MAE directly instead of MSE -- more robust to outliers, recap the MAE vs MSE tradeoff from scikit-learn metrics notes
```

## Ranking objective (a genuinely different, less common use case worth knowing exists)
```python
XGBClassifier(objective="rank:pairwise")     # for LEARNING TO RANK problems -- e.g. search result ranking, recommendation ordering, not simple classification/regression
```
Haven't used this myself, but noting it exists since XGBoost genuinely supports this whole separate category of problem beyond classification/regression — ranking where the goal is relative ORDER of items, not an absolute label/value per item.

## Visualizing a single tree from the ensemble (recap connects to scikit-learn's plot_tree from the classification algorithms notes)
```python
from xgboost import plot_tree
import matplotlib.pyplot as plt

plt.figure(figsize=(20, 10))
plot_tree(model, num_trees=0)     # visualize the FIRST tree specifically (tree index 0) out of the whole ensemble
plt.show()
```
Useful for understanding what an individual tree in the boosting sequence actually learned, though with `n_estimators=100+`, looking at every single tree individually isn't practical — this is more of a spot-check/understanding tool than a full model interpretation approach (feature importance, file 07, is more practical for that).

## A realistic full workflow — combining everything so far
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split, cross_val_score

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", XGBClassifier(n_estimators=100, max_depth=4, learning_rate=0.1, random_state=42))
])

cv_scores = cross_val_score(pipeline, X_train, y_train, cv=5)     # recap cross-validation from scikit-learn file 03
print(f"CV accuracy: {cv_scores.mean():.3f} +/- {cv_scores.std():.3f}")

pipeline.fit(X_train, y_train)
final_score = pipeline.score(X_test, y_test)
```
This is genuinely the realistic shape of an actual XGBoost project — everything from the scikit-learn workflow (Pipeline, cross-validation, train/test split) applies directly, with XGBClassifier just slotting in as the model step.

## Comparing XGBoost against the baseline escalation path (recap from scikit-learn ensemble notes)
```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from xgboost import XGBClassifier

models = {
    "Logistic Regression": LogisticRegression(),
    "Random Forest": RandomForestClassifier(random_state=42),
    "XGBoost": XGBClassifier(random_state=42)
}

for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5)
    print(f"{name}: {scores.mean():.3f}")
```
This exact loop-based comparison (recap from scikit-learn classification algorithms notes) is genuinely how I'd validate whether XGBoost's added complexity is actually earning its keep on a specific real dataset, rather than assuming it's automatically the best choice.
