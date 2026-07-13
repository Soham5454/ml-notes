# 06 — Ensemble Methods

Ensemble methods combine MULTIPLE models to get better performance than any single model alone — genuinely one of the most practically important topics in classical ML, since ensemble methods (especially gradient boosting) tend to win a huge share of real-world tabular data competitions.

## The core idea — why combining models helps
A single decision tree overfits easily (recap from file 05). But if you train MANY trees, each slightly different, and combine their predictions (majority vote for classification, averaging for regression), the individual trees' mistakes tend to cancel out while the genuine patterns they've all picked up on reinforce each other. This is the "wisdom of crowds" principle applied to models.

## Bagging — Bootstrap Aggregating
```python
from sklearn.ensemble import BaggingClassifier
from sklearn.tree import DecisionTreeClassifier

model = BaggingClassifier(estimator=DecisionTreeClassifier(), n_estimators=50, random_state=42)
model.fit(X_train, y_train)
```
Bagging trains many copies of the SAME algorithm, each on a different random SAMPLE of the training data (sampled WITH replacement — "bootstrap" samples), then averages their predictions. This reduces variance (overfitting) without needing to change the underlying algorithm at all.

## Random Forest — bagging specifically applied to decision trees, plus one extra trick
```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, max_depth=10, random_state=42)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```
Random Forest is genuinely just bagged decision trees, with one additional twist: at each split, each tree only considers a RANDOM SUBSET of features (not all of them). This extra randomness makes the individual trees even MORE different from each other, which further reduces overfitting when combined.

### Feature importance — a genuinely useful bonus from tree-based models
```python
importances = model.feature_importances_
feature_importance_df = pd.DataFrame({"feature": X.columns, "importance": importances}).sort_values("importance", ascending=False)
```
This is one of my favorite things about tree-based ensembles — after training, you get a genuinely useful ranking of which features actually mattered most for predictions, which connects directly back to the feature engineering/selection work from the pandas and upcoming file 11 notes.

### Key hyperparameters
```python
RandomForestClassifier(
    n_estimators=100,       # number of trees -- more trees = generally better but slower, with diminishing returns
    max_depth=10,              # limits how deep each individual tree can grow
    min_samples_split=2,          # minimum samples needed to split a node further
    max_features="sqrt"              # how many features each tree considers per split
)
```

## Boosting — a fundamentally different ensemble strategy
Unlike bagging (which trains models independently and combines them), boosting trains models SEQUENTIALLY, where each new model specifically focuses on fixing the mistakes of the previous ones.

## AdaBoost
```python
from sklearn.ensemble import AdaBoostClassifier

model = AdaBoostClassifier(n_estimators=50, random_state=42)
model.fit(X_train, y_train)
```
Each new "weak learner" (typically a very shallow decision tree, sometimes just a single split called a "stump") gets trained with EXTRA WEIGHT placed on the examples the previous learners got wrong — literally forcing each new model to pay more attention to the hard cases.

## Gradient Boosting
```python
from sklearn.ensemble import GradientBoostingClassifier

model = GradientBoostingClassifier(n_estimators=100, learning_rate=0.1, max_depth=3, random_state=42)
model.fit(X_train, y_train)
```
Similar sequential idea to AdaBoost, but instead of reweighting examples, each new tree is trained to predict the RESIDUAL ERRORS (recap the residual concept from the Seaborn regression plots notes) of the combined previous trees. Genuinely one of the most powerful "out of the box" algorithms for structured/tabular data — this is the conceptual foundation that XGBoost (next in my actual roadmap after this) builds on and optimizes further.

### Key hyperparameters
```python
GradientBoostingClassifier(
    n_estimators=100,        # number of sequential trees
    learning_rate=0.1,          # how much each new tree's correction contributes -- smaller = more conservative, needs more trees to compensate
    max_depth=3                    # gradient boosting trees are typically SHALLOW (unlike random forest's deeper trees) -- boosting relies on many weak learners combined, not individually strong ones
)
```
The `learning_rate` and `n_estimators` genuinely trade off against each other — a smaller learning rate needs more trees to reach the same performance, but often generalizes better. This exact learning_rate concept reappears identically in neural network training (recap connects forward to file 14/PyTorch).

## Voting Classifier — combining genuinely DIFFERENT algorithms together
```python
from sklearn.ensemble import VotingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC

voting_model = VotingClassifier(estimators=[
    ("lr", LogisticRegression()),
    ("rf", RandomForestClassifier()),
    ("svc", SVC(probability=True))
], voting="soft")     # 'soft' averages predicted PROBABILITIES, 'hard' takes a majority vote on final class predictions
model.fit(X_train, y_train)
```
Different from bagging/boosting (which combine many copies of the SAME algorithm) — this combines fundamentally DIFFERENT algorithms, on the idea that different algorithms might make different KINDS of mistakes, so combining them smooths things out further.

## Bagging vs Boosting — the core practical difference to remember
- **Bagging (Random Forest)** — trains models INDEPENDENTLY/in parallel, reduces VARIANCE (overfitting), generally more robust to noisy data
- **Boosting (Gradient Boosting)** — trains models SEQUENTIALLY, reduces BIAS (underfitting) by focusing on mistakes, often achieves higher raw accuracy but is more prone to overfitting if not carefully tuned (via learning_rate, n_estimators, max_depth)

## Honest practical takeaway
For most real tabular data problems, I now understand the reasonable "escalation path": try a simple linear/logistic baseline first (file 04), then a Random Forest as a strong, fairly safe default, then Gradient Boosting (or XGBoost specifically, next in my roadmap) if squeezing out more performance is worth the added tuning complexity.
