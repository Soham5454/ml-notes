# 06 — Word Embeddings: Word2Vec & GloVe

Recap file 05's closing note — moving beyond sparse, count-based representations (BoW, TF-IDF, n-grams) into DENSE, LEARNED vectors that capture actual semantic meaning. Genuinely the biggest conceptual leap in this whole NLP folder, and the direct ancestor of PyTorch file 11's `nn.Embedding`.

## The core problem with everything covered so far

```python
# In TF-IDF/BoW, "cat" and "dog" and "car" are all EQUALLY dissimilar to each other --
# just different vocabulary indices, no notion that "cat" and "dog" are semantically
# CLOSER to each other than either is to "car"
```

Genuinely the fundamental limitation: sparse representations (files 03-05) treat every word as an independent, unrelated dimension — there's no built-in concept of MEANING or SIMILARITY between words, only co-occurrence counts. Word embeddings fix this by learning a DENSE vector for each word, positioned in space such that semantically similar words end up CLOSE together.

## The distributional hypothesis — the core idea behind ALL word embeddings

```
"You shall know a word by the company it keeps." -- J.R. Firth, 1957
```

Genuinely the foundational linguistic insight: words that appear in SIMILAR CONTEXTS tend to have SIMILAR MEANINGS. "cat" and "dog" both frequently appear near words like "pet," "feed," "walk," "vet" — so a model that learns to predict context from words (or words from context) will naturally position "cat" and "dog" close together in the learned vector space, WITHOUT ever being explicitly told they're both animals.

## Word2Vec — two training approaches, briefly conceptual

```python
# CBOW (Continuous Bag of Words): predict the CENTER word from surrounding CONTEXT words
# "the ___ sat on the mat" -> predict "cat"

# Skip-gram: predict the surrounding CONTEXT words from the CENTER word (the reverse)
# "cat" -> predict "the", "sat", "on", "the", "mat" (nearby words)
```

Genuinely both are trained using a SHALLOW neural network (recap PyTorch file 04's `nn.Linear` — Word2Vec's original architecture is essentially one hidden layer), where the LEARNED WEIGHTS of that hidden layer, after training on massive text, BECOME the word embeddings. The actual prediction task (CBOW/skip-gram) is genuinely just a means to an end — the useful byproduct is the dense vector representation each word ends up with.

## Using pretrained Word2Vec — the practical approach

```python
import gensim.downloader as api

model = api.load('word2vec-google-news-300')      # pretrained on Google News, 300-dimensional vectors

vector = model['cat']
print(vector.shape)   # (300,) -- a dense, 300-dimensional vector
print(vector[:5])       # array of real numbers, NOT interpretable individually
```

Genuinely important, honest note: individual DIMENSIONS of a word embedding are NOT directly interpretable (unlike TF-IDF, where each dimension corresponds to a specific known word) — the MEANING lives in the overall geometric relationships between vectors, not in any single coordinate.

## Word similarity — the genuinely satisfying payoff

```python
print(model.similarity('cat', 'dog'))       # ~0.76 -- genuinely high, both are pets/animals
print(model.similarity('cat', 'car'))         # ~0.21 -- genuinely low, unrelated concepts

print(model.most_similar('cat', topn=5))
# [('cats', 0.83), ('dog', 0.76), ('kitten', 0.75), ('feline', 0.73), ('puppy', 0.71)]
```

Recap Math Foundations file 01's cosine similarity derivation, and file 04's TF-IDF document similarity — this is EXACTLY the same operation (cosine similarity between vectors), just applied to WORD embeddings instead of document TF-IDF vectors, and now capturing genuine semantic relationships rather than just co-occurrence overlap.

## The famous "king - man + woman = queen" example

```python
result = model.most_similar(positive=['king', 'woman'], negative=['man'], topn=3)
print(result)   # [('queen', 0.71), ...]
```

Genuinely one of the most famous demonstrations in all of NLP — word embeddings capture RELATIONSHIPS as consistent VECTOR ARITHMETIC. The "royalty" direction and the "gender" direction in the embedding space are apparently consistent enough that `king - man + woman` (subtracting the "male" direction, adding the "female" direction) lands near "queen." This genuinely surprised even the researchers who first discovered it — a real emergent property of training on the distributional hypothesis at scale, not something explicitly programmed in.

## GloVe — a different training approach, similar end result

```python
# GloVe (Global Vectors) trains on GLOBAL word co-occurrence STATISTICS across the
# whole corpus (a big co-occurrence matrix, recap Math Foundations file 03's
# matrix/SVD concepts -- GloVe is genuinely related to matrix factorization)
# rather than Word2Vec's LOCAL, sliding-window prediction task

import gensim.downloader as api
glove_model = api.load('glove-wiki-gigaword-100')      # pretrained GloVe vectors, 100-dimensional

print(glove_model.similarity('cat', 'dog'))
```

Genuinely worth knowing the philosophical difference: Word2Vec learns from LOCAL context windows (predicting nearby words), GloVe learns from GLOBAL co-occurrence COUNTS across the entire corpus (directly related to Math Foundations file 03's matrix factorization/SVD ideas — GloVe is genuinely closer in spirit to factorizing a giant word-co-occurrence matrix). In PRACTICE, both produce genuinely similar-quality embeddings for most tasks — the choice between them is often more about availability of pretrained vectors than a strong theoretical preference.

## Using embeddings as features for classification

```python
import numpy as np

def sentence_to_vector(sentence, model):
    words = sentence.lower().split()
    vectors = [model[w] for w in words if w in model]
    if not vectors:
        return np.zeros(model.vector_size)
    return np.mean(vectors, axis=0)      # genuinely simple: average all word vectors in the sentence

sentence_vec = sentence_to_vector("the cat sat on the mat", model)
print(sentence_vec.shape)   # (300,)
```

Genuinely a real, common (if simplistic) technique — averaging word vectors to get a single SENTENCE-level vector, usable as input features for any scikit-learn classifier (recap that folder), similar in spirit to how TF-IDF vectors get used but now carrying genuine semantic information instead of just weighted counts. The honest limitation, worth flagging clearly: averaging LOSES word order entirely (recap file 03's bag-of-words limitation, reappearing here in a new form) — this is genuinely one of several real motivations for moving to RNN/transformer-based sentence representations (PyTorch files 11/15), which process the FULL sequence rather than collapsing it into an average.

## Word2Vec/GloVe vs contextual embeddings — the honest limitation, bridging forward

```python
# Genuinely important limitation: Word2Vec/GloVe give EVERY occurrence of a word
# the SAME vector, regardless of context
print(model.similarity('bank', 'river'))     # some similarity, from "river bank" contexts
print(model.similarity('bank', 'money'))       # ALSO some similarity, from "bank account" contexts
# But "bank" has ONE fixed vector -- it can't distinguish "river bank" from "money bank"
# in any given sentence, since the word "bank" has the SAME representation every time
```

This is genuinely THE key limitation that Hugging Face's transformer-based models solve — recap PyTorch file 15's attention mechanism and the upcoming Hugging Face bridge — modern contextual embeddings (from BERT, GPT, etc.) generate a DIFFERENT vector for "bank" depending on its surrounding sentence, correctly distinguishing the "river bank" and "money bank" senses. Word2Vec/GloVe are genuinely foundational, historically important, and still useful for lightweight tasks — but this fixed-vector-per-word limitation is precisely the gap that motivated the shift toward the transformer-based contextual embeddings covered later in this roadmap.

## Try this

```python
# Using a pretrained Word2Vec or GloVe model, explore:
# 1. model.most_similar('good', topn=5) -- do the results make intuitive sense?
# 2. Try the vector arithmetic pattern with a DIFFERENT relationship, e.g.
#    'Paris' - 'France' + 'Italy' -- does it land near 'Rome'?
# 3. Build sentence vectors (via averaging) for 3 sentences on similar topics and 
#    3 sentences on a very different topic, then use cosine_similarity to confirm
#    the similar-topic sentences cluster closer together than the different-topic ones
```

Testing the vector-arithmetic pattern on a SELF-CHOSEN relationship (not just the famous king/queen example already given) is genuinely the best way to confirm this isn't a cherry-picked one-off demonstration, but a real, general property of the learned embedding space.

## Classical NLP — halfway point. What's next
Files 01-06 covered the preprocessing and representation foundations. File 07 begins applying these representations to actual TASKS — Text Classification with classical ML, directly connecting this whole folder back to the scikit-learn/XGBoost notes.
