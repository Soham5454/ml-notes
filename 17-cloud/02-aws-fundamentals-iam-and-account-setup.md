# 02 — AWS Fundamentals: IAM & Account Setup

Recap file 01's closing note — covering AWS's Identity and Access Management system, genuinely the foundational security layer underlying every other AWS service in this folder.

## What IAM actually controls

```
IAM (Identity and Access Management) answers two questions for EVERY
request to AWS: WHO is making this request, and WHAT are they ALLOWED to do
Genuinely the security foundation everything else in this folder depends on --
misconfigured IAM is one of the most common REAL sources of cloud security
incidents (recap file 01's credential-safety warning directly, now extended
to PERMISSIONS, not just credential exposure)
```

## The root account — genuinely important, use sparingly

```
The ROOT account (created when first signing up) has UNRESTRICTED access
to everything -- genuinely important practice: use it ONLY for genuinely
account-level tasks (billing setup, initial IAM configuration), then
create separate IAM USERS (covered next) for actual day-to-day work
```

Genuinely worth internalizing this as a real, standard security practice — never using the root account for routine work means even if ONE set of credentials gets compromised, the damage is genuinely limited to whatever that specific IAM user was permitted to do, not full account control.

## IAM Users — individual identities

```bash
aws iam create-user --user-name soham-dev

aws iam create-access-key --user-name soham-dev      # generates the actual credentials
```

Genuinely worth flagging directly, recapping file 01's warning: the access key ID and SECRET access key returned here are shown ONLY ONCE — genuinely important to store them securely (a password manager, environment variables, recap Flask/MLOps folders' secret-management discipline) rather than losing them or, far worse, committing them anywhere near a git repository.

## IAM Policies — defining WHAT is allowed, genuinely just JSON documents

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-ml-bucket/*"
    }
  ]
}
```

Genuinely worth reading this structure carefully — `Effect: Allow` (or `Deny`), `Action` (WHICH operations, e.g. `s3:GetObject` = reading from S3, recap file 04), `Resource` (WHICH specific resources this applies to, using an ARN — Amazon Resource Name, a genuinely standard unique identifier format across all AWS services). This policy genuinely grants ONLY read/write access to objects within one specific S3 bucket, nothing more.

## The Principle of Least Privilege — genuinely the single most important IAM concept

```
Genuinely important security principle, worth stating directly: grant EACH
identity (user, service, application) ONLY the permissions it genuinely
NEEDS to do its job, nothing more
Recap the general "reduce blast radius" theme from MLOps file 10's deployment
strategies directly -- the SAME underlying philosophy, applied here to
PERMISSIONS instead of deployment traffic
```

Genuinely worth grounding this in a concrete, realistic example: a Flask application (recap that whole folder) that only needs to READ from one specific S3 bucket should genuinely be granted a policy allowing ONLY `s3:GetObject` on THAT bucket — NOT broad `AdministratorAccess`, even though the broader permission would technically also work. If that application were ever compromised, least-privilege access genuinely limits what an attacker could do with its credentials.

## IAM Roles — genuinely different from Users, worth the distinction

```
IAM USER: represents a PERSON (or a long-lived application) with permanent
  credentials

IAM ROLE: a set of permissions that can be TEMPORARILY ASSUMED -- genuinely
  no permanent credentials attached directly, used by AWS SERVICES themselves
  (e.g. an EC2 instance, file 03, or a Lambda function) to access OTHER
  AWS services securely
```

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Genuinely important, real best practice worth understanding clearly: an EC2 instance (file 03) running a training script that needs to read data from S3 (file 04) should genuinely use an IAM ROLE attached to that instance, NOT hardcoded access keys stored on the instance itself — recap the general "never hardcode secrets" discipline directly, roles are precisely AWS's mechanism for avoiding that anti-pattern entirely, since the temporary credentials are managed automatically and never need to be stored as static secrets at all.

## IAM Groups — genuinely useful for managing multiple users efficiently

```bash
aws iam create-group --group-name ml-engineers
aws iam attach-group-policy --group-name ml-engineers --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam add-user-to-group --user-name soham-dev --group-name ml-engineers
```

Genuinely worth understanding the practical benefit — rather than attaching policies to EVERY individual user one at a time, GROUPS let a whole TEAM's permissions be managed in ONE place; adding a new team member genuinely just means adding them to the right group, automatically inheriting the correct permission set.

## Multi-Factor Authentication (MFA) — genuinely non-negotiable for real accounts

```
Genuinely important, real security practice worth stating directly: enable
MFA on the root account IMMEDIATELY, and genuinely encourage/require it for
IAM users too -- recap file 01's credential-exposure warning, MFA provides
a real, additional layer of protection even if a password/access key IS
somehow compromised
```

## Access keys vs temporary credentials — genuinely important, evolving best practice

```python
# Genuinely worth knowing: AWS's CURRENT best practice increasingly discourages
# long-lived access keys (which can be accidentally leaked, recap file 01)
# in favor of TEMPORARY credentials via IAM Roles (this file) or AWS SSO/
# Identity Center for human users -- worth being aware this is the direction
# the field is genuinely moving, even though long-lived keys remain common
# in many existing systems and simpler tutorials
```

## Try this

```bash
# In your AWS free tier account (recap file 01's setup):
# 1. Enable MFA on the root account
# 2. Create a new IAM user for yourself (NOT using root for daily work)
# 3. Write a custom IAM policy granting ONLY read access to a single S3
#    bucket (create an empty test bucket first, recap file 04 for the
#    actual bucket creation command)
# 4. Attach that policy to your new IAM user
# 5. Configure the aws CLI with THIS user's credentials (not root) and
#    confirm you CAN list objects in that one bucket, but CANNOT perform
#    an action outside that policy's scope (e.g. try creating a NEW bucket
#    and confirm it's correctly denied)
```

Deliberately testing that a DENIED action is correctly rejected is genuinely the clearest, most concrete proof that least-privilege IAM configuration is actually working, rather than just assuming the policy JSON is correct.

## What's next
File 03 — AWS Compute: EC2 & Serverless (Lambda), covering the two fundamental ways to actually RUN code on AWS — genuinely the compute foundation for training models and serving applications in the cloud.
