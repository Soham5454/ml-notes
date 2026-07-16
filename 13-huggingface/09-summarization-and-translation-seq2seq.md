# 09 — Summarization & Translation (Seq2Seq Models)

Recap file 08's closing note — extending generation into ENCODER-DECODER (Seq2Seq) architectures, genuinely combining BERT-style understanding (file 07's encoder) with GPT-style generation (file 08's decoder) in a single model.

## The encoder-decoder architecture — why these tasks need BOTH halves

```python
# Summarization/Translation genuinely need TWO distinct capabilities:
# 1. UNDERSTAND the full input (like BERT's bidirectional encoder, files 05-07)
# 2. GENERATE new, coherent output text (like GPT's autoregressive decoder, file 08)
# A pure encoder (BERT) can't generate. A pure decoder (GPT) can only see LEFT context
# while generating, genuinely limiting its ability to fully process a long input FIRST
```

Genuinely the core architectural insight: an encoder-decoder model has an ENCODER that processes the entire input bidirectionally (building a rich understanding, recap PyTorch file 15's attention), and a separate DECODER that generates output autoregressively (recap file 08), while also attending BACK to the encoder's output at every generation step via **cross-attention** — a genuinely important addition to the self-attention already covered in PyTorch file 15.

## Cross-attention — the piece that's genuinely new here

```python
# Self-attention (PyTorch file 15): tokens attend to OTHER TOKENS IN THE SAME SEQUENCE
# Cross-attention (this file): the DECODER's tokens attend to the ENCODER's output
# This is precisely how the decoder "looks back" at the original input while generating
# each new output token -- e.g., while generating a French translation word, cross-attention
# lets the decoder focus on the RELEVANT English word(s) in the encoder's representation
```

Genuinely worth understanding this as a natural, motivated EXTENSION of PyTorch file 15's Q/K/V attention formula — in cross-attention, the Query comes from the DECODER's current state, but the Key and Value come from the ENCODER's output. Same underlying `softmax(Q@K^T/sqrt(d_k))@V` formula, just drawing Q from one sequence and K/V from another.

## Summarization with a pipeline

```python
from transformers import pipeline

summarizer = pipeline("summarization", model="facebook/bart-large-cnn")

article = """
The rapid advancement of artificial intelligence has transformed numerous industries
over the past decade. From healthcare diagnostics to autonomous vehicles, machine
learning models are increasingly making decisions that impact daily life. However,
this progress raises important questions about safety, fairness, and accountability
that researchers and policymakers are still working to address.
"""

summary = summarizer(article, max_length=50, min_length=15, do_sample=False)
print(summary[0]['summary_text'])
```

Genuinely worth noticing this uses BART, an encoder-decoder model — distinct from file 05's BERT (encoder-only) and file 08's GPT (decoder-only), precisely the architectural distinction just covered.

## Translation with a pipeline

```python
translator = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
result = translator("Machine learning is fascinating")
print(result[0]['translation_text'])
```

## Manual Seq2Seq generation — recap file 08's manual approach, now with an encoder involved

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM

model_name = "facebook/bart-large-cnn"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSeq2SeqLM.from_pretrained(model_name)

inputs = tokenizer(article, return_tensors="pt", max_length=1024, truncation=True)
summary_ids = model.generate(inputs['input_ids'], max_length=50, min_length=15, num_beams=4)
print(tokenizer.decode(summary_ids[0], skip_special_tokens=True))
```

Recap file 08's `num_beams` note directly — beam search is genuinely used constantly for summarization/translation specifically, since these tasks benefit strongly from optimizing the OVERALL output sequence quality rather than greedily committing word-by-word, which can more easily go astray in longer, structured generation tasks like these.

## Fine-tuning a summarization model — recap files 05-07's Trainer pattern

```python
from datasets import load_dataset

dataset = load_dataset("cnn_dailymail", "3.0.0")

def preprocess_function(examples):
    inputs = tokenizer(examples['article'], max_length=512, truncation=True)
    labels = tokenizer(text_target=examples['highlights'], max_length=128, truncation=True)      # note: text_target
    inputs['labels'] = labels['input_ids']
    return inputs

tokenized_dataset = dataset.map(preprocess_function, batched=True)
```

Genuinely important, new detail worth flagging: `text_target=` tokenizes the SUMMARY (the target output) using the tokenizer's DECODER-side settings, distinct from tokenizing the regular input — necessary because encoder and decoder inputs are genuinely handled slightly differently internally for Seq2Seq models.

```python
from transformers import Seq2SeqTrainingArguments, Seq2SeqTrainer, DataCollatorForSeq2Seq

data_collator = DataCollatorForSeq2Seq(tokenizer=tokenizer, model=model)      # recap file 04/06's DataCollator pattern

training_args = Seq2SeqTrainingArguments(
    output_dir="./summarization_results",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    predict_with_generate=True          # genuinely important -- tells the trainer to use .generate() during eval,
                                           # not just compute loss, since generation quality is what actually matters
)

trainer = Seq2SeqTrainer(
    model=model, args=training_args,
    train_dataset=tokenized_dataset['train'],
    eval_dataset=tokenized_dataset['validation'],
    data_collator=data_collator, tokenizer=tokenizer
)
trainer.train()
```

Genuinely worth recognizing `Seq2SeqTrainer`/`Seq2SeqTrainingArguments` as SPECIALIZED versions of files 05-07's `Trainer`/`TrainingArguments` — the same underlying pattern, extended specifically to handle the generate-during-eval need that classification/token-classification tasks didn't have.

## Evaluating with ROUGE — recap Classical NLP file 10 directly

```python
import evaluate
rouge = evaluate.load("rouge")

def compute_metrics(eval_pred):
    predictions, labels = eval_pred
    decoded_preds = tokenizer.batch_decode(predictions, skip_special_tokens=True)
    decoded_labels = tokenizer.batch_decode(labels, skip_special_tokens=True)
    return rouge.compute(predictions=decoded_preds, references=decoded_labels)
```

Genuinely the exact ROUGE metric from Classical NLP file 10, now computed via the standardized `evaluate` library instead of the `rouge_score` package used there directly — same underlying formula, same recall-oriented interpretation, same honest limitations (semantic-blindness, recap that file's BLEU discussion) still apply here.

## T5 — a genuinely elegant "everything is text-to-text" framing

```python
from transformers import T5Tokenizer, T5ForConditionalGeneration

tokenizer = T5Tokenizer.from_pretrained("t5-small")
model = T5ForConditionalGeneration.from_pretrained("t5-small")

# T5 genuinely frames EVERY task as text-to-text, using task PREFIXES
input_text = "summarize: " + article
inputs = tokenizer(input_text, return_tensors="pt", max_length=512, truncation=True)
output = model.generate(inputs['input_ids'], max_length=50)
print(tokenizer.decode(output[0], skip_special_tokens=True))

# Translation with the SAME model, just a different prefix
translation_input = "translate English to German: " + "The weather is nice today"
```

Genuinely worth knowing T5's philosophy explicitly — rather than needing separate task-specific model heads (recap file 03's `AutoModelForX` variety), T5 reframes summarization, translation, classification, and more ALL as "generate the right text given this text and a task description prefix." A genuinely elegant unification, and conceptually a real step toward the instruction-following paradigm that dominates the large language models covered in the upcoming Frontier GenAI phase.

## Try this

```python
# Using the pretrained BART summarization pipeline, summarize a genuinely long
# article (500+ words) of your choice, trying 3 different max_length values
# (20, 50, 100) and comparing how the summary quality/completeness changes
# Then use the Helsinki-NLP translation pipeline to translate a paragraph to a
# language you know, and manually judge the translation quality against your
# own understanding -- note any genuinely awkward phrasings, which is a good
# reminder that even strong pretrained models have real, honest limitations
```

Manually judging translation quality against personal language knowledge is genuinely a good calibration exercise — building a realistic sense of where these models are impressively strong versus where they still make real mistakes.

## What's next
File 10 — Evaluation with Hugging Face, formalizing the `evaluate` library used across files 05-09 and covering the broader landscape of metrics for different task types.
