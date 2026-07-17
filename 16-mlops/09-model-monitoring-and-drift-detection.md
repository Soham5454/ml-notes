# 09 — Model Monitoring & Drift Detection

Recap file 08's closing note and file 01's "the model has been silently getting worse for weeks and nobody noticed" problem — this file directly solves it. Genuinely essential once a model is actually deployed and serving real, live traffic.

## Why a model can genuinely degrade WITHOUT any code changing

```python
# Recap file 01's core three-way-risk insight directly:
# The MODEL and CODE haven't changed at all -- but the real-world DATA
# the model encounters in production can genuinely SHIFT over time,
# away from the distribution it was originally trained on
```

Genuinely worth grounding this in a concrete example: a sentiment classifier trained on movie reviews (recap Classical NLP/Hugging Face folders' IMDB examples) might genuinely start performing worse if suddenly used on PRODUCT reviews, or if slang/language usage patterns shift over time (recap the general "language evolves" observation) — the code and model weights are IDENTICAL, but the real-world input distribution has genuinely moved.

## Data drift vs Concept drift — genuinely important, distinct failure modes

```
DATA DRIFT: the INPUT distribution changes, but the relationship between
  inputs and outputs stays the same
  (e.g. genuinely more customers from a new country start using the app,
  shifting the feature distributions, but "positive review words" still
  genuinely mean positive sentiment)

CONCEPT DRIFT: the RELATIONSHIP between inputs and outputs itself changes
  (e.g. a word that used to be neutral becomes genuinely associated with
  negative sentiment due to a cultural/contextual shift -- the SAME input
  now genuinely means something different)
```

Genuinely worth being able to distinguish these precisely — they require DIFFERENT responses: data drift might just need the model to see more diverse training examples (recap the general "more representative training data" principle), while concept drift genuinely means the model's learned patterns are becoming actively WRONG, and needs more urgent retraining.

## Statistical drift detection — recap Math Foundations' distributions directly

```python
from scipy.stats import ks_2samp      # recap Math Foundations file 14's hypothesis testing directly

def detect_feature_drift(training_data, production_data, feature_name, threshold=0.05):
    statistic, p_value = ks_2samp(training_data[feature_name], production_data[feature_name])

    if p_value < threshold:      # recap Math Foundations file 14's exact hypothesis-testing framework
        print(f"DRIFT DETECTED in {feature_name}: p-value={p_value:.4f}")
        return True
    return False
```

Genuinely worth recognizing this as a DIRECT application of Math Foundations file 14's hypothesis testing — the Kolmogorov-Smirnov test asks "are these two samples (training data's feature distribution vs. recent production data's feature distribution) drawn from the SAME underlying distribution?" A small p-value (recap that file's exact interpretation) suggests genuinely significant drift has occurred — the production data's distribution has meaningfully diverged from what the model was trained on.

## Monitoring prediction distributions — a genuinely simpler, practical proxy

```python
import pandas as pd      # recap the pandas folder directly

def monitor_prediction_distribution(predictions_log):
    prediction_counts = predictions_log['predicted_label'].value_counts(normalize=True)
    print(prediction_counts)
    # Genuinely worth watching for: has the ratio of predicted classes shifted
    # dramatically? (e.g. a spam classifier suddenly flagging 40% of emails as
    # spam, when historically it flagged 5% -- genuinely worth investigating,
    # even without directly knowing WHY)
```

Genuinely a simpler, often more PRACTICAL first-line monitoring signal than full statistical drift tests — recap MLOps file 09's own prediction-logging discussion (Flask file 09's database logging) — if the model's OUTPUT distribution shifts substantially, something is genuinely different, even before diagnosing the exact root cause.

## Confidence score monitoring — genuinely useful, easy to implement

```python
def monitor_confidence_scores(predictions_log):
    avg_confidence = predictions_log['confidence'].mean()
    low_confidence_rate = (predictions_log['confidence'] < 0.6).mean()

    if low_confidence_rate > 0.20:      # genuinely worth alerting if >20% of predictions are low-confidence
        print(f"WARNING: {low_confidence_rate*100:.1f}% of recent predictions have low confidence")
```

Genuinely worth connecting to PyTorch file 05's softmax/confidence discussion directly — a rising rate of LOW-CONFIDENCE predictions is often a genuinely early, practical warning sign that the model is encountering INPUTS it wasn't well-trained on, even before ground-truth labels are available to measure actual accuracy degradation.

## The genuinely hard problem — monitoring accuracy WITHOUT ground truth labels

```python
# Recap the general supervised-learning setup from scikit-learn folder --
# accuracy/F1 require KNOWING the true label. In production, the true label
# is often GENUINELY UNKNOWN at prediction time (that's precisely WHY the
# model is being used) -- this is a real, honest challenge, not a solved problem
```

Genuinely important, honest acknowledgment worth making directly — this is why the PROXY signals covered above (prediction distribution shifts, confidence score trends, statistical feature drift) matter so much: they're detectable WITHOUT waiting for ground truth, which might arrive much later (if at all) or never at all for some tasks.

## Delayed ground truth — when it eventually DOES arrive

```python
# Genuinely common pattern: for some tasks, the true label eventually becomes
# known (e.g. did the user actually churn, was the loan actually repaid,
# did the customer actually return the product) -- even if delayed by days/weeks
def evaluate_with_delayed_labels(predictions_log, actual_outcomes):
    merged = predictions_log.merge(actual_outcomes, on='request_id')      # recap SQL file 05/pandas' merge directly
    accuracy = (merged['predicted_label'] == merged['actual_label']).mean()
    return accuracy
```

Genuinely worth knowing this pattern exists for tasks where ground truth eventually catches up — recap Flask file 09's database logging discussion directly, storing a `request_id` with every prediction genuinely enables JOINING predictions against outcomes once they become available, even with a real time delay.

## Setting up alerts — genuinely important, closing the monitoring loop

```python
# Genuinely a real, practical pattern -- connecting monitoring checks to actual
# notifications, not just logs nobody reads
def check_and_alert(drift_detected, low_confidence_rate):
    if drift_detected or low_confidence_rate > 0.20:
        send_alert("Model performance may be degrading -- investigation needed")
        # In a real system: Slack webhook, email, PagerDuty, etc.
```

Genuinely important, practical closing point — recap file 01's "nobody noticed" framing directly: monitoring DATA (dashboards, logs) that nobody actively watches is genuinely no better than no monitoring at all. Real production systems connect monitoring checks to ACTIVE alerting, ensuring a human genuinely gets notified when something looks wrong, rather than relying on someone remembering to check a dashboard.

## Dashboards — visualizing monitoring signals over time

```python
# Genuinely worth knowing tools like Grafana (paired with a time-series database
# like Prometheus) are the STANDARD production choice for this -- recap MLOps
# file 05's W&B live-dashboard discussion, a conceptually related idea, just
# for ONGOING PRODUCTION metrics rather than TRAINING-time metrics specifically
```

## Try this

```python
# Using the Flask app's prediction-logging database (recap Flask file 09's
# exercise), simulate a period of "drift" by logging predictions on text
# samples deliberately DIFFERENT in style/vocabulary from the model's original
# training data (e.g. train on movie reviews, then "monitor" using product reviews)
# Compute the KS test (recap Math Foundations file 14 directly) comparing a
# simple text-length or vocabulary-based feature between the original training
# data and this "drifted" simulated production data
# Also compute the average confidence score on both sets and confirm it's
# genuinely lower on the drifted data -- connecting both signals (statistical
# test AND confidence monitoring) to the SAME underlying real phenomenon
```

Simulating drift deliberately and confirming BOTH detection signals (statistical test and confidence monitoring) agree is genuinely the clearest way to build real confidence that these techniques would actually catch a genuine production problem, not just pass a synthetic exercise.

## What's next
File 10 — Model Registry & Deployment Strategies, covering HOW a new, validated model version actually gets rolled out to replace the current production model SAFELY — blue-green deployments, canary releases, and shadow testing.
