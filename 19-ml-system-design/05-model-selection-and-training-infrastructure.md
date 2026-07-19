# 05 — Model Selection & Training Infrastructure Design

Recap file 04's closing note — covering how to choose and JUSTIFY a modeling approach, and design the infrastructure that trains it reliably at scale, directly recapping the XGBoost bridge file's original decision framework at system-design scale.

## Model selection — recap the XGBoost bridge file's ENTIRE framework, directly applied

```
Recap the XGBoost bridge file's founding decision framework, word for
word: "Structured/tabular data, need for interpretability, moderate
dataset size, limited compute -> XGBoost is genuinely often the smarter
first choice. Unstructured data, very large datasets, willing to trade
interpretability for performance -> Deep learning becomes necessary/superior"
```

Genuinely worth recognizing this exact framework as directly APPLICABLE, verbatim, to system design interview model-selection discussions — recap the recurring theme from EVERY bridge file in this roadmap: this isn't a new decision to make, it's the SAME engineering judgment already deeply internalized, now articulated in an interview context.

## Starting simple — genuinely a real, important practical principle

```
Genuinely important, real advice worth stating explicitly in an
interview: propose starting with a SIMPLE baseline (recap scikit-learn's
LogisticRegression, or XGBoost, directly) even if a more sophisticated
deep learning approach is ultimately expected -- recap Math Foundations
file 12's bias-variance tradeoff directly: a simple model is FASTER to
build, easier to DEBUG, and provides a genuine BASELINE to measure
whether added complexity is actually worth its cost
```

## Choosing between fine-tuning, RAG, and prompting — recap Frontier GenAI files 08/13 directly

```
Genuinely worth recognizing this as DIRECTLY reusing Hugging Face file 13
and Frontier GenAI file 08's exact decision framework, now for a system
design interview involving GenAI:
"Need consistent behavior/style" -> fine-tuning (recap Frontier GenAI
  file 08's LoRA/PEFT discussion)
"Need access to specific, updatable facts" -> RAG (recap Frontier GenAI
  files 06-07)
"Need flexible, rapidly-iterable behavior" -> prompting (recap Frontier
  GenAI file 02)
```

## Training infrastructure — recap Cloud file 06's SageMaker and MLOps folder directly

```
Genuinely important components to mention explicitly in a complete
answer, recap Cloud file 06 and MLOps files 02-06 directly:
- COMPUTE: CPU vs GPU (recap PyTorch file 13, Cloud file 03's GPU-
  instance discussion), single-machine vs DISTRIBUTED training (covered next)
- EXPERIMENT TRACKING (recap MLOps files 04-05's MLflow/W&B discussion directly)
- DATA VERSIONING (recap MLOps file 06's DVC discussion directly)
- REPRODUCIBILITY (recap MLOps file 01's core reproducibility theme directly)
```

## Distributed training — genuinely important for large-scale systems

```
Genuinely worth knowing this concept exists at a system-design level,
even without deep implementation: when a SINGLE machine's GPU memory or
compute is insufficient (recap PyTorch file 13's GPU-memory constraints
directly), training gets DISTRIBUTED across multiple machines/GPUs
DATA PARALLELISM: each machine holds a FULL COPY of the model, processes
  a DIFFERENT batch of data (recap PyTorch file 08's batching discussion
  directly), gradients are SYNCHRONIZED across machines
MODEL PARALLELISM: the MODEL ITSELF is split across multiple machines
  (genuinely necessary when a single model is too large to fit on ONE
  GPU/machine at all, recap Frontier GenAI file 08's QLoRA discussion of
  large-model memory constraints directly)
```

## Retraining cadence — recap MLOps file 08's automated retraining discussion directly

```
Genuinely important, real question worth raising proactively: HOW OFTEN
does this model need to be retrained? Recap MLOps file 09's drift-
detection discussion directly -- systems with FAST-CHANGING underlying
patterns (e.g. fraud detection, file 09, where fraudsters actively adapt
their techniques) genuinely need MORE FREQUENT retraining than systems
with genuinely STABLE underlying patterns
```

```
Genuinely worth explicitly proposing a retraining STRATEGY:
- SCHEDULED (recap MLOps file 08's cron-based example directly) -- e.g.
  weekly, for genuinely stable-pattern systems
- TRIGGERED by drift detection (recap MLOps file 09's monitoring
  discussion directly) -- retrain when monitoring signals genuinely
  indicate meaningful degradation
- CONTINUOUS/online learning -- genuinely worth knowing this exists:
  some systems update model weights continuously as new data arrives,
  rather than full periodic retraining -- a real, more complex approach,
  worth mentioning as an option for GENUINELY fast-changing systems, with
  an honest note about its added complexity
```

## Handling class imbalance — recap scikit-learn's imbalance discussion directly

```
Genuinely important, real consideration for MANY system design problems
(fraud detection, file 09, spam detection, rare-event prediction) --
recap scikit-learn's class_weight/SMOTE/resampling discussion directly:
worth explicitly raising HOW severe the imbalance is likely to be (e.g.
fraud is genuinely a TINY fraction of transactions) and which
mitigation technique fits
```

## Cold start problem — genuinely important, a real, common system design challenge

```
Genuinely worth knowing this concept by name, directly connecting to
file 07's recommendation-system preview: a NEW user or NEW item has NO
historical interaction data -- recap the general "the model needs SOME
signal to work with" theme from throughout this roadmap: common
mitigations include using CONTENT-based features (recap file 04's item-
feature discussion directly) instead of purely collaborative signals
for new entities, or falling back to POPULARITY-based defaults until
enough real interaction data accumulates
```

## Offline evaluation before deployment — recap scikit-learn/Frontier GenAI file 11 directly

```
Genuinely essential, worth explicitly stating as a REQUIRED step before
any deployment discussion (file 06): a proper TRAIN/VALIDATION/TEST
split (recap scikit-learn's entire discipline directly), evaluated
using the metrics defined during requirements-gathering (recap file 02
of this folder directly) -- and recap MLOps file 08's "compare against
CURRENT production model" discipline directly, not just an absolute
threshold
```

## The honest, complete model-selection narrative — a realistic example

```
Genuinely worth practicing narrating a complete, honest justification:
"Given the tabular, structured nature of this data and the need for
some interpretability for the compliance team, I'd start with XGBoost
(recap the XGBoost bridge file directly) as a baseline. If the offline
evaluation shows genuinely insufficient performance, and given we have
millions of labeled examples, I'd consider a deep learning approach
(recap PyTorch folder directly) next, accepting a real interpretability
tradeoff in exchange for potentially better raw performance."
```

## Try this

```
# Genuinely worth practicing out loud, for a search ranking system
# (preview of file 08): justify a complete model-selection narrative
# explicitly stating: why this task type (ranking, not classification),
# why this SPECIFIC model family, what the retraining cadence should be
# and why, and how the cold-start problem would be handled for new items
# entering the search index
```

Practicing the FULL narrative (not just naming a model) is genuinely the real skill — a system design interview rewards REASONED justification at every step, directly recapping this entire roadmap's honest tradeoff-communication discipline.

## What's next
File 06 — Serving Architecture & Latency Optimization, covering how a trained model actually reaches users reliably and quickly — the final piece before this folder's case studies pull everything together.
