# 03 — AutoModel & Pipelines

Recap files 01-02 — this file covers the two genuinely distinct ways to RUN a Hugging Face model: the highest-level `pipeline` API (fastest to use), and the full manual `AutoModel` + tokenizer control (most flexible, needed for fine-tuning later).

## `pipeline` — the fastest way to get a working result

```python
from transformers import pipeline

classifier = pipeline("sentiment-analysis")
result = classifier("I absolutely loved this movie!")
print(result)   # [{'label': 'POSITIVE', 'score': 0.9998}]
```

Genuinely astonishing how little code this is compared to Classical NLP file 08's full sentiment pipeline (VADER lexicon rules, or a trained TF-IDF+LogisticRegression model) — `pipeline()` downloads a sensible DEFAULT pretrained model for the task, handles tokenization, runs inference, and post-processes the output, all in one call.

## Common pipeline tasks — genuinely covering most classical NLP file topics directly

```python
# Text classification (recap Classical NLP file 07)
classifier = pipeline("text-classification")

# Named Entity Recognition (recap Classical NLP file 11)
ner = pipeline("ner", grouped_entities=True)
print(ner("Apple was founded by Steve Jobs in California"))

# Question Answering (full treatment file 07 of THIS folder)
qa = pipeline("question-answering")
print(qa(question="Who founded Apple?", context="Apple was founded by Steve Jobs in California"))

# Text generation (full treatment file 08)
generator = pipeline("text-generation")
print(generator("The future of AI is", max_length=30))

# Summarization (full treatment file 09)
summarizer = pipeline("summarization")

# Translation
translator = pipeline("translation_en_to_fr")

# Zero-shot classification -- genuinely powerful, no training data needed at all
zero_shot = pipeline("zero-shot-classification")
print(zero_shot("This is a tutorial about machine learning", candidate_labels=["education", "sports", "cooking"]))
```

Genuinely worth pausing on `zero-shot-classification` specifically — recap Classical NLP file 07's classification pipeline, which required LABELED TRAINING DATA. Zero-shot classification uses a model trained differently (typically on Natural Language Inference) to classify text into ARBITRARY categories NEVER SEEN during training, just by describing the categories in plain language. Genuinely one of the clearest demonstrations of how much more FLEXIBLE pretrained transformer understanding is compared to the classical, must-be-trained-per-task approach from that whole earlier folder.

## Specifying a particular model in a pipeline

```python
classifier = pipeline("sentiment-analysis", model="distilbert-base-uncased-finetuned-sst-2-english")
```

Genuinely important practical habit — pipelines use a DEFAULT model if none is specified, which is fine for quick experimentation but often not the best choice for a specific real use case. Explicitly naming a model (browsing the Hub, recap file 01) is the right practice for anything beyond casual testing.

## Batch processing with pipelines

```python
texts = ["I loved it!", "Terrible experience", "It was okay I guess"]
results = classifier(texts)
for text, result in zip(texts, results):
    print(f"{text} -> {result['label']} ({result['score']:.3f})")
```

Recap PyTorch file 08's batching motivation — pipelines handle batching internally, genuinely more efficient than looping and calling the pipeline on one text at a time.

## The manual approach — AutoModel + tokenizer, full control

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

model_name = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

inputs = tokenizer("I absolutely loved this movie!", return_tensors="pt")

with torch.no_grad():          # recap PyTorch file 03 -- no gradients needed for inference
    outputs = model(**inputs)

print(outputs.logits)            # raw logits, recap PyTorch file 06's cross-entropy note --
                                    # NOT yet softmaxed into probabilities

probs = torch.softmax(outputs.logits, dim=-1)      # recap PyTorch file 05's softmax
print(probs)

predicted_class = torch.argmax(probs, dim=-1).item()
print(model.config.id2label[predicted_class])          # maps the class index back to a readable label
```

Genuinely this is EXACTLY what `pipeline("sentiment-analysis")` does internally — the manual version reveals every step: tokenize, forward pass (`model(**inputs)`, recap PyTorch's `**kwargs` unpacking pattern from file 19), extract raw logits, softmax, argmax, map back to a label. Worth building this manually at least once — `pipeline` is genuinely convenient, but understanding what it's hiding matters for debugging and for the fine-tuning work coming in file 05.

## `AutoModelFor...` — task-specific model heads

```python
from transformers import (
    AutoModel,                          # base model, no task-specific head -- raw embeddings/hidden states
    AutoModelForSequenceClassification,   # adds a classification head on top (recap file 02's [CLS] token note)
    AutoModelForTokenClassification,        # adds a per-token classification head (NER, file 06)
    AutoModelForQuestionAnswering,            # adds a span-prediction head (file 07)
    AutoModelForCausalLM,                       # adds a next-token-prediction head (GPT-style generation, file 08)
    AutoModelForSeq2SeqLM                          # encoder-decoder head (translation/summarization, file 09)
)
```

Genuinely the key architectural insight worth understanding: the SAME pretrained base model's weights (e.g. BERT's core transformer layers) get REUSED across all these variants — only the final TASK-SPECIFIC HEAD (a small additional layer, recap PyTorch file 04's `nn.Linear` — genuinely just one or two more layers on top) differs. This is precisely why the AutoClass naming convention exists — `AutoModelForX` tells the library exactly which head to attach to the shared base architecture.

## Pipeline vs manual approach — the honest decision guide

```
Quick prototyping, exploring what a task/model can do          -> pipeline()
Need to inspect intermediate outputs (embeddings, attention)     -> manual AutoModel + tokenizer
Fine-tuning a model on custom data                                -> manual approach REQUIRED (file 05)
Production inference, need fine control over batching/latency       -> manual approach, usually
Simple, one-off classification/generation task                        -> pipeline() is genuinely fine
```

## Try this

```python
# Use the zero-shot-classification pipeline to classify 5 genuinely different sentences
# into categories of your choice (not standard sentiment/topic categories -- try something
# creative like ["urgent", "casual", "formal"])
# Then take ONE of those sentences and replicate the same classification using the manual
# AutoModel + tokenizer approach for regular sentiment-analysis instead, printing the raw
# logits, softmaxed probabilities, and final predicted label at each step
```

Building the manual pipeline by hand, comparing its output against the equivalent `pipeline()` call, is genuinely the clearest way to confirm the high-level API isn't doing anything mysterious — just automating exactly the steps just written manually.

## What's next
File 04 — The Datasets Library, moving from single-example inference into properly loading and processing DATA AT SCALE — the necessary foundation before file 05's fine-tuning can happen.
