# 12 — Expectation, Variance & Covariance

Files 09-11 covered probability and distributions conceptually. This file covers the actual numerical summaries computed FROM a distribution — genuinely the building blocks of loss functions, regularization, and PCA's covariance matrix (recap file 03).

## Expectation (expected value) — the "average outcome" of a random variable

```
E[X] = sum( x * P(x) )   for discrete X
```

```python
import numpy as np

# Fair die: E[X] = 1*(1/6) + 2*(1/6) + ... + 6*(1/6)
outcomes = np.array([1, 2, 3, 4, 5, 6])
probs = np.array([1/6]*6)
expected_value = np.sum(outcomes * probs)
print(expected_value)   # 3.5
```

Genuinely just a weighted average, where the weights are the probabilities. This is the theoretical version of the "mean" — the sample mean (recap pandas `.mean()`) is an ESTIMATE of the true expectation, computed from actual observed data rather than known probabilities.

## Why expectation matters for loss functions — the direct payoff

Recap PyTorch file 06's `nn.MSELoss` — the loss computed over a batch is literally an EXPECTATION (average) of the squared error across samples: `Loss = E[(y_pred - y_true)^2]`. Training a model to minimize loss is, precisely, minimizing the EXPECTED error across the whole data distribution, not just the specific training batch — the batch is a sample used to ESTIMATE that expectation, which is exactly why train/val splits (recap scikit-learn/XGBoost/PyTorch notes) matter: low loss on the training batch doesn't guarantee low EXPECTED loss on unseen data.

## Variance — how spread out a random variable is

```
Var(X) = E[(X - E[X])^2]
```

```python
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])
mean = np.mean(data)
variance = np.mean((data - mean)**2)
print(variance)   # 4.0 -- matches np.var(data)
print(np.var(data))
```

Genuinely just "the average squared distance from the mean" — squaring ensures negative and positive deviations don't cancel out, and penalizes large deviations more (recap file 05's MSE intuition — same squaring reasoning applies here for the exact same purpose).

## Standard deviation — variance's more interpretable cousin

```python
std = np.sqrt(variance)
print(std)   # 2.0
```

Standard deviation is in the SAME units as the original data (variance is in squared units, less intuitive) — this is precisely why `StandardScaler` (scikit-learn) divides by standard deviation, not variance, when standardizing features.

## The bias-variance tradeoff — genuinely one of the most important ML concepts, now with real math grounding

Recap the overfitting discussions from scikit-learn/XGBoost/PyTorch (files on regularization/early stopping) — here's the formal decomposition:

```
Total Error = Bias^2 + Variance + Irreducible Noise
```

- **Bias** — error from a model being too SIMPLE to capture the true pattern (underfitting). A linear model trying to fit a curved relationship has high bias.
- **Variance** — error from a model being too SENSITIVE to the specific training data (overfitting) — small changes in training data cause big changes in the learned model. A very deep decision tree with no regularization has high variance.
- **Irreducible noise** — genuine randomness in the data that NO model can predict away, a hard floor on achievable performance.

This formal decomposition is genuinely the mathematical backing for every regularization technique covered across the whole repo (`reg_lambda` in XGBoost, dropout/weight decay in PyTorch) — they all work by deliberately INCREASING bias slightly in exchange for a bigger REDUCTION in variance, when the model is in an overfitting regime.

## Covariance — how two variables move together

```
Cov(X, Y) = E[(X - E[X]) * (Y - E[Y])]
```

```python
x = np.array([1, 2, 3, 4, 5])
y = np.array([2, 4, 5, 4, 5])

cov_matrix = np.cov(x, y)
print(cov_matrix)
# [[var(x), cov(x,y)],
#  [cov(x,y), var(y)]]
```

Positive covariance means the two variables tend to increase together; negative means one tends to increase as the other decreases; near-zero means little linear relationship. Recap file 03's PCA derivation directly — the covariance MATRIX (containing every pairwise covariance between features) is EXACTLY what gets eigen-decomposed to find principal components. This file closes the loop on that: now the covariance matrix's actual entries are computable and understandable, not just "the input to PCA."

## Correlation — covariance, normalized to be interpretable

```python
correlation_matrix = np.corrcoef(x, y)
print(correlation_matrix)
```

Correlation is covariance divided by the product of both standard deviations, which forces it into a fixed range of -1 to 1, making it comparable across different variable scales (covariance's magnitude depends on the variables' units, correlation's doesn't). File 16 covers correlation properly in the context of regression theory and the classic "correlation does not imply causation" caveat.

## Try this

```python
# Simulate two scenarios with numpy:
# Scenario A: y = 2*x + small random noise (strong positive relationship)
# Scenario B: y = completely random, unrelated to x
# Compute covariance and correlation for both scenarios
# Confirm Scenario A shows strong positive covariance/correlation, Scenario B shows near-zero
```

Seeing covariance/correlation numerically confirm an intentionally-built strong vs weak relationship is genuinely the clearest way to trust these numbers when they show up unexplained in a real dataset's correlation matrix (a very common EDA step, recap Seaborn's heatmap notes).

## What's next — moving from Probability into Statistics
File 13 — Descriptive Statistics, the practical toolkit (mean, median, quartiles, outlier detection) used constantly during EDA (recap pandas/Seaborn notes), now grounded in the expectation/variance theory just covered.
