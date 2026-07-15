# 01 — Text Preprocessing Fundamentals

Starting the Classical NLP phase — genuinely the missing bridge flagged back when planning the roadmap: PyTorch file 11's `nn.Embedding` and file 15's tokenization notes assumed text was already numbers-ready. This folder covers everything BEFORE that point.

## Why raw text can't go directly into any model

Recap PyTorch file 11's embedding note — every model (classical ML or deep learning) needs NUMBERS as input, not raw strings. Text preprocessing is the pipeline that turns messy human language into a clean, consistent, numeric-ready form.

## Setting up — NLTK and basic text

```python
import nltk
nltk.download('punkt')
nltk.download('stopwords')
nltk.download('wordnet')

text = "The quick brown foxes are jumping over the lazy dogs! Isn't that amazing?"
```

`nltk` (Natural Language Toolkit) is genuinely one of the standard classical NLP libraries in Python — the downloads above pull small reference datasets (tokenizer rules, stopword lists, a lemmatization dictionary) needed for the functions in this whole file.

## Lowercasing — the simplest, most common first step

```python
text_lower = text.lower()
print(text_lower)
```

Genuinely important reasoning: without lowercasing, "The" and "the" are treated as completely DIFFERENT tokens by any downstream vectorization (file 03) — inflating vocabulary size and splitting what should be the same signal across two separate features. Worth knowing the honest exception: lowercasing can lose real information in some cases (e.g. "US" the country vs "us" the pronoun) — a real tradeoff, not a rule to apply blindly everywhere.

## Tokenization — splitting text into units

```python
from nltk.tokenize import word_tokenize, sent_tokenize

words = word_tokenize(text)
print(words)   # ['The', 'quick', 'brown', 'foxes', 'are', 'jumping', ..., '!', 'Is', "n't", 'that', 'amazing', '?']

sentences = sent_tokenize(text)
print(sentences)   # ['The quick brown foxes are jumping over the lazy dogs!', "Isn't that amazing?"]
```

Genuinely more subtle than just splitting on whitespace — `word_tokenize` correctly handles punctuation (`!`, `?` become separate tokens) and contractions (`"Isn't"` correctly splits into `"Is"` and `"n't"`, preserving the negation as a distinguishable unit). A naive `text.split()` would instead produce messy tokens like `"dogs!"` and `"Isn't"` stuck together, hurting the quality of everything downstream.

## Removing punctuation

```python
import string

def remove_punctuation(tokens):
    return [word for word in tokens if word not in string.punctuation]

clean_words = remove_punctuation(words)
print(clean_words)
```

Genuinely a real judgment call, not always a strict "must remove" step — punctuation carries real signal in some tasks (e.g. `!` might correlate with sentiment intensity, covered in file 08). Worth deciding based on the specific downstream task rather than removing it reflexively every time.

## Stopword removal — filtering out low-information common words

```python
from nltk.corpus import stopwords

stop_words = set(stopwords.words('english'))
print(list(stop_words)[:10])   # ['the', 'a', 'an', 'is', 'are', ...]

filtered_words = [w for w in clean_words if w.lower() not in stop_words]
print(filtered_words)   # ['quick', 'brown', 'foxes', 'jumping', 'lazy', 'dogs', 'amazing']
```

Genuinely the core idea: words like "the," "is," "a" appear CONSTANTLY across almost every English sentence, carrying very little discriminating information for most tasks (classification, topic modeling) — removing them reduces noise and vocabulary size. Important honest caveat worth flagging: stopword removal can genuinely HURT some tasks — sentiment analysis (file 08) sometimes needs words like "not" (a common stopword) preserved, since removing it can flip a sentence's actual meaning. Always a task-dependent decision, never a universal default.

## Removing numbers and special characters — regex-based cleaning

```python
import re

def clean_text(text):
    text = re.sub(r'\d+', '', text)              # remove digits
    text = re.sub(r'[^\w\s]', '', text)             # remove non-alphanumeric, non-whitespace characters
    text = re.sub(r'\s+', ' ', text).strip()          # collapse multiple spaces into one, trim
    return text

print(clean_text("Hello!!! I have 123 apples...   and 456 oranges."))
# "Hello I have  apples   and  oranges"  (spaces collapsed by the final regex)
```

Genuinely worth knowing basic regex patterns for text cleaning — `\d` matches digits, `\w` matches word characters (letters/digits/underscore), `\s` matches whitespace. These three patterns cover a genuinely large fraction of real-world text cleaning needs.

## Handling contractions properly

```python
contractions_dict = {"isn't": "is not", "aren't": "are not", "won't": "will not", "can't": "cannot"}

def expand_contractions(text):
    for contraction, expansion in contractions_dict.items():
        text = text.replace(contraction, expansion)
    return text

print(expand_contractions("I can't believe it isn't working"))
# "I cannot believe it is not working"
```

Genuinely useful for tasks where the "not" needs to be a clearly separate, unambiguous token — expanding contractions BEFORE tokenization avoids relying on the tokenizer's own contraction-splitting behavior (which, recap the `word_tokenize` example above, does handle SOME contractions reasonably but not perfectly for every edge case).

## The full preprocessing pipeline — putting it all together

```python
def preprocess_text(text, remove_stopwords=True):
    text = text.lower()
    text = expand_contractions(text)
    text = clean_text(text)
    tokens = word_tokenize(text)
    if remove_stopwords:
        tokens = [w for w in tokens if w not in stop_words]
    return tokens

result = preprocess_text("The quick brown foxes aren't lazy at all!")
print(result)   # ['quick', 'brown', 'foxes', 'lazy']
```

Genuinely the realistic shape of a preprocessing function used at the start of almost every classical NLP pipeline in this whole folder — each subsequent file (stemming/lemmatization, vectorization, classification) assumes clean, tokenized input roughly like this as its starting point.

## Try this

```python
# Take 3 genuinely messy real-world text samples (a tweet with hashtags/emojis-as-text,
# a product review with numbers and exclamation marks, a formal paragraph)
# Run each through the full preprocess_text() pipeline above
# For EACH one, explicitly decide and justify: should stopwords be removed for this
# specific task? Should punctuation be kept? Write out the reasoning, not just the code
```

Explicitly reasoning through the task-dependent tradeoffs (stopwords, punctuation) for different text TYPES is genuinely the real skill this file builds — preprocessing isn't one-size-fits-all, and interview/practical questions often probe exactly this judgment.

## What's next
File 02 — Stemming & Lemmatization, reducing words to their root forms — the next standard step after tokenization in most classical NLP pipelines.
