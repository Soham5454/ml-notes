# 04 — Feature Engineering & Feature Stores at Scale

Recap file 03's closing note — covering how raw data becomes the actual SIGNALS a model consumes, and the infrastructure that serves those signals reliably. Genuinely connects directly back to the pandas/scikit-learn folders' feature engineering discussions, now at production scale.

## Feature categories — genuinely important vocabulary for system design discussions

```
Genuinely worth categorizing features explicitly, recap the pandas/
scikit-learn folders' feature engineering discussions directly:
- USER features (demographics, historical behavior aggregates)
- ITEM features (recap file 07's recommendation-system preview directly --
  product/content attributes)
- CONTEXTUAL features (time of day, device type, genuinely often
  computed AT REQUEST TIME)
- INTERACTION features (user-item combinations, recap file 07 directly)
```

## Batch features vs Real-time features — recap file 03's preview, now fully treated

```python
# BATCH feature: computed periodically (recap MLOps file 07's scheduled-
# workflow discussion directly), e.g. nightly
def compute_user_avg_purchase_30d(user_id):
    # Recap SQL file 07's window functions / file 04's aggregation discussion directly
    return db.query(f"SELECT AVG(amount) FROM transactions WHERE user_id={user_id} AND date > NOW() - INTERVAL 30 DAY")

# REAL-TIME feature: computed AT request time, using the CURRENT request's data
def compute_is_first_transaction_today(user_id, current_timestamp):
    # Genuinely needs a FAST lookup (recap Cloud file 05's DynamoDB
    # low-latency discussion directly) -- can't afford a slow batch query
    return not has_transaction_today(user_id)
```

Genuinely important, real tradeoff worth articulating explicitly — BATCH features can be RICHER/more complex (recap SQL folder's full aggregation/window-function toolkit, computed with time to spare) but are genuinely STALE by the time they're used (computed last night, used today). REAL-TIME features are genuinely FRESH but must be computed FAST (recap Cloud file 03's latency discussion directly), constraining their complexity.

## The training-serving skew problem — genuinely one of the most important, real ML systems concerns

```
Genuinely THE most commonly cited real-world ML systems failure mode,
worth understanding deeply: a feature computed ONE WAY during TRAINING
(e.g. using a full SQL query with complete historical context, recap SQL
folder directly) but computed a SLIGHTLY DIFFERENT way during SERVING
(e.g. an approximated, faster real-time version) -- the model was
TRAINED on one distribution of feature values but SERVED a subtly
different one, genuinely degrading real-world performance in a way
that's hard to detect without specifically checking for it
```

Genuinely worth connecting this directly to MLOps file 09's drift-detection discipline — training-serving skew is genuinely a DISTINCT failure mode from the DATA drift covered there (the underlying data hasn't necessarily changed, but the FEATURE COMPUTATION LOGIC differs between training and serving) — this is PRECISELY the core problem Feature Stores (covered next) are built to solve.

## Feature Stores — genuinely the standard solution to training-serving skew

```
A FEATURE STORE is a centralized system that computes features ONCE and
serves them CONSISTENTLY to BOTH training and serving pipelines -- genuinely
the same underlying computation logic, used in both contexts, eliminating
the skew problem by CONSTRUCTION rather than by careful manual discipline alone
```

```python
# Conceptually, using a feature store (e.g. Feast, or a cloud-native
# equivalent like SageMaker Feature Store, recap Cloud file 06 directly)
feature_store.get_online_features(      # genuinely FAST lookup, for serving (recap Cloud file 05's
    entity_rows=[{"user_id": 123}],         # DynamoDB-speed discussion directly)
    features=["user_avg_purchase_30d", "user_days_since_signup"]
)

feature_store.get_historical_features(      # genuinely used for TRAINING, pulling the
    entity_df=training_labels_df,               # SAME features but for a historical, labeled dataset
    features=["user_avg_purchase_30d", "user_days_since_signup"]
)
```

Genuinely worth understanding the ARCHITECTURE behind this — a feature store typically maintains BOTH an OFFLINE store (recap Cloud file 07's BigQuery/data warehouse discussion directly, optimized for large-scale historical queries during training) and an ONLINE store (recap Cloud file 05's DynamoDB/low-latency discussion directly, optimized for fast single-entity lookups during serving), with the SAME feature-computation DEFINITIONS feeding both — genuinely the direct architectural solution to the training-serving skew problem.

## Point-in-time correctness — genuinely a subtle, important concept

```
Genuinely a real, subtle bug worth knowing about explicitly: when
building a TRAINING dataset from historical data, a feature must be
computed using ONLY information that would have been ACTUALLY AVAILABLE
at that historical point in time -- recap SQL file 13's ML train/test-
split-by-date discussion directly (the SAME "avoid data leakage from the
future" principle), now applied specifically to FEATURE computation
timing, not just the train/test split itself
```

Genuinely worth a concrete example — computing "user's average purchase over ALL history" for a training example from 6 months ago, but ACCIDENTALLY including purchases from the last 6 months (which hadn't happened yet at that point in time) is genuinely a real, damaging form of data leakage, directly recapping the general "don't let future information leak into training" discipline from scikit-learn/XGBoost/PyTorch folders, now at the individual FEATURE level.

## Feature versioning — recap MLOps file 06's DVC/reproducibility discussion directly

```
Genuinely worth explicitly mentioning: feature DEFINITIONS themselves
change over time (a "user_engagement_score" formula gets refined) --
recap MLOps file 06's entire data/model versioning discipline directly,
now applied to FEATURE DEFINITIONS -- a genuinely mature system tracks
WHICH VERSION of each feature's computation logic was used for any
given trained model, for the SAME reproducibility reasons covered
extensively in that whole MLOps file
```

## Feature selection at scale — recap scikit-learn's feature importance discussion directly

```
Genuinely important, practical consideration: at real scale, hundreds or
thousands of CANDIDATE features might exist -- recap XGBoost file 07's
feature importance discussion and Math Foundations file 12's bias-
variance tradeoff directly: MORE features isn't automatically better --
genuinely worth discussing feature SELECTION/importance analysis as part
of a complete system design answer, not just "we compute lots of features"
```

## Embeddings as features — recap the ENTIRE Hugging Face/Frontier GenAI folders directly

```
Genuinely worth connecting directly: for recommendation systems (file 07)
and search ranking (file 08) specifically, LEARNED EMBEDDINGS (recap
Hugging Face file 06's contextual embeddings, and Frontier GenAI file 05's
vector-database discussion directly) are often genuinely powerful
features themselves -- a user's embedding, an item's embedding, and
their SIMILARITY (recap Math Foundations file 01's cosine similarity,
appearing YET AGAIN) are frequently among the most predictive features
in real, modern recommendation/ranking systems
```

## Try this

```
# Genuinely worth practicing out loud: for a fraud detection system
# (preview of file 09), list 8-10 CONCRETE features across the categories
# from this file (user, item/transaction, contextual, interaction)
# For EACH feature, explicitly state whether it's BATCH or REAL-TIME
# computed, and explain WHY that choice makes sense given the feature's
# nature and the system's latency requirements (recap file 02's
# requirements-gathering discussion directly)
```

Explicitly justifying batch-vs-real-time for EACH feature (not just listing features) is genuinely the skill being tested here — a real system design interview rewards this kind of specific, reasoned tradeoff articulation over a generic feature list.

## What's next
File 05 — Model Selection & Training Infrastructure Design, covering how to choose and justify a modeling approach, and design the infrastructure that trains it reliably at scale.
