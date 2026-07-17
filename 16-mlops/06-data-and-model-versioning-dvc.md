# 06 — Data & Model Versioning (DVC)

Recap file 05's closing note and file 01's "the data has changed since I trained this model, and I can't tell exactly how" problem — this file directly solves it, extending Git's versioning discipline (recap this entire roadmap's git workflow) to large data files and models, which Git alone genuinely handles poorly.

## Why Git alone genuinely doesn't work for data/models

```bash
# Recap this whole roadmap's git add/commit/push workflow directly --
# genuinely excellent for TEXT files (code, markdown notes), but:
# - A trained PyTorch model checkpoint can be hundreds of MB to several GB
# - A training dataset can be many GB
# - Git genuinely stores the FULL HISTORY of every version of every file --
#   committing large binary files repeatedly would make the repo genuinely
#   balloon in size, and Git's diffing (built for TEXT) doesn't work
#   meaningfully on binary model weights anyway
```

Genuinely important, real practical limitation worth understanding clearly — this is PRECISELY the gap DVC (Data Version Control) fills: it uses Git's FAMILIAR commands and workflow, but stores the actual LARGE FILES separately (in cloud storage, covered below), keeping the Git repo itself small and fast.

## How DVC works — genuinely elegant, worth understanding the mechanism

```bash
dvc init      # sets up DVC in an existing git repo

dvc add data/training_data.csv      # genuinely does NOT put the actual CSV into git
```

```bash
# What dvc add actually does:
# 1. Moves the real data file into DVC's own storage/cache
# 2. Creates a SMALL, TEXT-based .dvc file (e.g. training_data.csv.dvc) containing
#    a hash/pointer to the actual data
# 3. THIS SMALL POINTER FILE is what gets committed to Git, not the data itself
```

```bash
git add data/training_data.csv.dvc data/.gitignore
git commit -m "Add v1 of training data"
```

Genuinely the key insight worth internalizing: Git tracks the POINTER (tiny, text-based, diffable), while DVC tracks the actual DATA (large, binary, stored separately) — directly analogous to how a Python reference/pointer (recap the general concept from DSA's linked list discussion) points to actual data stored elsewhere, rather than containing the data directly.

## Remote storage — where the actual data files live

```bash
dvc remote add -d myremote s3://my-bucket/dvc-storage      # genuinely can be S3, GCS, Azure, or even a local/network path

dvc push      # uploads the actual data files to the remote storage
dvc pull      # downloads them -- genuinely similar to git push/pull, but for the LARGE files
```

Genuinely worth recognizing the deliberate parallel to Git's own `push`/`pull` — DVC is genuinely designed to feel familiar to anyone already comfortable with Git (recap this whole roadmap's git workflow), just operating on the large-file layer that Git itself skips over.

## Reproducing an exact past state — directly solving file 01's core problem

```bash
git log --oneline data/training_data.csv.dvc      # see the history of DATA VERSIONS, via Git's log

git checkout <commit_hash> -- data/training_data.csv.dvc      # revert the POINTER to an old version
dvc checkout                                                        # pulls the ACTUAL old data file matching that pointer
```

Genuinely the direct, complete answer to file 01's "I can't reproduce the exact data used to train a model from 3 months ago" problem — checking out an old Git commit that includes an old `.dvc` pointer file, then running `dvc checkout`, restores the EXACT data file that existed at that point in time, genuinely enabling true reproducibility.

## DVC pipelines — versioning the entire training PROCESS, not just files

```yaml
# dvc.yaml
stages:
  prepare:
    cmd: python prepare_data.py
    deps:
      - raw_data.csv
      - prepare_data.py
    outs:
      - processed_data.csv

  train:
    cmd: python train.py
    deps:
      - processed_data.csv
      - train.py
    params:
      - learning_rate
      - epochs
    outs:
      - model.pkl
    metrics:
      - metrics.json:
          cache: false
```

Genuinely a powerful, important feature — `dvc.yaml` defines the ENTIRE pipeline (prepare data → train model) as a sequence of stages with explicit DEPENDENCIES. Running `dvc repro` re-executes ONLY the stages whose dependencies have genuinely changed (recap file 02's Docker layer-caching philosophy directly — the SAME "skip unchanged work" optimization idea, now applied to an ML pipeline instead of container image layers).

```bash
dvc repro      # re-runs only what's genuinely needed based on what's changed
```

## Tracking experiments with DVC — a genuinely lighter-weight alternative to files 04-05

```bash
dvc exp run --set-param learning_rate=0.01
dvc exp run --set-param learning_rate=0.001
dvc exp show      # genuinely displays a table comparing all experiment runs and their metrics
```

Genuinely worth knowing DVC has its OWN lightweight experiment tracking too, tightly integrated with the pipeline/versioning system covered above — some teams use DVC alone for smaller projects, others combine it with MLflow/W&B (files 04-05) for the richer visualization those tools provide, using DVC specifically for the DATA/pipeline versioning piece those tools don't handle as thoroughly.

## Params files — genuinely important for the "which hyperparameters produced this model" question

```yaml
# params.yaml
learning_rate: 0.001
epochs: 20
batch_size: 32
```

```python
import yaml      # recap the general config-file pattern from Flask file 08 directly

with open('params.yaml') as f:
    params = yaml.safe_load(f)

model = LogisticRegression(C=params['learning_rate'])      # genuinely reads from the VERSIONED params file,
                                                                 # not a hardcoded value buried in code
```

Genuinely important, real discipline — keeping hyperparameters in a SEPARATE, VERSIONED file (rather than hardcoded inline in training scripts) means CHANGING a hyperparameter is a genuinely trackable, diffable Git change, directly connecting to `dvc.yaml`'s `params:` section shown above, which automatically triggers a pipeline re-run when this file changes.

## Data lineage — the honest, complete picture DVC provides

```
Recap file 01's full ML lifecycle diagram directly -- DVC genuinely provides
a complete, VERSIONED, REPRODUCIBLE record connecting:
RAW DATA (versioned) -> PROCESSING CODE (versioned via git) -> PROCESSED DATA (versioned)
  -> TRAINING CODE (versioned via git) -> HYPERPARAMETERS (versioned via params.yaml)
  -> TRAINED MODEL (versioned) -> METRICS (versioned)
Every single link in this chain is traceable back to an EXACT git commit
```

Genuinely the complete, honest payoff of this file — recap file 01's audit exercise directly: "can you reproduce the exact training run that produced this model, right now, from scratch" — with DVC properly set up, the honest answer becomes genuinely YES, traceable through Git history alone.

## Try this

```bash
# Take a small dataset and a simple training script from anywhere in this repo
# Initialize DVC in a git repo, dvc add the dataset, commit the resulting
# .dvc pointer file to git
# Set up a local DVC remote (a local folder is genuinely fine for practice,
# doesn't require real cloud storage) and dvc push the actual data to it
# Modify the dataset slightly (add/remove a few rows), dvc add it again,
# and commit the updated pointer -- confirm git log shows TWO distinct
# versions of the .dvc file
# Use git checkout on the OLDER commit's .dvc file plus dvc checkout to
# restore the ORIGINAL version of the data, confirming true reproducibility
```

Actually reverting to an old data version and confirming it matches the original exactly is genuinely the clearest, most concrete proof that this whole versioning discipline works as intended, rather than trusting it in the abstract.

## MLOps — halfway point. What's next
Files 01-06 covered infrastructure (Docker/Compose) and tracking/versioning (MLflow, W&B, DVC) — genuinely the foundation for reproducible, trackable ML work. File 07 shifts into AUTOMATION — CI/CD Fundamentals & GitHub Actions, automating the testing and deployment processes that have been done manually throughout this entire roadmap so far.
