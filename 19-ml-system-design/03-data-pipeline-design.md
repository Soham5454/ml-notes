# 03 — Data Pipeline Design

Recap file 02's closing note — covering how data actually gets collected, stored, and processed at scale, genuinely the foundation every subsequent design decision (features, model, serving) builds on top of.

## Data collection — recap SQL and Cloud folders' storage discussions directly

```
Genuinely important sources to consider explicitly:
- USER-GENERATED events (clicks, purchases, searches -- recap the general
  "log everything meaningfully" discipline from MLOps file 09 directly)
- SYSTEM logs (recap Flask file 09's prediction-logging discussion, and
  MLOps file 09's monitoring discipline, directly)
- EXTERNAL data sources (recap Frontier GenAI file 04's API-integration
  discussion directly, or Cloud file 04's S3-based data ingestion)
```

## Batch vs Streaming ingestion — genuinely an important, real architectural decision

```
BATCH: data collected and processed in scheduled CHUNKS (recap MLOps
  file 07's scheduled-workflow/cron discussion directly) -- e.g. nightly
  ETL jobs pulling from a production database (recap SQL/Cloud file 05's
  RDS discussion) into a data warehouse (recap Cloud file 07's BigQuery
  discussion directly)

STREAMING: data processed CONTINUOUSLY, as it arrives (genuinely a NEW
  concept for this roadmap, worth knowing: tools like Kafka/Kinesis
  provide a durable, ordered EVENT STREAM that multiple downstream
  consumers can process in real time)
```

Genuinely worth connecting this decision directly back to file 02's latency-requirement discussion — a REAL-TIME fraud detection system (file 09) genuinely needs STREAMING ingestion (a transaction must be scored within milliseconds of occurring), while a weekly recommendation refresh (file 07) is genuinely well-served by BATCH processing, which is simpler and cheaper to build/operate.

## The Lambda Architecture — genuinely a useful, real pattern combining both

```
Genuinely worth knowing this real, named pattern: maintain BOTH a batch
layer (processes ALL historical data periodically, recap MLOps file 07's
scheduled-workflow discussion, producing a comprehensive but slightly
STALE view) and a speed/streaming layer (processes RECENT data
immediately, providing FRESH but potentially less complete results) --
serving layer COMBINES both views
```

Genuinely worth an honest note — this pattern adds real, meaningful complexity (recap the recurring "is this complexity genuinely justified" tradeoff theme from throughout this roadmap), worth proposing ONLY when a system genuinely needs BOTH comprehensive historical accuracy AND real-time freshness simultaneously — not a default architecture to reach for automatically.

## Data storage — recap Cloud files 04/05/07's storage-choice framework directly

```
Recap Cloud file 05's RDS-vs-DynamoDB honest tradeoff table directly,
now applied at system-design scale:
- RAW EVENT DATA -> genuinely often object storage (recap Cloud file 04's
  S3 discussion) -- cheap, scalable, good for large volumes of
  semi-structured logs
- STRUCTURED, QUERYABLE data -> recap SQL folder + Cloud file 05's RDS
  discussion directly, for genuinely relational data needing JOINs/aggregations
- ANALYTICAL queries at scale -> recap Cloud file 07's BigQuery discussion
  directly -- a genuinely different tool for "run complex SQL over
  terabytes of historical data" versus RDS's transactional-workload focus
- FEATURE VECTORS for low-latency serving -> recap Cloud file 05's
  DynamoDB discussion directly, or a genuinely dedicated feature store
  (full treatment in file 04)
```

## Data validation and quality — recap MLOps file 08's data-validation-gate discussion directly

```python
# Recap MLOps file 08's EXACT validation pattern, now at system-design scale
def validate_incoming_data(batch):
    assert batch['user_id'].notna().all(), "Missing user IDs"      # recap pandas folder directly
    assert (batch['transaction_amount'] >= 0).all(), "Negative amounts detected"
    # Genuinely important at SCALE: log/alert on validation failures
    # (recap MLOps file 09's alerting discussion directly) rather than
    # silently dropping bad data or crashing the entire pipeline
```

Genuinely worth explicitly mentioning this in a system design interview — recap MLOps file 01's "code, data, model" three-way risk discussion directly: a genuinely complete data pipeline design ALWAYS includes a validation step, not just "data flows from A to B."

## Handling data drift and schema evolution — recap MLOps file 09's drift discussion directly

```
Genuinely important, real consideration worth raising proactively:
production data SCHEMAS change over time (a new field gets added, an old
one deprecated) and DISTRIBUTIONS shift (recap MLOps file 09's data-drift
discussion directly) -- a genuinely robust pipeline design ANTICIPATES
this, rather than assuming the schema/distribution observed today will
remain fixed forever
```

## ETL vs ELT — genuinely worth knowing this distinction

```
ETL (Extract, Transform, Load): transform data BEFORE loading into the
  final storage/warehouse -- genuinely traditional, ensures only clean
  data reaches the warehouse
ELT (Extract, Load, Transform): load RAW data first, transform LATER,
  often directly within the warehouse (recap Cloud file 07's BigQuery
  SQL-based transformation capability directly) -- genuinely more common
  now that warehouses like BigQuery can handle heavy transformation
  workloads efficiently themselves
```

## Feature computation timing — genuinely important, connects directly to file 04

```
Genuinely critical distinction, previewing file 04's full treatment:
OFFLINE/BATCH features (computed periodically, e.g. "average purchase
  amount over the last 30 days," recap MLOps file 07's scheduled-job
  discussion) vs ONLINE/REAL-TIME features (computed at request time,
  e.g. "is this the user's first transaction today")
```

## A complete, realistic data pipeline diagram — pulling this file together

```
Raw Events (app/website) -> Streaming ingestion (Kafka-style, for
  real-time needs) OR Batch ETL (scheduled, recap MLOps file 07) ->
  Data Lake / Warehouse (recap Cloud files 04/07) -> Validation (recap
  MLOps file 08) -> Feature computation (file 04) -> Feature Store
  (file 04) -> Model training (file 05) / Model serving (file 06)
```

Genuinely worth being able to draw and narrate this exact diagram fluently — recap file 01's whiteboarding-practice discussion directly — this is genuinely the realistic backbone diagram underlying nearly every case study covered in files 07-11 of this folder.

## Try this

```
# Genuinely worth practicing this out loud (recap file 01's practice
# discipline directly): for a recommendation system (preview of file 07),
# sketch the FULL data pipeline from raw user click events to a trained
# model, explicitly deciding: batch or streaming ingestion? What storage
# for raw events vs processed features? What validation checks would you
# add, and where in the pipeline would they run?
```

Explicitly justifying EACH storage/processing choice (not just naming a tool) is genuinely the real skill being practiced — recap the recurring tradeoff-communication theme from this entire roadmap's bridge files directly.

## What's next
File 04 — Feature Engineering & Feature Stores at Scale, covering how raw data gets transformed into the actual SIGNALS a model consumes, and the infrastructure that serves those signals reliably and consistently.
