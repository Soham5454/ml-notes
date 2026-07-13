# 09 — Handling Missing Data & Imbalanced Classes

## Missing Data — XGBoost's genuinely unique built-in handling
Recap from scikit-learn's preprocessing notes (file 02 there) — most algorithms REQUIRE missing values to be handled beforehand (via SimpleImputer or similar) before training. XGBoost is genuinely different here.

```python
model = XGBClassifier()
model.fit(X_train_with_nans, y_train)     # XGBoost can handle NaN values DIRECTLY, no imputation required
```

### How this actually works internally
During training, for every split, XGBoost learns the BEST DEFAULT DIRECTION for missing values at that specific split — essentially, it tries sending all the missing values left, then tries sending them all right, and picks whichever gives better results, learning this per-split as part of the normal training process. This is genuinely different from imputation (which fills in a single guessed value BEFORE training) — XGBoost lets the missing-ness itself be informative, per split, rather than assuming one universal replacement value works everywhere.

### Should I still impute anyway?
Honest take: even though XGBoost handles NaNs natively, I'd still consider WHY data is missing before deciding. If missingness is genuinely random, letting XGBoost handle it natively is simpler and often works well. If missingness itself carries meaning (e.g. a survey question left blank because it wasn't applicable to that respondent), that's actually information XGBoost's native handling can already exploit — potentially better than blindly imputing a mean/median that might erase that meaningful signal. This connects back to the pandas data cleaning notes' broader point about being deliberate rather than mechanical about how missing data gets treated.

## Setting a custom missing value marker (if NaN isn't how missingness is represented in the data)
```python
model = XGBClassifier(missing=-999)     # tells XGBoost to treat -999 as the missing value marker instead of NaN, useful for older/legacy datasets using a specific sentinel value
```

## Imbalanced Classes — a genuinely important, common real-world problem
Recap from the Seaborn EDA class imbalance notes and scikit-learn's stratification notes — when one class vastly outnumbers another (fraud detection, rare disease diagnosis, etc.), a naive model can achieve high accuracy by just predicting the majority class every time, without genuinely learning anything useful.

### scale_pos_weight — XGBoost's primary tool for binary imbalanced classification
```python
# A common calculation: ratio of negative to positive class counts
scale_pos_weight = (y_train == 0).sum() / (y_train == 1).sum()

model = XGBClassifier(scale_pos_weight=scale_pos_weight)
model.fit(X_train, y_train)
```
This parameter effectively tells XGBoost to weight the MINORITY class more heavily during training, so mistakes on the rare class are penalized more than mistakes on the common class — directly addressing the core problem rather than just hoping accuracy alone reflects real performance.

### Checking whether scale_pos_weight actually helped — using the RIGHT metrics (recap from scikit-learn evaluation notes)
```python
from sklearn.metrics import classification_report, roc_auc_score

predictions = model.predict(X_test)
probabilities = model.predict_proba(X_test)[:, 1]

print(classification_report(y_test, predictions))     # look specifically at RECALL for the minority class, not just overall accuracy
print(roc_auc_score(y_test, probabilities))
```
Genuinely important recap from scikit-learn file 08 — plain accuracy remains a misleading metric here even after applying scale_pos_weight; precision/recall/F1 (specifically for the minority class) and ROC-AUC are the metrics that actually reveal whether the imbalance handling worked.

## Combining with resampling techniques (from outside XGBoost, but commonly paired with it)
```python
from imblearn.over_sampling import SMOTE     # a separate library, imbalanced-learn, genuinely popular for this specific problem

smote = SMOTE(random_state=42)
X_resampled, y_resampled = smote.fit_resample(X_train, y_train)     # creates SYNTHETIC minority-class examples to balance the dataset

model = XGBClassifier()
model.fit(X_resampled, y_resampled)
```
SMOTE (Synthetic Minority Oversampling Technique) generates new, synthetic minority-class examples by interpolating between existing minority examples, rather than just duplicating them — a genuinely different approach from `scale_pos_weight`'s "reweight the existing data" strategy. Worth noting: SMOTE should ONLY be applied to training data (recap the fit-on-train-only principle from scikit-learn preprocessing), never to validation/test data, since creating synthetic test examples would give a dishonestly easy evaluation.

## scale_pos_weight vs SMOTE — which to use
- **scale_pos_weight** — simpler, genuinely built directly into XGBoost, no extra library needed, works by reweighting the LOSS function
- **SMOTE** — genuinely creates new synthetic data points, can help when the minority class is SEVERELY underrepresented and reweighting alone isn't enough, but adds a real preprocessing step and a dependency on an external library

My honest starting approach: try `scale_pos_weight` first since it's simpler and built-in — only reach for SMOTE if that alone doesn't yield strong enough minority-class recall/F1.

## Multi-class imbalance (a genuinely trickier case than binary)
```python
from sklearn.utils.class_weight import compute_sample_weight

sample_weights = compute_sample_weight(class_weight="balanced", y=y_train)
model.fit(X_train, y_train, sample_weight=sample_weights)
```
`scale_pos_weight` is specifically for BINARY classification — for multi-class imbalance, `sample_weight` (a per-row weight array, computed here via scikit-learn's helper) is the more general approach, giving individual training examples different importance during training regardless of how many classes exist.
