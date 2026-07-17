# 10 — Model Registry & Deployment Strategies

Recap file 09's closing note — covering HOW a new, validated model version actually gets rolled out to replace the current production model SAFELY, without risking a broken deployment or degraded user experience.

## Recap the Model Registry — file 04's MLflow foundation, now in full context

```python
# Recap MLOps file 04 directly: the Model Registry maintains a clear record
# of every model VERSION and its current STAGE (Staging, Production, Archived)
from mlflow.tracking import MlflowClient
client = MlflowClient()

client.transition_model_version_stage(
    name="sentiment-classifier", version=5, stage="Staging"      # promote a NEW candidate to Staging first
)
```

Genuinely worth recognizing the Registry as the SOURCE OF TRUTH every deployment strategy in this file operates on — "deploying a new model" genuinely means "changing WHICH registered version is being served," a concept made precise and traceable specifically because of file 04's tracking discipline.

## The naive approach — and why it's genuinely risky

```python
# Genuinely risky, naive approach: just REPLACE the production model file directly
# If the new model is genuinely WORSE (recap file 08's evaluation gates — even
# with gates, genuinely unexpected real-world behavior can still occur), EVERY
# user is immediately affected, with no easy, fast way to limit the damage
```

Genuinely the core motivation for every strategy covered in this file — reduce the BLAST RADIUS of a bad deployment, rather than betting everything on a new model being correct on the very first try.

## Blue-Green Deployment — genuinely the simplest safe strategy

```
BLUE environment: the CURRENT, live production model, serving all real traffic
GREEN environment: the NEW model version, deployed but NOT yet receiving traffic

Once the GREEN environment is verified healthy (recap Flask file 06's /health
endpoint, and file 09's monitoring signals), traffic is SWITCHED from Blue to
Green -- ideally, genuinely INSTANTLY
```

```yaml
# Conceptually, using Docker Compose (recap MLOps file 03) with two services:
services:
  model-blue:
    image: sentiment-classifier:v4      # current production
  model-green:
    image: sentiment-classifier:v5      # new candidate

  load-balancer:
    # routes traffic to EITHER blue or green based on current configuration
```

Genuinely the key advantage: if the Green environment shows problems AFTER receiving real traffic, ROLLING BACK is genuinely instant — just switch traffic back to Blue, which was never actually taken down. The honest cost: running BOTH environments simultaneously genuinely doubles resource usage during the transition period.

## Canary Deployment — genuinely a more gradual, cautious alternative

```
Instead of switching 100% of traffic at once (blue-green), CANARY releases
route a SMALL PERCENTAGE of traffic (e.g. 5%) to the new model, monitor it
closely (recap file 09's monitoring signals directly), and GRADUALLY increase
the percentage if it continues to look healthy
```

```python
import random

def route_request(user_id):
    if random.random() < 0.05:      # genuinely 5% of traffic
        return "model-v5"      # the new candidate
    return "model-v4"      # the current, trusted production model
```

Genuinely worth understanding WHY this is often preferred over blue-green for HIGH-STAKES deployments: a bug affecting only 5% of traffic is genuinely far less damaging than one affecting 100% instantly — canary deployments trade a SLOWER rollout for a genuinely SMALLER blast radius if something goes wrong, directly connecting to file 09's monitoring signals being watched closely during the gradually-increasing rollout.

## Shadow Deployment — recap file 08's brief preview, now fully explained

```python
@app.route('/api/v1/predict', methods=['POST'])
def predict():
    data = request.get_json()

    result = current_model.predict(data['text'])      # genuinely the ONLY result actually RETURNED to the user

    # Shadow: run the new model TOO, but genuinely discard its result for now,
    # just LOG it for comparison (recap Flask file 09's prediction logging)
    shadow_result = new_model.predict(data['text'])
    log_shadow_comparison(data['text'], result, shadow_result)

    return jsonify(result)
```

Genuinely the SAFEST possible strategy — the new model runs on REAL production traffic, but its output NEVER actually affects what the user sees, purely used for OFFLINE comparison against the current model's real-world behavior. Genuinely more expensive computationally (running TWO models on every request), but genuinely eliminates ALL risk to the user experience during evaluation — a real, honest tradeoff between cost and safety.

## A/B Testing — genuinely related, but with a DIFFERENT goal than the above strategies

```python
# Genuinely important distinction: canary/blue-green/shadow are about SAFELY
# rolling out a model believed to be an improvement
# A/B testing is about MEASURING whether one genuinely IS better, often when
# the answer isn't obvious from offline evaluation alone (recap file 08's
# evaluation-gate discussion -- sometimes REAL user behavior reveals things
# offline metrics genuinely can't capture)
def assign_ab_group(user_id):
    return "A" if hash(user_id) % 2 == 0 else "B"      # genuinely consistent per-user assignment
```

Genuinely worth being precise: A/B testing typically runs LONGER (weeks, not hours) and specifically measures a BUSINESS metric (click-through rate, conversion, user satisfaction) — recap Math Foundations file 14's hypothesis testing directly, the SAME statistical framework (comparing two groups, computing a p-value) genuinely applies here to determine if the observed difference is statistically significant or just noise.

## Rollback strategy — genuinely essential, and directly enabled by file 04's Registry

```python
client.transition_model_version_stage(
    name="sentiment-classifier", version=4, stage="Production"      # genuinely instant rollback to the
                                                                          # PREVIOUS known-good version
)
```

Genuinely worth emphasizing directly: EVERY deployment strategy in this file needs a clear, FAST rollback plan — and this is PRECISELY why file 04's Model Registry and file 06's DVC versioning matter so much. Without clean versioning, "roll back to the previous model" can become a genuinely slow, error-prone manual process instead of a single, confident command.

## The honest decision framework — which strategy for which situation

```
Low-stakes, well-tested change, confident in quality        -> Blue-Green (fast, simple)
High-stakes, uncertain about real-world behavior               -> Canary (gradual, limits blast radius)
GENUINELY unsure if new model is even an improvement              -> Shadow deployment first, THEN canary/blue-green
Need to prove a business metric improvement, not just model metrics -> A/B testing
```

Genuinely the same honest engineering-tradeoff framework echoed throughout this entire roadmap — no single strategy is universally correct; the right choice depends on genuine confidence level, the cost of a mistake, and how measurable "success" actually is for the specific model/task.

## Try this

```python
# Using the Flask app with MLflow model registry integration (recap files 04
# and 09's exercises), implement a SIMPLE canary routing function: 90% of
# requests go to the "Production" stage model, 10% go to the "Staging" stage
# model, with BOTH predictions logged to the database (recap Flask file 09)
# along with WHICH model served each request
# After simulating a batch of requests, query the logged data to compare
# the confidence score distributions between the two models' served
# predictions -- write a short, honest assessment of whether the Staging
# model looks ready for full promotion based on this data
```

Writing an honest assessment based on real (even if simulated) comparison data is genuinely the realistic decision-making exercise this whole file has been building toward — deployment decisions are genuinely judgment calls informed by data, not automatic pass/fail checks alone.

## What's next
File 11 — Orchestration & Kubernetes Intro, extending Docker Compose (file 03) into the industry-standard tool for running and scaling containerized applications (including everything covered across this whole folder) at genuine production scale.
