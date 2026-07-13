# 07 — Feature Importance & Interpretation

Recap from scikit-learn's Random Forest notes (file 06 there) — tree-based models give a genuinely useful bonus: a ranking of which features actually mattered for predictions. XGBoost offers this too, but with MORE ways to measure "importance," each answering a slightly different question.

## Basic feature importance (the scikit-learn-compatible way)
```python
importances = model.feature_importances_
feature_importance_df = pd.DataFrame({
    "feature": X.columns,
    "importance": importances
}).sort_values("importance", ascending=False)
```
Genuinely identical syntax to scikit-learn's RandomForestClassifier — same `.feature_importances_` attribute, same DataFrame-and-sort pattern from the scikit-learn feature selection notes.

## Visualizing feature importance (built directly into XGBoost)
```python
from xgboost import plot_importance
import matplotlib.pyplot as plt

plot_importance(model, max_num_features=15)
plt.show()
```
A genuinely convenient built-in plotting shortcut — no need to manually build a bar chart with matplotlib (though you certainly could, using the skills from the matplotlib notes, if more customization was needed).

## The DIFFERENT importance types — this is the part that genuinely matters more with XGBoost than scikit-learn's simpler Random Forest importance
```python
model.get_booster().get_score(importance_type="weight")     # 'weight' -- how many times a feature is used to split across ALL trees
model.get_booster().get_score(importance_type="gain")           # 'gain' -- the average improvement in accuracy brought by splits using that feature
model.get_booster().get_score(importance_type="cover")             # 'cover' -- the average number of samples affected by splits using that feature
```
This is a genuine nuance I didn't expect — "importance" isn't just one single number, it depends on WHAT you're measuring:
- **weight** — a feature used in many splits, even if each split's impact is small, gets a high weight score. Can be misleading if a feature is used often but doesn't actually improve predictions much each time.
- **gain** — genuinely the metric I trust most for understanding TRUE predictive value, since it directly measures how much each split actually improved the model's objective.
- **cover** — useful for understanding how broadly a feature's splits affect the DATASET, rather than how much they improve accuracy.

## Setting which importance type to use for `.feature_importances_` and `plot_importance`
```python
model = XGBClassifier(importance_type="gain")     # sets which type .feature_importances_ will report
plot_importance(model, importance_type="gain")
```
Given the note above, I'd default to `"gain"` for genuinely understanding which features matter most, rather than the default `"weight"`.

## SHAP values — a deeper, more rigorous interpretation method (worth knowing exists, genuinely more principled than basic feature importance)
```python
import shap

explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

shap.summary_plot(shap_values, X_test)     # shows each feature's impact on EVERY individual prediction, not just an aggregate importance score
```
Basic feature importance gives an AGGREGATE, dataset-wide ranking. SHAP values go further — they explain each INDIVIDUAL prediction, showing exactly how much each feature pushed that specific prediction up or down from the baseline. Genuinely more rigorous and increasingly the standard approach for real model interpretability work, though it requires the separate `shap` library (`pip install shap`) and is computationally heavier than basic importance.

## A single-prediction SHAP explanation (the genuinely powerful part)
```python
shap.force_plot(explainer.expected_value, shap_values[0], X_test.iloc[0])     # explains exactly why THIS ONE prediction came out the way it did
```
This is genuinely valuable for real applications where "why did the model predict THIS for THIS specific person/case" matters — like explaining a loan rejection or a medical risk score to someone affected by it, not just understanding the model in the aggregate.

## Practical use of feature importance — informing further feature engineering (recap connects to scikit-learn file 11 and pandas ML/DL notes)
```python
top_features = feature_importance_df.head(10)["feature"].tolist()
X_train_reduced = X_train[top_features]     # retrain using ONLY the most important features -- can simplify the model, speed up training, sometimes even improve generalization by removing noisy low-importance features
```
This connects the whole loop together — feature engineering (pandas) creates candidate features, feature selection (scikit-learn file 11) systematically narrows them, and feature IMPORTANCE (here) provides genuine evidence, AFTER training, for which of those engineering/selection decisions actually paid off.

## Honest caution about feature importance
Feature importance shows CORRELATION with predictive value within THIS specific model, not necessarily true causal importance in the real world — a feature could rank highly just because it happens to correlate with another, truly causal factor that wasn't included in the dataset at all. Genuinely useful for guiding modeling decisions, but not something I'd treat as definitive real-world causal insight without further domain investigation.
