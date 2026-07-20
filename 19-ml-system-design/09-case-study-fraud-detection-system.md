# 09 — Case Study: Fraud Detection System Design

Recap file 08's closing note — applying the framework to a genuinely different challenge profile: extreme class imbalance, hard real-time requirements, and adversarial, ADAPTING behavior, directly closing the loop on file 02's original worked example.

## Step 1: Clarify requirements — recap file 02's EXACT worked example directly

```
Recap file 02's fraud-detection clarifying questions directly, now
answered and built upon:
- Real-time BLOCKING required (recap file 02/06's latency discussion --
  genuinely the strictest latency requirement of any case study in this
  folder, since a legitimate transaction being delayed is a direct,
  real user-experience cost)
- Labels are GENUINELY DELAYED (recap MLOps file 09's delayed-ground-
  truth discussion directly -- fraud confirmed via chargebacks, often
  weeks later)
- Class imbalance is GENUINELY EXTREME (recap scikit-learn/file 05's
  imbalance discussion directly -- fraud is often <1% of transactions)
- False positives have a REAL, direct business cost (blocking legitimate
  customers, recap Math Foundations file 14's Type I/Type II framing directly)
```

## Step 2: High-level architecture — recap file 03's streaming-ingestion discussion directly

```
Transaction request -> Real-time feature retrieval (recap file 04's
  online feature store directly) -> Model inference (recap file 06's
  online serving, STRICT latency budget) -> Decision (allow/block/flag
  for review) -> Async: log for delayed-label collection (recap MLOps
  file 09's prediction-logging discussion directly)
```

## Step 3: The adversarial, adapting nature of fraud — genuinely the most distinctive aspect of this case study

```
Genuinely important, real insight worth stating explicitly: UNLIKE
recommendation (file 07) or search (file 08), fraud detection faces an
genuinely ADVERSARIAL opponent who ACTIVELY ADAPTS to evade the current
model -- recap MLOps file 09's drift-detection discussion directly, but
here the "drift" is GENUINELY intentional and adversarial, not incidental
```

Genuinely worth this insight directly shaping several downstream design decisions: MORE FREQUENT retraining (recap file 05's retraining-cadence discussion directly) is genuinely justified here specifically because of this adversarial adaptation, more so than in file 07/08's case studies.

## Step 4: Handling extreme class imbalance — recap scikit-learn's ENTIRE discussion directly

```
Genuinely worth proposing MULTIPLE techniques explicitly, recap
scikit-learn folder directly: class_weight adjustments, SMOTE/resampling,
and choosing evaluation metrics that GENUINELY reflect the imbalance
(recap scikit-learn's precision-recall curve discussion directly --
accuracy alone is GENUINELY meaningless here, since predicting "not
fraud" for every single transaction would achieve >99% accuracy while
being completely useless)
```

## Step 5: Feature engineering — recap file 04's ENTIRE discussion, now with concrete fraud-specific examples

```
Genuinely worth listing CONCRETE, real features across file 04's
categories:
- REAL-TIME: transaction amount vs this user's typical amount, time
  since last transaction, is this a NEW device/location (recap file 04's
  real-time-feature discussion directly, computed at request time)
- BATCH: user's historical fraud-report count, account age, average
  transaction frequency (recap file 04's batch-feature discussion directly)
- GRAPH-based features -- genuinely worth knowing this exists: does this
  transaction share a device/IP/card with KNOWN fraudulent accounts
  (recap DSA file 09's graph/connected-components discussion directly --
  genuinely a real, powerful technique: modeling users/devices/cards as
  a GRAPH and detecting suspicious CONNECTED COMPONENTS or clusters)
```

## Step 6: Model selection — recap the XGBoost bridge file's decision framework directly

```
Genuinely worth an honest, direct justification: "Given this is
structured, tabular transaction data needing FAST inference (recap file
06's latency-budget discussion directly) and some interpretability for
compliance/dispute-resolution purposes (recap the XGBoost bridge file's
interpretability discussion directly), XGBoost or a similar gradient-
boosted model is genuinely a strong, standard choice for this specific
problem profile"
```

## Step 7: The three-tier decision system — genuinely important, a real, common pattern

```
Genuinely worth proposing this explicitly, recap Math Foundations file
14's Type I/Type II tradeoff directly: rather than a binary allow/block
decision, propose THREE tiers based on the model's confidence score:
- HIGH confidence fraud -> BLOCK automatically
- LOW confidence (clearly legitimate) -> ALLOW automatically
- AMBIGUOUS/medium confidence -> FLAG for human review (recap Frontier
  GenAI file 09's human-in-the-loop discussion directly, the SAME
  underlying "reduce blast radius of an uncertain automated decision" principle)
```

Genuinely worth recognizing this three-tier pattern as directly recapping MLOps file 10's "reduce blast radius" deployment philosophy, now applied to individual PREDICTION decisions rather than model deployment rollouts — the SAME underlying risk-management principle, appearing at a different layer of the system.

## Step 8: Handling delayed labels — recap MLOps file 09's delayed-ground-truth discussion directly

```
Genuinely important, real challenge worth addressing explicitly: since
TRUE fraud labels arrive weeks later (via chargebacks), the model's
REAL performance can't be immediately validated after deployment --
recap MLOps file 09's "delayed labels eventually catch up" pattern
directly: propose a system that JOINS predictions against eventual
outcomes once available (recap that file's exact join-based approach)
to continuously validate and retrain
```

## Step 9: Evaluation — recap scikit-learn and MLOps file 08's model-quality-gate discussion directly

```
OFFLINE: Precision-Recall AUC (genuinely more meaningful than ROC-AUC
  under extreme imbalance, recap scikit-learn's exact discussion of this
  distinction directly), evaluated against a GENUINELY held-out, time-
  based split (recap SQL file 13's time-based-split discussion directly,
  avoiding the adversarial-adaptation leakage of a random split)
ONLINE: false positive RATE on real, live legitimate transactions
  (genuinely a critical business metric), and eventual fraud-catch rate
  once delayed labels arrive
```

## The honest, complete tradeoffs to articulate

```
- Precision vs recall: recap Math Foundations file 14's Type I/Type II
  framing directly, an EXPLICIT, honest business tradeoff, not a purely
  technical one
- Automation vs human review: recap Step 7's three-tier system directly
  -- full automation is faster/cheaper but riskier at the margins
- Retraining frequency vs stability: recap file 05's retraining-cadence
  discussion directly, MORE frequent retraining fights adversarial
  adaptation but risks model instability if done carelessly
```

## Try this

```
# Genuinely worth practicing this complete case study out loud, then
# explicitly comparing it against files 07 and 08's case studies --
# write down which ELEMENTS of the framework (files 01-06) appeared in
# ALL THREE case studies unchanged, and which elements were GENUINELY
# reshaped by this problem's distinct challenges (extreme imbalance,
# adversarial adaptation, delayed labels)
```

This cross-case-study comparison exercise is genuinely valuable — building the real, transferable skill of recognizing which parts of the framework are UNIVERSAL versus which require genuine, problem-specific adaptation.

## What's next
File 10 — Case Study: Feed Ranking / Content Moderation System Design, covering a genuinely different challenge profile again: balancing engagement optimization against safety, and operating at massive content-generation scale.
