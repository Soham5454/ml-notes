# Classical NLP — A to Z

Notes on classical (pre-transformer) NLP — the missing bridge between PyTorch's brief tokenization/embedding mentions and the full Hugging Face phase ahead. Covers text preprocessing, vectorization, classical text tasks, and evaluation metrics, with every file cross-referencing back to PyTorch, Math Foundations, DSA, and scikit-learn.

## Index

| # | Topic | File |
|---|---|---|
| 01 | Text Preprocessing Fundamentals | [01-text-preprocessing-fundamentals.md](01-text-preprocessing-fundamentals.md) |
| 02 | Stemming & Lemmatization | [02-stemming-and-lemmatization.md](02-stemming-and-lemmatization.md) |
| 03 | Bag of Words & Text Vectorization | [03-bag-of-words-and-text-vectorization.md](03-bag-of-words-and-text-vectorization.md) |
| 04 | TF-IDF | [04-tf-idf.md](04-tf-idf.md) |
| 05 | N-grams & Language Modeling Basics | [05-ngrams-and-language-modeling-basics.md](05-ngrams-and-language-modeling-basics.md) |
| 06 | Word Embeddings: Word2Vec & GloVe | [06-word-embeddings-word2vec-glove.md](06-word-embeddings-word2vec-glove.md) |
| 07 | Text Classification with Classical ML | [07-text-classification-with-classical-ml.md](07-text-classification-with-classical-ml.md) |
| 08 | Sentiment Analysis | [08-sentiment-analysis.md](08-sentiment-analysis.md) |
| 09 | Topic Modeling (LDA) | [09-topic-modeling-lda.md](09-topic-modeling-lda.md) |
| 10 | NLP Evaluation Metrics (BLEU, ROUGE, Perplexity) | [10-nlp-evaluation-metrics.md](10-nlp-evaluation-metrics.md) |
| 11 | Named Entity Recognition & POS Tagging | [11-ner-and-pos-tagging.md](11-ner-and-pos-tagging.md) |
| 12 | Classical NLP → Hugging Face Bridge | [12-classical-nlp-to-huggingface-bridge.md](12-classical-nlp-to-huggingface-bridge.md) |

## How to read these
Files 01-02 (preprocessing) feed into files 03-06 (representation/vectorization), which feed into files 07-09 (tasks: classification, sentiment, topic modeling). File 10 covers evaluation, file 11 covers structured extraction (NER/POS), and file 12 closes the loop with a full concept map into Hugging Face.

## Roadmap position
Python Basics → NumPy → pandas → Matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Math Foundations → DSA → SQL → **Classical NLP (done)** → Hugging Face (next) → OpenCV → Flask → MLOps → Cloud → Frontier GenAI → ML System Design → Interview Prep
