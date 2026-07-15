# 07 — Question Answering with Transformers

Recap file 06's closing note and file 02's sentence-pair tokenization preview — this file covers extractive Question Answering: given a QUESTION and a CONTEXT passage, predict the SPAN of text within the context that answers it.

## The task, precisely defined

```python
context = "The Eiffel Tower is located in Paris, France. It was completed in 1889 and stands 330 meters tall."
question = "How tall is the Eiffel Tower?"

# The model doesn't GENERATE an answer -- it predicts the START and END position
# of the answer SPAN within the context: "330 meters"
```

Genuinely important distinction worth being precise about: this is EXTRACTIVE QA — the answer must be a literal, contiguous span copied from the context. This is genuinely different from GENERATIVE QA (where a model writes a novel answer in its own words, closer to file 08's text generation) — extractive QA is more constrained but also more verifiable/grounded, since the answer can always be traced back to an exact location in the source text.

## Using the pretrained pipeline first — recap file 03

```python
from transformers import pipeline

qa_pipeline = pipeline("question-answering")
result = qa_pipeline(question=question, context=context)
print(result)
# {'score': 0.89, 'start': 78, 'end': 88, 'answer': '330 meters'}
```

Genuinely worth noticing the output structure — `start`/`end` are CHARACTER positions in the original context string, `answer` is the extracted text, `score` is the model's confidence.

## How the model actually predicts a span — the architecture, briefly

```python
# The QA model outputs TWO separate probability distributions over EVERY token position:
# 1. start_logits -- how likely each token is to be the START of the answer
# 2. end_logits -- how likely each token is to be the END of the answer
# The final answer span is the (start, end) pair with the highest COMBINED probability
```

Recap PyTorch file 05's softmax notes — genuinely two separate classification problems running in parallel over the same token sequence, each token getting a "start-ness" score and an "end-ness" score, rather than one single classification per token like file 06's NER.

## Fine-tuning setup — recap file 02's sentence-pair tokenization

```python
from datasets import load_dataset
from transformers import AutoTokenizer, AutoModelForQuestionAnswering

dataset = load_dataset("squad")      # the genuinely standard QA benchmark dataset
model_name = "distilbert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForQuestionAnswering.from_pretrained(model_name)

print(dataset['train'][0])
# {'question': '...', 'context': '...', 'answers': {'text': ['...'], 'answer_start': [515]}}
```

## The genuinely tricky preprocessing — mapping CHARACTER positions to TOKEN positions

```python
def preprocess_qa(examples):
    questions = [q.strip() for q in examples['question']]
    inputs = tokenizer(
        questions, examples['context'],
        max_length=384, truncation="only_second",      # only truncate the CONTEXT, never the question
        return_offsets_mapping=True, padding="max_length"
    )

    offset_mapping = inputs.pop("offset_mapping")
    start_positions, end_positions = [], []

    for i, offsets in enumerate(offset_mapping):
        answer = examples['answers'][i]
        start_char = answer['answer_start'][0]
        end_char = start_char + len(answer['text'][0])

        sequence_ids = inputs.sequence_ids(i)          # tells us WHICH tokens belong to the context (vs question)
        context_start = sequence_ids.index(1)
        context_end = len(sequence_ids) - 1 - sequence_ids[::-1].index(1)

        if offsets[context_start][0] > start_char or offsets[context_end][1] < end_char:
            start_positions.append(0)          # answer isn't fully within this truncated window
            end_positions.append(0)
        else:
            idx = context_start
            while idx <= context_end and offsets[idx][0] <= start_char:
                idx += 1
            start_positions.append(idx - 1)
            idx = context_end
            while idx >= context_start and offsets[idx][1] >= end_char:
                idx -= 1
            end_positions.append(idx + 1)

    inputs["start_positions"] = start_positions
    inputs["end_positions"] = end_positions
    return inputs

tokenized_squad = dataset.map(preprocess_qa, batched=True, remove_columns=dataset['train'].column_names)
```

Genuinely the most complex preprocessing code in this whole folder, worth understanding the CORE idea even if the exact implementation feels dense: the dataset gives the answer's position in CHARACTERS within the raw context string, but the model needs the answer's position in TOKEN INDICES after tokenization (recap file 02's subword splitting — character positions and token positions genuinely don't line up 1-to-1). `return_offsets_mapping=True` gives, for every token, exactly which character range it corresponds to in the original text — this mapping is precisely what makes converting character positions to token positions possible.

`truncation="only_second"` is a genuinely important detail — recap file 02's sentence-pair tokenization, where the question comes FIRST and context SECOND; if the combined length exceeds `max_length`, only truncating the "second" sequence (context) preserves the full question, which is obviously essential.

## Training — recap file 05/06's Trainer pattern, genuinely unchanged in shape

```python
from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(
    output_dir="./qa_results",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    evaluation_strategy="epoch"
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_squad['train'],
    eval_dataset=tokenized_squad['validation'],
    tokenizer=tokenizer
)
trainer.train()
```

Genuinely worth recognizing the SAME pattern for the third time now (files 05, 06, 07) — `Trainer` genuinely doesn't care what the specific task is, as long as the model outputs a loss (recap PyTorch file 19's `outputs.loss` note) and the preprocessing correctly shapes the labels (here: `start_positions`/`end_positions` instead of file 05's single class label or file 06's per-token labels).

## Extracting the actual answer from raw model outputs — the manual version

```python
import torch

inputs = tokenizer(question, context, return_tensors="pt")
with torch.no_grad():
    outputs = model(**inputs)

start_idx = torch.argmax(outputs.start_logits)
end_idx = torch.argmax(outputs.end_logits)

answer_tokens = inputs['input_ids'][0][start_idx:end_idx+1]
answer = tokenizer.decode(answer_tokens)
print(answer)   # "330 meters"
```

Genuinely closes the loop with the pipeline example from earlier — this manual version is EXACTLY what `qa_pipeline()` does internally, using `argmax` (recap PyTorch file 05) on both logit arrays to find the most likely start and end positions, then decoding (recap file 02) the token span in between.

## The honest limitation — what happens when there's no answer

```python
# SQuAD 2.0 (an extension of the dataset used above) includes UNANSWERABLE questions --
# genuinely important real-world case: a good QA model should be able to say
# "this context doesn't contain the answer" rather than confidently extracting
# a WRONG span just because it has to output SOMETHING
```

Genuinely worth knowing this exists as a real, harder version of the task — handling unanswerable questions typically involves a special "no answer" position (often pointing to the `[CLS]` token, recap file 02) that the model can predict when appropriate, rather than always being forced to extract some span.

## Try this

```python
# Using the pretrained QA pipeline (no fine-tuning needed for this exercise), write
# 3 context passages on a topic of your choice, each with 2 questions
# For ONE of the 6 questions, deliberately ask something the passage does NOT
# actually answer, and observe how the pipeline responds -- does it give a low
# confidence score, or does it confidently extract an irrelevant span?
# This is genuinely a good way to build honest calibration about extractive QA's
# real limitations, directly connecting to this file's "no answer" discussion
```

Deliberately probing the model with an unanswerable question is genuinely the most instructive part of this exercise — seeing firsthand how a model trained without explicit "no answer" handling behaves when faced with one.

## Hugging Face — halfway point. What's next
Files 01-07 covered the fine-tuning workflow for classification, token classification, and question answering — genuinely the core Trainer-based pattern, now internalized. File 08 shifts into GENERATIVE tasks — Text Generation with GPT-style causal language models.
