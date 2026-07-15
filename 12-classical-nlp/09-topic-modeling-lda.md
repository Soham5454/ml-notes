# 09 — Topic Modeling (LDA)

Recap file 08's closing note — shifting from SUPERVISED classification (sentiment, spam — files 07-08, needing labeled data) to UNSUPERVISED discovery of themes within a document collection, genuinely no labels required at all.

## The core idea — documents as mixtures of topics

LDA (Latent Dirichlet Allocation) assumes every document is a MIXTURE of several topics, and every topic is a DISTRIBUTION over words. Genuinely the goal: given only raw documents (no labels), automatically discover what the underlying "topics" are, and how much each document draws from each one.

```python
# Conceptual example, before any code:
# Document: "The team won the championship game with a great defensive strategy"
# LDA might discover this is: 70% "Sports" topic, 30% "Strategy/Business" topic
# where "Sports" topic itself is a distribution like: {game: 0.15, team: 0.12, win: 0.10, ...}
```

## Setting up — preprocessing recap, then LDA

```python
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.decomposition import LatentDirichletAllocation

documents = [
    "the team scored a goal in the final minute of the game",
    "the stock market rallied after the earnings report",
    "the striker's performance won them the championship",
    "investors reacted positively to the quarterly revenue growth",
    "the coach praised the defense after the match",
    "the company's profits exceeded analyst expectations this quarter"
]

vectorizer = CountVectorizer(stop_words='english')      # recap file 03 -- LDA genuinely uses raw COUNTS, not TF-IDF
doc_term_matrix = vectorizer.fit_transform(documents)

lda = LatentDirichletAllocation(n_components=2, random_state=42)      # asking for 2 topics
lda.fit(doc_term_matrix)
```

Genuinely important, sometimes-missed detail: LDA traditionally works on raw WORD COUNTS (Bag of Words, file 03), NOT TF-IDF weighted values — this is because LDA's underlying probabilistic model is built around counting word occurrences directly, and TF-IDF's re-weighting can actually distort the statistical assumptions LDA relies on.

## Inspecting the discovered topics

```python
def print_topics(model, vectorizer, n_words=5):
    feature_names = vectorizer.get_feature_names_out()
    for topic_idx, topic in enumerate(model.components_):
        top_words = [feature_names[i] for i in topic.argsort()[-n_words:][::-1]]
        print(f"Topic {topic_idx}: {', '.join(top_words)}")

print_topics(lda, vectorizer)
# Topic 0: game, team, championship, striker, coach   -- genuinely reads as "Sports"
# Topic 1: market, earnings, revenue, profits, investors  -- genuinely reads as "Finance"
```

Genuinely satisfying to see LDA discover these coherent topic clusters WITHOUT ever being told the documents were about sports vs finance — the algorithm purely finds statistical word co-occurrence patterns and groups them, using the exact same `argsort` top-k pattern from file 04's keyword extraction.

## Getting each document's topic distribution

```python
doc_topics = lda.transform(doc_term_matrix)
for i, doc in enumerate(documents):
    print(f"Doc {i}: {doc[:40]}... -> Topic distribution: {doc_topics[i].round(2)}")

# Doc 0: "the team scored a goal..." -> [0.85, 0.15]  -- mostly Topic 0 (Sports)
# Doc 1: "the stock market rallied..." -> [0.12, 0.88]  -- mostly Topic 1 (Finance)
```

Genuinely the core output of LDA — a probability distribution over topics FOR EACH document, directly usable as a dense feature representation (similar in spirit to file 06's word embeddings, but at the DOCUMENT level and specifically interpretable as "topic proportions" rather than an opaque semantic vector).

## Choosing the number of topics — a genuinely real, honest challenge

```python
# Unlike supervised classification (files 07-08), there's no ground-truth "correct"
# number of topics -- n_components is a HYPERPARAMETER that needs judgment/tuning
# (recap the general unsupervised-learning challenges from the scikit-learn folder's
# clustering notes -- k-means has the SAME "how many clusters" problem)
```

Genuinely worth recognizing this as the SAME fundamental challenge as choosing `k` for k-means clustering (recap scikit-learn notes) — LDA is genuinely a form of "soft clustering" over words/documents, sharing this exact hyperparameter-selection difficulty. Common practical approaches: try several values, use **coherence scores** (a metric measuring how semantically consistent the top words in each discovered topic actually are), or rely on domain knowledge about how many distinct themes are genuinely expected.

## Coherence score — a genuinely useful evaluation tool

```python
# Using the gensim library, which has more built-in topic-modeling tooling than scikit-learn
from gensim.models import CoherenceModel
from gensim.corpora import Dictionary

tokenized_docs = [doc.split() for doc in documents]
dictionary = Dictionary(tokenized_docs)
corpus = [dictionary.doc2bow(doc) for doc in tokenized_docs]

# (after training a gensim LDA model, not shown here for brevity)
# coherence_model = CoherenceModel(model=lda_gensim, texts=tokenized_docs, dictionary=dictionary, coherence='c_v')
# print(coherence_model.get_coherence())
```

Genuinely worth knowing this metric exists as the standard, more rigorous way to compare different `n_components` choices objectively, rather than just eyeballing whether the printed top-words per topic "look reasonable" (which is a genuinely fine STARTING point, but not a fully rigorous evaluation method on its own).

## LDA vs clustering — the honest comparison

Recap scikit-learn's k-means clustering notes — genuinely a related but DIFFERENT idea:
- **k-means clustering** assigns each document to exactly ONE cluster (hard assignment).
- **LDA** assigns each document a DISTRIBUTION across ALL topics (soft assignment) — a document can genuinely be "70% sports, 30% finance" simultaneously, which is often a more realistic model of real documents that blend multiple themes.

## Practical use cases — genuinely real applications

```
- Organizing a large, unlabeled document collection (news articles, research papers,
  customer support tickets) into discoverable themes without manual labeling
- Recommendation systems: recommend documents with SIMILAR topic distributions
  (recap Math Foundations file 01's cosine similarity, now applied to topic-distribution
  vectors instead of raw word vectors)
- Trend detection over time: track how topic proportions shift across a time-ordered
  document collection (recap SQL file 12's date-bucketing notes for how this data
  might get organized before analysis)
```

## Try this

```python
# Using 10-15 short documents spanning 3 genuinely distinct topics (e.g. sports, cooking,
# technology), build an LDA model asking for 3 topics
# 1. Print the top 5 words per discovered topic -- do they align with the 3 real themes?
# 2. Check a document's topic distribution -- does it match the theme you'd expect?
# 3. Deliberately write ONE document that GENUINELY blends two themes (e.g. "the new
#    smart kitchen gadget uses AI to recommend recipes") and confirm LDA assigns it
#    a genuinely MIXED distribution rather than forcing it into one topic
```

That third document is genuinely the most interesting test — seeing LDA correctly represent a document's mixed nature (rather than being forced into a single hard category, like k-means would) is the clearest demonstration of what makes LDA's "soft" topic modeling genuinely valuable over simpler clustering approaches.

## What's next
File 10 — NLP Evaluation Metrics, formalizing BLEU, ROUGE, and perplexity — the specific metrics used for text GENERATION and language modeling tasks, distinct from the classification metrics (precision/recall/F1) already covered via the scikit-learn folder.
