# 01 — Introduction to MLOps & the ML Lifecycle

Starting the MLOps phase — genuinely the discipline that turns everything built across the PyTorch, Hugging Face, and Flask folders into something that runs RELIABLY, not just once in a demo. Recap Flask file 12's closing bridge directly: this is precisely "scaling the discipline built in Flask for systems that need to run reliably, not just work once."

## What MLOps actually is

MLOps (Machine Learning Operations) — genuinely the practices and tooling for taking a model from "works in my notebook" to "runs reliably in production, stays reliable over time, and can be improved safely." A direct extension of DevOps principles (continuous integration, automation, monitoring — recap the general software engineering discipline touched on throughout this roadmap), specifically adapted for the unique challenges ML systems introduce.

## Why ML systems need MORE than regular software engineering discipline

```python
# Regular software: code changes are the main source of bugs/regressions
# ML systems: THREE things can independently change and cause problems:
# 1. CODE (the usual software concern)
# 2. DATA (the training data, or the distribution of data seen in production)
# 3. MODEL (the actual learned weights/parameters, which can be RETRAINED
#    and silently produce different behavior even with identical code)
```

Genuinely the core insight that makes MLOps a distinct discipline, not just "DevOps with extra steps" — recap the general train/test discipline emphasized throughout scikit-learn, XGBoost, and PyTorch folders. A model that performs well in testing can genuinely DEGRADE over time purely because the real-world DATA it encounters shifts away from its training distribution — a failure mode that doesn't exist in traditional software, where code that works today keeps working tomorrow (barring genuine bugs).

## The complete ML lifecycle — the map this whole folder fills in

```
1. Data collection & preparation      -- recap pandas, SQL folders
2. Experimentation & model training     -- recap scikit-learn, XGBoost, PyTorch, Hugging Face
3. Experiment tracking                    -- files 04-05 of THIS folder
4. Model versioning & data versioning       -- file 06
5. Model packaging (containerization)         -- files 02-03
6. CI/CD for automated testing/deployment       -- files 07-08
7. Deployment (serving)                           -- recap Flask folder directly, files 10-11 of this folder
8. Monitoring & drift detection                     -- file 09
9. Retraining & iteration                             -- closes the loop back to step 1
```

Genuinely worth seeing this whole cycle laid out explicitly upfront — every file in this folder fills in ONE piece of this loop, and by the end, the full "how does a real ML product actually get built and maintained" picture will be complete, connecting directly back to nearly every previous folder in this roadmap.

## The "it worked on my machine" problem — recap Flask file 11's Docker motivation

```python
# Recap Flask file 11 directly: "works on my machine" problems arise from
# dependency version differences, OS differences, missing environment setup
# For ML specifically, this problem is GENUINELY WORSE:
# - Different CUDA/GPU driver versions (recap PyTorch file 13) can silently
#   change numerical results or crash entirely
# - Different library versions (a newer scikit-learn/PyTorch release) can
#   genuinely change a model's behavior even with IDENTICAL code
# - Random seeds (recap the general reproducibility habit from scikit-learn/
#   PyTorch notes) need to be controlled carefully for genuinely reproducible results
```

Genuinely important, real motivation for file 02's deep Docker treatment — the packaging problem is MORE severe for ML workloads than typical web apps, precisely because of GPU/CUDA dependencies and the sensitivity of numerical results to exact library versions.

## Reproducibility — a genuinely foundational MLOps concern

```python
# Recap the general "random_state=42" habit seen throughout scikit-learn/PyTorch notes
# MLOps formalizes this into a REAL DISCIPLINE:
# - Same code + same data + same random seed + same library versions
#   = should GENUINELY produce the same trained model, every single time
# This is genuinely HARDER to guarantee than it sounds, and is precisely
# why experiment tracking (files 04-05) and data versioning (file 06) matter --
# without them, "reproduce the model that was in production 3 months ago"
# can become genuinely impossible
```

## The honest, real costs of skipping MLOps discipline

```
Without experiment tracking:     "Which of these 47 notebook runs was the best one? I genuinely don't remember."
Without data versioning:           "The data has changed since I trained this model, and I can't tell exactly how."
Without model versioning:            "Which exact model is currently running in production right now?"
Without monitoring:                    "The model has been silently getting worse for weeks and nobody noticed."
Without CI/CD:                           "Deploying a new model version is a manual, error-prone, 2-hour process."
```

Genuinely worth internalizing these as REAL, common failure modes, not hypothetical concerns — every single one of these is a genuinely frequent, well-documented problem in real ML teams, and each file in this folder addresses one directly.

## MLOps maturity levels — a genuinely useful mental framework

```
Level 0: Manual process -- train in a notebook, manually deploy, no tracking, no monitoring
Level 1: ML pipeline automation -- automated training pipelines, but manual deployment
Level 2: CI/CD pipeline automation -- fully automated training AND deployment,
         with proper testing, versioning, and monitoring at every stage
```

Genuinely worth knowing this maturity framework (originally from Google's MLOps guidance) as a real, practical way to assess where any given project or team actually sits — most personal/early-stage projects genuinely start at Level 0, and this whole folder is about building the skills to move deliberately toward Level 2 as a project's stakes and scale genuinely justify the added complexity.

## Try this

```python
# Take a model already trained somewhere in this repo (recap scikit-learn, XGBoost,
# or PyTorch folders) and honestly audit it against the MLOps lifecycle above:
# - Can you reproduce the EXACT training run that produced it, right now, from scratch?
# - Do you know exactly which version of the training data was used?
# - If this model's performance silently degraded tomorrow, would you notice?
# Write down honest answers to these 3 questions BEFORE reading further into this
# folder -- this is genuinely the most useful "try this" exercise in the whole
# folder, since it makes the real, personal gap between "I can train a model" and
# "I can operate a model reliably" concrete before the tooling to close that gap
# gets introduced
```

Genuinely worth doing this audit honestly rather than skipping it — the discomfort of realizing "I actually can't easily reproduce that model" is precisely the motivation that makes files 04-06's tooling feel necessary rather than academic.

## What's next
File 02 — Docker Fundamentals, going properly deep into containerization (recap Flask file 11's brief introduction) — the packaging foundation nearly everything else in this folder builds on.
