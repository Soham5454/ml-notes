# 11 — Infrastructure as Code (Terraform)

Recap file 10's closing note — moving from MANUALLY running CLI commands (files 02-09) into DECLARATIVELY defining cloud infrastructure as version-controlled CODE, genuinely the final piece for truly reproducible, professional cloud infrastructure management.

## The problem with manual CLI commands — recap MLOps file 01's reproducibility theme directly

```bash
# Recap files 02-09's CLI commands directly -- genuinely fine for LEARNING
# and one-off experimentation, but for REAL infrastructure:
# - Commands run once, in a terminal, are genuinely NOT tracked anywhere
# - "What exact configuration is THIS resource currently running?" becomes
#   genuinely hard to answer after enough manual changes accumulate
# - Recreating an ENTIRE environment (recap MLOps file 06's reproducibility
#   discussion directly) from scattered CLI history is genuinely error-prone
```

## Infrastructure as Code — genuinely applying this whole roadmap's Git discipline to infrastructure itself

```
Recap this ENTIRE roadmap's git add/commit/push workflow directly --
Infrastructure as Code (IaC) means describing cloud infrastructure in
TEXT FILES, version-controlled in git, EXACTLY like application code
Genuinely the same reproducibility principle from MLOps file 06's DVC
discussion, now applied to the INFRASTRUCTURE layer instead of data/models
```

## Terraform — genuinely the most popular, provider-agnostic IaC tool

```hcl
# main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "ml_training_instance" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.medium"            # recap file 03's instance type discussion directly

  tags = {
    Project = "sentiment-classifier"       # recap file 10's tagging discipline directly
  }
}

resource "aws_s3_bucket" "training_data" {
  bucket = "my-ml-training-data-terraform"      # recap file 04's S3 discussion directly
}
```

Genuinely worth recognizing this file's structure directly — it DECLARATIVELY describes the DESIRED STATE ("I want an EC2 instance with these properties, an S3 bucket with this name"), rather than the IMPERATIVE step-by-step commands from files 02-09 ("run this CLI command, then that one"). Terraform figures out HOW to achieve that desired state automatically.

## The Terraform workflow — genuinely the core commands to know

```bash
terraform init          # downloads the necessary provider plugins (AWS, GCP, etc.)
terraform plan             # genuinely shows WHAT WOULD CHANGE, without actually changing anything yet
terraform apply               # actually creates/modifies the infrastructure to match the .tf files
terraform destroy                # genuinely tears down everything Terraform created -- important for
                                     # file 10's cost-management discipline, cleanly removing exercise resources
```

Genuinely worth understanding `terraform plan` as a REAL, important safety feature — recap Flask file 10's "verify with SELECT before UPDATE/DELETE" safety discipline directly: `plan` shows EXACTLY what will be created, modified, or destroyed BEFORE it actually happens, letting a genuinely risky or unintended change be caught and reviewed before real infrastructure gets affected.

## State — genuinely important, worth understanding how Terraform tracks reality

```
Terraform maintains a STATE FILE (terraform.tfstate) recording what
infrastructure it has ACTUALLY created and its current configuration
Genuinely important, real practice: for team environments, this state file
should be stored REMOTELY (e.g. in an S3 bucket, recap file 04 directly,
with locking to prevent simultaneous conflicting changes) rather than
just a local file, which would genuinely be lost or inconsistent across
different team members' machines
```

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state-bucket"
    key    = "ml-infrastructure/terraform.tfstate"
    region = "us-east-1"
  }
}
```

## Variables — genuinely important for reusable, environment-specific configuration

```hcl
# variables.tf
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "development"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

```hcl
# main.tf, using the variables
resource "aws_instance" "app_server" {
  instance_type = var.instance_type
  tags = {
    Environment = var.environment      # recap file 10's tagging discipline directly
  }
}
```

```bash
terraform apply -var="environment=production" -var="instance_type=t3.large"
```

Genuinely worth recognizing this as the SAME configuration-management discipline from MLOps file 06's `params.yaml` and Flask file 08's config classes — separating CONFIGURATION from the underlying STRUCTURE/logic, letting the same `.tf` files deploy DIFFERENT environments (development vs production) with different resource sizes/settings.

## A complete, realistic example — recap this entire folder's architecture

```hcl
# Recap file 09's public/private subnet architecture directly, now as code
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_subnet" "private" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"
}

resource "aws_security_group" "web_sg" {      # recap file 09's security group discussion directly
  vpc_id = aws_vpc.main.id
  ingress {
    from_port = 443
    to_port   = 443
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_db_instance" "app_db" {      # recap file 05's RDS discussion directly
  engine         = "postgres"
  instance_class = "db.t3.micro"
  subnet_group_name = aws_db_subnet_group.private.name      # placed in the PRIVATE subnet, recap file 09
}
```

Genuinely satisfying to see this — EVERY concept from files 02-09 (VPCs, subnets, security groups, RDS, EC2) genuinely gets expressed in ONE reviewable, version-controlled file, rather than a scattered sequence of manual CLI commands that would need to be perfectly remembered and re-run in exact order to recreate.

## Terraform and CI/CD — recap MLOps files 07-08 directly

```yaml
# .github/workflows/terraform.yml -- recap MLOps file 07's GitHub Actions structure directly
jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Terraform Init
        run: terraform init
      - name: Terraform Plan
        run: terraform plan
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'      # recap MLOps file 08's deployment-gating discussion directly
        run: terraform apply -auto-approve
```

Genuinely worth recognizing this as directly connecting Infrastructure as Code to the ENTIRE CI/CD discipline from MLOps files 07-08 — infrastructure CHANGES can genuinely go through the SAME automated review/testing/deployment pipeline as application code changes, a real, mature practice at genuinely professional organizations.

## Modules — genuinely important for reusable infrastructure patterns

```hcl
module "ml_training_environment" {
  source = "./modules/ml-training"
  instance_type = "p3.2xlarge"      # recap file 03's GPU instance discussion directly
  bucket_name   = "training-data-v2"
}
```

Genuinely worth knowing modules exist as Terraform's REUSABLE COMPONENT system — recap the general "don't repeat yourself" software engineering principle, and Flask file 08's Blueprint discussion directly: a well-designed module (e.g. "a standard ML training environment: VPC + GPU instance + S3 bucket") can genuinely be reused across multiple projects, rather than rewriting the same infrastructure definition repeatedly.

## Try this

```hcl
# Write Terraform configuration recreating file 09's exercise architecture:
# a VPC with public/private subnets, an EC2 instance in the public subnet,
# an RDS instance in the private subnet, and correctly scoped security groups
# Run terraform plan FIRST and carefully review exactly what would be created
# Run terraform apply and confirm the infrastructure matches what file 09's
# exercise built manually
# Make ONE small change (e.g. change the EC2 instance_type), run terraform
# plan again, and confirm it correctly shows ONLY that one resource needing
# modification, not a full recreation of everything
# Finally, run terraform destroy and confirm EVERYTHING created is fully
# and cleanly removed -- directly connecting to file 10's cost-management
# cleanup discipline
```

Confirming `terraform destroy` cleanly removes everything is genuinely valuable, real practice — directly closing the loop on file 10's cost-management concerns with a single, reliable command, rather than manually hunting down and deleting each resource individually.

## What's next
File 12 — the final Cloud → Frontier GenAI Bridge, pulling every concept from files 01-11 together and connecting them directly to cloud-hosted LLM APIs and serverless GenAI infrastructure — the immediate next phase of this roadmap.
