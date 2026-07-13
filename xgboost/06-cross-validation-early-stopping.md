# 06 — Cross-Validation & Early Stopping

## Early Stopping — genuinely one of XGBoost's most practical features
Instead of guessing the "right" `n_estimators` upfront and hoping it's neither too few (underfit) nor too many (overfit), early stopping lets you set a genuinely large `n_estimators` and have training STOP automatically once validation performance stops improving.

```python
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split

X_train, X_val, y_train, y_val = train_test_split(X_train_full, y_train_full, test_size=0.2, random_state=42)     # need a validation set specifically for this, recap the three-way split from scikit-learn file 03

model = XGBClassifier(n_estimators=1000, learning_rate=0.05, random_state=42)     # deliberately set HIGH, early stopping will cut it short
model.fit(
    X_train, y_train,
    eval_set=[(X_train, y_train), (X_val, y_val)],
    early_stopping_rounds=10,     # stop if validation performance doesn't improve for 10 consecutive rounds
    verbose=False
)

print(model.best_iteration)     # the actual number of trees that ended up being used
```

## How early stopping actually decides when to stop
It watches the validation metric (recap `eval_metric` from file 03) after each new tree is added. If the validation score hasn't IMPROVED for `early_stopping_rounds` consecutive rounds, training stops, and the model reverts to using only the trees up through the BEST round (not literally the last round trained) — genuinely a built-in safeguard against the exact overfitting pattern from file 05.

## Why this matters more than manually tuning n_estimators
Manually searching for the "right" n_estimators via GridSearchCV (recap from scikit-learn file 09) works but is genuinely wasteful — it means training separate full models for EVERY candidate n_estimators value just to compare. Early stopping achieves a similar effect within a SINGLE training run, since it naturally finds a good stopping point as part of one fit() call.

## Getting predictions using only the best iteration
```python
predictions = model.predict(X_test, iteration_range=(0, model.best_iteration + 1))     # ensures predictions use only the trees up to the best-found point, not any extra ones trained AFTER early stopping triggered
```
Genuinely, the scikit-learn wrapper handles this automatically in most cases when early stopping is used, but worth knowing this explicit control exists, especially when working with the native API.

## Cross-Validation with XGBoost — the built-in xgb.cv function (native API)
```python
import xgboost as xgb

dtrain = xgb.DMatrix(X_train, label=y_train)     # recap DMatrix from file 01

params = {"max_depth": 4, "learning_rate": 0.1, "objective": "binary:logistic"}

cv_results = xgb.cv(
    params, dtrain,
    num_boost_round=500,
    nfold=5,                        # recap K-Fold concept from scikit-learn cross-validation notes
    early_stopping_rounds=10,
    metrics="logloss",
    seed=42
)
print(cv_results.tail())     # shows train/test metric history across boosting rounds, PER FOLD averaged
```
This is genuinely a nice built-in combination of cross-validation AND early stopping in one call, specific to XGBoost's native API — finds a good `n_estimators` value that's validated across multiple folds, not just a single train/validation split.

## Using scikit-learn's cross_val_score with XGBoost (recap the pattern from scikit-learn file 03)
```python
from sklearn.model_selection import cross_val_score

model = XGBClassifier(n_estimators=100, max_depth=4, learning_rate=0.1, random_state=42)
scores = cross_val_score(model, X_train, y_train, cv=5, scoring="accuracy")
print(scores.mean(), scores.std())
```
Genuinely works identically to how it worked for any other scikit-learn-compatible model in the earlier notes — no special handling needed since XGBClassifier follows the same estimator interface.

## Stratified K-Fold with XGBoost (recap from scikit-learn file 03, genuinely important for imbalanced classification, connects to file 09)
```python
from sklearn.model_selection import StratifiedKFold, cross_val_score

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X_train, y_train, cv=skf)
```

## Combining early stopping WITHIN a cross-validation loop (a genuinely more advanced, careful pattern)
```python
from sklearn.model_selection import KFold
import numpy as np

kf = KFold(n_splits=5, shuffle=True, random_state=42)
scores = []

for train_idx, val_idx in kf.split(X_train_full):
    X_tr, X_val = X_train_full.iloc[train_idx], X_train_full.iloc[val_idx]
    y_tr, y_val = y_train_full.iloc[train_idx], y_train_full.iloc[val_idx]

    model = XGBClassifier(n_estimators=1000, learning_rate=0.05, random_state=42)
    model.fit(X_tr, y_tr, eval_set=[(X_val, y_val)], early_stopping_rounds=10, verbose=False)
    scores.append(model.score(X_val, y_val))

print(np.mean(scores), np.std(scores))
```
This manual loop is genuinely more code than the simple `cross_val_score` version, but it's necessary specifically when combining early stopping WITH cross-validation properly, since `cross_val_score` alone doesn't have a clean built-in way to pass a per-fold validation set for early stopping — a real practical detail worth remembering rather than assuming the simple approach handles everything.

## Practical honest takeaway
Early stopping is genuinely close to a "should almost always use it" default for real XGBoost projects — it's a cleaner, more automatic, less wasteful way to find a good `n_estimators` than a manual grid search, and directly guards against the overfitting risk that boosting is genuinely prone to if left unchecked.
