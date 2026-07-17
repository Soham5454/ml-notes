# 03 — AWS Compute: EC2 & Serverless (Lambda)

Recap file 02's closing note — covering the two fundamental ways to actually RUN code on AWS, genuinely the compute foundation for training models and serving applications in the cloud.

## EC2 — genuinely just a rentable virtual machine

```
EC2 (Elastic Compute Cloud) -- recap file 01's IaaS definition directly:
genuinely a virtual machine you fully control, choose the OS, install
whatever software is needed, run it as long as required
```

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.medium \
  --key-name my-key-pair \
  --iam-instance-profile Name=my-ec2-role      # recap file 02's IAM Role discussion directly
```

## Instance types — genuinely important to choose correctly

```
t3.micro, t3.medium -- GENERAL PURPOSE, balanced CPU/memory, good for
  web apps (recap Flask folder), light workloads
c5.xlarge -- COMPUTE-OPTIMIZED, more CPU relative to memory, good for
  CPU-heavy processing
r5.xlarge -- MEMORY-OPTIMIZED, good for large in-memory datasets (recap
  pandas' memory-usage discussions)
p3.2xlarge, g4dn.xlarge -- GPU INSTANCES, genuinely essential for PyTorch
  training at real scale (recap PyTorch file 13's GPU/CUDA discussion directly --
  these instance types provide actual NVIDIA GPUs with CUDA support)
```

Genuinely important, real cost-awareness point worth stating directly — GPU instances (p3, g4dn families) are GENUINELY significantly more expensive per hour than CPU instances, recap PyTorch file 13's GPU necessity discussion for TRAINING specifically, but recap the same file's honest note that INFERENCE often runs acceptably on CPU — choosing the RIGHT instance type for the actual workload (training vs inference) is a genuinely important cost-optimization decision, not just a technical one.

## SSH access — connecting to a running EC2 instance

```bash
ssh -i my-key-pair.pem ec2-user@<instance-public-ip>
```

Genuinely worth understanding the key pair mechanism — EC2 uses SSH key-based authentication (recap the general public/private key cryptography concept, genuinely related to the password-hashing one-way-transformation idea from Flask file 10, though a distinct cryptographic mechanism) rather than passwords by default, and the `.pem` private key file needs the SAME careful protection as any other credential (recap file 01/02's credential-safety discipline directly).

## Setting up a training environment on EC2 — recap PyTorch/MLOps directly

```bash
# Once connected via SSH:
sudo yum update -y
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

pip install torch torchvision      # recap PyTorch file 01's installation instructions directly
```

Genuinely worth recognizing this as literally the SAME environment setup covered throughout the PyTorch folder, just performed on a remote, rentable machine instead of a local one — the actual ML CODE (recap PyTorch files 07-19's training loops, fine-tuning, etc.) is genuinely UNCHANGED, only the underlying hardware is now cloud-hosted.

## Stopping vs Terminating — genuinely important, real cost implications

```bash
aws ec2 stop-instances --instance-ids i-0123456789abcdef0        # STOPPED -- genuinely stops billing
                                                                       # for COMPUTE, but storage (EBS) still bills
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0     # TERMINATED -- genuinely deletes the
                                                                        # instance and its storage entirely
```

Genuinely important, real practical distinction worth internalizing directly — recap file 01's billing-alert warning: forgetting a RUNNING EC2 instance (especially a GPU one) is a genuinely common, real way people receive unexpectedly large bills. STOPPING an instance when not actively in use, and TERMINATING it entirely once genuinely done with it, is a real, important cost-management habit.

## Auto Scaling Groups — recap MLOps file 11's HPA concept, now at the VM level

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-app-asg \
  --min-size 2 --max-size 10 --desired-capacity 3 \
  --launch-template LaunchTemplateName=my-app-template
```

Genuinely worth recognizing this as the EXACT same elasticity concept from file 01 and MLOps file 11's Kubernetes HorizontalPodAutoscaler, just applied at the EC2 virtual-machine level instead of the container-pod level — automatically adding/removing EC2 instances based on demand, genuinely the same underlying "pay for what you use" philosophy.

## Lambda — genuinely serverless compute, a fundamentally different model

```python
# lambda_function.py
import json

def lambda_handler(event, context):
    text = event.get('text', '')
    # ... run a lightweight prediction, recap Flask file 06's inference logic directly ...
    prediction = "positive"      # placeholder

    return {
        'statusCode': 200,
        'body': json.dumps({'prediction': prediction})
    }
```

Genuinely the core conceptual shift worth understanding clearly: with EC2, you rent a MACHINE that runs continuously (billed whether it's actively processing a request or sitting idle). With Lambda, you deploy a FUNCTION, and AWS runs it ONLY when triggered (an API request, a file upload, a scheduled event) — genuinely billed per INVOCATION and execution time, not for idle time at all.

## When Lambda genuinely makes sense — the honest tradeoff

```
Genuinely good for Lambda: infrequent, short-duration tasks (a lightweight
  prediction endpoint with low/sporadic traffic, a data-processing trigger
  when a new file lands in S3, recap file 04)
Genuinely NOT good for Lambda: long-running training jobs (Lambda has a
  hard MAXIMUM execution time limit, genuinely 15 minutes), workloads
  needing GPU access (Lambda genuinely doesn't support GPU), or applications
  needing to maintain STATE between requests (recap Flask's model-loaded-
  once-at-startup pattern from file 06 -- Lambda's stateless, ephemeral
  nature genuinely conflicts with that pattern, covered more below)
```

## The "cold start" problem — genuinely important for ML workloads specifically

```
Genuinely important honest limitation: a Lambda function that hasn't been
invoked recently needs to "cold start" -- AWS provisions a fresh execution
environment, which can take genuinely noticeable TIME, especially if the
function needs to LOAD A LARGE MODEL FILE first (recap Flask file 06's
"load model once at startup" pattern -- Lambda's ephemeral nature means
this loading can genuinely happen repeatedly on cold starts, rather than
truly ONCE)
```

Genuinely worth knowing this is a REAL, well-documented challenge for serving large ML models via Lambda specifically — some teams mitigate it with "provisioned concurrency" (keeping some instances genuinely warm at extra cost) or by using Lambda only for lightweight models, while heavier models genuinely run better on EC2 (this file) or SageMaker endpoints (file 06).

## API Gateway — genuinely the standard way to expose a Lambda as an HTTP API

```
Recap Flask file 07's REST API discussion directly -- API Gateway is
AWS's managed service for creating HTTP endpoints that TRIGGER a Lambda
function, genuinely the serverless equivalent of a Flask route (recap
that folder's @app.route pattern), just with AWS managing the actual
HTTP server infrastructure entirely
```

## Try this

```bash
# Launch a small EC2 t3.micro instance (genuinely within the free tier),
# SSH into it, install Python and a lightweight scikit-learn model
# (recap Classical NLP file 07's classification pipeline), and confirm
# it can serve predictions -- then STOP the instance when done to avoid
# ongoing charges
# Separately, write a simple Lambda function that accepts a JSON payload
# and returns a hardcoded/simple prediction, deploy it, and connect it
# to an API Gateway endpoint -- test it with curl (recap Flask file 07's
# curl testing habit directly) and confirm you get a genuine, real HTTP
# response from a fully serverless setup
```

Building BOTH an EC2-based and a Lambda-based mini-deployment is genuinely the best way to feel the real, practical difference between "rent a persistent machine" and "deploy a function that runs on demand" firsthand, rather than just reading about the distinction.

## What's next
File 04 — AWS Storage: S3 & Data Management, covering where data, model artifacts, and datasets actually LIVE in AWS — genuinely the storage foundation nearly every other service in this folder depends on.
