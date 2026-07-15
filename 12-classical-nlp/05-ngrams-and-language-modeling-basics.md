# 05 — N-grams & Language Modeling Basics

Recap files 03-04's n-gram previews — this file formalizes them properly, and introduces the classical (pre-neural-network) approach to modeling language sequences, genuinely the conceptual ancestor of what GPT-style models (recap PyTorch file 15) do at a vastly larger scale.

## What an n-gram actually is

A contiguous sequence of N tokens from a text — genuinely just a sliding window (recap DSA file 13's sliding window technique) over the token sequence.

```python
from nltk import ngrams
from nltk.tokenize import word_tokenize

text = "the quick brown fox jumps"
tokens = word_tokenize(text)

unigrams = list(ngrams(tokens, 1))
bigrams = list(ngrams(tokens, 2))
trigrams = list(ngrams(tokens, 3))

print(unigrams)   # [('the',), ('quick',), ('brown',), ('fox',), ('jumps',)]
print(bigrams)     # [('the', 'quick'), ('quick', 'brown'), ('brown', 'fox'), ('fox', 'jumps')]
print(trigrams)      # [('the', 'quick', 'brown'), ('quick', 'brown', 'fox'), ('brown', 'fox', 'jumps')]
```

Recap file 03's `dog bites man` vs `man bites dog` limitation directly — bigrams/trigrams partially recover word-order information that pure unigram Bag of Words genuinely loses, since `('dog', 'bites')` and `('bites', 'dog')` are now DIFFERENT features.

## The vocabulary size tradeoff — genuinely important, worth understanding clearly

```python
# Vocabulary size grows MUCH faster with n-grams than with unigrams alone
# For a vocabulary of V unique words, the number of POSSIBLE bigrams is up to V^2,
# possible trigrams up to V^3 -- exponential growth (recap DSA file 01's Big O intuition)
```

Genuinely the real, practical tension: higher-order n-grams (trigrams, 4-grams) capture MORE context/word-order information, but the feature space EXPLODES in size, and most possible n-grams never actually occur in any real training data — leading to extreme sparsity. In practice, bigrams and trigrams are genuinely the common sweet spot; higher orders are rarely used in classical (non-neural) approaches specifically because of this tradeoff.

## N-gram Language Models — predicting the next word, classically

```python
from collections import defaultdict, Counter

def build_bigram_model(tokens):
    model = defaultdict(Counter)
    for w1, w2 in ngrams(tokens, 2):
        model[w1][w2] += 1
    return model

corpus = "the cat sat on the mat the cat ran on the road".split()
bigram_model = build_bigram_model(corpus)

print(dict(bigram_model['the']))   # {'cat': 2, 'mat': 1, 'road': 1}
```

Genuinely the core idea behind a classical N-gram Language Model: given the PREVIOUS word (or previous N-1 words), what's the PROBABILITY distribution over what comes NEXT? This directly recaps Math Foundations file 09's conditional probability — `P(next_word | previous_word)` is precisely what this bigram model estimates, just counting occurrences rather than using Bayes' theorem directly.

```python
def predict_next_word(model, word):
    if word not in model:
        return None
    total = sum(model[word].values())
    probabilities = {w: count/total for w, count in model[word].items()}
    return probabilities

print(predict_next_word(bigram_model, 'the'))
# {'cat': 0.5, 'mat': 0.25, 'road': 0.25}
```

Genuinely a real, if crude, language model — given "the," it predicts "cat" is the most likely next word based purely on frequency counts observed in the training corpus. This is precisely the conceptual ancestor of what GPT-style transformer models (recap PyTorch files 15/19) do — predict a probability distribution over the next token — just replacing simple frequency counting with a massively more sophisticated learned neural network.

## The Markov assumption — the genuinely important simplifying assumption

```python
# An N-gram model assumes: P(next_word | ALL previous words) ≈ P(next_word | last N-1 words)
# This is called the MARKOV ASSUMPTION -- genuinely a simplification, not literally true
# (real language has long-range dependencies, recap PyTorch file 11/15's RNN/attention
# motivation for handling exactly this limitation properly)
```

Worth understanding this honestly: N-gram models genuinely CAN'T capture long-range context ("The trophy didn't fit in the suitcase because IT was too big" — recap PyTorch file 15's exact example — resolving "it" needs context far beyond the last 2-3 words). This limitation is PRECISELY why RNNs (PyTorch file 11) and then attention/transformers (PyTorch file 15) were developed — n-gram models are a genuinely useful STARTING point for understanding language modeling conceptually, but hit a real, fundamental ceiling that neural approaches were built to overcome.

## Generating text with an n-gram model — a genuinely fun, illustrative exercise

```python
import random

def generate_text(model, start_word, length=10):
    current = start_word
    result = [current]
    for _ in range(length - 1):
        if current not in model:
            break
        next_word_probs = predict_next_word(model, current)
        next_word = random.choices(list(next_word_probs.keys()),
                                     weights=list(next_word_probs.values()))[0]
        result.append(next_word)
        current = next_word
    return ' '.join(result)

print(generate_text(bigram_model, 'the', length=8))
```

Genuinely worth actually running this — the generated text will be locally coherent (each word choice makes sense given the ONE word before it) but globally nonsensical/repetitive over longer stretches, precisely BECAUSE of the Markov assumption's short memory. This limitation, seen firsthand, is a genuinely good motivator for why RNN/transformer-based generation (much later in this roadmap, once actually generating text with real language models) represents such a significant improvement.

## Smoothing — handling unseen n-grams, briefly

```python
# Problem: if a bigram never appeared in training data, its probability is exactly 0
# This is genuinely too harsh -- absence of evidence isn't evidence of impossibility

# Laplace (add-one) smoothing -- a simple, classical fix
def laplace_smoothed_prob(model, word, next_word, vocab_size):
    count = model[word][next_word]
    total = sum(model[word].values())
    return (count + 1) / (total + vocab_size)      # add 1 to numerator, vocab_size to denominator
```

Genuinely the same underlying idea as Math Foundations file 11's Bayesian prior — rather than assigning a hard ZERO probability to unseen events, smoothing assumes a small baseline probability for everything, updated as more evidence (data) arrives. A simple, classical technique, still conceptually useful even though modern neural language models handle this differently (softmax over a fixed vocabulary, recap PyTorch file 05, naturally avoids assigning literal zero probability to any vocabulary token).

## Perplexity — a preview, fully covered in file 10

```python
# Perplexity measures how "surprised" a language model is by real text --
# LOWER perplexity means the model assigns HIGHER probability to what actually happened,
# meaning it's a BETTER model of the language
# Full treatment, including the exact formula, in file 10's NLP evaluation metrics
```

## Try this

```python
# Build a TRIGRAM language model (using ngrams(tokens, 3)) on a larger piece of text
# (e.g. a few paragraphs of any public domain text)
# Compare text GENERATED from the trigram model against text generated from the
# bigram model above -- does the trigram version feel more coherent over longer stretches?
# Then deliberately query the model for a word/context that never appeared in training
# and confirm it correctly returns None/no prediction, motivating why smoothing matters
```

Comparing bigram vs trigram generation quality directly is genuinely a good hands-on way to feel the tradeoff between "more context = more coherent" and "more context = sparser, more unseen combinations" that this whole file has been building toward.

## What's next
File 06 — Word Embeddings (Word2Vec & GloVe), moving beyond sparse, count-based representations (BoW, TF-IDF, n-grams) into DENSE, LEARNED vector representations that capture actual semantic meaning.
