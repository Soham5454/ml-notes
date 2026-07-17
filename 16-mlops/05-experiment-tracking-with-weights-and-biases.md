# 05 — Experiment Tracking with Weights & Biases (W&B)

Recap file 04's closing note — the other genuinely major, industry-standard experiment tracking tool, with particularly strong visualization and deep-learning-specific features that make it especially popular for PyTorch/Hugging Face work.

## MLflow vs W&B — the honest, practical comparison upfront

```
MLflow (file 04):  genuinely great for self-hosting, open-source, integrates
  cleanly with the Model Registry concept, a real standard in many companies'
  internal ML infrastructure
W&B (this file):     genuinely excellent, polished visualizations, especially
  strong for deep learning (live-updating loss curves, gradient/weight
  histograms), a hosted SaaS product (though self-hosting options exist),
  very popular in research and PyTorch/Hugging Face-heavy teams
```

Genuinely worth knowing BOTH exist and are widely used in real industry settings — many teams genuinely use one or the other (or occasionally both for different purposes), and familiarity with both makes a stronger, more adaptable profile.

## Setting up W&B

```python
import wandb

wandb.login()      # genuinely requires a free account at wandb.ai

wandb.init(project="sentiment-classification", config={
    "learning_rate": 0.001,
    "epochs": 20,
    "batch_size": 32,
    "architecture": "DistilBERT"
})
```

Genuinely worth noticing the `config` dict passed directly at `init` time — this is W&B's convention for logging hyperparameters UPFRONT, all at once, contrasted with MLflow's more incremental `log_param()` calls (file 04) — a genuinely minor syntactic difference reflecting a similar underlying purpose.

## Logging metrics during training — recap PyTorch file 07's training loop directly

```python
for epoch in range(20):
    # ... recap PyTorch file 07's five-step training loop exactly ...
    train_loss = ...
    val_loss = ...
    val_accuracy = ...

    wandb.log({
        "epoch": epoch,
        "train_loss": train_loss,
        "val_loss": val_loss,
        "val_accuracy": val_accuracy
    })
```

Genuinely the same core idea as MLflow's `step=` parameter (file 04) — W&B automatically plots these as LIVE-UPDATING curves in its web dashboard AS TRAINING RUNS, genuinely useful for watching PyTorch file 09's overfitting pattern (train/val divergence) develop in real time rather than only after training completes.

## W&B's genuinely standout feature — live dashboards during training

```python
# Genuinely worth knowing this distinguishes W&B from many alternatives:
# the dashboard updates LIVE while training is still running, letting a
# long-running PyTorch training job (recap PyTorch file 13's GPU training
# discussion) be monitored REMOTELY, from a phone or another machine, without
# needing to be actively watching the terminal output
```

Genuinely valuable for real, long training runs — recap PyTorch file 13's mixed-precision/GPU training discussion, where a real fine-tuning job (recap Hugging Face file 05's Trainer-based fine-tuning) might run for hours; being able to check progress remotely and catch a diverging loss EARLY (rather than discovering it hours later) is a genuine, practical time-saver.

## Integration with Hugging Face's Trainer — genuinely seamless

```python
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    report_to="wandb",      # genuinely this ONE line enables full W&B tracking
    run_name="distilbert-sentiment-v1"
)
```

Recap Hugging Face file 05's `Trainer`/`TrainingArguments` directly — genuinely one of the most convenient integrations in this whole folder: `report_to="wandb"` automatically logs every training metric, learning rate schedule step, and evaluation result to W&B, with ZERO manual `wandb.log()` calls needed inside the training loop, since `Trainer` already handles that loop internally.

## Logging images, tables, and other rich media — genuinely useful for CV/NLP work

```python
# Recap the OpenCV folder directly -- logging example predictions visually
wandb.log({"example_predictions": [wandb.Image(img, caption=f"Predicted: {label}") for img, label in samples]})

# Genuinely useful for NLP -- logging a table of text examples and their predictions
table = wandb.Table(columns=["text", "predicted_sentiment", "confidence"])
for text, pred, conf in results:
    table.add_data(text, pred, conf)
wandb.log({"predictions_table": table})
```

Genuinely a real, practical advantage over plain metric logging — being able to VISUALLY inspect actual model predictions (recap OpenCV file 04's bounding-box visualization habit, or Classical NLP's qualitative error-checking discipline) directly within the same dashboard as the numeric metrics, rather than needing separate tooling.

## Hyperparameter sweeps — W&B's genuinely powerful automated tuning tool

```python
sweep_config = {
    'method': 'bayes',      # recap Math Foundations file 11's Bayesian thinking directly --
                                # genuinely the SAME Bayesian optimization philosophy
                                # briefly previewed in that file's closing notes
    'metric': {'name': 'val_accuracy', 'goal': 'maximize'},
    'parameters': {
        'learning_rate': {'min': 0.0001, 'max': 0.1},
        'batch_size': {'values': [16, 32, 64]}
    }
}

sweep_id = wandb.sweep(sweep_config, project="sentiment-classification")

def train():
    wandb.init()
    lr = wandb.config.learning_rate
    batch_size = wandb.config.batch_size
    # ... training code using these hyperparameters ...

wandb.agent(sweep_id, function=train, count=20)      # runs 20 trials automatically
```

Genuinely worth connecting directly to Math Foundations file 11's Bayesian preview and XGBoost file 08's hyperparameter tuning discussion — `method: 'bayes'` uses Bayesian optimization (recap that file's prior/posterior updating concept) to intelligently choose WHICH hyperparameters to try next, based on results from PREVIOUS trials, genuinely more efficient than random or grid search for expensive-to-train models, since it focuses search effort on genuinely promising regions of the hyperparameter space rather than exploring uniformly.

## Comparing runs visually — genuinely W&B's strongest UI feature

```python
# Genuinely worth knowing this exists: W&B's "Parallel Coordinates" plot visualizes
# MANY hyperparameters and their resulting metric SIMULTANEOUSLY, letting patterns
# like "high learning_rate + low batch_size consistently underperforms" be spotted
# visually across dozens of runs at once -- genuinely harder to see in MLflow's
# more tabular comparison view
```

## Try this

```python
# Take a PyTorch fine-tuning run (recap Hugging Face file 05's Trainer-based
# text classification) and enable W&B tracking via report_to="wandb"
# Watch the live dashboard update during training, and after it completes,
# log a wandb.Table with 10 example predictions (text, true label, predicted
# label, confidence) for qualitative inspection
# Then set up a small hyperparameter sweep (2-3 learning rates, 2 batch sizes)
# using the Bayesian method, run it, and use the parallel coordinates view
# to identify which hyperparameter combination performed best
```

Actually setting up and running a sweep, then visually inspecting the parallel coordinates plot, is genuinely the best way to appreciate W&B's real strength over simpler, purely tabular tracking approaches.

## What's next
File 06 — Data & Model Versioning (DVC), extending the versioning discipline from files 04-05's model tracking to the DATA itself — directly solving file 01's "the data has changed since I trained this model, and I can't tell exactly how" problem.
