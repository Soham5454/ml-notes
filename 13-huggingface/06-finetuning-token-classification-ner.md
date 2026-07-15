# 06 — Fine-tuning Token Classification (NER)

Recap file 05's closing note and Classical NLP file 11's NER introduction — this file extends the same Trainer-based workflow to a task where EVERY TOKEN needs its own label, not one label for the whole sequence.

## The core structural difference from file 05

```python
# Text classification (file 05): ONE label per ENTIRE sequence
# "This movie was great" -> label: POSITIVE

# Token classification (this file): ONE label PER TOKEN
# "Apple" "was" "founded" "by" "Steve" "Jobs"
#  B-ORG   O      O         O    B-PER   I-PER
```

Genuinely the key new concept: the **BIO tagging scheme** — `B-` (Beginning of an entity), `I-` (Inside/continuation of an entity), `O` (Outside, not part of any entity). "Steve Jobs" needs TWO tokens tagged `B-PER` and `I-PER` specifically to represent it as ONE multi-token entity, rather than two separate single-token entities — genuinely the standard, practical solution to representing multi-word named entities in a per-token labeling scheme.

## Loading a NER dataset

```python
from datasets import load_dataset

dataset = load_dataset("conll2003")      # a genuinely standard, classic NER benchmark dataset
print(dataset['train'][0])
# {'tokens': ['EU', 'rejects', 'German', 'call', ...], 'ner_tags': [3, 0, 7, 0, ...]}

label_names = dataset['train'].features['ner_tags'].feature.names
print(label_names)   # ['O', 'B-PER', 'I-PER', 'B-ORG', 'I-ORG', 'B-LOC', 'I-LOC', 'B-MISC', 'I-MISC']
```

Genuinely important, immediately noticeable difference from file 05: `tokens` is ALREADY a list of pre-split words, not a raw string — and `ner_tags` are already numeric labels PER TOKEN, aligned position-by-position with `tokens`.

## The genuinely tricky part — aligning labels with SUBWORD tokenization

```python
# Recap file 02's subword tokenization -- "Washington" might become ["wash", "##ington"]
# But the ORIGINAL dataset has ONE label for "Washington" as a whole word
# This creates a MISALIGNMENT that needs to be explicitly handled
```

This is genuinely THE core technical challenge of this whole file, worth understanding deeply rather than treating as boilerplate:

```python
def tokenize_and_align_labels(examples):
    tokenized_inputs = tokenizer(examples['tokens'], truncation=True, is_split_into_words=True)

    labels = []
    for i, label in enumerate(examples['ner_tags']):
        word_ids = tokenized_inputs.word_ids(batch_index=i)      # maps each SUBWORD token back to its ORIGINAL word index
        previous_word_idx = None
        label_ids = []
        for word_idx in word_ids:
            if word_idx is None:
                label_ids.append(-100)                              # special tokens ([CLS], [SEP], padding) get -100
            elif word_idx != previous_word_idx:
                label_ids.append(label[word_idx])                     # first subword of a word gets the REAL label
            else:
                label_ids.append(-100)                                  # subsequent subwords of the SAME word get -100
            previous_word_idx = word_idx
        labels.append(label_ids)

    tokenized_inputs['labels'] = labels
    return tokenized_inputs

tokenized_dataset = dataset.map(tokenize_and_align_labels, batched=True)
```

Genuinely important detail worth flagging explicitly: `-100` is a SPECIAL VALUE that PyTorch's `CrossEntropyLoss` (recap PyTorch file 06) is configured to IGNORE by default — meaning special tokens and continuation subwords don't contribute to the loss calculation at all. Only the FIRST subword of each original word gets the real label and genuinely counts toward training — a clean, standard way of handling the alignment problem without needing to invent a new loss function.

`is_split_into_words=True` is the tokenizer flag telling it the input is ALREADY a list of words (not raw text needing its own splitting) — genuinely necessary here since `tokens` (from the dataset) is pre-split, unlike file 05's raw text strings.

## Loading the token classification model

```python
from transformers import AutoModelForTokenClassification

model = AutoModelForTokenClassification.from_pretrained(
    "bert-base-cased",      # NOTE: cased, not uncased -- capitalization is genuinely a strong
                                # signal for named entities (e.g. "Apple" the company vs "apple" the fruit)
    num_labels=len(label_names),
    id2label={i: label for i, label in enumerate(label_names)},
    label2id={label: i for i, label in enumerate(label_names)}
)
```

Genuinely worth flagging the `cased` vs `uncased` choice explicitly — recap Classical NLP file 01's lowercasing caveat ("US" vs "us") — for NER specifically, capitalization is genuinely a strong, useful signal (proper nouns are typically capitalized), so a CASED model is usually the better choice here, unlike file 05's sentiment task where case mattered less.

## Training — genuinely the same Trainer pattern as file 05

```python
from transformers import TrainingArguments, Trainer, DataCollatorForTokenClassification

data_collator = DataCollatorForTokenClassification(tokenizer=tokenizer)      # recap file 04's DataCollator,
                                                                                  # this variant correctly pads
                                                                                  # the LABELS array too, not just input_ids

training_args = TrainingArguments(
    output_dir="./ner_results",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    evaluation_strategy="epoch"
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset['train'],
    eval_dataset=tokenized_dataset['validation'],
    data_collator=data_collator,
    tokenizer=tokenizer
)
trainer.train()
```

Genuinely worth recognizing how much of file 05's `Trainer` setup transfers UNCHANGED — the only real new pieces are the label-alignment preprocessing above and the specialized `DataCollatorForTokenClassification`, which correctly pads the per-token LABELS array (using -100) alongside the usual input padding.

## Evaluation for token classification — genuinely different from simple accuracy

```python
import evaluate
seqeval = evaluate.load("seqeval")      # a genuinely standard NER-specific evaluation metric

def compute_metrics(eval_pred):
    predictions, labels = eval_pred
    predictions = np.argmax(predictions, axis=2)

    true_predictions = [
        [label_names[p] for p, l in zip(pred, label) if l != -100]      # filter out the -100 positions
        for pred, label in zip(predictions, labels)
    ]
    true_labels = [
        [label_names[l] for p, l in zip(pred, label) if l != -100]
        for pred, label in zip(predictions, labels)
    ]

    results = seqeval.compute(predictions=true_predictions, references=true_labels)
    return {"precision": results["overall_precision"], "recall": results["overall_recall"], "f1": results["overall_f1"]}
```

`seqeval` genuinely computes ENTITY-LEVEL precision/recall/F1 (recap scikit-learn's exact definitions), not token-level — meaning a multi-token entity like "Steve Jobs" only counts as CORRECT if BOTH tokens are predicted correctly with the right BIO tags, a genuinely stricter and more meaningful evaluation than naive per-token accuracy would give.

## Using the fine-tuned NER model

```python
from transformers import pipeline

ner_pipeline = pipeline("ner", model=model, tokenizer=tokenizer, aggregation_strategy="simple")
print(ner_pipeline("Elon Musk founded SpaceX in California"))
```

`aggregation_strategy="simple"` genuinely handles merging `B-`/`I-` tagged subwords back into complete, readable entity spans automatically — converting the raw per-token BIO predictions back into the clean, human-readable entity list format seen back in file 03's `ner` pipeline example.

## Try this

```python
# Using the conll2003 dataset, fine-tune bert-base-cased for token classification
# on a small subset (2000-3000 examples) for 2 epochs
# Run the resulting model through the ner pipeline on 3 sentences of your own choosing
# (not from the training data) and manually verify whether it correctly identifies
# PERSON, ORG, and LOC entities
# Deliberately test it on a sentence with an AMBIGUOUS word (recap Classical NLP file 11's
# "Washington" example) and see if your fine-tuned model handles the context correctly
```

Testing on genuinely novel, self-written sentences (not just the training data) is the real test of whether the model learned generalizable entity-recognition patterns rather than memorizing the specific training examples.

## What's next
File 07 — Question Answering with Transformers, another token-level task, but predicting a SPAN of an answer within a context passage rather than classifying every token independently.
