# 11 — Bayes' Theorem & Bayesian Thinking

Picking up file 09's unanswered spam-filter question directly. Bayes' theorem is genuinely the correct, precise way to update a belief given new evidence — and it's surprisingly easy to get wrong by intuition alone.

## The formula

```
P(A | B) = [ P(B | A) * P(A) ] / P(B)
```

In words: the probability of A given B equals the probability of B given A, times the prior probability of A, divided by the overall probability of B. Each term has a name genuinely worth knowing:
- `P(A)` — the **prior** (what I believed about A before seeing evidence B)
- `P(B | A)` — the **likelihood** (how likely the evidence is, if A is true)
- `P(A | B)` — the **posterior** (updated belief about A, after seeing evidence B)
- `P(B)` — the **evidence** (overall probability of seeing B at all, normalizing everything)

## Solving file 09's spam filter question properly

```python
# Given:
P_spam = 0.30                    # prior: 30% of emails are spam
P_not_spam = 0.70
P_free_given_spam = 0.90          # likelihood: 90% of spam contains "free"
P_free_given_not_spam = 0.10      # 10% of non-spam also contains "free"

# Step 1: law of total probability (recap file 09) to get P(free)
P_free = (P_free_given_spam * P_spam) + (P_free_given_not_spam * P_not_spam)
print(P_free)   # 0.34

# Step 2: Bayes' theorem
P_spam_given_free = (P_free_given_spam * P_spam) / P_free
print(P_spam_given_free)   # ~0.794
```

So: an email containing "free" is about 79.4% likely to be spam — NOT 90% (the number most people's intuition jumps to, mistakenly using `P(free | spam)` as if it were `P(spam | free)`). This exact mistake — confusing `P(B|A)` with `P(A|B)` — is genuinely one of the most common statistical reasoning errors, worth remembering by name: **confusion of the inverse**.

## Naive Bayes classifier — the direct ML application

Recap scikit-learn notes on classification algorithms — Naive Bayes IS this formula, applied to classify based on multiple features, with a simplifying ("naive") assumption that features are conditionally independent given the class.

```python
import numpy as np

# Simplified: classify email as spam/not-spam using word presence as features
# P(spam | word1, word2) proportional to P(word1|spam) * P(word2|spam) * P(spam)
# (the "naive" independence assumption avoids needing P(word1 AND word2 | spam) jointly)

def naive_bayes_score(features_likelihoods, prior):
    score = prior
    for likelihood in features_likelihoods:
        score *= likelihood
    return score

spam_score = naive_bayes_score([0.9, 0.6], P_spam)         # P("free")*P("winner")|spam * prior
not_spam_score = naive_bayes_score([0.1, 0.05], P_not_spam)

# normalize to get actual probabilities
total = spam_score + not_spam_score
P_spam_given_evidence = spam_score / total
print(P_spam_given_evidence)
```

The "naive" independence assumption is genuinely often technically wrong (words in real emails ARE correlated) but works surprisingly well in practice — a good early lesson that a simplifying assumption doesn't have to be perfectly true to be practically useful, a theme that reappears often in ML.

## Bayesian vs Frequentist thinking — the philosophical split, briefly

**Frequentist** view: probability represents long-run frequency across repeated trials. A parameter (like a coin's true bias) is a FIXED but unknown constant — doesn't have a probability distribution itself.

**Bayesian** view: probability represents a DEGREE OF BELIEF, which can apply to fixed unknown parameters too. A coin's true bias can have a probability DISTRIBUTION representing uncertainty about it, and that distribution gets updated as more data arrives.

```python
# Bayesian coin-bias estimation: start with a "prior belief" the coin might be biased anywhere,
# then update after observing flips

from scipy import stats

# Prior: Beta distribution, genuinely the standard choice for modeling an unknown probability
prior_alpha, prior_beta = 1, 1     # Beta(1,1) = uniform, i.e. "no idea yet, could be any bias"

# Observed: 8 heads, 2 tails
heads, tails = 8, 2
posterior_alpha = prior_alpha + heads
posterior_beta = prior_beta + tails

posterior_mean = posterior_alpha / (posterior_alpha + posterior_beta)
print(posterior_mean)   # 0.818 -- updated belief the coin is biased toward heads
```

This Beta-Binomial update is genuinely a clean, closed-form example of Bayesian updating: start with a prior, observe data, get a posterior — and the posterior can become the prior for the NEXT batch of observed data, updating continuously. This general pattern (prior → likelihood → posterior) reappears in Bayesian optimization (used in hyperparameter tuning, recap XGBoost file 08) and Bayesian deep learning approaches to uncertainty estimation.

## Where this connects forward — genuinely practical payoff

- **Naive Bayes classifiers** — direct scikit-learn application, already covered above.
- **A/B testing and hypothesis testing** (file 14) — the frequentist vs Bayesian framing genuinely affects how experiment results get interpreted; important to know both exist.
- **Bayesian hyperparameter optimization** — smarter alternatives to grid/random search (recap XGBoost file 08) use Bayesian methods to model which hyperparameter regions are likely to perform well, updating that belief after each trial.
- **RAG systems** (upcoming, Frontier GenAI phase) — retrieval scoring in RAG pipelines is conceptually related to updating belief about which documents are relevant given a query, though not literally computed via Bayes' formula in most implementations — flagging the conceptual kinship here.

## Try this

```python
# A medical test is 95% accurate (both for true positives and true negatives).
# The disease being tested for affects only 1% of the population.
# Using Bayes' theorem, compute P(has disease | positive test result)
# The answer is genuinely much lower than 95% -- work through why, using the same
# "confusion of the inverse" mistake flagged above as the thing to consciously avoid
```

This is the single most famous real-world Bayes' theorem example (the "base rate fallacy") — working through it by hand is genuinely one of the best exercises for permanently fixing the intuition that a highly ACCURATE test on a RARE condition can still produce mostly false positives among all positive results.

## What's next
File 12 — Expectation, Variance & Covariance, the numerical summaries (mean, spread, and how variables move together) that show up constantly in loss functions, regularization, and PCA (recap file 03).
