# Cloud — A to Z

Notes on cloud computing fundamentals for AI/ML — the managed, provider-hosted equivalents of everything covered throughout the MLOps folder. Covers AWS (IAM, compute, storage, databases, SageMaker) and GCP (compute, storage, Vertex AI), plus networking, cost management, and infrastructure as code.

## Index

| # | Topic | File |
|---|---|---|
| 01 | Introduction to Cloud Computing & Core Concepts | [01-introduction-to-cloud-computing.md](01-introduction-to-cloud-computing.md) |
| 02 | AWS Fundamentals: IAM & Account Setup | [02-aws-fundamentals-iam-and-account-setup.md](02-aws-fundamentals-iam-and-account-setup.md) |
| 03 | AWS Compute: EC2 & Lambda | [03-aws-compute-ec2-and-lambda.md](03-aws-compute-ec2-and-lambda.md) |
| 04 | AWS Storage: S3 & Data Management | [04-aws-storage-s3-and-data-management.md](04-aws-storage-s3-and-data-management.md) |
| 05 | AWS Databases: RDS & DynamoDB | [05-aws-databases-rds-and-dynamodb.md](05-aws-databases-rds-and-dynamodb.md) |
| 06 | AWS ML Services: SageMaker Deep Dive | [06-aws-ml-services-sagemaker-deep-dive.md](06-aws-ml-services-sagemaker-deep-dive.md) |
| 07 | GCP Fundamentals: Compute & Storage | [07-gcp-fundamentals-compute-and-storage.md](07-gcp-fundamentals-compute-and-storage.md) |
| 08 | GCP ML Services: Vertex AI | [08-gcp-ml-services-vertex-ai.md](08-gcp-ml-services-vertex-ai.md) |
| 09 | Cloud Networking Basics | [09-cloud-networking-basics.md](09-cloud-networking-basics.md) |
| 10 | Cost Management & Optimization | [10-cost-management-and-optimization.md](10-cost-management-and-optimization.md) |
| 11 | Infrastructure as Code (Terraform) | [11-infrastructure-as-code-terraform.md](11-infrastructure-as-code-terraform.md) |
| 12 | Cloud → Frontier GenAI Bridge | [12-cloud-to-frontier-genai-bridge.md](12-cloud-to-frontier-genai-bridge.md) |

## How to read these
File 01 covers universal cloud concepts (IaaS/PaaS/SaaS, regions, elasticity) that apply across every provider. Files 02-06 go deep on AWS specifically — IAM, compute, storage, databases, and SageMaker. Files 07-08 cover GCP's equivalents, reinforcing that the underlying concepts transfer directly. Files 09-11 cover networking, cost discipline, and infrastructure as code. File 12 closes the loop into the Frontier GenAI phase ahead. Every file explicitly cross-references the MLOps, PyTorch, Flask, and SQL folders' equivalent self-managed concepts.

## Roadmap position
Python Basics → NumPy → pandas → Matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Math Foundations → DSA → SQL → Classical NLP → Hugging Face → OpenCV → Flask → MLOps → **Cloud (done)** → Frontier GenAI → ML System Design → Interview Prep
