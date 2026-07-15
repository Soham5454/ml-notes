# 10 — NLP Evaluation Metrics

Recap file 09's closing note, and file 05's perplexity preview — this file formalizes the metrics SPECIFIC to text generation and language modeling, genuinely distinct from the classification metrics (precision/recall/F1, recap scikit-learn) already covered for tasks like file 07's classification and file 08's sentiment analysis.

## Why classification metrics don't work for generation tasks

```python
# For classification (files 07-08): there's exactly ONE correct label, precision/recall/F1 apply directly
# For generation (translation, summarization, upcoming Hugging Face text generation):
# there are MANY possible "correct" outputs -- comparing generated text to a reference
# needs fundamentally different metrics
```

Genuinely the core challenge: "translate this sentence to French" or "summarize this article" doesn't have ONE single correct answer — multiple genuinely valid phrasings exist. Metrics for these tasks need to measure SIMILARITY to reference text(s), not exact match.

## Perplexity — recap file 05's preview, now fully explained

```python
import numpy as np

def perplexity(model_probabilities):
    # model_probabilities: the probability the model assigned to each ACTUAL word that occurred
    log_probs = np.log(model_probabilities)
    avg_log_prob = np.mean(log_probs)
    return np.exp(-avg_log_prob)

# Example: a model that's very confident and correct about what actually happened
good_model_probs = [0.9, 0.85, 0.8, 0.9]
print(perplexity(good_model_probs))   # LOW perplexity -- genuinely good

# A model that's uncertain/often wrong about what actually happened
bad_model_probs = [0.1, 0.15, 0.05, 0.2]
print(perplexity(bad_model_probs))   # HIGH perplexity -- genuinely bad
```

Recap Math Foundations file 15's MLE/log-likelihood derivation directly — perplexity is genuinely just `exp(-average log-likelihood)`, i.e. a transformed version of the negative log-likelihood loss (recap PyTorch file 06's cross-entropy connection) that's more INTERPRETABLE: perplexity can be read as "the model was, on average, as confused as if it were choosing uniformly among THIS MANY options." A perplexity of 1 is perfect (always fully confident and correct); lower is genuinely better. Recap file 05's n-gram language model — perplexity is precisely the standard way to evaluate HOW GOOD a language model (n-gram OR neural, recap PyTorch file 15's transformers) is at predicting real text.

## BLEU Score — the classic machine translation metric

```python
from nltk.translate.bleu_score import sentence_bleu

reference = [["the", "cat", "is", "on", "the", "mat"]]      # note: list of reference(s), can have multiple
candidate = ["the", "cat", "sat", "on", "the", "mat"]

score = sentence_bleu(reference, candidate)
print(score)
```

BLEU (Bilingual Evaluation Understudy) genuinely measures N-GRAM OVERLAP (recap file 05) between the generated text and one or more REFERENCE translations — specifically, what fraction of the candidate's n-grams (unigrams, bigrams, up through 4-grams typically) also appear in the reference(s), with a penalty for outputs that are too SHORT (to prevent gaming the score by generating a tiny, "safe" but incomplete output).

```python
from nltk.translate.bleu_score import SmoothingFunction

smoothie = SmoothingFunction().method1
score = sentence_bleu(reference, candidate, smoothing_function=smoothie)
```

Genuinely a real, practical necessity for SHORT sentences — without smoothing, BLEU can be harshly, misleadingly ZERO if even one 4-gram fails to match, even when the translation is genuinely quite good. Smoothing functions soften this for realistic, short-sentence evaluation.

## The honest limitations of BLEU — genuinely important to know

```python
# BLEU is PURELY n-gram overlap -- it has NO understanding of MEANING
reference = [["the", "weather", "is", "nice", "today"]]
candidate1 = ["the", "weather", "is", "nice", "today"]        # perfect match
candidate2 = ["today", "the", "weather", "is", "pleasant"]       # GENUINELY equally good translation,
                                                                     # but scores much LOWER on BLEU due to
                                                                     # word reordering and synonym substitution
```

Genuinely a real, well-known weakness — BLEU can penalize translations that are SEMANTICALLY correct but phrased differently from the reference. This is precisely why more recent metrics (like BERTScore, which uses contextual embeddings — recap file 06's limitation and the Hugging Face bridge ahead — to measure SEMANTIC similarity rather than exact n-gram overlap) have become increasingly preferred in modern NLP evaluation, though BLEU remains genuinely standard and widely reported for historical/comparability reasons.

## ROUGE — the standard metric for summarization

```python
from rouge_score import rouge_scorer

scorer = rouge_scorer.RougeScorer(['rouge1', 'rouge2', 'rougeL'], use_stemmer=True)      # recap file 02's stemming

reference_summary = "the cat sat on the mat and looked outside"
generated_summary = "the cat sat on the mat"

scores = scorer.score(reference_summary, generated_summary)
print(scores)
# {'rouge1': ..., 'rouge2': ..., 'rougeL': ...}
```

Genuinely the RECALL-oriented counterpart to BLEU's precision-oriented approach — ROUGE asks "how much of the REFERENCE's content is captured in the generated summary" (recap scikit-learn's precision/recall distinction — same underlying philosophical difference, just applied to n-gram overlap instead of classification labels). `ROUGE-1`/`ROUGE-2` measure unigram/bigram overlap (recap file 05); `ROUGE-L` measures the LONGEST COMMON SUBSEQUENCE (recap DSA file 15's exact LCS algorithm — genuinely the identical technique, applied here to compare word sequences instead of arbitrary strings) between generated and reference text, which is more forgiving of word-order variations than plain n-gram matching.

## F1 Score for span-based tasks — recap scikit-learn's F1, applied to NLP-specific tasks

```python
# For tasks like Named Entity Recognition (full treatment in file 11) or Question
# Answering, F1 is often computed at the TOKEN or SPAN level rather than the whole
# document level -- genuinely the same precision/recall/F1 formula from scikit-learn,
# just applied to "did the predicted span/entity boundaries match the true ones"
# rather than "did the predicted class label match"
```

## Human evaluation — the honest gold standard, briefly

```
Genuinely worth knowing: for many generation tasks (especially open-ended ones like
chatbot responses or creative writing), NO automatic metric fully captures quality.
Real NLP research and production systems often rely on HUMAN evaluation (rating
outputs on fluency, relevance, correctness) as the actual ground truth, using automatic
metrics like BLEU/ROUGE/perplexity as faster, cheaper PROXIES during development
```

Genuinely important honest framing to carry forward — automatic metrics are useful for rapid iteration and comparison, but they're APPROXIMATIONS of what's actually wanted (good, meaningful text), not the true target itself. This same "the metric is a proxy for the real goal" caution echoes the general ML evaluation philosophy from the scikit-learn/XGBoost folders — a model or system optimizing purely for a metric can genuinely find ways to score well without truly solving the intended problem.

## Try this

```python
# Take a reference sentence and write 3 candidate variations:
# 1. A near-perfect paraphrase (different words, same meaning)
# 2. A word-for-word match with ONE word changed
# 3. A genuinely poor, unrelated response
# Score all 3 against the reference using sentence_bleu
# Confirm variation 2 (near word-for-word) scores HIGHER than variation 1 (good paraphrase)
# despite variation 1 being arguably the BETTER response -- this concretely demonstrates
# BLEU's semantic-blindness limitation discussed in this file, rather than just reading about it
```

Seeing BLEU concretely REWARD a lazier, more literal match over a genuinely better paraphrase is the clearest way to internalize why modern evaluation increasingly supplements these classical metrics with semantic, embedding-based approaches (bridging directly into the Hugging Face phase).

## What's next
File 11 — Named Entity Recognition & POS Tagging, the last classical NLP TASK category — extracting structured information (names, dates, organizations) from unstructured text.
