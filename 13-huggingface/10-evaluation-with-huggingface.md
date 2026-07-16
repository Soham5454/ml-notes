# 10 — Evaluation with Hugging Face

Recap the `evaluate` library used casually across files 05, 06, and 09 — this file formalizes it properly and covers the broader landscape of metrics across different task types, tying together scikit-learn's metrics (recap that folder) and Classical NLP file 10's generation metrics into one unified interface.

## The `evaluate` library — a standardized interface

```python
import evaluate

accuracy = evaluate.load("accuracy")
f1 = evaluate.load("f1")
precision = evaluate.load("precision")
recall = evaluate.load("recall")

predictions = [1, 0, 1, 1, 0]
references = [1, 0, 0, 1, 0]

print(accuracy.compute(predictions=predictions, references=references))
print(f1.compute(predictions=predictions, references=references))
```

Genuinely the exact same metrics from scikit-learn's classification report (recap that folder), just accessed through Hugging Face's standardized `evaluate.load()` interface — worth recognizing this as a CONVENIENCE/CONSISTENCY layer, not a fundamentally different set of metrics. The real value: every metric, from simple accuracy to complex ones like `seqeval` (recap file 06) or `rouge` (recap file 09), uses the SAME `.compute(predictions=..., references=...)` calling pattern.

## Combining multiple metrics

```python
from evaluate import combine

clf_metrics = combine(["accuracy", "f1", "precision", "recall"])
results = clf_metrics.compute(predictions=predictions, references=references)
print(results)   # all four metrics computed together, one call
```

Genuinely useful for the `compute_metrics` function pattern used repeatedly across files 05-07's `Trainer` setup — computing several metrics at once without redundant separate calls.

## Metrics by task type — the honest, complete map

```python
# Classification (file 05): accuracy, f1, precision, recall -- recap scikit-learn directly
# Token Classification / NER (file 06): seqeval -- entity-level precision/recall/F1
# Question Answering (file 07): squad metric -- exact match + token-level F1 on the answer span
# Generation / Summarization / Translation (files 08-09): rouge, bleu, meteor
# Perplexity (Classical NLP file 05's preview, fully explained in Classical NLP file 10): 
#     genuinely used for evaluating raw language modeling quality, less common as a
#     FINE-TUNING metric since it doesn't directly measure task performance
```

## The SQuAD metric — exact match, a genuinely stricter companion to F1

```python
squad_metric = evaluate.load("squad")

predictions = [{"id": "1", "prediction_text": "330 meters"}]
references = [{"id": "1", "answers": {"text": ["330 meters"], "answer_start": [78]}}]

results = squad_metric.compute(predictions=predictions, references=references)
print(results)   # {'exact_match': 100.0, 'f1': 100.0}
```

Recap file 07's QA task directly — `exact_match` is genuinely a STRICT binary check (does the predicted span match a reference EXACTLY, character for character), while `f1` (token-overlap based, recap scikit-learn's F1 formula applied at the token level) gives PARTIAL credit for close-but-not-perfect answers — e.g. predicting "330 meters tall" instead of "330 meters" would score 0 on exact match but still reasonably high on F1, since most of the tokens genuinely overlap.

## BLEU and ROUGE via `evaluate` — recap Classical NLP file 10 and file 09

```python
bleu = evaluate.load("bleu")
rouge = evaluate.load("rouge")

predictions = ["the cat sat on the mat"]
references = [["the cat is sitting on the mat"]]

print(bleu.compute(predictions=predictions, references=references))
print(rouge.compute(predictions=predictions, references=references))
```

Genuinely the SAME metrics, same honest limitations flagged back in Classical NLP file 10 (n-gram overlap doesn't capture semantic meaning) — worth carrying that skepticism forward rather than assuming a standardized library interface somehow fixes a metric's fundamental conceptual limitations.

## BERTScore — a genuinely more semantic alternative

```python
bertscore = evaluate.load("bertscore")
results = bertscore.compute(predictions=predictions, references=references, lang="en")
print(results)   # {'precision': [...], 'recall': [...], 'f1': [...]}
```

Recap Classical NLP file 10's honest note about BLEU's semantic-blindness, and file 06's fixed-vs-contextual embedding distinction — BERTScore genuinely uses CONTEXTUAL EMBEDDINGS (from a BERT-family model) to measure semantic SIMILARITY between generated and reference text, rather than exact n-gram overlap. This directly solves the "good paraphrase scores worse than lazy near-copy" problem demonstrated in that earlier file's "try this" exercise — a paraphrase with different WORDS but the same MEANING will genuinely score well on BERTScore, unlike BLEU/ROUGE.

## Using a custom metric function — for genuinely task-specific needs

```python
def custom_metric(predictions, references):
    exact_matches = sum(1 for p, r in zip(predictions, references) if p.strip().lower() == r.strip().lower())
    return {"exact_match_rate": exact_matches / len(predictions)}

print(custom_metric(["Paris", "berlin"], ["Paris", "Berlin"]))
```

Genuinely worth knowing standardized metrics aren't always sufficient — real projects sometimes need a CUSTOM evaluation function tailored to the specific business/research question, and `compute_metrics` (recap files 05-07) genuinely accepts any function following the right signature, not just built-in `evaluate` metrics.

## The honest limitation, echoed one more time — metrics are proxies

```python
# Recap Classical NLP file 10's closing framing directly:
# Every metric in this file is a PROXY for "is this output actually good/useful,"
# not the true goal itself. High ROUGE doesn't guarantee a genuinely GOOD summary
# (it could just closely copy phrasing without capturing the real key points).
# High accuracy on an imbalanced dataset can be genuinely misleading (recap
# scikit-learn's class imbalance caution, echoed AGAIN here for the umpteenth time
# across this whole repo, because it keeps mattering).
```

Genuinely worth this repetition — this exact caution has now appeared in the scikit-learn folder, Classical NLP file 10, and here — a real, recurring theme precisely because it's one of the most common genuine mistakes in applied ML: optimizing for a proxy metric while losing sight of the actual goal it's supposed to represent.

## Comparing multiple models on the SAME metric — a genuinely realistic workflow

```python
models_to_compare = ["distilbert-base-uncased-finetuned-sst-2-english", "bert-base-uncased"]
results = {}

for model_name in models_to_compare:
    clf = pipeline("sentiment-analysis", model=model_name)
    # ... run predictions on a shared test set, compute accuracy for each ...
    # results[model_name] = accuracy.compute(...)

print(results)
```

Genuinely the realistic shape of real model-selection work — using the SAME evaluation metric(s), on the SAME held-out test set (recap scikit-learn's train/test discipline), across multiple candidate models, to make an honest, fair comparison before choosing which one to actually deploy.

## Try this

```python
# Using the fine-tuned model from file 05 (or a pretrained sentiment pipeline if
# file 05 wasn't run), compute accuracy, precision, recall, and F1 together using
# evaluate.combine() on a held-out test set
# Then take 2 genuinely different generated summaries of the SAME article (one from
# file 09's BART pipeline, one written by hand, deliberately paraphrased rather than
# copying exact phrases) and score BOTH against the same reference summary using
# rouge AND bertscore -- confirm bertscore treats the hand-paraphrased version more
# fairly than rouge does, directly replicating this file's core lesson
```

Replicating the BLEU/BERTScore contrast from this file's own examples, using genuinely self-generated text, is the strongest way to confirm the lesson has actually landed rather than just being read.

## What's next
File 11 — Model Hub: Sharing, Versioning & Model Cards, covering the OTHER half of the Hub (recap file 01's brief mention) — not just downloading models, but properly sharing and documenting one's own.
