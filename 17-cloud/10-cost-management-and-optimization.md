# 10 — Cost Management & Optimization

Recap file 09's closing note — formalizing the cost-awareness themes touched on throughout files 01-09 (file 01's billing alerts, file 03's stop-vs-terminate, file 04's storage classes, file 07's preemptible VMs) into a genuinely systematic discipline.

## Why cloud costs genuinely get out of control easily — the honest, real problem

```
Recap file 01's original warning directly -- cloud billing is GENUINELY
different from a fixed monthly subscription: costs scale with ACTUAL
USAGE, which means a forgotten resource (file 03's stop-vs-terminate
warning), an inefficient architecture, or unexpectedly high traffic can
ALL produce a genuinely surprising, real bill -- this is a well-documented,
common problem, not a hypothetical concern
```

## Billing alerts and budgets — recap file 01's setup instruction, now fully explained

```bash
aws budgets create-budget \
  --account-id 123456789 \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

```json
{
  "BudgetName": "monthly-ml-budget",
  "BudgetLimit": {"Amount": "50", "Unit": "USD"},
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}
```

Genuinely worth setting this up as the FIRST, non-negotiable step in ANY real cloud account — recap file 01's opening instruction directly, this file finally shows the actual configuration behind that earlier advice: a budget alert genuinely notifies you (via email/SNS) BEFORE costs spiral, rather than discovering a large bill only at the end of the month.

## Cost Explorer / Billing reports — genuinely understanding WHERE money is actually going

```bash
aws ce get-cost-and-usage \
  --time-period Start=2026-06-01,End=2026-07-01 \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE
```

Genuinely worth building the habit of actually LOOKING at cost breakdowns BY SERVICE regularly — a genuinely common, real pattern: a forgotten GPU instance (file 03/06) or an over-provisioned RDS instance (file 05) can be identified and fixed QUICKLY once actually visible in a cost breakdown, rather than remaining a hidden, ongoing expense.

## Right-sizing — genuinely matching resources to actual need

```
Recap file 03's instance-type discussion directly -- a GENUINELY common,
real cost mistake: provisioning a large, powerful instance "just in case,"
when the actual workload needs far less
Right-sizing means genuinely MEASURING actual CPU/memory/GPU utilization
(recap the general profiling/monitoring discipline from MLOps file 09)
and matching the instance type to REAL, observed need, not assumed need
```

Genuinely worth connecting this directly to file 03's honest note about GPU instances being significantly more expensive — training a genuinely small model on an expensive multi-GPU instance, when a single smaller GPU (or even CPU, recap PyTorch file 13's honest note about inference often not needing GPU at all) would suffice, is a real, common, avoidable cost.

## Reserved Instances / Committed Use Discounts — genuinely important for STEADY, predictable workloads

```
Genuinely worth knowing this cost lever exists: committing to use a
SPECIFIC instance type for 1-3 years upfront (Reserved Instances on AWS,
Committed Use Discounts on GCP) genuinely offers SIGNIFICANT discounts
(often 30-70%) over on-demand pricing, for workloads with GENUINELY
predictable, steady usage (e.g. a production Flask app, recap that folder,
running continuously)
```

Genuinely worth understanding the honest tradeoff — recap file 07's Preemptible/Spot discussion directly, forming a genuinely complete picture of AWS/GCP's pricing spectrum: On-Demand (most flexible, most expensive) → Reserved/Committed (cheaper, but requires a genuine, real usage-duration commitment) → Spot/Preemptible (cheapest, but genuinely can be reclaimed) — choosing the right MIX for different workload types (steady production traffic vs. flexible batch training) is a real, practical cost-optimization skill.

## Storage cost optimization — recap file 04's storage classes and lifecycle policies directly

```json
{
  "Rules": [{"Status": "Enabled", "Transitions": [{"Days": 30, "StorageClass": "STANDARD_IA"}]}]
}
```

Genuinely worth recapping file 04's lifecycle policy example directly as a real, automated cost-optimization tool — recap MLOps file 06's DVC discussion too: OLD dataset versions, genuinely rarely accessed but retained for reproducibility, are a real, common candidate for automatic transition to cheaper storage tiers.

## Auto-scaling for cost efficiency — recap file 01/03/MLOps file 11's elasticity theme directly

```
Genuinely the DIRECT cost-optimization payoff of the elasticity concept
covered repeatedly throughout this whole folder -- scaling DOWN during
genuinely low-traffic periods (nights, weekends, depending on the
application's real usage pattern) directly reduces cost, rather than
paying for PEAK capacity around the clock
```

## Serverless for cost efficiency — recap file 03's Lambda discussion directly

```
Recap file 03's Lambda "pay per invocation, not for idle time" discussion
directly -- for GENUINELY sporadic, low-traffic ML inference workloads,
Lambda (or Cloud Functions, GCP's equivalent) can be significantly cheaper
than keeping an EC2 instance running continuously, precisely because
there's ZERO cost during idle periods
```

## Spot/Preemptible instances for training — recap file 07's discussion, now the FULL cost picture

```python
# Recap file 07's checkpoint-and-resume discussion directly -- combining
# PyTorch's checkpointing discipline (recap PyTorch file 14) with Spot/
# Preemptible instances is genuinely one of the highest-value cost
# optimizations for TRAINING workloads specifically, since training is
# genuinely often fault-tolerant (can resume from a checkpoint) in a way
# that a live-serving production API (which needs continuous uptime) is not
```

## Tagging resources — genuinely important for cost ATTRIBUTION, not just tracking

```bash
aws ec2 create-tags --resources i-0123456789 --tags Key=Project,Value=sentiment-classifier Key=Environment,Value=training
```

Genuinely worth building this habit — recap the general "understand where money is going" theme directly: tagging every resource with its PROJECT/PURPOSE lets cost reports (via Cost Explorer, shown earlier) be broken down MEANINGFULLY ("how much did the sentiment classifier project cost this month" rather than just "how much did EC2 cost overall") — genuinely essential once managing multiple projects/experiments simultaneously.

## The honest, complete cost-optimization checklist

```
[ ] Billing alerts/budgets configured (genuinely the FIRST step, recap file 01)
[ ] Resources STOPPED or TERMINATED when not actively needed (recap file 03)
[ ] Instance types RIGHT-SIZED to actual measured need, not assumed need
[ ] Storage lifecycle policies configured for aging, rarely-accessed data (recap file 04)
[ ] Spot/Preemptible instances used for fault-tolerant training workloads (recap file 07)
[ ] Reserved/Committed pricing considered for GENUINELY steady, predictable workloads
[ ] Resources properly TAGGED for cost attribution and visibility
[ ] Regular (e.g. weekly) review of Cost Explorer/Billing reports, not just a one-time setup
```

## Try this

```bash
# In your AWS/GCP account, set up a genuinely real budget alert at a small,
# meaningful threshold (e.g. $10)
# Review your account's current resources (EC2 instances, RDS databases,
# S3 buckets from previous exercises) and confirm NOTHING is running that
# shouldn't be -- genuinely stop/terminate/delete anything left over from
# earlier exercises in this folder
# Add meaningful tags (Project, Environment) to any remaining resources
# Pull a Cost Explorer report for the current month and identify the
# single LARGEST cost driver -- write down whether it's genuinely
# necessary or could be optimized using one of this file's techniques
```

Genuinely worth treating this exercise as real, ongoing hygiene, not a one-time task — actually cleaning up leftover resources from this folder's earlier exercises is a genuinely practical, real habit worth building permanently.

## What's next
File 11 — Infrastructure as Code (Terraform), moving from MANUALLY running CLI commands (files 02-09) into DECLARATIVELY defining cloud infrastructure as version-controlled code — genuinely the final piece needed for truly reproducible, professional cloud infrastructure management.
