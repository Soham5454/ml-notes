# 11 — Named Entity Recognition & POS Tagging

Recap file 02's brief `pos_tag` usage for lemmatization — this file gives POS tagging its full, proper treatment, and introduces NER, the last classical NLP TASK category before this folder's Hugging Face bridge.

## Part-of-Speech (POS) Tagging — full treatment

```python
import nltk
from nltk import pos_tag, word_tokenize

sentence = "The quick brown fox jumps over the lazy dog"
tokens = word_tokenize(sentence)
tags = pos_tag(tokens)
print(tags)
# [('The', 'DT'), ('quick', 'JJ'), ('brown', 'JJ'), ('fox', 'NN'), ('jumps', 'VBZ'),
#  ('over', 'IN'), ('the', 'DT'), ('lazy', 'JJ'), ('dog', 'NN')]
```

## The genuinely important POS tag categories to recognize

```
DT  - determiner (the, a, an)
JJ  - adjective
NN  - noun (singular)         NNS - noun (plural)
VB  - verb (base form)         VBZ - verb (3rd person singular present)
RB  - adverb
IN  - preposition
PRP - pronoun
CC  - coordinating conjunction (and, or, but)
```

Recap file 02's `get_wordnet_pos` function directly — these ARE the exact tags that function was translating into WordNet's simpler noun/verb/adjective/adverb categories for lemmatization. Full tag reference available via `nltk.help.upenn_tagset()`.

## How POS taggers actually work — briefly, the classical approach

```python
# Classical POS taggers (like NLTK's default) are trained using statistical models
# (Hidden Markov Models, or Maximum Entropy classifiers) on large HAND-LABELED corpora,
# learning patterns like: "a word following 'the' is very likely a noun or adjective"
# Genuinely related to file 05's n-gram language modeling -- POS tagging uses similar
# SEQUENCE-based statistical reasoning, just predicting a GRAMMATICAL TAG instead of
# the next WORD
```

Genuinely worth recognizing this connection — POS tagging is fundamentally a SEQUENCE LABELING task (recap PyTorch file 11's RNN motivation — assigning a label to EVERY element in a sequence, using surrounding context), which is precisely why modern POS taggers increasingly use RNN/transformer-based architectures rather than classical HMMs, for the same reasons neural sequence models outperformed n-gram language models (recap file 05's honest limitation section).

## Named Entity Recognition (NER) — extracting structured entities from text

```python
import spacy

nlp = spacy.load("en_core_web_sm")      # spaCy, genuinely a more production-oriented NLP library than NLTK

doc = nlp("Apple is looking at buying a UK startup for $1 billion, according to Reuters.")

for ent in doc.ents:
    print(ent.text, ent.label_)
# Apple ORG
# UK GPE
# $1 billion MONEY
# Reuters ORG
```

Genuinely the core task: identify SPANS of text referring to real-world entities (people, organizations, locations, dates, monetary amounts) and classify WHAT KIND of entity each one is. spaCy is genuinely the standard, production-grade library for this — faster and more accurate out-of-the-box than NLTK's built-in NER for most practical use.

## Common entity types

```
PERSON   - people's names
ORG      - organizations, companies
GPE      - geopolitical entities (countries, cities, states)
DATE     - dates or date ranges
MONEY    - monetary values
PERCENT  - percentages
TIME     - times
```

## Why NER is genuinely harder than it first looks

```python
doc = nlp("Washington announced new policies while George Washington's portrait hung nearby")
for ent in doc.ents:
    print(ent.text, ent.label_)
# Washington GPE   (correctly identified as the city/government, from context)
# George Washington PERSON   (correctly identified as a person, from context)
```

Genuinely the SAME word ("Washington") gets correctly classified DIFFERENTLY based on surrounding CONTEXT — this is precisely the same context-dependence challenge flagged in file 06's "bank" example (river bank vs money bank). Real NER systems need genuine contextual understanding, not just a lookup dictionary of known entity names — a classical rule-based/dictionary approach would genuinely fail on cases like this, correctly motivating why modern NER increasingly uses the same contextual embedding techniques (Hugging Face bridge, file 12) that solve file 06's word-sense ambiguity problem.

## Building a simple rule-based NER — for genuinely understanding the classical approach

```python
import re

def simple_date_extractor(text):
    pattern = r'\b(?:January|February|March|April|May|June|July|August|September|October|November|December)\s+\d{1,2},?\s+\d{4}\b'
    return re.findall(pattern, text)

print(simple_date_extractor("The meeting is on March 15, 2026 and the deadline is June 1, 2026"))
# ['March 15, 2026', 'June 1, 2026']
```

Genuinely worth building at least one rule-based extractor by hand — it demonstrates the classical, pre-ML approach to entity extraction (pattern matching via regex, recap file 01's regex introduction), and makes clear WHY it's brittle: this pattern would completely miss "3/15/2026" or "15th March 2026," genuinely different date formats requiring separate, hand-crafted rules for every variation — precisely the maintenance burden that trained NER models are built to avoid.

## Combining POS tagging and NER — a genuinely useful pattern

```python
def extract_noun_phrases(text):
    doc = nlp(text)
    return [chunk.text for chunk in doc.noun_chunks]

print(extract_noun_phrases("The quick brown fox jumps over the lazy dog"))
# ['The quick brown fox', 'the lazy dog']
```

`noun_chunks` genuinely combines POS information (identifying nouns) with dependency parsing (understanding which adjectives/determiners MODIFY which noun) to extract complete meaningful PHRASES, not just individual tagged words — a genuinely practical building block for tasks like keyword/topic extraction (recap file 04's TF-IDF-based keyword extraction — noun phrases are often a MORE meaningful unit than single TF-IDF-ranked words).

## Dependency parsing — a brief, genuinely important preview

```python
doc = nlp("The cat chased the mouse")
for token in doc:
    print(token.text, token.dep_, token.head.text)
# The   det     cat
# cat   nsubj   chased      <- cat is the SUBJECT of "chased"
# chased ROOT   chased
# the   det     mouse
# mouse dobj    chased      <- mouse is the OBJECT of "chased"
```

Genuinely a deeper structural analysis than POS tagging alone — dependency parsing reveals the GRAMMATICAL RELATIONSHIPS between words (who did what to whom), not just each word's individual category. Worth knowing this exists as the more complete classical parsing approach, though full mastery of dependency grammar is genuinely a deep sub-specialty of linguistics/NLP beyond this folder's A-to-Z scope — the practical takeaway is knowing it's available via spaCy for tasks that genuinely need relationship extraction (e.g. building simple information-extraction systems: "who chased what").

## Try this

```python
# Using spaCy, process a short news-style paragraph (write or find 3-4 sentences
# mentioning at least 2 people, 1 organization, 1 location, and 1 date)
# 1. Extract and print all named entities with their types
# 2. Extract all noun_chunks and compare them against the entities -- which noun
#    chunks were NOT flagged as named entities, and does that distinction make sense?
# 3. Manually verify by re-reading the text: did spaCy correctly resolve any
#    ambiguous words based on context, similar to the Washington example in this file?
```

Manually cross-checking spaCy's output against a careful human reading is genuinely the best way to build calibrated trust in these tools — understanding both where they work impressively well and where they still make genuine mistakes.

## Classical NLP — almost complete. What's next
File 12 — the final Classical NLP → Hugging Face Bridge, pulling every technique from files 01-11 together and connecting them explicitly to the transformer-based tools this whole folder has been building toward.
