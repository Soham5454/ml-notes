# 09 — Cloud Networking Basics

Recap file 08's closing note — covering VPCs, security groups, and load balancers, genuinely the networking layer underlying every service covered across files 02-08.

## VPC — genuinely your own private, isolated network within the cloud

```
VPC (Virtual Private Cloud): a genuinely ISOLATED network space within
AWS/GCP -- recap file 01's core cloud value proposition directly: just as
compute/storage are virtualized and rented, NETWORKING is too -- a VPC is
genuinely your own private slice of the provider's network infrastructure
```

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

Genuinely worth understanding the CIDR notation directly — `10.0.0.0/16` defines a RANGE of IP addresses (genuinely 65,536 addresses in this case) available WITHIN this VPC for resources (EC2 instances, file 03; RDS databases, file 05) to use.

## Subnets — genuinely dividing a VPC into smaller, purpose-specific segments

```bash
aws ec2 create-subnet --vpc-id vpc-0123456789 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a      # PUBLIC subnet
aws ec2 create-subnet --vpc-id vpc-0123456789 --cidr-block 10.0.2.0/24 --availability-zone us-east-1b      # PRIVATE subnet
```

Genuinely the key, important distinction worth understanding clearly:
- **Public subnet**: resources here CAN have a direct route to the internet (e.g. a Flask web server, recap that whole folder, that genuinely needs to receive public traffic)
- **Private subnet**: resources here have NO direct internet route — genuinely used for things that should NEVER be directly internet-accessible (a database, recap file 05's RDS directly — genuinely should NEVER be publicly reachable, only accessible FROM the application tier within the same VPC)

Genuinely worth connecting this directly to file 04's public-bucket security warning — the SAME underlying security principle (don't expose things to the internet unnecessarily) applied at the network-architecture level instead of the storage level.

## Security Groups — genuinely a firewall for individual resources

```bash
aws ec2 create-security-group --group-name web-sg --description "Web server security group" --vpc-id vpc-0123456789

aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789 \
  --protocol tcp --port 443 --cidr 0.0.0.0/0      # genuinely allows HTTPS traffic from ANYWHERE

aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789 \
  --protocol tcp --port 22 --cidr 203.0.113.0/24      # genuinely allows SSH ONLY from a specific, known IP range
```

Genuinely important, real security principle worth stating directly, recapping file 02's least-privilege discussion: a security group defines EXACTLY which traffic (protocol, port, source IP range) is allowed IN and OUT of a resource — genuinely the network-level equivalent of file 02's IAM least-privilege principle, applied to network TRAFFIC instead of API PERMISSIONS. Opening `0.0.0.0/0` (genuinely "from anywhere") on a sensitive port like SSH (22) is a real, common, dangerous misconfiguration — restricting SSH access to KNOWN IP ranges is genuinely standard, important practice.

## The genuinely realistic architecture — connecting files 03-05 together

```
Recap files 03-05 directly, now properly networked:
PUBLIC SUBNET: EC2 instance running the Flask app (file 03), security
  group allowing HTTPS (443) from anywhere, SSH (22) only from known IPs

PRIVATE SUBNET: RDS database instance (file 05), security group allowing
  PostgreSQL traffic (5432) ONLY from the web app's security group --
  genuinely NEVER directly reachable from the public internet
```

Genuinely the realistic, complete network architecture underlying the "Flask + RDS" pattern from file 05's exercise — the DATABASE genuinely should never be directly internet-accessible, only reachable from the application tier that legitimately needs it, a real, important security architecture worth understanding explicitly rather than just running commands without grasping WHY this structure matters.

## Load Balancers — recap MLOps file 11's Kubernetes Service concept directly

```bash
aws elbv2 create-load-balancer \
  --name my-app-lb \
  --subnets subnet-public1 subnet-public2 \      # genuinely spans MULTIPLE AZs, recap file 01's AZ redundancy
  --security-groups sg-0123456789
```

Genuinely worth recognizing this as the SAME concept from MLOps file 11's Kubernetes `Service` (type: LoadBalancer) discussion, and file 03's Auto Scaling Group — distributing incoming traffic across MULTIPLE running instances (recap file 03's ASG directly), automatically routing away from any UNHEALTHY instance (recap Flask file 06's `/health` endpoint and MLOps file 11's liveness/readiness probes directly — the SAME health-check concept, now at the cloud load-balancer level).

## DNS and domain routing — genuinely important, connecting a domain to actual infrastructure

```bash
# Route 53 (AWS) / Cloud DNS (GCP) -- genuinely maps a human-readable domain
# name to the actual IP address of a load balancer/instance
aws route53 change-resource-record-sets --hosted-zone-id Z123456 --change-batch file://dns-record.json
```

Genuinely worth knowing this exists as the FINAL piece connecting a real, memorable URL (like a portfolio project's domain, recap Flask file 11's deployment discussion) to the actual cloud infrastructure built throughout this folder.

## VPC Peering and private connectivity — genuinely important for multi-service architectures

```
Genuinely worth knowing this concept exists: VPC PEERING connects two
SEPARATE VPCs (e.g. one for the ML training environment, another for the
production serving environment) so resources in each can communicate
PRIVATELY, without traffic ever traversing the public internet -- a real,
important security/performance consideration for larger, multi-environment
ML systems
```

## NAT Gateways — genuinely important, a real, common point of confusion

```
Genuinely worth understanding this directly: a resource in a PRIVATE subnet
(recap this file's subnet discussion) has NO direct internet route -- but
it might still genuinely NEED outbound internet access (e.g. to download
a package via pip, recap Flask file 01's requirements.txt installation,
or to pull a pretrained model from Hugging Face's Hub, recap that folder)
A NAT Gateway provides this: OUTBOUND-only internet access for private
subnet resources, WITHOUT making them directly reachable FROM the internet
```

Genuinely a real, common confusion worth clarifying directly — "private subnet" doesn't mean "no internet access at all," it means "not directly REACHABLE from the internet" — a NAT Gateway is precisely the mechanism that lets private resources still initiate OUTBOUND connections (installing packages, calling external APIs) while remaining protected from unsolicited INBOUND connections.

## Try this

```bash
# Genuinely worth doing this exercise using the AWS console's VPC wizard
# (genuinely more visual/manageable for a first attempt than raw CLI commands)
# to build the realistic architecture described in this file:
# A VPC with one public and one private subnet, an EC2 instance (file 03) in
# the public subnet running your Flask app, an RDS instance (file 05) in
# the private subnet, and security groups correctly restricting the database
# to ONLY accept connections from the EC2 instance's security group
# Confirm you CAN connect to the Flask app from your browser (public route
# working), and confirm you CANNOT connect directly to the RDS instance
# from your own local machine (private subnet correctly isolated)
```

Confirming the database is genuinely UNREACHABLE from your local machine (while the Flask app IS reachable) is the clearest, most concrete proof that the public/private subnet architecture is correctly protecting the sensitive resource, exactly as intended.

## What's next
File 10 — Cost Management & Optimization, formalizing the cost-awareness themes touched on throughout files 01-09 into a genuinely systematic discipline for keeping cloud spending under control.
