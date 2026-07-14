# 09 — Probability Fundamentals

Files 01-08 covered how a model computes and learns. Probability is the language for reasoning about UNCERTAINTY — genuinely underlies every loss function that measures "how wrong" a prediction is, and every generative model that samples new data.

## What probability actually measures

Probability quantifies how likely an event is, as a number between 0 (impossible) and 1 (certain). `P(A)` denotes "the probability of event A."

```python
# Simple example: fair 6-sided die
outcomes = [1, 2, 3, 4, 5, 6]
P_rolling_4 = 1 / len(outcomes)
print(P_rolling_4)   # 0.1666...
```

## Sample space and events

The **sample space** is the set of ALL possible outcomes. An **event** is any subset of that sample space.

```python
sample_space = {1, 2, 3, 4, 5, 6}
event_even = {2, 4, 6}
P_even = len(event_even) / len(sample_space)
print(P_even)   # 0.5
```

## Independence — a genuinely critical concept for ML

Two events are **independent** if one happening doesn't affect the probability of the other: `P(A and B) = P(A) * P(B)`.

```python
# Two fair coin flips, independent
P_heads = 0.5
P_two_heads = P_heads * P_heads
print(P_two_heads)   # 0.25
```

Recap the dataset/DataLoader shuffle note from PyTorch file 08 — mini-batch training implicitly ASSUMES samples are drawn roughly independently, which is precisely why shuffling matters: without it, consecutive batches can be correlated (e.g. all one class in a row if the dataset was sorted), violating this independence assumption and destabilizing training.

## Conditional probability — probability GIVEN some information

```
P(A | B) = P(A and B) / P(B)
```

Read as "the probability of A, GIVEN that B has already happened." Worked example:

```python
# Deck of cards: P(king | face card)
# P(king and face card) = P(king) = 4/52  (all kings are face cards)
# P(face card) = 12/52

P_king_and_face = 4/52
P_face = 12/52
P_king_given_face = P_king_and_face / P_face
print(P_king_given_face)   # 0.333... -- 1/3, since kings are 1 of 3 face-card ranks
```

Genuinely the exact concept underlying every classification model's output — `model(x)` in a classifier is literally trying to estimate `P(class | features)` — "given these features, what's the probability of each class." This connects directly to file 06's cross-entropy loss and Bayes' theorem in file 11.

## Mutually exclusive events

```python
# Rolling a die: can't be both 2 AND 5 on the same roll
P_2_or_5 = (1/6) + (1/6)   # for mutually exclusive events, P(A or B) = P(A) + P(B)
print(P_2_or_5)   # 0.333...
```

If events CAN overlap, the formula needs adjusting: `P(A or B) = P(A) + P(B) - P(A and B)`, subtracting the overlap so it isn't double-counted.

## The connection to loss functions — the genuinely important payoff of this file

Recap PyTorch file 06's cross-entropy loss. Cross-entropy is DIRECTLY built from probability theory — specifically, it measures how far a model's PREDICTED probability distribution is from the TRUE distribution (which, for a labeled example, is 100% certainty on the correct class, 0% on everything else).

```python
import numpy as np

# true label: class 1 (one-hot: [0, 1, 0])
true_dist = np.array([0, 1, 0])

# model's predicted probabilities (from softmax, recap PyTorch file 05)
predicted_dist = np.array([0.2, 0.7, 0.1])

# cross-entropy: -sum(true * log(predicted))
cross_entropy = -np.sum(true_dist * np.log(predicted_dist))
print(cross_entropy)   # -log(0.7) approx 0.357
```

Genuinely worth internalizing: if the model had predicted `0.99` for the correct class instead of `0.7`, cross-entropy would be much smaller (`-log(0.99) ≈ 0.01`) — the loss function IS a probability concept (negative log-likelihood, formally), not just an arbitrary formula that happens to penalize wrong answers. File 15 (MLE) makes this connection even more explicit.

## Law of total probability — combining conditional probabilities

```
P(A) = P(A|B1)*P(B1) + P(A|B2)*P(B2) + ...
```

Useful when an event's probability depends on which of several distinct scenarios occurred. This shows up directly in file 11's Bayes' theorem derivation — flagging it now so the derivation there doesn't feel like it's introducing a brand new idea out of nowhere.

## Try this

```python
# A spam filter dataset: 30% of emails are spam.
# 90% of spam emails contain the word "free". 10% of non-spam emails also contain "free".
# 1. Compute P(email contains "free") using the law of total probability
# 2. Manually reason: if an email contains "free", does it feel like it's more likely spam or not?
# (File 11 will show exactly how to compute P(spam | contains "free") properly using Bayes' theorem)
```

Trying to reason about this by intuition first, before file 11 gives the exact formula, is genuinely the best setup for appreciating why Bayes' theorem is needed at all — intuition alone gets this wrong surprisingly often (the classic "base rate" mistake).

## What's next
File 10 — Random Variables & Distributions, formalizing "outcomes with probabilities" into the mathematical objects (Gaussian, Bernoulli, etc.) that show up constantly in ML — weight initialization, noise modeling, and the assumptions behind loss functions.
