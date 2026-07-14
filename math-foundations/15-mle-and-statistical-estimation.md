# 15 — MLE & Statistical Estimation

This is genuinely the file that answers a question quietly sitting unanswered since file 09: WHY are MSE and cross-entropy the "correct" loss functions to minimize, rather than some other arbitrary formula that also happens to penalize wrong predictions? Maximum Likelihood Estimation is the answer.

## The core idea behind MLE

Given some observed data, and a model with unknown parameters, MLE asks: **which parameter values make the OBSERVED data most probable?** Instead of guessing parameters and checking if they fit, MLE flips the question — search over all possible parameter values for the ones under which the actual observed data would have been the most likely outcome.

## A simple worked example — estimating a coin's bias

```python
import numpy as np
from scipy.optimize import minimize_scalar

# Observed: 7 heads, 3 tails out of 10 flips
heads, tails = 7, 3

# Likelihood of observing this data, as a function of p (probability of heads)
def negative_log_likelihood(p):
    if p <= 0 or p >= 1:
        return np.inf
    log_likelihood = heads * np.log(p) + tails * np.log(1 - p)
    return -log_likelihood   # negative, since we're using a MINIMIZER to find the MAXIMUM

result = minimize_scalar(negative_log_likelihood, bounds=(0.001, 0.999), method='bounded')
print(result.x)   # should be very close to 0.7 -- exactly heads/total, matching intuition
```

Genuinely satisfying result: MLE mathematically CONFIRMS that "just use the observed proportion" (7/10 = 0.7) is the statistically optimal estimate for a Bernoulli parameter — not just a convenient shortcut, but the actual maximum-likelihood answer, derivable from calculus (setting the derivative of the log-likelihood to zero, recap file 05).

## Why "log-likelihood" instead of raw likelihood — a genuinely practical reason

```python
# Raw likelihood involves multiplying many small probabilities together
# e.g. for 1000 data points, likelihood = p1 * p2 * ... * p1000
# This underflows to numerically 0 almost immediately on a computer

# Taking the LOG converts products into SUMS (log(a*b) = log(a) + log(b))
# which is numerically stable and, since log is monotonic, maximizing log-likelihood
# gives the EXACT SAME answer as maximizing raw likelihood
```

This is genuinely why "log-likelihood" (or negative log-likelihood, since optimizers conventionally MINIMIZE) appears constantly in ML rather than raw likelihood — pure numerical necessity, not a different underlying idea.

## The connection to MSE — linear regression IS maximum likelihood, under a Gaussian assumption

Recap file 10's normal distribution notes. If the ERRORS in a regression problem are assumed to be normally distributed (a very common, often reasonable assumption), maximizing the likelihood of the observed data turns out to be MATHEMATICALLY IDENTICAL to minimizing mean squared error.

```python
# The Gaussian likelihood for a single data point:
# L(y | y_pred, sigma) = (1 / (sigma*sqrt(2*pi))) * exp(-(y - y_pred)^2 / (2*sigma^2))
#
# Taking the log and dropping constants that don't depend on y_pred:
# log L proportional to  -(y - y_pred)^2
#
# MAXIMIZING this is the same as MINIMIZING (y - y_pred)^2 -- literally squared error
```

This is genuinely one of the most satisfying derivations in this whole folder — `nn.MSELoss` (recap PyTorch file 06) was never an arbitrary "let's punish big errors more" choice. It's the mathematically PROVEN correct loss function under the assumption that prediction errors follow a normal distribution. If a different noise assumption were more appropriate (e.g. Laplace-distributed errors), the "correct" MLE-derived loss would instead be Mean Absolute Error — this is precisely why MAE exists as an alternative, not just a stylistic choice.

## The connection to cross-entropy — classification IS maximum likelihood too

Recap file 09's cross-entropy connection to probability. For classification, the model outputs a probability distribution over classes (via softmax, recap PyTorch file 05), and the TRUE label represents "what actually happened." MLE for this setup:

```python
# For a single correctly-labeled example, class c:
# Likelihood = P(class = c | model's predicted distribution) = predicted_prob[c]
#
# Log-likelihood = log(predicted_prob[c])
#
# NEGATIVE log-likelihood = -log(predicted_prob[c])
# ... which is EXACTLY the cross-entropy loss formula from file 09 / PyTorch file 06
```

Genuinely closes the loop completely: cross-entropy loss is not a heuristic penalty function, it IS negative log-likelihood under the categorical distribution — the mathematically correct thing to minimize if the goal is "find model parameters that make the observed labels most probable."

## MLE vs MAP (Maximum A Posteriori) — bringing back file 11's Bayesian thinking

MLE finds parameters that maximize `P(data | parameters)`. MAP instead maximizes `P(parameters | data)`, which by Bayes' theorem (recap file 11) equals `P(data | parameters) * P(parameters) / P(data)` — MAP incorporates a PRIOR belief about the parameters, MLE doesn't.

```python
# Genuinely important practical connection: L2 regularization (weight decay, recap PyTorch file 09)
# is mathematically equivalent to MAP estimation with a Gaussian PRIOR on the weights
# (a prior belief that weights should tend to be small/near zero, unless the data strongly argues otherwise)
```

This is genuinely the deepest, most satisfying connection in this entire Math Foundations folder: `weight_decay` in PyTorch's Adam optimizer isn't just "add a penalty term because big weights are bad" — it's literally performing Bayesian MAP estimation with a specific prior, expressed through ordinary gradient descent machinery. Regularization was Bayesian statistics in disguise the entire time.

## Try this

```python
# Generate 200 samples from a Normal(mean=7, std=2) distribution using np.random.normal
# Write the negative log-likelihood function for a Normal distribution (mean and std as parameters)
# Use scipy.optimize.minimize to find the MLE estimates of mean and std
# Confirm they closely match the TRUE values used to generate the data (7 and 2)
```

Recovering the true generating parameters purely through MLE optimization is genuinely the clearest hands-on proof that "fitting a model" and "maximizing likelihood" are the same operation, whether it's a two-parameter Gaussian here or a billion-parameter neural network trained via `loss.backward()` + `optimizer.step()`.

## What's next
File 16 — Correlation & Regression Theory, the statistical view of the regression models already used throughout scikit-learn/XGBoost notes, closing the loop between "fit a line to data" and everything covered in this entire Math Foundations folder.
