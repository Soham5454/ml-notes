# Scikit-learn — A to Z

This is the big one — the first real contact with actual machine learning algorithms after all the data handling/visualization groundwork (Python, NumPy, pandas, matplotlib, Seaborn). Everything before this was preparation; this is where models actually get trained.

## Index

| # | Topic | File |
|---|-------|------|
| 01 | Introduction & The Estimator API | [01-introduction-estimator-api.md](01-introduction-estimator-api.md) |
| 02 | Preprocessing — Scaling & Encoding | [02-preprocessing-scaling-encoding.md](02-preprocessing-scaling-encoding.md) |
| 03 | Train-Test Split & Cross-Validation | [03-train-test-split-cross-validation.md](03-train-test-split-cross-validation.md) |
| 04 | Linear Models & Regression | [04-linear-models-regression.md](04-linear-models-regression.md) |
| 05 | Classification Algorithms | [05-classification-algorithms.md](05-classification-algorithms.md) |
| 06 | Ensemble Methods | [06-ensemble-methods.md](06-ensemble-methods.md) |
| 07 | Clustering & Unsupervised Learning | [07-clustering-unsupervised.md](07-clustering-unsupervised.md) |
| 08 | Model Evaluation & Metrics | [08-model-evaluation-metrics.md](08-model-evaluation-metrics.md) |
| 09 | Hyperparameter Tuning | [09-hyperparameter-tuning.md](09-hyperparameter-tuning.md) |
| 10 | Pipelines & Workflow | [10-pipelines-workflow.md](10-pipelines-workflow.md) |
| 11 | Feature Selection & Engineering | [11-feature-selection-engineering.md](11-feature-selection-engineering.md) |
| 12 | Dimensionality Reduction (PCA, t-SNE, LDA) | [12-dimensionality-reduction.md](12-dimensionality-reduction.md) |
| 13 | Model Persistence & Deployment Basics | [13-model-persistence-deployment.md](13-model-persistence-deployment.md) |
| 14 | ML to DL Bridge | [14-ml-to-dl-bridge.md](14-ml-to-dl-bridge.md) |

## Setup
```bash
pip install scikit-learn
```
```python
from sklearn import ...     # scikit-learn is modular -- you import specific things from specific submodules, not the whole library at once
```

## Roadmap position
Python Basics → NumPy → pandas → matplotlib → Seaborn → **scikit-learn (you are here)** → XGBoost → PyTorch → Hugging Face

## Why this file matters more than the previous ones
Everything up to Seaborn was about understanding and visualizing data. This is the first point where an algorithm actually LEARNS something from that data and makes predictions — genuinely a different category of skill from what came before, even though it builds directly on top of it.
