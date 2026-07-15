# 02 — Stemming & Lemmatization

Recap file 01's preprocessing pipeline — this is the next standard step: reducing words to their ROOT form, so that "running," "runs," and "ran" all get treated as referring to the same underlying concept rather than three separate vocabulary entries.

## Why root-form reduction matters

Without it, a vectorizer (file 03) treats "run," "running," "runs," and "ran" as FOUR completely unrelated tokens — genuinely fragmenting what should be one strong signal across multiple weak ones, and needlessly inflating vocabulary size (recap file 01's lowercasing reasoning — same underlying motivation).

## Stemming — crude, rule-based, fast

```python
from nltk.stem import PorterStemmer

stemmer = PorterStemmer()

words = ["running", "runs", "ran", "easily", "fairly", "studies", "studying"]
stemmed = [stemmer.stem(w) for w in words]
print(stemmed)   # ['run', 'run', 'ran', 'easili', 'fairli', 'studi', 'studi']
```

Genuinely important, commonly-surprising observation: stemming can produce results that AREN'T real words (`"easili"`, `"studi"`) — the Porter Stemmer works by chopping off common suffixes using a fixed set of RULES, without any actual understanding of the language. It's fast and simple, but genuinely crude — notice `"ran"` didn't get reduced to `"run"` at all, since rule-based suffix-stripping can't handle irregular verb forms.

```python
# Different stemmers exist with different rule aggressiveness
from nltk.stem import SnowballStemmer

snowball = SnowballStemmer('english')
print(snowball.stem('running'))   # 'run' -- genuinely a refined, slightly more accurate successor to Porter
```

## Lemmatization — dictionary-based, slower, more accurate

```python
from nltk.stem import WordNetLemmatizer

lemmatizer = WordNetLemmatizer()

print(lemmatizer.lemmatize('running'))          # 'running' -- WITHOUT part-of-speech info, defaults to noun!
print(lemmatizer.lemmatize('running', pos='v'))    # 'run' -- CORRECT, when told it's a verb
print(lemmatizer.lemmatize('better', pos='a'))       # 'good' -- genuinely handles irregular forms correctly
```

Genuinely the key advantage over stemming: lemmatization uses an actual DICTIONARY (WordNet) and returns REAL words — `"better"` correctly lemmatizes to `"good"`, something rule-based stemming could never achieve since there's no suffix-stripping rule that could derive that relationship. The tradeoff: lemmatization needs to know the word's PART OF SPEECH (`pos`) to work correctly, and is computationally slower than stemming.

## Stemming vs Lemmatization — the honest, practical comparison

| | Stemming | Lemmatization |
|---|---|---|
| Method | Rule-based suffix stripping | Dictionary lookup |
| Speed | Fast | Slower |
| Output | May not be a real word | Always a real word |
| Accuracy | Cruder, can over/under-stem | More accurate |
| Needs POS info | No | Yes, for best results |
| When to use | Large-scale, speed-critical, approximate needs (e.g. search engine indexing) | Accuracy-critical tasks, readability matters |

Genuinely a real engineering tradeoff — a search engine indexing billions of documents might prefer stemming's speed even with crude output, while a chatbot or text-generation-adjacent task genuinely needs lemmatization's real-word accuracy.

## Getting part-of-speech tags for proper lemmatization

```python
from nltk import pos_tag
from nltk.tokenize import word_tokenize

sentence = "The foxes are running quickly"
tokens = word_tokenize(sentence)
tags = pos_tag(tokens)
print(tags)   # [('The', 'DT'), ('foxes', 'NNS'), ('are', 'VBP'), ('running', 'VBG'), ('quickly', 'RB')]
```

`pos_tag` labels each token with its grammatical role (`NN`=noun, `VB`=verb, `JJ`=adjective, `RB`=adverb, etc — full treatment in file 11). Genuinely needed as an input to lemmatization for it to work correctly at scale, since manually specifying `pos='v'` for every word (as in the earlier example) isn't practical for real text.

```python
def get_wordnet_pos(treebank_tag):          # converts NLTK's tags into WordNet's simpler format
    if treebank_tag.startswith('J'):
        return 'a'      # adjective
    elif treebank_tag.startswith('V'):
        return 'v'        # verb
    elif treebank_tag.startswith('R'):
        return 'r'          # adverb
    else:
        return 'n'            # default to noun

def lemmatize_sentence(sentence):
    tokens = word_tokenize(sentence)
    tagged = pos_tag(tokens)
    return [lemmatizer.lemmatize(word, get_wordnet_pos(tag)) for word, tag in tagged]

print(lemmatize_sentence("The foxes are running quickly"))
# ['The', 'fox', 'be', 'run', 'quickly']
```

Genuinely the realistic, complete lemmatization pipeline — POS tagging feeding directly into lemmatization, rather than guessing or defaulting to noun for every word (which would have incorrectly left "running" unchanged, as shown in the earlier standalone example).

## Combining with file 01's full pipeline

```python
def full_preprocess(text):
    text = text.lower()
    text = clean_text(text)                          # recap file 01
    tokens = word_tokenize(text)
    tokens = [w for w in tokens if w not in stop_words]   # recap file 01
    tagged = pos_tag(tokens)
    lemmatized = [lemmatizer.lemmatize(w, get_wordnet_pos(t)) for w, t in tagged]
    return lemmatized

print(full_preprocess("The foxes were running quickly through the studies"))
```

## Try this

```python
# Take the word list ["better", "geese", "wolves", "mice", "went", "am"]
# 1. Run each through PorterStemmer and observe how many produce non-words or fail
#    to find the correct root
# 2. Run each through WordNetLemmatizer WITH correct POS tags and compare results
# This is genuinely the clearest way to see stemming's crudeness versus lemmatization's
# accuracy on IRREGULAR word forms specifically, which is exactly where the two techniques
# diverge most dramatically
```

Irregular forms (plurals like "geese"/"mice," irregular verbs like "went") are genuinely the sharpest test case for seeing WHY lemmatization's dictionary-based approach outperforms stemming's rule-based guessing — regular words often stem/lemmatize similarly, but irregulars expose the real gap.

## What's next
File 03 — Bag of Words & Text Vectorization, finally converting the clean tokens from files 01-02 into actual NUMBERS a model can use.
