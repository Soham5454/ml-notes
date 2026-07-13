# 09 — Hyperparameter Tuning

Recap from file 01: hyperparameters are set BEFORE training (like `n_neighbors` for KNN, `max_depth` for a decision tree, `alpha` for Ridge) — they can't be "learned" through normal `.fit()`. This file covers systematically SEARCHING for good hyperparameter values instead of guessing.

## Why manual guessing doesn't scale
With one or two hyperparameters, manually trying a few values and comparing scores is fine. But most real models have MULTIPLE hyperparameters that interact with each other (e.g. a Random Forest's `n_estimators`, `max_depth`, and `min_samples_split` all affect each other's ideal values) — manually trying every combination becomes impractical fast.

## GridSearchCV — exhaustively trying every combination
```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

param_grid = {
    "n_estimators": [50, 100, 200],
    "max_depth": [5, 10, 15, None],
    "min_samples_split": [2, 5, 10]
}

grid_search = GridSearchCV(RandomForestClassifier(), param_grid, cv=5, scoring="accuracy")
grid_search.fit(X_train, y_train)

print(grid_search.best_params_)     # the best combination found
print(grid_search.best_score_)         # the cross-validated score for that best combination
```
This tries EVERY SINGLE combination of the given parameter values (3 × 4 × 3 = 36 combinations here), running full cross-validation (recap from file 03) on each one. Genuinely thorough, but the number of combinations grows multiplicatively — adding more hyperparameters or more values per hyperparameter can make this extremely slow, since it's literally training `(combinations × cv folds)` separate models.

## Using the best model directly
```python
best_model = grid_search.best_estimator_     # the actual trained model with the best hyperparameters, ready to use
predictions = best_model.predict(X_test)
```

## RandomizedSearchCV — sampling randomly instead of trying everything
```python
from sklearn.model_selection import RandomizedSearchCV

param_distributions = {
    "n_estimators": [50, 100, 150, 200, 250, 300],
    "max_depth": [5, 10, 15, 20, None],
    "min_samples_split": [2, 5, 10, 15]
}

random_search = RandomizedSearchCV(RandomForestClassifier(), param_distributions, n_iter=20, cv=5, random_state=42)
random_search.fit(X_train, y_train)
```
Instead of trying EVERY combination, this randomly samples `n_iter` combinations to try — genuinely more practical when the search space is large, since it can often find a near-optimal combination much faster than exhaustively checking everything. There's a real, somewhat counterintuitive research finding behind this: random sampling often finds nearly-as-good hyperparameters as full grid search, in a fraction of the computation time, especially when only a few hyperparameters actually matter much.

## When to use GridSearch vs RandomizedSearch
- **GridSearch** — when the search space is genuinely small (few hyperparameters, few values each), and you want the guaranteed best combination within that space
- **RandomizedSearch** — when the search space is large, or when compute time/budget is a real constraint (genuinely relevant given my RTX 3050's limitations for anything more compute-heavy down the line)

## Nested cross-validation — a subtlety worth knowing about (avoiding a genuine mistake)
If you use the SAME cross-validation to both tune hyperparameters AND report final performance, you can get an overly optimistic estimate, since you've effectively "peeked" at validation performance while choosing hyperparameters. The more rigorous approach:
```python
# Outer loop: honest performance estimate
# Inner loop: hyperparameter tuning
from sklearn.model_selection import cross_val_score

grid_search = GridSearchCV(RandomForestClassifier(), param_grid, cv=5)
nested_scores = cross_val_score(grid_search, X, y, cv=5)     # outer cv wraps the ENTIRE tuning process
```
Honestly, for most practical/smaller projects, a simpler train/validation/test split (recap file 03) with tuning done only against the validation set, and a FINAL untouched test set for reporting, is a more common real-world compromise than full nested cross-validation — nested CV is more rigorous but also genuinely expensive to compute.

## Visualizing hyperparameter tuning results (connects to matplotlib/Seaborn skills)
```python
results_df = pd.DataFrame(grid_search.cv_results_)
sns.lineplot(data=results_df, x="param_max_depth", y="mean_test_score", hue="param_n_estimators")
```
`grid_search.cv_results_` contains the FULL results for every combination tried, not just the best one — genuinely useful for visualizing how performance changes as each hyperparameter varies, which can reveal patterns (like "performance plateaus after n_estimators=100, no need to go higher") that the single best-combination number alone wouldn't show.

## A realistic tuning workflow
```python
# 1. Get a baseline with default hyperparameters first
baseline_model = RandomForestClassifier(random_state=42)
baseline_score = cross_val_score(baseline_model, X_train, y_train, cv=5).mean()

# 2. Do a broad RandomizedSearch to find a promising region
random_search = RandomizedSearchCV(RandomForestClassifier(), param_distributions, n_iter=20, cv=5, random_state=42)
random_search.fit(X_train, y_train)

# 3. Optionally, narrow down with a focused GridSearch around the best region found
# 4. Evaluate the FINAL tuned model on the held-out test set exactly ONCE
final_score = random_search.best_estimator_.score(X_test, y_test)
```
This "baseline → broad random search → focused grid search → final honest test evaluation" progression is genuinely the realistic path I plan to follow, rather than jumping straight to an expensive exhaustive grid search on a huge parameter space from the start.

## Connection forward to Deep Learning
This exact hyperparameter tuning problem reappears in DL, just with a different (often larger) set of hyperparameters — learning rate, batch size, number of layers, number of neurons per layer, dropout rate. Tools like Optuna or Keras Tuner exist specifically for DL hyperparameter search, using similar underlying ideas (random/grid search, plus smarter Bayesian optimization approaches) to what's covered here.
