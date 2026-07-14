# 03 — Core Parameters

XGBoost has genuinely a LOT of parameters — this file covers the ones that actually matter most day-to-day, grouped by what they control.

## General Parameters

### objective — what kind of problem is being solved
```python
XGBClassifier(objective="binary:logistic")        # binary classification, outputs probability
XGBClassifier(objective="multi:softmax")             # multi-class classification, outputs class label directly
XGBClassifier(objective="multi:softprob")               # multi-class classification, outputs probabilities per class
XGBRegressor(objective="reg:squarederror")                 # standard regression, minimizing squared error
```
Genuinely the scikit-learn wrapper (`XGBClassifier`/`XGBRegressor`) usually picks a sensible default objective automatically based on the target data it sees, but knowing this exists matters for understanding what's happening under the hood, and is essential when using the native API directly (file 01).

### booster — the type of model used at each boosting round
```python
XGBClassifier(booster="gbtree")     # tree-based boosting (the default, and what's covered throughout these notes)
XGBClassifier(booster="gblinear")      # linear boosting instead of trees -- rarely used, genuinely niche
XGBClassifier(booster="dart")             # Dropout Additive Regression Trees -- adds dropout (recap concept from the scikit-learn to DL bridge notes) to gradient boosting, can help reduce overfitting further
```
`gbtree` is genuinely the default and what I'd use for the vast majority of problems — `dart` is worth knowing exists specifically because of that dropout connection to neural networks, but adds training complexity that's not always worth it.

## Tree-Specific Parameters (controlling how each individual tree is built)

### max_depth — how deep each tree can grow
```python
XGBClassifier(max_depth=6)     # default is 6 -- genuinely shallower than typical Random Forest trees (recap from scikit-learn notes, where Random Forest trees are often allowed to grow much deeper)
```
Recap the reasoning from the scikit-learn ensemble notes: boosting relies on MANY weak learners combined, not individually strong ones — shallow trees (3-8 levels deep) are standard for boosting specifically, unlike bagging/Random Forest where deeper individual trees are more common.

### min_child_weight — minimum sum of instance weight needed in a child node to allow a further split
```python
XGBClassifier(min_child_weight=1)     # higher values = more conservative, prevents the model from creating splits based on too little data
```
Conceptually similar to scikit-learn's `min_samples_split`/`min_samples_leaf` from the decision tree notes — a regularization control preventing overly specific, overfit splits.

### gamma — minimum loss reduction required to make a further split
```python
XGBClassifier(gamma=0)     # default 0 -- higher values make the algorithm more conservative, only splitting when it genuinely improves the model meaningfully
```

## Boosting Process Parameters

### n_estimators — number of boosting rounds (number of trees)
```python
XGBClassifier(n_estimators=100)     # same concept as scikit-learn's Random Forest/GradientBoosting n_estimators
```

### learning_rate (also called eta) — how much each tree's correction contributes
```python
XGBClassifier(learning_rate=0.1)     # default 0.3 in XGBoost specifically, though 0.1 or lower is commonly used in practice
```
Recap directly from the scikit-learn gradient boosting notes — smaller learning_rate needs MORE n_estimators to reach similar performance, but often generalizes better. This exact tradeoff genuinely reappears identically in neural network training later.

### subsample — fraction of training data used per tree (row sampling)
```python
XGBClassifier(subsample=0.8)     # each tree trains on a random 80% SAMPLE of the training rows, not all of them
```
Adds randomness similar to bagging's bootstrap sampling (recap from scikit-learn ensemble notes) INTO the boosting process, which can help reduce overfitting further.

### colsample_bytree — fraction of features used per tree (column sampling)
```python
XGBClassifier(colsample_bytree=0.8)     # each tree only considers a random 80% of FEATURES, not all of them
```
Genuinely the same idea as Random Forest's feature-subset-per-split trick (recap from scikit-learn notes) — reduces correlation between individual trees in the ensemble, improving generalization.

## Regularization Parameters (deeper coverage in file 05)
```python
XGBClassifier(reg_alpha=0, reg_lambda=1)     # reg_alpha = L1 penalty, reg_lambda = L2 penalty -- same recap from Ridge/Lasso in scikit-learn linear models notes
```

## A genuinely reasonable set of "starting point" parameters
```python
model = XGBClassifier(
    n_estimators=200,
    max_depth=4,
    learning_rate=0.05,
    subsample=0.8,
    colsample_bytree=0.8,
    random_state=42
)
```
This isn't a magic universal setting — it's a genuinely reasonable STARTING point for hyperparameter tuning (file 08) based on common practice, not something to treat as automatically optimal for every dataset.

## eval_metric — what metric to track during training
```python
model = XGBClassifier(eval_metric="logloss")     # common options: 'logloss', 'error', 'auc' for classification; 'rmse', 'mae' for regression
```
Connects directly to the metrics covered in the scikit-learn model evaluation notes — this just tells XGBoost which metric to actually MONITOR during the boosting process itself (relevant for early stopping, file 06).
