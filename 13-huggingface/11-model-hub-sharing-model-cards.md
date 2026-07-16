# 11 — Model Hub: Sharing, Versioning & Model Cards

Recap file 01's brief Hub mention and file 10's closing note — this file covers the OTHER half of the Hub: not just downloading pretrained models, but properly sharing, versioning, and documenting a fine-tuned model of your own.

## Authenticating with the Hub

```python
from huggingface_hub import login

login()      # prompts for a token, or pass token="hf_..." directly
# Genuinely important practical note: NEVER hardcode a real token in a notebook/script
# that gets shared or committed to a public repo (recap general credential-safety
# habits from the SQL/Flask-adjacent parts of this roadmap) -- use environment
# variables or the interactive login prompt instead
```

## Pushing a fine-tuned model to the Hub

```python
model.push_to_hub("my-username/my-sentiment-model")
tokenizer.push_to_hub("my-username/my-sentiment-model")
```

Genuinely the direct continuation of file 05's `trainer.save_model()` — instead of (or in addition to) saving locally, `push_to_hub` uploads the model weights, config, and tokenizer files to a repository on huggingface.co, making it loadable by ANYONE (or privately, by just you/your team) via the exact same `from_pretrained()` pattern used throughout this whole folder.

## Pushing directly from the Trainer

```python
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="./results",
    push_to_hub=True,                    # automatically pushes checkpoints during training
    hub_model_id="my-username/my-sentiment-model"
)
# After trainer.train() completes:
trainer.push_to_hub()
```

Genuinely convenient — recap file 05's full `Trainer` setup, this is just one additional flag turning on automatic Hub syncing throughout training, rather than needing a separate manual push step afterward.

## Model repository structure — what actually gets uploaded

```
my-sentiment-model/
├── config.json           -- architecture settings (recap file 01's model.config inspection)
├── pytorch_model.bin      -- the actual weights (recap PyTorch file 14's state_dict, essentially)
├── tokenizer_config.json    -- tokenizer settings
├── vocab.txt                 -- the tokenizer's vocabulary (recap file 02)
├── special_tokens_map.json     -- recap file 02's special token definitions
└── README.md                     -- the MODEL CARD, covered next
```

Genuinely worth recognizing this as directly analogous to a GitHub repo (recap the whole ml-notes repo this entire roadmap has been building) — version-controlled, documented, shareable — just specifically structured for ML model artifacts rather than code/notes.

## Writing a proper Model Card — genuinely important, often-skipped practice

```markdown
---
language: en
license: mit
tags:
- sentiment-analysis
- text-classification
datasets:
- imdb
metrics:
- accuracy
---

# My Sentiment Model

Fine-tuned DistilBERT for binary sentiment classification on the IMDB dataset.

## Training Data
Trained on 2000 examples from the IMDB movie review dataset (recap file 04's subset selection).

## Performance
Accuracy: 0.87 on a held-out 500-example test set (recap file 10's evaluation practices).

## Limitations
- Trained on movie reviews specifically -- may not generalize well to other domains
  (recap PyTorch file 12's transfer learning honesty about domain shift)
- Small training set (2000 examples) -- genuinely limited compared to production-scale models
- English only

## Intended Use
Educational/portfolio project demonstrating the Hugging Face fine-tuning workflow.
NOT intended for production sentiment analysis without further validation.
```

Genuinely the SAME "document your work honestly" discipline that should accompany every model, echoing this repo's own README-writing habit (recap the XGBoost/PyTorch folders' README structure) — a model card's `## Limitations` section is genuinely as important as its performance numbers, directly connecting to the general responsible-AI framing touched on in file 01.

## Model versioning — genuinely using Git under the hood

```python
# Every Hub model repository IS a git repository (recap the whole ml-notes Git workflow
# used throughout this roadmap) -- commits, branches, and history all work the same way
from huggingface_hub import Repository

repo = Repository(local_dir="my-model", clone_from="my-username/my-sentiment-model")
# Make changes, then:
repo.push_to_hub(commit_message="Improved model after additional fine-tuning")
```

Genuinely satisfying to recognize — the EXACT `git add`/`git commit`/`git push` workflow used to push every single folder in this ml-notes repo throughout this whole roadmap is the SAME underlying mechanism the Hub uses for model versioning. Different content (model weights vs markdown notes), identical version-control foundation.

## Loading a private model

```python
model = AutoModelForSequenceClassification.from_pretrained(
    "my-username/my-private-model",
    token="hf_..."      # or already logged in via login() above
)
```

Genuinely useful for personal/work projects where a model shouldn't be publicly downloadable — the Hub supports private repositories exactly like GitHub does, requiring authentication to access.

## Model cards' license field — a genuinely important, easy-to-overlook detail

```python
# license: mit / apache-2.0 / cc-by-4.0 / etc.
# Genuinely important when USING someone else's model too -- always check the license
# before using a pretrained model in anything beyond personal experimentation,
# especially for commercial or redistributable projects
```

Recap the general IP/copyright-awareness themes present throughout this roadmap's more careful moments — a pretrained model's license genuinely determines what's legally permissible to build on top of it, and this is worth checking explicitly rather than assuming all "free to download" models are equally free to use for any purpose.

## Discoverability — tags, and why they matter

```python
model.push_to_hub("my-username/my-sentiment-model", tags=["sentiment-analysis", "distilbert", "imdb"])
```

Genuinely practical — tags are what make a model SEARCHABLE and FILTERABLE on the Hub's model browser (recap file 01's Hub overview), directly analogous to how good README/tag hygiene on a GitHub repo (recap this whole roadmap's repo) helps others (or future-you) actually find and understand the work.

## Try this

```python
# Take the model fine-tuned in file 05 (or train a quick small one now if that
# exercise wasn't run) and push it to the Hub under your own account
# Write a genuinely complete model card including: training data description,
# a real performance number from file 10's evaluation, at least 2 honest limitations,
# and appropriate tags
# Then, in a FRESH Python session/script, load the model back using ONLY
# AutoModelForSequenceClassification.from_pretrained("your-username/your-model-name")
# and confirm it produces the same predictions as before pushing
```

Actually completing the full push-then-reload cycle is genuinely the best way to confirm real understanding of the Hub as a practical tool, not just a theoretical concept — and having even one real, documented model on your own Hub profile is a genuinely good portfolio artifact for the interview-prep phase later in this roadmap.

## What's next
File 12 — Efficient Fine-tuning Preview: PEFT & LoRA, a focused look at parameter-efficient fine-tuning techniques — genuinely essential once models get too large to fully fine-tune with normal compute, and a direct preview of what the Frontier GenAI phase will cover in depth.
