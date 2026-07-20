# 08 — Case Study: Search Ranking System Design

Recap file 07's closing note — applying the same complete framework to a genuinely different task type: ranking, with its own distinct considerations from recommendation.

## Step 1: Clarify requirements — recap file 02's framework directly

```
Prompt: "Design a search ranking system for an e-commerce platform"

Genuinely important clarifying questions, recap file 02 directly:
- "Is this searching PRODUCTS specifically, or general content?" (assume products)
- Latency: "Search is genuinely ALWAYS real-time -- users expect
  immediate results" (recap file 06's online-serving discussion directly,
  genuinely no batch option here, unlike file 07's homepage recommendations)
- Scale: "How many searchable products, and queries per second at peak?"
- Success metric: "Click-through rate on results, and ultimately purchase
  conversion from search" (recap file 02's business-vs-ML-metric distinction)
```

## Recommendation vs Search Ranking — genuinely an important, distinct framing

```
Genuinely important distinction worth stating explicitly, recap file 02's
task-type-identification discipline directly: recommendation (file 07)
ranks items for a user WITHOUT an explicit query -- search ranking ranks
items GIVEN an explicit user query. This changes the core signal: query-
item RELEVANCE (does this product genuinely match "red running shoes")
becomes the PRIMARY signal, with personalization as a SECONDARY refinement
```

## Step 2: High-level architecture — recap file 07's two-stage pattern, now for search

```
Query -> Query Understanding/Processing -> Retrieval (fast, broad) ->
  Ranking (slower, precise) -> Results
```

Genuinely worth recognizing this as directly recapping file 07's candidate-generation + ranking two-stage architecture — the SAME underlying pattern (recap Frontier GenAI file 07's retrieve-then-rerank discussion too, a third appearance of this exact idea), now with RETRIEVAL matching against a query instead of CANDIDATE GENERATION matching against a user's history.

## Step 3: Query understanding — recap Classical NLP folder directly

```
Genuinely worth recapping the ENTIRE Classical NLP folder's preprocessing
toolkit directly: tokenization, stemming/lemmatization (recap that
folder's files 01-02), spell correction, and QUERY EXPANSION (recognizing
"sneakers" and "running shoes" as related, recap Hugging Face file 06's
embedding-similarity discussion directly)
```

Genuinely worth explicitly mentioning query INTENT classification too — recap scikit-learn/Hugging Face's classification discipline directly: is this a NAVIGATIONAL query (looking for a specific known product) or an EXPLORATORY query (browsing a category) — genuinely affects which ranking signals should matter most.

## Step 4: Retrieval — recap Classical NLP file 04 and Frontier GenAI file 05/07 directly

```
Genuinely worth proposing HYBRID retrieval explicitly, recap Frontier
GenAI file 07's hybrid-search discussion directly:
1. KEYWORD/lexical matching (recap Classical NLP file 04's TF-IDF/BM25
   discussion directly) -- genuinely essential for exact product names,
   SKUs, brand names
2. SEMANTIC/embedding-based matching (recap Frontier GenAI file 05
   directly) -- genuinely handles synonyms/related concepts ("sneakers"
   matching "athletic shoes") that pure keyword matching misses
```

## Step 5: Ranking — genuinely the core, most nuanced part of this case study

```
Genuinely important signals to combine explicitly, recap file 04's
feature-categorization discipline directly:
- TEXT RELEVANCE (query-product text match, recap Classical NLP folder)
- PRODUCT quality signals (recap file 04's item-feature discussion --
  ratings, review count, historical conversion rate)
- PERSONALIZATION (recap file 07's user-feature discussion directly --
  this user's past purchase history/preferences)
- BUSINESS signals (recap the general "business objective" framing from
  file 02 directly -- inventory levels, margin, sponsored placements)
```

Genuinely worth an honest, direct note about LEARNING TO RANK — recap scikit-learn's classification discipline directly, but genuinely a DISTINCT ML formulation: rather than predicting a single label/score per item independently, learning-to-rank objectives (pairwise or listwise approaches) directly optimize for the CORRECT RELATIVE ORDERING of a list of results — genuinely worth knowing this distinct objective exists, since naive independent classification/regression per item can produce a technically accurate but poorly-ORDERED final list.

## Step 6: Training data — genuinely a real, subtle challenge for search specifically

```
Genuinely important, real consideration: USER CLICKS on search results
are a genuinely BIASED signal -- recap Math Foundations file 14's
hypothesis-testing/bias-awareness discipline directly: users
disproportionately click on TOP-RANKED results regardless of TRUE
relevance (a well-known real phenomenon called "position bias") --
worth explicitly proposing techniques to correct for this (e.g.
randomizing result order occasionally to collect unbiased training signal)
rather than naively training on raw click data
```

## Step 7: Evaluation — recap file 07's ranking metrics discussion, now expanded

```
Genuinely worth introducing NDCG (Normalized Discounted Cumulative Gain)
explicitly -- a genuinely standard ranking metric that rewards relevant
results appearing HIGHER in the list MORE than the same relevant result
appearing lower, recap DSA file 08's priority/ranking discussion
directly for the underlying "position matters" intuition
ONLINE: A/B testing (recap MLOps file 10 directly) on actual click-
  through rate and conversion, same discipline as file 07
```

## Step 8: Scale and reliability — recap file 06/Cloud folder directly

```
Genuinely worth discussing INDEX FRESHNESS explicitly: how quickly do
NEW products become searchable? (Recap file 03's streaming-vs-batch
ingestion discussion directly -- a genuinely real, practical
consideration for e-commerce specifically, where new inventory needs to
be discoverable quickly)
```

## The honest, complete tradeoffs to articulate

```
- Keyword vs semantic retrieval: precision on exact matches vs recall
  on related concepts (recap Frontier GenAI file 07's hybrid-search
  tradeoff discussion directly)
- Relevance vs personalization vs business signals: genuinely competing
  objectives requiring an honest, explicit weighting decision
- Index freshness vs system complexity: recap file 03's batch-vs-
  streaming tradeoff directly, now applied to search index updates
```

## Try this

```
# Genuinely worth practicing this complete case study out loud, in under
# 45 minutes, explicitly contrasting it against file 07's recommendation
# case study -- write down 3-4 SPECIFIC ways this design differs from
# the recommendation system (not just "it's for search instead"), e.g.
# the query-understanding step, the position-bias correction, NDCG vs
# Precision@K
```

Explicitly articulating the DIFFERENCES from the previous case study is genuinely valuable practice — recognizing which parts of a system design GENERALIZE across problem types versus which parts are genuinely problem-SPECIFIC is a real, transferable interview skill.

## What's next
File 09 — Case Study: Fraud Detection System Design, applying the framework to a genuinely different challenge profile: extreme class imbalance, real-time requirements, and adversarial, adapting behavior.
