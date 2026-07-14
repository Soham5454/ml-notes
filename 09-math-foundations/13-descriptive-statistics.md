# 13 — Descriptive Statistics

Files 09-12 built the probability theory. This file is the practical toolkit for SUMMARIZING actual data — genuinely the exact operations already run during EDA in the pandas/Seaborn notes, now with the "why" behind each one.

## Measures of central tendency — mean, median, mode

```python
import numpy as np
import pandas as pd

data = pd.Series([2, 4, 4, 4, 5, 5, 7, 9, 100])   # deliberately includes an outlier

print(data.mean())     # ~15.56 -- pulled heavily toward the outlier
print(data.median())   # 5.0 -- unaffected by the outlier
print(data.mode()[0])  # 4 -- most frequent value
```

Genuinely important, practical distinction: **mean is sensitive to outliers, median is not.** This is exactly why median household income is reported in the news instead of mean — one billionaire in a sample would distort the mean wildly, while median stays representative of the "typical" value. Recap pandas notes on `.describe()` — this is precisely what that function is computing under the hood.

## Measures of spread — range, variance, standard deviation, IQR

```python
print(data.std())                    # recap file 12's standard deviation
print(data.max() - data.min())        # range -- also sensitive to outliers

Q1 = data.quantile(0.25)
Q3 = data.quantile(0.75)
IQR = Q3 - Q1
print(IQR)   # interquartile range -- robust to outliers, unlike range/std
```

The **IQR** (interquartile range) is genuinely the outlier-robust alternative to standard deviation for measuring spread — this is the exact mathematical basis for boxplots (recap Seaborn's boxplot notes): the "whiskers" typically extend to `Q1 - 1.5*IQR` and `Q3 + 1.5*IQR`, and anything beyond that gets flagged as an outlier point.

```python
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR
outliers = data[(data < lower_bound) | (data > upper_bound)]
print(outliers)   # correctly flags the 100
```

## Skewness — measuring asymmetry

```python
print(data.skew())   # positive value -- distribution has a long tail toward high values
```

Positive skew means the tail stretches toward larger values (mean > median, exactly the pattern seen above with the outlier). Negative skew is the opposite. Genuinely useful during EDA for deciding whether a feature needs a transformation (like a log transform) before feeding it into a model that assumes roughly symmetric/normal input distributions.

## Kurtosis — measuring "tailedness"

```python
print(data.kurtosis())
```

Measures how heavy a distribution's tails are compared to a normal distribution — high kurtosis means more extreme outliers than a normal distribution would predict. Less commonly used day-to-day than skewness, but worth recognizing when it shows up in `.describe()`-adjacent statistical summaries.

## The five-number summary and `.describe()`

```python
print(data.describe())
# count, mean, std, min, 25%, 50%, 75%, max
```

This single pandas call — used constantly without a second thought throughout the earlier notes — is genuinely just: count (sample size), mean (file 12), std (file 12), and the five-number summary (min, Q1, median, Q3, max) that a boxplot visualizes directly. Nothing mysterious left in this function anymore.

## Z-scores — standardizing individual values for comparison

```python
z_scores = (data - data.mean()) / data.std()
print(z_scores)
```

A Z-score expresses how many standard deviations a value is from the mean. Recap file 10's 68-95-99.7 rule — a Z-score beyond ±2 or ±3 is a common practical threshold for flagging outliers, DIRECTLY connected to that empirical rule (roughly 95% of normally-distributed data falls within Z-scores of ±2, so a ±2 threshold flags roughly the most extreme 5%). This is also EXACTLY the computation `StandardScaler` performs on every feature, recap scikit-learn preprocessing notes.

## Correlation matrices in practice — connecting file 12 to real EDA

```python
df = pd.DataFrame({
    'x': np.random.randn(100),
    'y': np.random.randn(100) * 2 + 1,
})
df['z'] = df['x'] * 3 + np.random.randn(100) * 0.1   # z strongly correlated with x

print(df.corr())
```

Recap Seaborn's heatmap notes — `sns.heatmap(df.corr(), annot=True)` visualizes exactly this matrix. Now fully traceable back through file 12's covariance formula, all the way to file 01's dot product (correlation is, underneath everything, a normalized dot product between two centered vectors — genuinely every file in this Math Foundations folder eventually connects back to that single file 01 operation).

## Try this

```python
# Load a real-ish dataset (or generate one with a few outliers deliberately injected)
# 1. Compute mean vs median -- confirm they diverge specifically where outliers exist
# 2. Compute Z-scores for every value, flag anything beyond |Z| > 2.5 as a potential outlier
# 3. Cross-check against the IQR method's flagged outliers -- do the two methods agree?
```

Comparing the Z-score method against the IQR method on the same data is genuinely a good practical exercise — they usually roughly agree but can diverge on skewed distributions, which is worth seeing firsthand rather than assuming any single outlier-detection method is universally correct.

## What's next
File 14 — Inferential Statistics & Hypothesis Testing, moving from DESCRIBING data already collected to making justified CLAIMS about a larger population based on a sample — the actual statistical machinery behind A/B testing and "is this result significant" questions.
