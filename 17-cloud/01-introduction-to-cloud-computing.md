# 01 — Introduction to Cloud Computing & Core Concepts

Starting the Cloud phase — recap MLOps file 12's closing bridge directly: this is precisely where the self-managed tools from that whole folder (Docker, Kubernetes, MLflow) meet their MANAGED, provider-hosted equivalents at real organizational scale.

## What "the cloud" actually is — genuinely just someone else's computers, rented

```
Genuinely the simplest honest definition: cloud computing means renting
COMPUTE, STORAGE, and other infrastructure from a provider (AWS, GCP, Azure)
over the internet, paying for what's used, INSTEAD of buying and maintaining
physical servers yourself
```

Genuinely worth grounding this in the "why" — recap Flask file 11's hosting discussion and MLOps file 01's reproducibility concerns: buying physical hardware requires huge upfront cost, ongoing maintenance, and genuinely can't scale up/down quickly to match actual demand. Cloud computing converts this into a flexible, pay-as-you-go OPERATING expense instead.

## IaaS, PaaS, SaaS — genuinely the fundamental service-level taxonomy

```
IaaS (Infrastructure as a Service): rent raw compute/storage/networking --
  genuinely just virtual machines, you manage the OS, runtime, everything above it
  (e.g. AWS EC2, recap file 03)

PaaS (Platform as a Service): the provider manages the OS/runtime, you just
  deploy CODE -- genuinely less to manage, less flexibility
  (e.g. AWS Elastic Beanstalk, Google App Engine)

SaaS (Software as a Service): a COMPLETE application, ready to use --
  genuinely zero infrastructure management at all
  (e.g. Gmail, Salesforce -- not something you'd typically BUILD, just USE)
```

Genuinely worth understanding this as a SPECTRUM of how much infrastructure responsibility you take on versus hand off to the provider — directly connecting to MLOps file 12's "self-managed vs managed" tradeoff framework, just formalized with standard industry terminology.

## Regions and Availability Zones — genuinely important geographic concepts

```
REGION: a genuinely large geographic area (e.g. us-east-1 in Virginia,
  ap-south-1 in Mumbai) -- choosing the RIGHT region matters for latency
  (closer to users = faster) and for legal/compliance reasons (some data
  must genuinely stay within certain countries)

AVAILABILITY ZONE (AZ): within a region, MULTIPLE physically separate
  data centers -- genuinely isolated from each other's power/cooling/network
  failures
```

Genuinely worth understanding WHY AZs matter directly — recap MLOps file 11's Kubernetes self-healing discussion: deploying an application across MULTIPLE AZs within a region means a single data center failure (power outage, hardware failure) doesn't take the whole application down — genuinely the cloud-infrastructure-level version of the same redundancy principle that motivated running multiple pod replicas.

## Elasticity and scalability — genuinely the core value proposition

```python
# Recap MLOps file 11's Horizontal Pod Autoscaler directly -- this EXACT
# concept, generalized beyond Kubernetes specifically:
# ELASTICITY: automatically scale resources UP during high demand, DOWN
# during low demand -- genuinely paying only for what's actually needed
# at any given moment, rather than provisioning for PEAK capacity permanently
```

Genuinely worth connecting this directly back to MLOps file 11's HPA example — that WAS a cloud-computing elasticity concept, just scoped to Kubernetes pods specifically. The SAME idea applies at every layer: virtual machines, storage, database throughput.

## Managed vs self-managed services — recap MLOps file 12's core framework directly

```
Genuinely the SAME framework from MLOps file 12, now the central organizing
principle of this ENTIRE Cloud folder:
Self-managed: install and operate PostgreSQL yourself on a virtual machine
Managed: use AWS RDS (file 05) -- AWS handles backups, patching, failover
  automatically, at a genuinely real dollar cost premium over self-managing
```

Genuinely worth carrying this exact framework forward as the lens for evaluating EVERY service covered in this folder — the recurring question "is the operational-burden reduction worth the added cost/lock-in" applies to nearly every AWS/GCP service choice.

## AWS vs GCP vs Azure — the honest, practical landscape

```
AWS: genuinely the largest, most mature provider, broadest service catalog,
  most common in job postings/industry usage overall
GCP: genuinely strong specifically for AI/ML workloads (Google's own deep
  ML research heritage shows in Vertex AI's genuinely tight integration
  with TensorFlow/JAX, and TPU access), often preferred by ML-focused teams
Azure: genuinely strong in enterprise/Microsoft-ecosystem-heavy organizations
```

Genuinely worth an honest, practical note for this roadmap's focus — this folder covers BOTH AWS (files 02-06, the broadest industry-standard skill) and GCP (files 07-08, genuinely valuable for AI/ML-specific roles) deliberately, since real job postings genuinely vary in which they require, and the underlying CONCEPTS (compute, storage, managed ML services) transfer almost completely between them once understood on one platform.

## The free tier — genuinely important for actually practicing this folder's content

```
Both AWS and GCP offer GENUINELY real free tiers for new accounts --
worth signing up for both, but with real, important caution:
- Set up BILLING ALERTS immediately (genuinely a common story: forgetting
  to shut down a resource and receiving an unexpectedly large bill)
- NEVER commit AWS/GCP credentials to a GitHub repo (recap the extensive
  credential-safety discipline from Flask and MLOps folders directly --
  this is genuinely one of the most common, damaging real mistakes,
  since exposed cloud credentials can be exploited by others to run up
  charges on your account)
```

Genuinely worth taking this warning seriously and concretely — exposed AWS keys in a public GitHub repo are a genuinely well-documented, common way for people to receive shocking, real bills from unauthorized usage. The `.gitignore` discipline from this whole roadmap's Git workflow, and the environment-variable secret management from Flask/MLOps folders, apply with GENUINELY higher stakes here.

## Cloud shell and CLI tools — genuinely the practical entry point

```bash
# AWS CLI -- genuinely the standard way to interact with AWS from a terminal,
# rather than always using the web console
aws configure      # sets up credentials locally (recap the credential-safety
                       # warning above -- genuinely never commit the resulting
                       # ~/.aws/credentials file to git)
aws s3 ls              # example: list S3 buckets (recap file 04)

# GCP equivalent
gcloud auth login
gcloud config set project my-project-id
```

Genuinely worth building comfort with BOTH the web console (genuinely good for learning/exploring visually) and the CLI (genuinely essential for automation, and directly connects to MLOps file 07's CI/CD pipelines, which typically use CLI tools rather than manual console clicks).

## Try this

```bash
# Sign up for BOTH an AWS free tier account and a GCP free tier account
# Set up billing alerts on BOTH immediately (genuinely the first, most
# important step, before creating any real resources)
# Install and configure both the aws CLI and gcloud CLI locally
# Run aws s3 ls and gcloud projects list as a basic "is this working"
# sanity check
# Confirm your local credential files (~/.aws/credentials, gcloud's config)
# are NOT inside any git-tracked directory, or are properly gitignored if
# they happen to be
```

Setting up billing alerts and confirming credential safety BEFORE creating any real resources is genuinely the most important, practical first step in this entire folder — worth treating as non-negotiable, not optional housekeeping.

## What's next
File 02 — AWS Fundamentals: IAM & Account Setup, covering AWS's identity and access management system — genuinely the foundational security layer underlying every other AWS service covered in this folder.
