# 06 — AWS ML Services: SageMaker Deep Dive

Recap file 05's closing note and MLOps file 12's bridge discussion directly — AWS's genuinely dedicated, purpose-built ML platform, tying together files 01-05's infrastructure concepts with everything covered throughout the PyTorch, Hugging Face, and MLOps folders.

## What SageMaker actually is — genuinely a unified ML platform

```
Recap MLOps file 12's concept map directly -- SageMaker is genuinely AWS's
MANAGED answer to nearly every piece of the MLOps lifecycle (recap MLOps
file 01's full lifecycle diagram): experiment tracking, training
infrastructure, model registry, deployment, and monitoring, ALL in one
integrated platform, rather than assembling separate tools (MLflow, EC2,
Docker, Kubernetes) manually
```

## SageMaker Notebooks — genuinely a managed Jupyter environment

```
Genuinely just a managed EC2 instance (recap file 03 directly) pre-configured
with Jupyter and common ML libraries (PyTorch, scikit-learn, recap those
folders directly) -- worth knowing this exists as a convenient starting
point, though genuinely just a specific, convenient application of file 03's
EC2 concepts
```

## SageMaker Training Jobs — genuinely the core training abstraction

```python
import sagemaker
from sagemaker.pytorch import PyTorch      # recap the ENTIRE PyTorch folder directly

estimator = PyTorch(
    entry_point='train.py',                  # your actual PyTorch training script,
                                                  # recap PyTorch file 07's training loop directly
    role='arn:aws:iam::123456789:role/SageMakerRole',      # recap file 02's IAM Role discussion
    instance_type='ml.p3.2xlarge',              # recap file 03's GPU instance discussion directly
    instance_count=1,
    framework_version='2.1',
    hyperparameters={'learning_rate': 0.001, 'epochs': 10}      # recap MLOps file 06's params.yaml concept
)

estimator.fit({'training': 's3://my-ml-training-data/datasets/v1/'})      # recap file 04's S3 discussion directly
```

Genuinely worth understanding what's happening here explicitly — SageMaker provisions a TEMPORARY EC2 instance (file 03, with the GPU type specified), automatically pulls the training data from S3 (file 04), runs `train.py` (which is GENUINELY just a normal PyTorch training script, recap that whole folder unchanged), saves the resulting model artifact back to S3, and TEARS DOWN the instance automatically when training completes — genuinely paying only for the exact duration of training, a real, practical application of file 01's elasticity principle.

## The `train.py` script — genuinely just PyTorch, nothing SageMaker-specific required

```python
# train.py -- genuinely almost IDENTICAL to any training script from the PyTorch folder
import torch
import argparse
import os

def train(args):
    model = MyModel()      # recap PyTorch file 04's nn.Module directly
    optimizer = torch.optim.Adam(model.parameters(), lr=args.learning_rate)      # recap PyTorch file 07

    for epoch in range(args.epochs):
        # ... recap PyTorch file 07's five-step training loop, completely unchanged ...
        pass

    torch.save(model.state_dict(), os.path.join(args.model_dir, 'model.pt'))      # recap PyTorch file 14

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--learning-rate', type=float, default=0.001)
    parser.add_argument('--epochs', type=int, default=10)
    parser.add_argument('--model-dir', type=str, default=os.environ.get('SM_MODEL_DIR'))      # SageMaker
                                                                                                     # convention
    args = parser.parse_args()
    train(args)
```

Genuinely the most reassuring realization in this entire file — recap the general pattern that's held true across every bridge file in this roadmap: SageMaker doesn't require learning a NEW way to write PyTorch code, it just requires accepting hyperparameters as command-line arguments (a genuinely standard Python pattern, recap the general `argparse` usage) and saving to a specific directory SageMaker expects, mapped via the `SM_MODEL_DIR` environment variable convention.

## SageMaker Experiments — recap MLOps file 04/05 directly

```python
from sagemaker.experiments.run import Run

with Run(experiment_name="sentiment-classifier-experiments") as run:
    run.log_parameter("learning_rate", 0.001)      # recap MLOps file 04's mlflow.log_param() EXACTLY
    run.log_metric("val_accuracy", 0.92)              # recap MLOps file 04's mlflow.log_metric() EXACTLY
```

Genuinely worth recognizing this AGAIN, directly recapping MLOps file 12's concept map — the API is genuinely almost identical to MLflow's, since it's solving the exact same "which run was best" problem (recap MLOps file 01's original framing) with AWS-managed infrastructure instead of a self-hosted MLflow server.

## Hyperparameter Tuning Jobs — recap XGBoost file 08 and MLOps file 05's Bayesian sweep directly

```python
from sagemaker.tuner import HyperparameterTuner, ContinuousParameter

hyperparameter_ranges = {
    'learning_rate': ContinuousParameter(0.0001, 0.1)      # recap MLOps file 05's sweep_config directly
}

tuner = HyperparameterTuner(
    estimator=estimator,
    objective_metric_name='val_accuracy',
    hyperparameter_ranges=hyperparameter_ranges,
    max_jobs=20,
    strategy='Bayesian'      # recap Math Foundations file 11's Bayesian optimization directly
)
tuner.fit({'training': 's3://my-ml-training-data/'})
```

Genuinely worth recognizing this as the SAME Bayesian hyperparameter search concept from MLOps file 05's W&B sweep discussion and Math Foundations file 11's Bayesian preview — SageMaker automatically runs MULTIPLE training jobs (each genuinely a separate instance of the training-job pattern above) in parallel, intelligently choosing which hyperparameters to try next.

## Deploying a SageMaker Endpoint — recap Flask/MLOps file 10 directly

```python
predictor = estimator.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.large'      # genuinely often CPU-only for inference, recap file 03's cost discussion
)

result = predictor.predict({"text": "This movie was fantastic!"})
```

Genuinely worth recognizing this as a MANAGED version of the ENTIRE Flask file 06 model-serving discussion — SageMaker handles provisioning the serving infrastructure, load balancing, and auto-scaling automatically, rather than manually building/deploying the Flask endpoint from that folder.

## SageMaker's built-in deployment strategies — recap MLOps file 10 directly

```python
predictor.update_endpoint(
    initial_instance_count=1,
    instance_type='ml.m5.large',
    # SageMaker supports canary/blue-green style traffic shifting NATIVELY,
    # recap MLOps file 10's entire deployment-strategy discussion directly --
    # the SAME concepts, now with AWS handling the traffic-splitting
    # infrastructure automatically
)
```

## SageMaker Model Monitor — recap MLOps file 09's drift detection directly

```python
from sagemaker.model_monitor import DefaultModelMonitor

monitor = DefaultModelMonitor(role=role, instance_count=1, instance_type='ml.m5.large')
monitor.suggest_baseline(baseline_dataset='s3://my-bucket/training-data.csv', dataset_format='csv')
```

Genuinely worth recognizing this as a DIRECT, managed implementation of MLOps file 09's entire drift-detection discussion — SageMaker automatically computes baseline statistics from training data (recap that file's KS-test-based comparison approach conceptually) and continuously compares live production traffic against it, alerting automatically when genuinely significant drift is detected.

## The honest cost/complexity tradeoff — recap file 01's framework directly

```
Genuinely worth an honest, direct assessment: SageMaker adds real convenience
and integration, at a genuinely real cost PREMIUM over assembling the
equivalent stack manually (EC2 + self-hosted MLflow + custom deployment
scripts, recap MLOps folder's entire self-managed toolkit)
Genuinely reasonable for TEAMS/PRODUCTION systems where the operational
time saved justifies the cost; genuinely worth starting with the self-managed
MLOps folder's tools for LEARNING and small personal projects, where the
cost/complexity tradeoff favors understanding the fundamentals directly
```

## Try this

```python
# Take a simple PyTorch training script from that folder's exercises
# (recap files 04-07's MLP/CNN examples) and adapt it to accept hyperparameters
# via argparse and save to SM_MODEL_DIR, following this file's train.py pattern
# Using SageMaker's free-tier-eligible instance types, launch a real training
# job via the PyTorch estimator, pointing to a small dataset uploaded to S3
# (recap file 04's exercise)
# Once training completes, deploy the resulting model as a SageMaker endpoint
# and send it a test prediction request, confirming the full training ->
# deployment pipeline works end-to-end on real AWS infrastructure
# Remember to DELETE the endpoint afterward (recap file 03's cost-management
# discipline directly) to avoid ongoing charges
```

Actually running this full pipeline on real infrastructure — even a tiny model on a tiny dataset — is genuinely the clearest way to feel how directly SageMaker builds on everything already deeply understood from the PyTorch and MLOps folders, rather than requiring an entirely separate skill set.

## What's next
File 07 — GCP Fundamentals: Compute & Storage, covering Google Cloud's equivalent core services — genuinely reinforcing that the underlying CONCEPTS from files 01-06 transfer directly, just with different service names and APIs.
