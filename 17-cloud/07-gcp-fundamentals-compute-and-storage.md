# 07 — GCP Fundamentals: Compute & Storage

Recap file 06's closing note — covering Google Cloud's equivalent core services, genuinely reinforcing that the CONCEPTS from files 01-06 transfer directly, just with different service names and APIs.

## The genuinely important realization upfront — same concepts, different names

```
Recap file 01's IaaS/PaaS/managed-vs-self-managed framework directly --
NOTHING conceptually new in this file. Compute Engine IS EC2 (file 03).
Cloud Storage IS S3 (file 04). The genuinely real work here is mapping
FAMILIAR concepts onto NEW vocabulary, not learning new ideas from scratch
```

## Compute Engine — recap file 03's EC2 discussion directly

```bash
gcloud compute instances create my-ml-instance \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

Genuinely the exact same virtual-machine concept from file 03 — `--zone` is genuinely GCP's version of file 01's Availability Zone concept, `--machine-type` is genuinely GCP's version of file 03's EC2 instance types (`e2-medium` roughly parallels a general-purpose `t3.medium`).

```bash
# GPU instances -- recap PyTorch file 13 and file 03's GPU discussion directly
gcloud compute instances create my-training-instance \
  --machine-type=n1-standard-8 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --maintenance-policy=TERMINATE
```

Genuinely worth knowing GCP's GPU attachment model is SLIGHTLY different from AWS's — rather than choosing a GPU-BUNDLED instance type (like AWS's `p3.2xlarge`, file 03), GCP lets a GPU ACCELERATOR be attached to a more general instance type separately, genuinely offering a bit more granular flexibility in matching CPU/memory/GPU combinations.

## Preemptible / Spot VMs — genuinely important, real cost-saving option

```bash
gcloud compute instances create my-training-instance --preemptible
```

Genuinely worth understanding this concept directly — recap file 01's elasticity/cost themes: PREEMPTIBLE (GCP) / SPOT (AWS equivalent) instances are GENUINELY much cheaper (often 60-90% off), but the provider can RECLAIM them with short notice if the capacity is needed elsewhere. Genuinely excellent for FAULT-TOLERANT workloads — recap MLOps file 04's experiment-tracking discussion directly: a training job that CHECKPOINTS regularly (recap PyTorch file 14's checkpointing discipline directly) can genuinely resume from its last checkpoint if preempted, making this a real, practical way to cut training costs significantly for non-urgent experimentation.

## Cloud Storage — recap file 04's S3 discussion directly

```bash
gsutil mb gs://my-ml-training-data      # genuinely GCP's equivalent of aws s3 mb

gsutil cp training_data.csv gs://my-ml-training-data/datasets/v1/
gsutil ls gs://my-ml-training-data/datasets/v1/
```

Genuinely the exact same object-storage concept from file 04 — BUCKETS, OBJECTS, storage CLASSES (Standard, Nearline, Coldline, Archive — genuinely GCP's naming for file 04's Standard/IA/Glacier tiers), versioning, and lifecycle policies all apply with the SAME underlying logic, just different CLI tool (`gsutil` instead of `aws s3`) and slightly different class names.

## Using Cloud Storage from Python — recap file 04's boto3 pattern directly

```python
from google.cloud import storage      # genuinely GCP's equivalent of boto3

client = storage.Client()
bucket = client.bucket('my-ml-training-data')
blob = bucket.blob('datasets/v1/training_data.csv')
blob.download_to_filename('local_copy.csv')

import pandas as pd
df = pd.read_csv('local_copy.csv')      # recap the pandas folder directly, UNCHANGED
```

Genuinely worth recognizing this as STRUCTURALLY identical to file 04's boto3 example — download to local, load into pandas, genuinely the same pattern regardless of cloud provider.

## IAM on GCP — recap file 02's AWS IAM discussion directly

```bash
gcloud projects add-iam-policy-binding my-project-id \
  --member="user:soham@example.com" \
  --role="roles/storage.objectViewer"      # genuinely GCP's least-privilege equivalent
                                                # of file 02's S3 read-only policy example
```

Genuinely worth recognizing the SAME least-privilege principle from file 02 directly — GCP calls its permission bundles "roles" (conceptually similar to AWS's policies, though GCP's role system is structured somewhat differently), but the underlying philosophy (recap file 02's "grant ONLY what's genuinely needed" principle) is identical.

## Service Accounts — recap file 02's IAM Roles discussion directly

```bash
gcloud iam service-accounts create ml-training-sa

gcloud projects add-iam-policy-binding my-project-id \
  --member="serviceAccount:ml-training-sa@my-project-id.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"
```

Genuinely the direct GCP equivalent of file 02's IAM Roles concept — a service account represents a NON-HUMAN identity (an application, a VM, a training job) that needs specific permissions, genuinely avoiding hardcoded credentials the exact same way file 02's EC2 IAM Role discussion described.

## Cloud SQL — recap file 05's RDS discussion directly

```bash
gcloud sql instances create my-ml-app-db \
  --database-version=POSTGRES_15 \
  --tier=db-f1-micro \
  --region=us-central1
```

Genuinely the direct GCP equivalent of file 05's RDS — a fully managed PostgreSQL/MySQL instance, and genuinely EVERY line of SQLAlchemy code from Flask file 09 works identically against it, exactly as file 05 described for RDS.

## Firestore / BigTable — recap file 05's DynamoDB discussion directly

```
Firestore -- genuinely GCP's document-store NoSQL database, roughly
  comparable to DynamoDB's use cases (file 05) for flexible-schema,
  key-based lookup patterns
BigTable -- genuinely GCP's WIDE-COLUMN store, built for MASSIVE scale
  time-series/sensor data -- a genuinely more specialized NoSQL variant
  without a direct AWS parallel covered so far in this folder
```

## BigQuery — genuinely GCP's standout, distinctive service worth knowing well

```sql
-- Recap the ENTIRE SQL folder directly -- BigQuery uses STANDARD SQL syntax
SELECT customer_id, COUNT(*) as order_count, AVG(order_value) as avg_value
FROM `my-project.sales.orders`
GROUP BY customer_id
HAVING COUNT(*) > 5
ORDER BY avg_value DESC;
```

Genuinely worth knowing BigQuery as GCP's most distinctive offering — a SERVERLESS, massively-scalable DATA WAREHOUSE, genuinely designed to run complex analytical queries (recap SQL files 04, 06, 07's GROUP BY/CTE/window-function toolkit, ALL directly applicable) over TERABYTES of data in seconds, without needing to provision or manage any database server at all — a genuinely different scale/use-case than RDS/Cloud SQL's transactional-workload focus.

```python
from google.cloud import bigquery      # genuinely queryable directly from Python, feeding into pandas
client = bigquery.Client()
df = client.query("SELECT * FROM `my-project.sales.orders` LIMIT 1000").to_dataframe()
```

Genuinely worth recognizing this as directly connecting to SQL file 14's SQL-to-pandas bridge discussion — the EXACT same "query → DataFrame → ML pipeline" pattern, now at genuinely massive analytical scale.

## The honest AWS vs GCP service mapping — a genuinely useful cheat sheet

| Need | AWS (files 02-06) | GCP (this file) |
|---|---|---|
| Virtual machines | EC2 | Compute Engine |
| Object storage | S3 | Cloud Storage |
| Serverless functions | Lambda | Cloud Functions |
| Managed relational DB | RDS | Cloud SQL |
| NoSQL key-value | DynamoDB | Firestore |
| Managed ML platform | SageMaker | Vertex AI (file 08) |
| Data warehouse | Redshift | BigQuery |
| IAM | IAM Policies/Roles | IAM Roles/Service Accounts |

Genuinely worth keeping this table as a real, practical reference — recognizing this mapping quickly is precisely what makes cross-provider fluency achievable without needing to relearn everything from scratch for each platform.

## Try this

```bash
# In your GCP free tier account (recap file 01's setup):
# Create a Compute Engine instance and SSH into it (recap file 03's EC2
# exercise directly, now on GCP)
# Create a Cloud Storage bucket, upload a small dataset, and write a Python
# script using google-cloud-storage to download it and load it into pandas
# Create a small Cloud SQL PostgreSQL instance and connect the SAME Flask
# app from file 05's exercise to it, confirming it works identically
# Run one query against BigQuery's public datasets (genuinely free to query
# small amounts, e.g. bigquery-public-data.samples.shakespeare) using the
# bigquery Python client, pulling the result directly into a pandas DataFrame
```

Running the SAME Flask app against Cloud SQL after already running it against RDS (file 05) is genuinely the clearest, most concrete proof that the application-level code is completely provider-agnostic — only the infrastructure underneath changes.

## What's next
File 08 — GCP ML Services: Vertex AI, covering Google's genuinely dedicated ML platform — the direct GCP counterpart to file 06's SageMaker deep dive.
