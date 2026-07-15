# 04 — TF-IDF

Recap file 03's closing note — TF-IDF refines raw Bag of Words counts into a smarter weighting scheme, accounting for how DISTINCTIVE a word is across the whole document collection, not just how often it appears in one document.

## The problem TF-IDF solves — why raw counts are misleading

```python
# Recap file 03's BoW example -- "the" appeared 2 times in doc1, "cat" appeared 1 time
# Raw counts suggest "the" is MORE important than "cat" -- genuinely backwards!
# "the" appears in almost every document (uninformative), "cat" is much more distinctive
```

Genuinely the core insight TF-IDF captures: a word's IMPORTANCE to a specific document should be HIGH if it appears often IN that document, but LOW if it appears in almost EVERY document across the whole collection (since that makes it uninformative for distinguishing documents from each other — same underlying motivation as file 01's stopword removal, but now expressed as a continuous WEIGHT instead of a binary keep/remove decision).

## The formula, broken into its two parts

```
TF-IDF(word, doc) = TF(word, doc) * IDF(word)

TF (Term Frequency)  = (count of word in doc) / (total words in doc)
IDF (Inverse Document Frequency) = log( (total documents) / (documents containing word) )
```

```python
import numpy as np

# Manual TF-IDF calculation for intuition
documents = [
    "the cat sat on the mat",
    "the dog sat on the log",
    "cats and dogs are great"
]

def compute_tf(word, doc):
    words = doc.split()
    return words.count(word) / len(words)

def compute_idf(word, documents):
    doc_count = sum(1 for doc in documents if word in doc.split())
    return np.log(len(documents) / doc_count)

word = "the"
tf = compute_tf(word, documents[0])
idf = compute_idf(word, documents)
print(f"TF: {tf:.3f}, IDF: {idf:.3f}, TF-IDF: {tf*idf:.3f}")

word2 = "cat"
tf2 = compute_tf(word2, documents[0])
idf2 = compute_idf(word2, documents)
print(f"TF: {tf2:.3f}, IDF: {idf2:.3f}, TF-IDF: {tf2*idf2:.3f}")
```

Genuinely worth running this by hand once — "the" appears in ALL 3 documents, so `IDF("the") = log(3/3) = log(1) = 0`, meaning its FULL TF-IDF score becomes ZERO regardless of how often it appears within a document. "cat" appears in only 1 document, giving it a genuinely higher IDF and therefore a higher overall importance score — exactly the behavior wanted.

## Using scikit-learn's TfidfVectorizer — the practical, real-world approach

```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer()
tfidf_matrix = vectorizer.fit_transform(documents)

print(vectorizer.get_feature_names_out())
print(tfidf_matrix.toarray())
```

Genuinely the same `.fit_transform()` interface as `CountVectorizer` (file 03) — a direct drop-in replacement in any scikit-learn pipeline, just producing WEIGHTED scores instead of raw counts. Note: scikit-learn's actual IDF formula includes small smoothing adjustments (avoiding division-by-zero, adding 1 to counts) that differ slightly from the simplified manual version above — genuinely fine for building intuition, but worth knowing the library version has minor, deliberate numerical safety differences.

## TfidfVectorizer parameters — genuinely the same controls as CountVectorizer

```python
vectorizer = TfidfVectorizer(
    max_features=5000,
    min_df=2,
    max_df=0.8,
    stop_words='english',
    ngram_range=(1, 2)      # recap file 03's n-gram preview, full treatment file 05
)
```

Recap file 03 directly — genuinely identical parameter names and behavior, since `TfidfVectorizer` is built on the same underlying vocabulary-building logic as `CountVectorizer`, just with the TF-IDF weighting applied on top instead of raw counts.

## Cosine similarity on TF-IDF vectors — a genuinely powerful, common application

```python
from sklearn.metrics.pairwise import cosine_similarity

similarity_matrix = cosine_similarity(tfidf_matrix)
print(similarity_matrix)
# A 3x3 matrix -- similarity_matrix[i][j] = how similar document i is to document j
```

Recap Math Foundations file 01's cosine similarity derivation directly — this is EXACTLY that formula (`a·b / (||a|| ||b||)`), now applied to TF-IDF document vectors instead of the toy "concept vectors" used there. This is genuinely the foundational technique behind basic document search/retrieval systems: represent a query the same way (as a TF-IDF vector), compute cosine similarity against every document, and rank by similarity score — a simplified but real ancestor of the retrieval step in RAG systems (upcoming in the Frontier GenAI phase).

```python
def find_most_similar(query, documents, vectorizer, tfidf_matrix):
    query_vec = vectorizer.transform([query])          # NOTE: .transform(), not .fit_transform() --
                                                           # genuinely important, recap scikit-learn's
                                                           # train/test transform discipline: never refit
                                                           # the vectorizer on new/query text
    similarities = cosine_similarity(query_vec, tfidf_matrix)
    best_match_idx = similarities.argmax()
    return documents[best_match_idx], similarities[0][best_match_idx]

match, score = find_most_similar("dogs are wonderful pets", documents, vectorizer, tfidf_matrix)
print(match, score)
```

Genuinely important, easy-to-miss detail flagged explicitly: using `.transform()` (not `.fit_transform()`) on new/query text — recap the scikit-learn preprocessing notes' train/test consistency discipline. Refitting the vectorizer on new text would build a DIFFERENT vocabulary/IDF weighting, making the resulting vectors incomparable to the original document vectors.

## TF-IDF for keyword extraction — a genuinely practical side use

```python
def top_keywords(doc_index, vectorizer, tfidf_matrix, n=3):
    feature_names = vectorizer.get_feature_names_out()
    doc_vector = tfidf_matrix[doc_index].toarray()[0]
    top_indices = doc_vector.argsort()[-n:][::-1]        # recap DSA file 11's argsort-based top-k pattern
    return [(feature_names[i], doc_vector[i]) for i in top_indices]

print(top_keywords(0, vectorizer, tfidf_matrix))
```

Genuinely a real, practical application — the words with the HIGHEST TF-IDF score within a single document are often a reasonable automatic summary of what that document is distinctively ABOUT, since the formula specifically rewards frequent-but-rare-elsewhere terms.

## TF-IDF vs Bag of Words — the honest comparison

| | Bag of Words (file 03) | TF-IDF |
|---|---|---|
| Weighting | Raw counts | Frequency, discounted by commonness |
| Common words like "the" | Can dominate | Genuinely down-weighted toward zero |
| Computational cost | Lower | Slightly higher (needs the full corpus for IDF) |
| Typical use | Simple baselines, when counts genuinely matter | Search/retrieval, keyword extraction, most classical text classification |

Genuinely the practical default: TF-IDF is usually the better starting point over plain BoW for classical text classification and retrieval tasks, precisely because of this smarter weighting — though both share the SAME fundamental "bag" limitation (word order still gets discarded, recap file 03's dog-bites-man example).

## Try this

```python
# Using 5-10 short documents on a topic of choice, build a TfidfVectorizer
# Write a simple "search engine": given a text query, use cosine_similarity to
# rank all documents by relevance and print the top 3 matches with their scores
# Then extract the top 5 keywords for each document using the top_keywords() function
# and manually judge whether they feel like a reasonable summary of each document's content
```

Building this tiny search engine end-to-end is genuinely the clearest way to feel TF-IDF's practical value — a working, if simplified, retrieval system built from concepts covered in this single file.

## What's next
File 05 — N-grams & Language Modeling Basics, formalizing the n-gram preview from files 03-04 and introducing the classical (pre-neural-network) approach to predicting/modeling language sequences.
