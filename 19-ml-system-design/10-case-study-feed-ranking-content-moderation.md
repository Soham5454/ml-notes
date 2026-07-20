# 10 — Case Study: Feed Ranking / Content Moderation System Design

Recap file 09's closing note — covering a genuinely different challenge profile: balancing engagement optimization against safety, operating at massive content-GENERATION scale (not just consumption scale, unlike files 07-09's case studies).

## Step 1: Clarify requirements — recap file 02's framework directly

```
Prompt: "Design a social media feed ranking system"

Genuinely important clarifying questions, recap file 02 directly:
- "What's the primary goal -- time spent, or a healthier engagement
  proxy?" (recap file 02's business-vs-ML-objective distinction directly
  -- genuinely worth raising this as an EXPLICIT, important discussion
  point, not just assuming "maximize engagement" blindly)
- Scale: "How many posts are CREATED per second, versus how many feed
  requests per second?" -- genuinely a NEW consideration compared to
  files 07-09: content GENERATION scale matters here, not just
  consumption/query scale
- "Does this system need to include content moderation/safety
  filtering, or is that handled separately?" (assume: INTEGRATED, since
  it's genuinely a core part of feed ranking done responsibly)
```

## The engagement-vs-wellbeing tension — genuinely important, worth raising proactively

```
Genuinely important, real, honest discussion point worth raising
explicitly, recap Frontier GenAI file 12's responsible-AI discipline
directly: a system optimizing PURELY for engagement/time-spent
(recap Classical NLP file 10 and Frontier GenAI file 11's "metrics are
proxies" discipline directly) can genuinely optimize toward
sensationalized, divisive, or otherwise harmful content, since that
content often GENUINELY drives short-term engagement
Genuinely worth proposing a MULTI-OBJECTIVE optimization explicitly,
rather than a single, narrow engagement metric
```

## Step 2: High-level architecture — combining files 07/08's patterns with a moderation layer

```
New content posted -> CONTENT MODERATION (safety check, covered below)
  -> Indexing/Feature extraction -> [Feed request] -> Candidate
  Generation (recap file 07's two-stage pattern directly) -> Ranking
  (multi-objective, covered below) -> Feed
```

Genuinely worth recognizing the moderation step as happening BOTH at content-creation time (a genuinely important gate BEFORE content is even eligible for ranking) AND potentially again during ranking (down-weighting borderline content rather than fully blocking it).

## Step 3: Content moderation — recap Classical NLP/Hugging Face/OpenCV folders directly

```
Genuinely worth proposing a MULTI-MODAL, MULTI-STAGE moderation pipeline:
- TEXT: recap Classical NLP file 08's sentiment analysis and Hugging
  Face file 05's fine-tuned classification discussion directly --
  fine-tuned classifiers detecting hate speech, harassment, spam
- IMAGE/VIDEO: recap OpenCV folder's classical techniques and PyTorch's
  CNN discussion directly -- classifiers detecting policy-violating imagery
- Genuinely important: a FAST, cheap first-pass classifier (recap file
  06/07's two-stage-architecture pattern directly, the SAME "fast broad
  pass, then slow precise pass" idea) flags LIKELY violations for a
  SLOWER, more careful model or HUMAN REVIEW (recap Frontier GenAI file
  09's human-in-the-loop discussion directly)
```

Genuinely worth an honest, direct note about the REAL, difficult tradeoff here — recap Math Foundations file 14's Type I/Type II framing directly: moderation false positives (removing legitimate content) have a real free-speech/user-trust cost; false negatives (missing genuine violations) have a real safety cost — this is genuinely one of the most consequential precision-recall tradeoffs in this entire folder's case studies, worth discussing with real honesty and care rather than glossing over.

## Step 4: Multi-objective ranking — genuinely the core technical challenge of this case study

```python
# Genuinely worth proposing a WEIGHTED COMBINATION of multiple predicted
# probabilities, rather than a single engagement score
final_score = (
    w1 * predicted_prob_click +
    w2 * predicted_prob_meaningful_interaction +      # e.g. comment, share -- a DEEPER signal than a click
    w3 * predicted_prob_report_or_hide -               # genuinely SUBTRACTED -- a negative signal
    w4 * content_safety_risk_score                     # genuinely down-weights borderline content
)
```

Genuinely worth recognizing this as directly recapping the general "combine multiple signals with explicit, tunable weights" theme from file 08's ranking-signal discussion — the weights (`w1, w2, w3, w4`) are genuinely a real, important LEVER for encoding the platform's actual values/priorities into the system, worth discussing explicitly as a business-and-ethics decision, not purely a technical one.

## Step 5: Feature engineering — recap file 04's discussion, now with feed-specific examples

```
Genuinely worth listing concrete features: content features (recap
OpenCV/Hugging Face embedding discussion directly -- topic, media type),
creator features (recap file 04's item-feature discussion directly --
account age, historical violation rate), user-content affinity (recap
file 07's collaborative-filtering discussion directly), and RECENCY
(genuinely important for feeds specifically -- content freshness decays
in relevance quickly, unlike file 07's product recommendations)
```

## Step 6: Scale considerations — genuinely a distinctive challenge here

```
Genuinely important, real scale consideration worth raising explicitly:
CONTENT VOLUME can be genuinely enormous and GROWING continuously (recap
file 03's streaming-ingestion discussion directly) -- moderation and
feature extraction need to happen at GENUINELY high throughput,
independent of any single user's feed-request rate, a real, distinct
scaling dimension from files 07-09's case studies
```

## Step 7: Evaluation — genuinely the hardest evaluation challenge in this folder

```
Genuinely worth an honest, direct acknowledgment: engagement metrics
(recap file 02's business-vs-ML metric distinction directly) are
GENUINELY insufficient alone here -- worth proposing SUPPLEMENTARY
metrics like "reported content rate," "time-well-spent surveys" (direct
user feedback, genuinely harder to scale but genuinely more honest), and
moderation precision/recall (recap Step 3's discussion directly),
evaluated together, not any single metric in isolation
```

## The honest, complete tradeoffs to articulate

```
- Engagement vs wellbeing: genuinely the central, honest tension of this
  entire case study (recap Step 1's discussion directly)
- Moderation precision vs recall: genuinely high-stakes on BOTH sides
  (recap Step 3 directly)
- Automated moderation vs human review cost: recap Frontier GenAI file
  09's human-in-the-loop tradeoff directly, now at genuinely massive scale
```

## Try this

```
# Genuinely worth practicing this case study with EXTRA emphasis on the
# engagement-vs-wellbeing discussion -- explicitly propose a multi-
# objective scoring formula (recap Step 4's example directly) and be
# prepared to JUSTIFY the relative weights you'd assign, out loud, as if
# challenged by an interviewer asking "why not just maximize clicks?"
```

Being able to articulate WHY pure engagement-maximization is a genuinely flawed objective, and propose a concrete, reasoned alternative, is genuinely one of the most valuable, differentiating skills a system design interview can reveal.

## What's next
File 11 — Case Study: RAG/GenAI Product Design, applying the framework to a genuinely current, cutting-edge system type, pulling together the entire Frontier GenAI folder into one complete case study.
