# 03 — Bag of Words & Text Vectorization

Recap files 01-02's closing note — this is the file that finally converts clean tokens into actual NUMBERS a model can use. Genuinely the foundational vectorization technique that TF-IDF (file 04) and even word embeddings (file 06) build on or contrast against.

## The core idea — a vocabulary and a count

Bag of Words (BoW) represents each document as a VECTOR of word counts, completely ignoring word ORDER (hence "bag" — like dumping all the words into a bag, losing sequence information entirely).

```python
from sklearn.feature_extraction.text import CountVectorizer

documents = [
    "the cat sat on the mat",
    "the dog sat on the log",
    "cats and dogs are great"
]

vectorizer = CountVectorizer()
bow_matrix = vectorizer.fit_transform(documents)

print(vectorizer.get_feature_names_out())
# ['and' 'are' 'cat' 'cats' 'dog' 'dogs' 'great' 'log' 'mat' 'on' 'sat' 'the']

print(bow_matrix.toarray())
# [[0 0 1 0 0 0 0 0 1 1 1 2]
#  [0 0 0 0 1 0 0 1 0 1 1 2]
#  [1 1 0 1 0 1 1 0 0 0 0 0]]
```

Recap scikit-learn's general `.fit_transform()` pattern — genuinely the exact same interface used throughout the scikit-learn folder, just applied to text here instead of numeric features. Each row is one document's vector; each column is one vocabulary word; each value is that word's COUNT in that document.

## Why "bag" — the genuinely important limitation to understand upfront

```python
doc1 = "the dog bit the man"
doc2 = "the man bit the dog"
# These produce the EXACT SAME Bag of Words vector -- despite having completely
# opposite meanings! BoW genuinely throws away all word order/structure.
```

Worth internalizing this limitation clearly and early — Bag of Words is a genuinely crude representation, deliberately simple, useful as a strong BASELINE and for tasks where word order matters less (broad topic classification, spam detection), but genuinely insufficient for anything requiring understanding of grammar, negation position, or sequence (recap PyTorch file 11's RNN motivation — sequence models exist precisely to fix this exact gap).

## Sparse matrices — a genuinely important practical detail

```python
print(type(bow_matrix))   # <class 'scipy.sparse._csr.csr_matrix'>
print(bow_matrix.shape)     # (3, 12) -- 3 documents, 12 vocabulary words
```

Recap the general efficiency-consciousness from the NumPy/pandas notes — real vocabularies can have TENS OF THOUSANDS of unique words, but any single document only uses a tiny fraction of them. Storing this as a normal dense array would waste enormous memory on mostly-zero values — `scipy`'s sparse matrix format stores ONLY the non-zero entries, genuinely essential for real-scale text data, not just an implementation detail to ignore.

## Binary Bag of Words — presence/absence instead of counts

```python
binary_vectorizer = CountVectorizer(binary=True)
binary_matrix = binary_vectorizer.fit_transform(documents)
print(binary_matrix.toarray())
# Same shape, but every non-zero value becomes exactly 1, regardless of how many times
# the word actually appeared
```

Genuinely useful when WHETHER a word appears matters more than HOW OFTEN — e.g. for some spam-detection features, "does this email contain the word 'winner'" might matter more than the exact count.

## Controlling vocabulary size — genuinely important parameters

```python
vectorizer = CountVectorizer(
    max_features=1000,        # keep only the 1000 most frequent words -- controls dimensionality
    min_df=2,                    # ignore words appearing in FEWER than 2 documents (likely noise/typos)
    max_df=0.8,                    # ignore words appearing in MORE than 80% of documents (too common to be useful)
    stop_words='english'              # built-in stopword removal, recap file 01
)
```

Genuinely a real, practical necessity — an unconstrained vocabulary from real-world text can easily exceed 100,000+ unique tokens, most of which are typos, rare proper nouns, or noise. `max_features`, `min_df`, and `max_df` are the standard controls for keeping the resulting feature space MANAGEABLE and genuinely informative, directly connecting back to the general "curse of dimensionality" concerns from the scikit-learn notes.

## N-grams in CountVectorizer — a preview, full treatment in file 05

```python
bigram_vectorizer = CountVectorizer(ngram_range=(1, 2))      # unigrams AND bigrams together
bigram_matrix = bigram_vectorizer.fit_transform(documents)
print(bigram_vectorizer.get_feature_names_out())
# includes both single words AND two-word phrases like 'the cat', 'cat sat', etc.
```

Genuinely a partial fix for BoW's word-order blindness — including two-word (or longer) sequences as vocabulary "words" recovers SOME local ordering information, though at the cost of a much larger vocabulary. File 05 covers this properly.

## Using BoW features in an actual classifier — closing the loop with scikit-learn

```python
from sklearn.naive_bayes import MultinomialNB          # recap Math Foundations file 11's Naive Bayes
from sklearn.model_selection import train_test_split

labels = [1, 1, 0]   # pretend: 1 = "about pets", 0 = "not about pets" -- toy example

X_train, X_test, y_train, y_test = train_test_split(bow_matrix, labels, test_size=0.33, random_state=42)

clf = MultinomialNB()
clf.fit(X_train, y_train)
predictions = clf.predict(X_test)
```

Genuinely the exact scikit-learn workflow already used throughout that entire folder — `CountVectorizer` output plugs directly into ANY scikit-learn classifier (Naive Bayes, Logistic Regression, XGBoost, recap that folder too) exactly like any other numeric feature matrix. `MultinomialNB` specifically is genuinely the classic pairing with Bag of Words counts, directly connecting back to Math Foundations file 11's Bayes' theorem derivation — the same math, now applied to word-count features instead of the spam-filter toy example used there.

## Try this

```python
# Take 5 short documents about 2 different topics (e.g. cooking and sports)
# Build a CountVectorizer with min_df=1 and inspect the resulting vocabulary size
# Then rebuild it with stop_words='english' and compare -- how much did the vocabulary shrink?
# Train a MultinomialNB classifier on the BoW features to distinguish the 2 topics,
# and check its accuracy on a couple of new, unseen test sentences
```

Seeing the vocabulary size shrink meaningfully after removing stopwords, and then watching a genuinely simple Naive Bayes classifier correctly distinguish topics from nothing but word counts, is a good concrete demonstration of BoW's real (if limited) power.

## What's next
File 04 — TF-IDF, refining raw word counts into a smarter weighting scheme that accounts for how DISTINCTIVE a word is, not just how often it appears.
