# 08 — GCP ML Services: Vertex AI

Recap file 07's closing note and file 06's SageMaker discussion directly — Google's genuinely dedicated ML platform, the direct GCP counterpart to that entire file, with a few genuinely distinctive strengths worth understanding.

## Vertex AI — genuinely GCP's unified ML platform, recap file 06's SageMaker framing directly

```
Recap MLOps file 12's concept map and file 06's SageMaker discussion
DIRECTLY -- Vertex AI is genuinely GCP's equivalent unified answer to the
SAME MLOps lifecycle (recap MLOps file 01's diagram): experiment tracking,
training infrastructure, model registry, deployment, monitoring, ALL
integrated -- genuinely the SAME value proposition as SageMaker, on GCP's
infrastructure instead of AWS's
```

## Vertex AI Training — recap file 06's SageMaker PyTorch estimator directly

```python
from google.cloud import aiplatform      # genuinely GCP's ML SDK, roughly the sagemaker package's counterpart

aiplatform.init(project='my-project-id', location='us-central1')

job = aiplatform.CustomTrainingJob(
    display_name='sentiment-classifier-training',
    script_path='train.py',                  # genuinely the SAME train.py concept from file 06,
                                                  # a normal PyTorch training script, recap that
                                                  # ENTIRE folder unchanged
    container_uri='us-docker.pkg.dev/vertex-ai/training/pytorch-gpu.2-1:latest',
    requirements=['torch', 'transformers']      # recap Hugging Face folder, applies unchanged
)

model = job.run(
    replica_count=1,
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_T4',      # recap file 07's GPU-attachment discussion directly
    accelerator_count=1,
    args=['--learning-rate=0.001', '--epochs=10']      # recap file 06's argparse pattern, UNCHANGED
)
```

Genuinely worth recognizing this as STRUCTURALLY the same pattern as file 06's SageMaker `PyTorch` estimator — a normal training script, a container specifying the framework, and a specified machine/GPU configuration, genuinely just different SDK syntax accomplishing the identical underlying goal.

## Vertex AI Experiments — recap MLOps file 04/05 and file 06's SageMaker Experiments directly

```python
aiplatform.init(experiment='sentiment-experiments')

with aiplatform.start_run('run-1') as run:
    run.log_params({'learning_rate': 0.001, 'epochs': 10})      # recap MLOps file 04's log_param() directly
    run.log_metrics({'val_accuracy': 0.92})                        # recap MLOps file 04's log_metric() directly
```

Genuinely worth recognizing this AGAIN, now for the THIRD time across MLflow (MLOps file 04), SageMaker (file 06), and Vertex AI — every genuinely major ML platform converges on nearly IDENTICAL experiment-tracking APIs, precisely because they're all solving the exact same, well-understood problem (recap MLOps file 01's original "which run was best" framing).

## Vertex AI's genuinely distinctive strength — deep integration with Google's own models

```python
# Genuinely worth knowing this as Vertex AI's REAL differentiator over
# SageMaker: native, first-class access to Google's own foundation models
from vertexai.generative_models import GenerativeModel      # genuinely a direct preview of
                                                                  # the Frontier GenAI phase ahead

model = GenerativeModel("gemini-1.5-pro")
response = model.generate_content("Explain the attention mechanism briefly")
print(response.text)
```

Genuinely worth flagging this as a real, meaningful distinction from AWS's SageMaker — recap PyTorch file 15 and the Hugging Face folder's transformer/generation discussions directly: Vertex AI provides GENUINELY native, tightly integrated access to Google's Gemini model family, in the SAME platform used for custom training/deployment — a real, practical advantage for teams wanting to COMBINE custom fine-tuned models with foundation-model API calls in one unified environment, directly foreshadowing file 12's bridge into the Frontier GenAI phase.

## AutoML — genuinely worth knowing exists, a different philosophy entirely

```python
from google.cloud import aiplatform

dataset = aiplatform.TabularDataset.create(
    display_name='customer-churn-data',
    gcs_source='gs://my-bucket/churn_data.csv'      # recap file 07's Cloud Storage discussion directly
)

job = aiplatform.AutoMLTabularTrainingJob(display_name='automl-churn-model')
model = job.run(dataset=dataset, target_column='churned', budget_milli_node_hours=1000)
```

Genuinely worth understanding AutoML's real, honest tradeoff — recap the general "understanding fundamentals vs using automated tools" tension touched on throughout this roadmap: AutoML genuinely AUTOMATES model selection, feature engineering, and hyperparameter tuning (recap scikit-learn/XGBoost's manual versions of all of this), producing a working model with GENUINELY minimal ML expertise required. The honest cost: LESS control and understanding of WHY a specific model/approach was chosen, and genuinely often a real, practical cost premium — worth knowing this exists as a real tool for certain business contexts, while recognizing that everything built across the scikit-learn/XGBoost/PyTorch folders provides genuinely deeper understanding and control that AutoML deliberately trades away.

## Vertex AI Endpoints — recap file 06's SageMaker deployment and Flask's serving discussion

```python
endpoint = model.deploy(
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=3      # recap MLOps file 11's HorizontalPodAutoscaler concept directly --
                                  # Vertex AI handles this scaling automatically
)

prediction = endpoint.predict(instances=[{"text": "This movie was fantastic!"}])
```

Genuinely worth recognizing `min_replica_count`/`max_replica_count` as file 01's elasticity principle and MLOps file 11's autoscaling concept, now expressed directly in Vertex AI's deployment API — the SAME underlying idea, consistently appearing across every layer of cloud infrastructure covered in this whole folder.

## Vertex AI Pipelines — recap MLOps file 08's CI/CD-for-ML discussion directly

```python
from kfp import dsl      # Kubeflow Pipelines, genuinely the underlying technology Vertex AI Pipelines builds on

@dsl.pipeline(name='training-pipeline')
def ml_pipeline():
    validate_task = validate_data_op()      # recap MLOps file 08's data validation gate directly
    train_task = train_model_op().after(validate_task)      # recap MLOps file 07's needs: dependency chain
    evaluate_task = evaluate_model_op(model=train_task.output).after(train_task)
    deploy_task = deploy_model_op(model=train_task.output).after(evaluate_task)
```

Genuinely worth recognizing this `.after()` dependency chaining as the EXACT same pipeline-dependency concept from MLOps file 07's GitHub Actions `needs:` discussion and file 08's complete validate→train→evaluate→deploy workflow — Vertex AI Pipelines is genuinely a managed way to define and run that SAME structure, using Kubeflow (built on Kubernetes, recap MLOps file 11 directly) under the hood.

## Model Monitoring on Vertex AI — recap MLOps file 09 and file 06's SageMaker Model Monitor directly

```python
from google.cloud.aiplatform import model_monitoring

monitoring_config = model_monitoring.ObjectiveConfig(
    skew_detection_config=model_monitoring.SkewDetectionConfig(
        data_source='gs://my-bucket/training-data.csv'      # genuinely the baseline for comparison,
                                                                  # recap MLOps file 09's drift-detection
                                                                  # baseline concept directly
    )
)
```

Genuinely worth recognizing this as the THIRD implementation of the same drift-monitoring concept covered in this roadmap (MLOps file 09's manual approach, file 06's SageMaker Model Monitor, now Vertex AI) — a genuinely strong signal that this specific MLOps concept is universally important enough that every major platform provides a managed version of it.

## The honest SageMaker vs Vertex AI comparison

```
SageMaker (file 06): genuinely broader AWS ecosystem integration, larger
  market share/job-posting prevalence, mature and battle-tested
Vertex AI (this file): genuinely stronger native foundation-model integration
  (Gemini), often cited as having a somewhat cleaner/more modern API design,
  genuinely strong choice for teams already using GCP's other services
  (BigQuery, file 07) or wanting tight foundation-model integration
```

Genuinely worth carrying forward the same honest, practical framing from file 01 directly — real job postings and team contexts vary in which they use; genuine fluency in the UNDERLYING MLOps concepts (this whole folder, plus the entire MLOps folder before it) transfers between them almost completely, which is precisely why understanding both, even briefly, is genuinely valuable.

## Try this

```python
# Take the SAME small PyTorch training script adapted for SageMaker in file
# 06's exercise, and adapt it (genuinely minimal changes needed, mostly just
# the argparse/SM_MODEL_DIR conventions) for a Vertex AI CustomTrainingJob
# Run it on Vertex AI's free-tier-eligible compute, deploy the resulting
# model as an endpoint, and send it a test prediction
# Compare your experience: which platform's SDK/API felt more intuitive?
# Write down 2-3 concrete, honest observations comparing the two -- this
# comparison is genuinely valuable, real interview-relevant knowledge
```

Actually running the SAME workload on both platforms and honestly comparing the experience is genuinely more valuable than reading about the differences abstractly — real, hands-on comparative knowledge is precisely what distinguishes genuine platform fluency from surface-level familiarity.

## What's next
File 09 — Cloud Networking Basics, covering VPCs, security groups, and load balancers — the networking layer underlying every service covered across files 02-08, genuinely essential for understanding how these services actually communicate securely.
