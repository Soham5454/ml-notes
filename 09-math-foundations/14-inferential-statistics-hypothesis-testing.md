# 14 — Inferential Statistics & Hypothesis Testing

File 13 described data already in hand. This file is about making JUSTIFIED claims about a bigger population using only a sample — genuinely the actual math behind "is this A/B test result real or just noise," which comes up constantly in real ML/product work.

## Population vs sample — the core distinction

The **population** is the entire group of interest (every user, every possible measurement). A **sample** is a subset actually observed. Almost nothing in real data science works with a full population — inferential statistics exists specifically to make defensible claims about the population FROM a sample, accounting honestly for the uncertainty that introduces.

## The Central Limit Theorem, properly — recap file 10's brief mention

```python
import numpy as np

population = np.random.exponential(scale=2.0, size=100000)   # deliberately skewed, NOT normal
print(population.mean())   # true population mean

sample_means = []
for _ in range(1000):
    sample = np.random.choice(population, size=50, replace=False)
    sample_means.append(sample.mean())

sample_means = np.array(sample_means)
print(sample_means.mean(), sample_means.std())   # sample_means themselves look normally distributed
```

Genuinely the foundational theorem for almost all classical inferential statistics: even though the POPULATION here is skewed (exponential), the distribution of SAMPLE MEANS (taken repeatedly) is approximately normal, and gets MORE normal and TIGHTER as sample size grows. This is precisely why so many statistical tests (which assume normality) still work reasonably well on real-world, non-normal data — they're often relying on the CLT applying to the sample mean, not requiring the raw data itself to be normal.

## Confidence intervals — quantifying uncertainty in an estimate

```python
from scipy import stats

sample = np.random.choice(population, size=50, replace=False)
sample_mean = sample.mean()
sample_std = sample.std(ddof=1)   # ddof=1 for sample standard deviation (Bessel's correction)
n = len(sample)

standard_error = sample_std / np.sqrt(n)
confidence_interval = stats.t.interval(0.95, df=n-1, loc=sample_mean, scale=standard_error)
print(f"Sample mean: {sample_mean:.2f}, 95% CI: {confidence_interval}")
```

A 95% confidence interval means: if this same sampling process were repeated many times, 95% of the resulting intervals would contain the TRUE population mean. Genuinely a common misinterpretation worth avoiding explicitly: it does NOT mean "95% probability the true mean is in THIS specific interval" — that's a subtly different (and technically Bayesian, recap file 11) claim than what a frequentist confidence interval actually guarantees.

`ddof=1` (delta degrees of freedom) matters genuinely — dividing by `n-1` instead of `n` when computing sample variance/std corrects for a slight bias that occurs when estimating a population parameter from a sample; NumPy defaults to `ddof=0` (population formula), so this has to be set explicitly for a proper SAMPLE standard deviation.

## Hypothesis testing — the formal framework

Every hypothesis test starts with two competing claims:
- **Null hypothesis (H0)** — "no effect" / "no difference" (the default, skeptical assumption)
- **Alternative hypothesis (H1)** — "there IS an effect/difference" (what's being tested for)

The test computes a **p-value**: the probability of seeing data at least as extreme as what was observed, IF the null hypothesis were actually true. A small p-value is evidence AGAINST the null hypothesis.

## A concrete A/B test example — t-test

```python
# Two versions of a website, measuring conversion rate (as a proxy metric, using time-on-site here)
version_A = np.random.normal(loc=50, scale=10, size=200)    # control
version_B = np.random.normal(loc=53, scale=10, size=200)     # variant, slightly higher

t_statistic, p_value = stats.ttest_ind(version_A, version_B)
print(f"t-statistic: {t_statistic:.3f}, p-value: {p_value:.4f}")

alpha = 0.05   # significance threshold, chosen BEFORE looking at the data
if p_value < alpha:
    print("Reject H0 -- the difference is statistically significant")
else:
    print("Fail to reject H0 -- not enough evidence of a real difference")
```

`alpha = 0.05` is a conventional threshold (5% chance of a false positive being considered acceptable) — genuinely arbitrary by tradition, not a law of nature, and worth knowing that different fields/companies sometimes use stricter thresholds (0.01) for higher-stakes decisions.

## Type I and Type II errors — the honest cost of any threshold choice

- **Type I error (false positive)** — rejecting H0 when it's actually true (concluding there's an effect when there isn't). Controlled by `alpha`.
- **Type II error (false negative)** — failing to reject H0 when it's actually false (missing a real effect). Related to a concept called **statistical power**.

```python
# Genuinely important practical tradeoff: lowering alpha (fewer false positives)
# increases the risk of Type II errors (missing real effects), all else equal
# This mirrors the precision/recall tradeoff from classification metrics (recap scikit-learn notes)
```

This is genuinely the SAME underlying tradeoff structure as precision vs recall in classification — being more conservative about claiming "positive" (low alpha, or a high classification threshold) reduces false positives but increases false negatives, and vice versa. Recognizing this parallel makes both concepts reinforce each other rather than feeling like two unrelated topics.

## Chi-square test — for categorical data relationships

```python
from scipy.stats import chi2_contingency

# Contingency table: rows = age group, columns = clicked ad (yes/no)
observed = np.array([[50, 30], [20, 40]])
chi2, p_value, dof, expected = chi2_contingency(observed)
print(f"p-value: {p_value:.4f}")
```

Used when testing whether two CATEGORICAL variables are related (e.g. "does age group affect click-through rate") — a genuinely common test for categorical feature analysis during EDA, complementing the correlation-based approach from file 13 which only works for numeric variables.

## Try this

```python
# Simulate two groups with genuinely no real difference (same mean, same std)
# Run a t-test 100 times on fresh random samples each time
# Count how many of those 100 tests produce p < 0.05 purely by chance
# This should land close to 5 (matching alpha=0.05) -- a direct, hands-on demonstration
# of what "5% false positive rate" actually means in practice
```

Actually seeing roughly 5 out of 100 "no real difference" tests still come back "significant" purely by chance is genuinely the clearest possible demonstration of why a single significant p-value from ONE test should never be treated as absolute proof — a healthy skepticism that's easy to state but much more convincing once witnessed directly in simulation.

## What's next
File 15 — MLE & Statistical Estimation, formalizing exactly HOW a model's parameters get chosen to "best fit" data — the deep statistical justification for why loss functions like MSE and cross-entropy are the RIGHT functions to minimize, not arbitrary choices.
