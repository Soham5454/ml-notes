# 04 — AWS Storage: S3 & Data Management

Recap file 03's closing note — covering where data, model artifacts, and datasets actually LIVE in AWS, genuinely the storage foundation nearly every other service in this folder depends on.

## S3 — genuinely object storage, worth understanding what that means

```
S3 (Simple Storage Service): stores OBJECTS (files) in BUCKETS -- genuinely
NOT a traditional file system with folders (though the console/CLI can
SIMULATE folder-like organization using "/" in object keys) -- each object
is identified by a unique KEY within its bucket
```

```bash
aws s3 mb s3://my-ml-training-data      # creates a bucket -- genuinely globally unique name required

aws s3 cp training_data.csv s3://my-ml-training-data/datasets/v1/training_data.csv      # recap MLOps
                                                                                              # file 06's DVC
                                                                                              # remote-storage discussion
                                                                                              # directly -- S3 is
                                                                                              # PRECISELY the kind of
                                                                                              # remote storage DVC uses
aws s3 ls s3://my-ml-training-data/datasets/v1/
```

## Why S3 specifically matters SO much for ML workflows

```
Genuinely the standard, foundational storage layer for real ML systems:
- Training DATASETS (recap the general pandas/SQL data-loading discussions,
  now sourced from S3 instead of local files)
- MODEL ARTIFACTS (recap PyTorch file 14's saving/loading, MLOps file 04's
  MLflow model logging -- these often store their actual artifact FILES in S3)
- LOGS and monitoring data (recap MLOps file 09's prediction logging,
  often archived to S3 for long-term storage)
```

## Storage classes — genuinely important, real cost-optimization lever

```
S3 Standard -- genuinely frequently-accessed data, higher cost per GB
S3 Standard-IA (Infrequent Access) -- genuinely cheaper storage, small
  retrieval fee, good for data accessed occasionally
S3 Glacier -- genuinely very cheap, but SLOW retrieval (minutes to hours) --
  good for long-term archival (old training data versions, recap MLOps
  file 06's data-versioning discussion for data that's genuinely rarely
  needed but must be RETAINED for reproducibility)
```

Genuinely worth connecting this directly to MLOps file 12's cost-tradeoff framework — choosing the RIGHT storage class for each dataset's actual access pattern is a real, meaningful cost lever; storing rarely-accessed archival training data in expensive Standard storage is genuinely a common, avoidable cost mistake.

## Bucket policies and permissions — recap file 02's IAM directly

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789:role/my-ec2-role"},
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-ml-training-data/*"
    }
  ]
}
```

Genuinely worth recognizing this as the SAME IAM policy structure from file 02, just attached directly to the BUCKET rather than to a user/role — recap file 02's least-privilege principle directly: this policy grants ONLY the specified role read access, nothing broader.

## Public vs private buckets — genuinely important, real security concern

```
Genuinely worth flagging directly, a REAL and historically common security
incident category: accidentally making an S3 bucket PUBLIC, exposing
sensitive training data or model artifacts to the entire internet
AWS's "Block Public Access" setting is genuinely ON by default for new
buckets specifically BECAUSE of how common and damaging this mistake
has historically been -- worth NEVER disabling it without a genuinely
deliberate, well-understood reason
```

## Versioning — recap MLOps file 06's DVC concept directly, natively in S3

```bash
aws s3api put-bucket-versioning --bucket my-ml-training-data --versioning-configuration Status=Enabled
```

Genuinely worth recognizing this as S3's OWN native version of MLOps file 06's DVC versioning discussion — once enabled, EVERY upload to the same object key creates a new VERSION rather than overwriting the previous one, letting an old dataset version genuinely be retrieved later. Recap MLOps file 12's concept map directly: this is precisely the "cloud storage versioning (S3 versioning) is genuinely the SAME underlying concept as DVC, just provider-managed" connection flagged in that bridge file.

## Uploading/downloading large files efficiently — genuinely important for real datasets

```bash
aws s3 cp large_dataset.csv s3://my-bucket/ --storage-class STANDARD_IA

aws s3 sync ./local_model_checkpoints/ s3://my-bucket/checkpoints/      # genuinely uploads
                                                                            # ONLY new/changed files,
                                                                            # recap the general
                                                                            # "skip unchanged work"
                                                                            # theme from MLOps
                                                                            # (Docker layer caching,
                                                                            # DVC's dvc repro) --
                                                                            # the SAME optimization
                                                                            # principle, now for S3 syncing
```

## Using S3 from Python — recap pandas' data-loading pattern directly

```python
import boto3      # the AWS SDK for Python
import pandas as pd

s3 = boto3.client('s3')
s3.download_file('my-ml-training-data', 'datasets/v1/training_data.csv', 'local_copy.csv')

df = pd.read_csv('local_copy.csv')      # recap the pandas folder directly from here onward, UNCHANGED
```

```python
# Or, genuinely more directly, pandas can read from S3 URLs without a
# separate download step first (requires s3fs installed)
df = pd.read_csv('s3://my-ml-training-data/datasets/v1/training_data.csv')
```

Genuinely worth recognizing this connects DIRECTLY back to SQL file 13's `pd.read_sql_query()` discussion — the SAME "pull data from wherever it actually lives, straight into a pandas DataFrame" pattern, just S3 instead of a SQL database as the source.

## S3 event triggers — connecting directly to file 03's Lambda discussion

```
Genuinely a real, powerful pattern: configure an S3 bucket to automatically
TRIGGER a Lambda function (recap file 03 directly) whenever a new object
is uploaded -- e.g. a new training data file lands in S3, automatically
triggering a data validation Lambda (recap MLOps file 08's data validation
gates directly) or kicking off a retraining pipeline
```

Genuinely worth recognizing this as a real, practical implementation of MLOps file 08's "automated retraining triggers" concept — instead of a SCHEDULED trigger (that file's cron example), an EVENT-DRIVEN trigger reacts immediately when new data genuinely arrives.

## Lifecycle policies — genuinely important, automated cost management

```json
{
  "Rules": [{
    "Status": "Enabled",
    "Transitions": [{"Days": 30, "StorageClass": "STANDARD_IA"}, {"Days": 90, "StorageClass": "GLACIER"}]
  }]
}
```

Genuinely a real, practical automation — automatically moving objects to CHEAPER storage classes as they age, without requiring manual intervention, directly connecting to this file's earlier storage-class cost discussion, now fully automated.

## Try this

```python
# Create an S3 bucket, enable versioning, and upload a small CSV dataset
# to it, using the aws CLI
# Write a Python script using boto3 to download it and load it into a
# pandas DataFrame, confirming the round-trip works correctly
# Modify the CSV slightly and re-upload it to the SAME key, then use
# aws s3api list-object-versions to confirm BOTH versions are genuinely
# retained
# Write a bucket policy granting read-only access to a specific IAM role
# (recap file 02's exercise directly) and confirm it correctly restricts
# access as expected
```

Confirming both dataset versions are genuinely retained after the second upload is the clearest, most concrete proof that S3 versioning is working as MLOps file 06's DVC-parallel discussion described.

## What's next
File 05 — AWS Databases: RDS & DynamoDB, covering managed database services — genuinely the cloud-hosted equivalent of everything covered throughout the SQL folder and Flask file 09's SQLAlchemy integration.
