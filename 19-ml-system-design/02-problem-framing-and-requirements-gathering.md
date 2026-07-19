# 02 — Problem Framing & Requirements Gathering

Recap file 01's closing note — going deep into the genuinely most important step of the whole framework, directly recapping DSA file 19's "understand the problem completely" discipline at system-design scale.

## Business objective vs ML objective — genuinely important, easy to conflate

```
Recap Classical NLP file 10 and Frontier GenAI file 11's "metrics are
proxies" discipline directly -- a business wants "increase user
engagement," but the ML SYSTEM optimizes something more concrete, like
"predict click-through probability" -- these are GENUINELY related but
DISTINCT, and conflating them is a real, common mistake
```

```
Genuinely worth asking directly, every time: "What business metric does
this system ultimately need to improve, and what's the PROXY ML metric
we'll actually optimize for?"
Example: Business goal = "reduce customer churn." ML objective = "predict
probability a customer churns in the next 30 days." These are RELATED
but the ML metric (recap scikit-learn's AUC/precision-recall discussion
directly) is a genuinely imperfect stand-in for the real business outcome
```

## Framing the problem as an ML TASK TYPE — recap the entire roadmap's task vocabulary directly

```
Genuinely worth explicitly categorizing the problem using vocabulary from
throughout this roadmap:
- Binary/multi-class classification (recap scikit-learn folder directly)
- Regression (recap scikit-learn/Math Foundations file 16 directly)
- Ranking (recap file 08 of THIS folder -- genuinely a DISTINCT task
  type from classification, worth knowing the difference)
- Recommendation (recap file 07 of THIS folder)
- Generation (recap Hugging Face/Frontier GenAI folders directly)
- Anomaly detection (recap file 09 of THIS folder -- fraud detection)
```

Genuinely important, real skill worth building — correctly identifying WHICH task type a business problem maps to determines the entire rest of the design (recap the general "pick the right tool for the actual problem" theme from every bridge file in this roadmap) — misidentifying "this is actually a RANKING problem, not classification" early leads to a genuinely weaker design later.

## Clarifying SCALE — recap DSA file 01's Big O discipline, now at a real, practical system level

```
Genuinely essential questions, always worth asking explicitly:
- How many USERS? (thousands, millions, billions -- recap Cloud file 01's
  elasticity discussion directly, this genuinely changes everything)
- How many REQUESTS per second, at PEAK? (recap Cloud file 09's load-
  balancer discussion directly)
- How much DATA exists for training? (recap the general "does this need
  a simple model or a complex one" framing from scikit-learn/XGBoost's
  bias-variance discussions)
- How does scale GROW over time? (recap Cloud file 01's auto-scaling
  discussion directly -- design for CURRENT scale, but discuss how it
  would evolve)
```

## Clarifying LATENCY requirements — recap Cloud file 03's compute-choice discussion directly

```
Genuinely important distinction, recap Cloud file 03's EC2-vs-Lambda
discussion and MLOps file 01's lifecycle directly:
- REAL-TIME/online serving: genuinely needs predictions in milliseconds
  (e.g. ad click prediction, fraud detection at transaction time) --
  recap Flask file 06 and Cloud file 06's SageMaker endpoint discussion directly
- BATCH/offline serving: genuinely can tolerate minutes/hours (e.g.
  nightly recommendation refresh, weekly churn scoring) -- recap MLOps
  file 07's scheduled-workflow discussion directly
```

Genuinely worth recognizing this distinction as determining ENTIRE architectural choices — a real-time fraud system (file 09) genuinely needs a fundamentally different serving architecture (file 06 of this folder) than a batch-computed weekly recommendation refresh (file 07).

## Clarifying the DATA landscape — recap MLOps file 08's data-validation discipline directly

```
Genuinely important questions: What LABELED data exists already? (Recap
the entire supervised-learning framing from scikit-learn folder directly
-- no labels genuinely changes the whole approach, potentially toward
unsupervised techniques, recap Classical NLP file 09's LDA discussion,
or toward a COLD START strategy, covered in file 07)
What's the LABEL DELAY? (Recap MLOps file 09's delayed-ground-truth
discussion directly -- fraud labels might arrive weeks later via
chargebacks, genuinely affecting how quickly a model can be validated/improved)
```

## Defining SUCCESS metrics — recap scikit-learn's entire metrics folder directly

```
Genuinely essential, worth being SPECIFIC: not just "accuracy," but
WHICH metric fits this SPECIFIC problem's honest tradeoffs (recap
scikit-learn's precision/recall/F1 discussion, and Math Foundations
file 14's Type I/Type II error framing, directly) -- a fraud detection
system (file 09) genuinely cares differently about false positives
(blocking a legitimate transaction) vs false negatives (missing real
fraud) than a spam filter does
```

```
Genuinely important, TWO kinds of metrics to define explicitly:
OFFLINE metrics: computed on held-out test data (recap scikit-learn's
  train/test discipline directly) BEFORE deployment
ONLINE metrics: measured on REAL, live traffic AFTER deployment (recap
  MLOps file 10's A/B testing/canary discussion directly) -- genuinely
  the metric that ACTUALLY matters, since offline metrics are a proxy
  (recap Frontier GenAI file 11's evaluation-honesty discipline directly)
  for real-world performance
```

## Constraints — genuinely important, often-overlooked

```
Genuinely worth asking: budget constraints? (Recap Cloud file 10's cost-
management discipline directly) Compliance/privacy constraints? (Recap
Frontier GenAI file 12's privacy discussion and Cloud file 09's data-
residency discussion directly -- e.g. GDPR requirements might mandate
EU user data stays within EU infrastructure) Interpretability
requirements? (Recap the XGBoost bridge file's classical-vs-deep-learning
interpretability tradeoff directly -- some regulated industries genuinely
REQUIRE explainable models, ruling out certain approaches regardless of
raw accuracy)
```

## A genuinely complete example — walking through the full requirements-gathering process

```
Prompt: "Design a system to detect fraudulent transactions"

Genuinely good clarifying questions to ask, in order:
1. "What's the transaction volume -- thousands or millions per day?"
2. "Does this need real-time blocking, or can flagged transactions be
   reviewed within a delay?" (recap this file's latency discussion directly)
3. "What labeled fraud data exists? Are labels genuinely delayed (recap
   MLOps file 09) via chargebacks/disputes?"
4. "What's the acceptable false-positive rate? (Recap Math Foundations
   file 14's Type I error discussion directly -- blocking legitimate
   transactions has a real, direct cost to the business)"
5. "Any regulatory/compliance constraints on the data or model
   interpretability?" (Recap this file's constraints discussion directly)
```

Genuinely worth recognizing this exact question sequence as the realistic, practical shape of the first 5-10 minutes of a genuinely well-conducted system design interview — file 09 of this folder will build the FULL fraud detection system design on top of exactly these clarified requirements.

## Try this

```
# Genuinely worth practicing this as a standalone, timed exercise:
# Take 3 different one-line prompts ("design a recommendation system,"
# "design a search ranking system," "design a content moderation system")
# and for EACH, write out (or say out loud, recap file 01's practice
# discipline directly) the 5-8 genuinely most important clarifying
# questions you'd ask BEFORE proposing any architecture at all
```

Practicing the QUESTIONS (not yet the answers/architecture) as a standalone skill is genuinely valuable — recap this file's core lesson directly: the quality of the clarifying questions asked is itself a real, evaluated signal in a system design interview, independent of the eventual architecture proposed.

## What's next
File 03 — Data Pipeline Design, covering how data actually gets collected, stored, and processed at scale — the foundation every subsequent design decision builds on.
