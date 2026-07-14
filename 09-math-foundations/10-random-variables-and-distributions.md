# 10 — Random Variables & Distributions

File 09 covered probability of individual events. A random variable formalizes "a quantity whose value depends on chance" — and a distribution describes ALL the possible values it can take, along with how likely each one is.

## Random variables — discrete vs continuous

A **discrete** random variable takes on a countable set of values (die roll: 1-6). A **continuous** random variable can take any value in a range (height, weight, model prediction error).

```python
import numpy as np

# Discrete: simulate 10000 die rolls
die_rolls = np.random.randint(1, 7, size=10000)
print(np.bincount(die_rolls)[1:] / 10000)   # roughly [1/6, 1/6, ...] for each face
```

## The Bernoulli distribution — the simplest possible distribution, and genuinely important

Models a single yes/no trial (coin flip, spam/not-spam, sick/healthy) with probability `p` of success.

```python
p = 0.3   # e.g. 30% probability of "positive" class
samples = np.random.binomial(n=1, p=p, size=10000)
print(samples.mean())   # should be close to 0.3
```

Recap PyTorch file 06's BCE loss (`nn.BCELoss`) — binary classification is LITERALLY modeling the output as a Bernoulli random variable, where the model's sigmoid output is its estimate of `p`. That's not an analogy, it's precisely what's happening mathematically.

## The Binomial distribution — repeated Bernoulli trials

Models the number of successes across `n` independent Bernoulli trials.

```python
# 20 coin flips, probability of heads = 0.5, repeated 10000 times -- how many heads typically?
results = np.random.binomial(n=20, p=0.5, size=10000)
print(results.mean())   # should be close to 10 (20 * 0.5)
```

## The Normal (Gaussian) distribution — genuinely the most important distribution in all of ML

```python
samples = np.random.normal(loc=0, scale=1, size=10000)   # mean=0, std=1 -- "standard normal"
print(samples.mean(), samples.std())   # close to 0 and 1
```

Formula (the famous "bell curve"):
```
f(x) = (1 / (σ√(2π))) * e^(-(x-μ)² / (2σ²))
```

`μ` (mu) is the mean (center), `σ` (sigma) is the standard deviation (spread). Genuinely everywhere in ML:
- **Weight initialization** (recap `nn.Linear`'s automatic init in PyTorch file 04) — weights are typically initialized by sampling from a normal distribution, specifically scaled to keep gradients well-behaved (Xavier/He initialization, both genuinely just "sample from a Normal distribution with a carefully chosen `σ`").
- **Feature scaling / standardization** (recap scikit-learn's `StandardScaler`) — transforms data to have mean 0, std 1, matching the standard normal shape, which is precisely why it's called "standardization."
- **VAE latent space** (recap PyTorch file 16) — explicitly forced to follow a standard normal distribution.
- **Noise modeling** — Gaussian noise is the default assumption in countless statistical/ML models (e.g. linear regression's error term).

## Why the Normal distribution shows up SO often — the Central Limit Theorem, briefly

Genuinely worth knowing conceptually: when many independent random effects add together, their SUM tends toward a Normal distribution, regardless of the shape of the individual effects' own distributions. This is a huge part of why Gaussian noise assumptions work reasonably well across so many unrelated real-world phenomena — measurement errors, biological traits, aggregated small effects.

```python
# Demonstrating CLT: sum of many uniform (NOT normal) random variables looks normal
uniform_sums = np.array([np.sum(np.random.uniform(0, 1, 30)) for _ in range(5000)])
print(uniform_sums.mean(), uniform_sums.std())
# plotting a histogram of uniform_sums would show a bell-curve shape, despite each individual
# component being uniformly (flat) distributed, not normal at all
```

## Probability Density Function (PDF) vs Cumulative Distribution Function (CDF)

```python
from scipy import stats

# PDF: the "height" of the curve at a specific point (NOT a probability itself for continuous variables)
pdf_value = stats.norm.pdf(0, loc=0, scale=1)
print(pdf_value)   # ~0.3989, the peak height of the standard normal curve

# CDF: probability that the random variable is LESS THAN OR EQUAL TO x
cdf_value = stats.norm.cdf(1.96, loc=0, scale=1)
print(cdf_value)   # ~0.975 -- genuinely the basis of the common "95% confidence" cutoff in file 14
```

Important distinction worth being precise about: for continuous distributions, `P(X = exact value)` is technically 0 (there are infinitely many possible values) — probabilities only make sense over a RANGE, which is what the CDF (or integrating the PDF over a range) actually computes.

## The Uniform distribution — every outcome equally likely

```python
samples = np.random.uniform(low=0, high=1, size=10000)
```

Recap PyTorch file 02's `torch.rand()` — that function samples from exactly this distribution. Used constantly for random initialization ranges and, notably, as the noise source `z` fed into a GAN's generator (recap PyTorch file 17), though normal-distributed noise is also common there.

## Try this

```python
# Generate 10000 samples from a Normal(mean=50, std=10) distribution using np.random.normal
# 1. Verify empirically that ~68% of samples fall within 1 std of the mean (40 to 60)
# 2. Verify ~95% fall within 2 std (30 to 70)
# This is the "68-95-99.7 rule" -- genuinely useful to have confirmed with real numbers, not just memorized
```

Confirming the 68-95-99.7 rule empirically (rather than just memorizing it) builds the intuition needed for file 14's confidence intervals and hypothesis testing, which lean on this exact rule constantly.

## What's next
File 11 — Bayes' Theorem & Bayesian Thinking, finally answering file 09's spam-filter question properly, and setting up the probabilistic reasoning behind Naive Bayes and modern Bayesian ML techniques.
