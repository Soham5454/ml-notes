# 05 — AWS Databases: RDS & DynamoDB

Recap file 04's closing note — covering managed database services, genuinely the cloud-hosted equivalent of everything covered throughout the SQL folder and Flask file 09's SQLAlchemy integration.

## RDS — managed relational databases, recap the ENTIRE SQL folder directly

```
RDS (Relational Database Service): AWS manages the actual DATABASE SERVER
(PostgreSQL, MySQL, or others) -- automatic backups, patching, failover --
genuinely everything covered throughout the SQL folder (SELECT, JOIN,
GROUP BY, transactions, all of it) applies COMPLETELY UNCHANGED, only the
database's HOSTING is different
```

```bash
aws rds create-db-instance \
  --db-instance-identifier my-ml-app-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password <genuinely-secure-password> \
  --allocated-storage 20
```

Genuinely worth recognizing directly, recapping file 01's managed-vs-self-managed framework: this is EXACTLY the "self-managed PostgreSQL on an EC2 instance vs. AWS RDS" comparison from that file, now made completely concrete — RDS handles backups, security patching, and failover automatically, at a real cost premium over self-hosting the SAME database on a raw EC2 instance.

## Connecting Flask to RDS — recap Flask file 09's SQLAlchemy setup directly

```python
# Recap Flask file 08's config management directly -- genuinely the ONLY
# change needed is the DATABASE_URL environment variable
DATABASE_URL = "postgresql://admin:password@my-ml-app-db.xxxxx.us-east-1.rds.amazonaws.com:5432/mydb"

app.config['SQLALCHEMY_DATABASE_URI'] = os.environ.get('DATABASE_URL')      # recap Flask file 09 EXACTLY
```

Genuinely worth pausing on this directly — EVERY line of SQLAlchemy code from Flask file 09 (the `Prediction` model, `.query.filter_by()`, `db.session.commit()`) works IDENTICALLY against RDS as it did against local SQLite, since RDS is genuinely just a real, managed PostgreSQL server reachable over the network — the application code doesn't need to know or care that it's now cloud-hosted.

## Read Replicas — genuinely important for scaling read-heavy workloads

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier my-ml-app-db-replica \
  --source-db-instance-identifier my-ml-app-db
```

Genuinely worth understanding the core idea: a read replica is a CONTINUOUSLY SYNCED COPY of the primary database, genuinely useful for offloading READ-heavy queries (recap MLOps file 09's prediction-logging analytics queries, or SQL file 04's aggregate reporting queries) onto the replica, keeping the PRIMARY database's capacity free for the application's actual write traffic. Directly connects to file 01's elasticity/scalability theme, applied specifically to databases.

## Multi-AZ deployments — recap file 01's Availability Zone discussion directly

```bash
aws rds create-db-instance --multi-az ...
```

Genuinely worth recognizing this as file 01's AZ redundancy concept, applied specifically to databases — a Multi-AZ RDS deployment maintains a SYNCHRONOUS STANDBY copy in a DIFFERENT availability zone, and automatically FAILS OVER to it if the primary AZ experiences an outage — genuinely important for production systems where database downtime is unacceptable.

## DynamoDB — a genuinely different database paradigm, NoSQL

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('predictions')

table.put_item(Item={
    'request_id': 'abc123',
    'text': 'This is great!',
    'prediction': 'positive',
    'confidence': 0.95
})

response = table.get_item(Key={'request_id': 'abc123'})
print(response['Item'])
```

Genuinely important, real conceptual distinction from RDS/SQL directly: DynamoDB is a KEY-VALUE / document store, NOT relational — recap SQL file 09's schema/normalization discussion directly: DynamoDB genuinely does NOT enforce a fixed schema across items the way SQL's `CREATE TABLE` does, and it doesn't natively support JOINs (recap SQL file 05 directly) the way relational databases do. Different items in the SAME table can genuinely have different attributes.

## When DynamoDB genuinely makes sense over RDS — the honest tradeoff

```
Genuinely good for DynamoDB: extremely high-scale, simple key-based lookups
  (recap DSA file 06's hashing discussion directly -- DynamoDB's core access
  pattern is genuinely a distributed hash table), data with a FLEXIBLE/
  EVOLVING schema, needing very LOW, predictable latency at massive scale

Genuinely good for RDS: complex queries needing JOINs/aggregations (recap
  SQL files 04-07 directly), genuinely relational data with clear structure,
  transactions spanning multiple related records (recap SQL file 10's
  ACID discussion directly -- RDS provides FULL ACID guarantees; DynamoDB's
  guarantees are more limited/different by design)
```

Genuinely the same honest engineering-tradeoff framework echoed throughout this entire roadmap — DynamoDB trades RELATIONAL query power (SQL folder's whole toolkit) for MASSIVE horizontal scalability and predictable low latency; RDS trades some of that raw scalability ceiling for the FULL expressive power of SQL already deeply covered.

## DynamoDB for ML use cases — genuinely practical, real applications

```python
# Genuinely common real pattern: storing FEATURE VECTORS for a feature store
# (recap the general "feature store" concept from the original roadmap
# gap-analysis conversation), where lookups are genuinely simple "get
# features for user_id X" queries at very high request volume

table.put_item(Item={
    'user_id': 'user_456',
    'features': {'age': 34, 'purchase_count': 12, 'avg_order_value': 45.30},
    'last_updated': '2026-07-17'
})
```

Genuinely worth knowing this real pattern — recap MLOps file 09's real-time prediction logging discussion directly: DynamoDB's low-latency key-based lookups make it a genuinely common choice for SERVING pre-computed features to a model at INFERENCE time, where speed and predictable latency matter more than complex relational querying.

## Provisioned vs On-Demand capacity — recap file 01's elasticity theme directly

```bash
aws dynamodb create-table --table-name predictions --billing-mode PAY_PER_REQUEST      # genuinely
                                                                                            # scales automatically,
                                                                                            # billed per actual request

# vs PROVISIONED mode -- genuinely specify exact read/write capacity units upfront,
# cheaper at STEADY, predictable traffic, but requires manual/auto-scaling configuration
```

Genuinely worth recognizing `PAY_PER_REQUEST` as file 01's elasticity concept in its purest form — zero capacity planning required, genuinely pay only for actual usage, ideal for unpredictable or new workloads where the real traffic pattern isn't yet well understood.

## Try this

```python
# Create a small RDS PostgreSQL instance (genuinely within the free tier)
# and connect the Flask app from Flask file 09's exercise to it, replacing
# the local SQLite DATABASE_URL with the RDS connection string
# Confirm every existing route (prediction logging, history, stats) works
# IDENTICALLY against the cloud-hosted database
# Separately, create a DynamoDB table and write a small script that stores
# and retrieves a few "feature vector" style records using put_item/get_item,
# confirming the key-value access pattern works as described in this file
```

Confirming the EXISTING Flask app works unchanged against RDS is genuinely the clearest proof that "managed cloud database" doesn't mean "different application code" — only the connection string genuinely changes.

## What's next
File 06 — AWS ML Services: SageMaker Deep Dive, covering AWS's genuinely dedicated, purpose-built ML platform — directly connecting to MLOps file 12's bridge discussion of managed experiment tracking, training, and deployment.
