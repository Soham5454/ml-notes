# 08 — Sentiment Analysis

Recap file 07's closing note — a focused, practical deep-dive into the specific task used as that file's running example, including techniques and pitfalls unique to sentiment analysis specifically.

## Lexicon-based sentiment analysis — the simplest possible approach, no training needed

```python
from nltk.sentiment import SentimentIntensityAnalyzer
import nltk
nltk.download('vader_lexicon')

sia = SentimentIntensityAnalyzer()

print(sia.polarity_scores("I absolutely loved this movie!"))
# {'neg': 0.0, 'neu': 0.286, 'pos': 0.714, 'compound': 0.6696}

print(sia.polarity_scores("This was okay, nothing special"))
# {'neg': 0.0, 'neu': 1.0, 'pos': 0.0, 'compound': 0.0}

print(sia.polarity_scores("Absolutely terrible, worst experience ever"))
# {'neg': 0.508, 'neu': 0.492, 'pos': 0.0, 'compound': -0.6249}
```

VADER (Valence Aware Dictionary and sEntiment Reasoner) is genuinely a LEXICON-based approach — it uses a pre-built DICTIONARY of words with known sentiment scores, plus hand-crafted RULES (handling negation, intensifiers, capitalization, punctuation) rather than any trained machine learning model. The `compound` score is a single normalized value from -1 (most negative) to +1 (most positive) — genuinely the most commonly used single number for a quick sentiment read.

## Why lexicon-based approaches genuinely struggle — the honest limitations

```python
print(sia.polarity_scores("The movie wasn't bad"))          # negation handling
# VADER handles simple negation reasonably: 'wasn't bad' correctly skews less negative

print(sia.polarity_scores("Yeah, GREAT service, only waited 3 hours"))    # sarcasm
# Genuinely FAILS here -- VADER sees "GREAT" (capitalized, intensified) and scores it
# strongly positive, completely missing the sarcasm
```

Genuinely important, honest limitation worth understanding clearly: lexicon-based methods have NO real understanding of context, sarcasm, or the kind of long-range meaning that recap PyTorch file 15's attention mechanism was built to capture. They work reasonably well for STRAIGHTFORWARD sentiment but genuinely break down on sarcasm, complex negation, or domain-specific language (e.g. "sick" meaning "great" in some slang contexts, versus "sick" meaning "ill").

## TextBlob — another simple lexicon-based option

```python
from textblob import TextBlob

blob = TextBlob("This product exceeded my expectations")
print(blob.sentiment)   # Sentiment(polarity=0.25, subjectivity=0.35)
```

`polarity` ranges from -1 to 1 (negative to positive, same idea as VADER's compound score), `subjectivity` ranges from 0 (purely factual/objective) to 1 (purely opinion-based) — a genuinely useful SECOND dimension VADER doesn't directly provide, useful for distinguishing factual statements from opinions even before considering their sentiment direction.

## ML-based sentiment analysis — recap file 07's full classification pipeline

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline

# Genuinely the SAME pipeline from file 07, applied specifically to sentiment labels
reviews = ["Great product, highly recommend", "Poor quality, broke immediately", "..."]
sentiments = [1, 0]   # 1 = positive, 0 = negative

pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(ngram_range=(1,2), stop_words='english')),      # recap file 05's n-grams
    ('clf', LogisticRegression())
])
pipeline.fit(reviews, sentiments)
```

Genuinely important reasoning for WHY bigrams (`ngram_range=(1,2)`) matter specifically for sentiment: recap file 05's word-order note — "not good" as a BIGRAM feature captures the negation directly, whereas unigram-only Bag of Words would see "not" and "good" as SEPARATE, disconnected features, genuinely losing the fact that "not" is negating "good" specifically. This is a real, practical reason n-grams matter more for sentiment than for some other classification tasks (like topic classification, where word order matters less).

## The negation handling problem — a genuinely deeper look

```python
# Recap file 01's stopword removal caveat, now fully explained in context
text = "The service was not good at all"
# If "not" gets removed as a stopword (file 01's default stopword list includes it!),
# the remaining text "service good" reads as POSITIVE -- exactly backwards!
```

Genuinely the concrete, worked example of file 01's earlier warning — this is PRECISELY why sentiment analysis pipelines typically customize the stopword list to EXCLUDE negation words:

```python
from nltk.corpus import stopwords

custom_stopwords = set(stopwords.words('english')) - {'not', 'no', 'never', "n't"}
vectorizer = TfidfVectorizer(stop_words=list(custom_stopwords))
```

## A common trick — negation scope marking

```python
def mark_negation(text):
    words = text.split()
    result = []
    negate = False
    for word in words:
        if word.lower() in ["not", "no", "never"]:
            negate = True
            result.append(word)
            continue
        if negate:
            result.append(word + "_NEG")
        else:
            result.append(word)
        if word.endswith(('.', ',', '!', '?')):
            negate = False
    return ' '.join(result)

print(mark_negation("The movie was not good and I did not enjoy it"))
# "The movie was not good_NEG and I did not enjoy_NEG it"
```

Genuinely a real, classical technique — appending `_NEG` to words following a negation turns "good" and "good_NEG" into COMPLETELY DIFFERENT vocabulary tokens for the vectorizer, letting a classical BoW/TF-IDF model learn that "good_NEG" correlates with NEGATIVE sentiment, without needing any word-order-aware architecture (RNN/transformer) to capture it. A genuinely clever workaround within the limitations of the bag-of-words paradigm, though obviously more brittle than a model that genuinely understands sequence and context.

## Aspect-based sentiment — a genuinely more advanced, practical extension

```python
# "The food was amazing but the service was terrible"
# Overall sentiment is AMBIGUOUS/mixed -- but ASPECT-level sentiment is clear:
# food: positive, service: negative
```

Genuinely worth knowing this exists as a real, common real-world need — simple document-level sentiment (files' examples so far) genuinely can't handle mixed reviews well. Aspect-based sentiment analysis identifies SPECIFIC aspects/entities mentioned and scores sentiment PER aspect — a genuinely more complex task, typically requiring either rule-based aspect extraction combined with nearby sentiment scoring, or (more robustly) the transformer-based approaches covered later in this roadmap (Hugging Face), which handle this kind of nuanced, context-dependent task far more naturally than classical bag-of-words methods.

## Try this

```python
# Take 10 genuinely mixed-sentiment product reviews (some clearly positive, some
# clearly negative, at least 2 with real negation like "not bad" or "wasn't great")
# 1. Score them all with VADER's compound score
# 2. Build a small TF-IDF + LogisticRegression pipeline using bigrams and a
#    negation-aware stopword list, trained on a SEPARATE small labeled set
# 3. Compare which approach handles the negation examples correctly
# 4. Write a genuinely sarcastic review and confirm BOTH approaches likely fail on it --
#    this failure is itself the useful data point, motivating file 12's bridge to
#    context-aware transformer models
```

Deliberately testing sarcasm and confirming BOTH classical approaches genuinely struggle with it is a valuable, honest exercise — not every technique needs to "work" for the exercise to be worthwhile; understanding WHERE and WHY a technique breaks down is real, practical knowledge.

## What's next
File 09 — Topic Modeling (LDA), shifting from SUPERVISED classification (sentiment, spam) to UNSUPERVISED discovery of themes/topics within a collection of documents.
