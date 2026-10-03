# Top Data Engineering Questions & Answers (2026)

A curated list of questions commonly asked in data engineering interviews today, grouped by topic.

---

## 1. Fundamentals & Architecture

### Q1. ETL vs ELT: what's the difference and when would you pick each?
**ETL** transforms data *before* loading it into the target; **ELT** loads raw data first and transforms inside the warehouse/lakehouse using its compute.
- Choose **ELT** with cheap storage and scalable engines (Redshift, Snowflake, BigQuery, Spark on a lakehouse), when you want to keep raw data for replay and reprocessing.
- Choose **ETL** when data must be masked, filtered or validated before landing (compliance), or the target has limited compute.

### Q2. What is the Medallion architecture?
A layered design: **Bronze/Raw** (as-ingested, immutable) → **Silver/Standardized** (cleaned, deduplicated, typed, conformed) → **Gold/Curated** (business-level aggregates, marts). It gives clear lineage, easier reprocessing, and separates quality concerns per layer.

### Q3. Data warehouse vs data lake vs lakehouse?
- **Warehouse**: structured, schema-on-write, optimized for SQL/BI.
- **Lake**: any data type on object storage, schema-on-read, cheap but historically weak on ACID/governance.
- **Lakehouse**: lake storage + table formats (Delta, Iceberg, Hudi) adding ACID transactions, schema evolution, time travel and upserts.

### Q4. Batch vs streaming vs micro-batch?
- **Batch**: periodic, high throughput, simpler.
- **Streaming**: record-by-record, low latency (Kafka, Flink, Kinesis).
- **Micro-batch**: small batches at short intervals (Spark Structured Streaming). Pick based on latency SLA, cost and complexity, not hype.

### Q5. What is Lambda vs Kappa architecture?
**Lambda** runs separate batch and speed layers and merges results (duplicate logic). **Kappa** uses a single streaming pipeline and replays the log for reprocessing. Kappa is simpler; Lambda is used when batch recomputation has different guarantees or tooling.

---

## 2. SQL & Data Modeling

### Q6. Explain star schema vs snowflake schema.
**Star**: a central fact table joined to denormalized dimensions, which means fewer joins and faster queries. **Snowflake**: dimensions are normalized into sub-dimensions, which saves storage and reduces redundancy but adds joins. Star is the default for analytics.

### Q7. What are Slowly Changing Dimensions (SCD)?
- **Type 1**: overwrite, no history.
- **Type 2**: add a new row with `effective_from`, `effective_to`, `is_current`; full history.
- **Type 3**: add a column for the previous value; limited history.

### Q8. Find the 2nd highest salary per department.
```sql
SELECT department_id, employee_id, salary
FROM (
  SELECT *,
         DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 2;
```
Use `DENSE_RANK` to handle ties properly (`ROW_NUMBER` would break them arbitrarily).

### Q9. How do you remove duplicates in SQL?
```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY updated_at DESC) AS rn
  FROM source_table
) WHERE rn = 1;
```

### Q10. Difference between `RANK`, `DENSE_RANK`, and `ROW_NUMBER`?
`ROW_NUMBER` is always unique; `RANK` leaves gaps after ties (1,1,3); `DENSE_RANK` has no gaps (1,1,2).

### Q11. What is the difference between a CTE and a temp table?
A CTE is a named query scoped to one statement (may be inlined or re-evaluated). A temp table is materialized, reusable across statements in a session, and can be indexed/analyzed. Use temp tables when a result is reused many times or is expensive.

### Q12. How do you optimize a slow SQL query?
Read the execution plan; filter early and select only needed columns; avoid functions on indexed/partition columns; use proper join order and types; partition/cluster/sort-key tables sensibly; pre-aggregate; avoid `SELECT *` and cross joins; update statistics.

---

## 3. Apache Spark & Big Data

### Q13. Explain narrow vs wide transformations.
**Narrow** (map, filter): each output partition depends on one input partition, with no shuffle. **Wide** (groupBy, join, distinct): data is redistributed across partitions, causing a **shuffle**, the most expensive operation in Spark.

### Q14. How do you handle data skew in Spark?
- Salting hot keys
- Broadcast joins for small tables
- Adaptive Query Execution (AQE) skew-join handling
- Repartitioning on a better key
- Filtering or isolating the skewed keys and processing them separately

### Q15. `repartition` vs `coalesce`?
`repartition(n)` does a full shuffle and can increase or decrease partitions with even distribution. `coalesce(n)` only reduces partitions and avoids a full shuffle, but may produce uneven sizes.

### Q16. What is a broadcast join and when is it used?
The small table is sent to every executor so the large table is joined locally without a shuffle. Use when one side fits comfortably in executor memory (`spark.sql.autoBroadcastJoinThreshold`, or an explicit `broadcast()` hint).

### Q17. How do you tune a Spark job?
Right-size executors/cores/memory; tune `spark.sql.shuffle.partitions` or enable AQE; use columnar formats (Parquet) with predicate/column pruning; cache only what is reused; avoid UDFs where built-ins work; fix small files and skew; use broadcast joins.

### Q18. What is the small files problem and how do you fix it?
Many tiny files cause metadata overhead and slow reads/listing on S3/HDFS. Fix with compaction jobs, `coalesce` before write, table-format maintenance (Hudi/Delta/Iceberg compaction/OPTIMIZE), and tuning writer parallelism.

### Q19. Parquet vs Avro vs ORC vs JSON/CSV?
- **Parquet/ORC**: columnar, compressed, great for analytics.
- **Avro**: row-based, strong schema evolution, ideal for streaming/Kafka.
- **JSON/CSV**: human-readable but inefficient and weakly typed.

---

## 4. Lakehouse & Table Formats

### Q20. Delta vs Iceberg vs Hudi: how do they differ?
All three add ACID, schema evolution and time travel on object storage.
- **Delta Lake**: tight Spark/Databricks integration, transaction log.
- **Iceberg**: engine-agnostic, hidden partitioning, strong metadata/snapshot model.
- **Hudi**: designed for upserts/incremental processing and CDC, with Copy-on-Write and Merge-on-Read table types.

### Q21. Copy-on-Write vs Merge-on-Read (Hudi)?
**CoW** rewrites whole files on update: slower writes, faster reads. **MoR** writes deltas to log files and merges at read/compaction: faster writes, slightly slower reads. Use MoR for write-heavy/near-real-time, CoW for read-heavy workloads.

### Q22. How do you implement SCD Type 1 with an upsert on a lakehouse?
Use the table format's merge/upsert (e.g., Hudi `upsert` with a record key and precombine field, or `MERGE INTO` in Delta/Iceberg) so the latest record overwrites the previous one per business key.

### Q23. What is time travel and why does it matter?
Querying a table as of a previous snapshot/version/timestamp. Useful for audits, rollback of bad loads, reproducing reports and debugging.

---

## 5. Orchestration & Pipeline Design

### Q24. How does Apache Airflow work? Key concepts?
DAGs define workflows as tasks with dependencies. Components: scheduler, executor, workers, metadata DB, webserver. Key ideas: operators, sensors, hooks, XComs, task groups, pools, retries, SLAs, and the logical (`data_interval`) date.

### Q25. How do you make pipelines idempotent?
Design so reruns produce the same result: overwrite partitions instead of appending, use upserts/merge on business keys, deterministic outputs based on the data interval (not `now()`), write to staging then atomically swap, and track state with watermarks/checkpoints.

### Q26. How do you handle backfills in Airflow?
Parameterize tasks by `data_interval_start/end`, make tasks idempotent, use `catchup`/`airflow dags backfill` with controlled `max_active_runs`, and avoid overloading sources by limiting concurrency with pools.

### Q27. What is CDC (Change Data Capture) and how do you implement it?
Capturing inserts/updates/deletes from a source as events. Approaches: **log-based** (Debezium, AWS DMS reading DB transaction logs; lowest impact), **trigger-based**, **timestamp/version column polling**, or **snapshot diffing**. Land changes in a raw layer, then apply them downstream with upserts and handle deletes (soft/hard).

### Q28. How do you handle late-arriving and out-of-order data?
Use event time (not processing time), watermarks with allowed lateness in streaming, reprocess affected partitions in batch, and upsert by key with a precombine/version field so the latest event wins.

### Q29. How do you handle schema evolution?
Use schema-aware formats (Avro/Parquet) and a schema registry or catalog; allow additive, backward-compatible changes; validate on ingest; quarantine incompatible records; version contracts with producers; and rely on table-format schema evolution.

---

## 6. Cloud (AWS-Focused)

### Q30. When would you use AWS Glue vs EMR vs Lambda?
- **Glue**: serverless Spark ETL, catalog, crawlers; low ops overhead.
- **EMR**: more control and tuning for large/long-running Spark/Hadoop jobs, often cheaper at scale.
- **Lambda**: small, event-driven, short tasks (under 15 minutes); not suited to heavy transforms.

### Q31. Redshift vs Athena vs Redshift Spectrum?
- **Redshift**: provisioned/serverless MPP warehouse for heavy, repeated BI workloads.
- **Athena**: serverless, pay-per-data-scanned SQL on S3 for ad hoc queries.
- **Spectrum**: lets Redshift query S3 external tables, combining warehouse and lake data.

### Q32. How do you optimize Redshift performance?
Choose good distribution keys (to colocate joins) and sort keys; use appropriate compression/encodings; `VACUUM`/`ANALYZE` (or rely on auto); avoid small frequent inserts (use `COPY` from S3); use materialized views; manage WLM / concurrency scaling.

### Q33. How do you reduce Athena cost and improve speed?
Store data as partitioned Parquet/ORC, compress, compact small files, query only needed columns and partitions, use partition projection, and avoid `SELECT *`.

### Q34. SNS vs SQS vs Kinesis vs Kafka?
- **SNS**: pub/sub fan-out notifications.
- **SQS**: durable queue for decoupling, with at-least-once delivery (FIFO for ordering).
- **Kinesis / Kafka**: ordered, replayable streams for high-throughput real-time pipelines (Kafka is open source with a richer ecosystem; Kinesis is managed).

### Q35. How would you secure a data platform on AWS?
IAM least privilege and roles (no long-lived keys), KMS encryption at rest, TLS in transit, VPC endpoints/private networking, Lake Formation for fine-grained access, S3 bucket policies and block public access, CloudTrail auditing, and column/row-level masking for PII.

---

## 7. Streaming

### Q36. Exactly-once vs at-least-once vs at-most-once?
At-most-once may lose data; at-least-once may duplicate; exactly-once requires idempotent writes plus transactional/checkpointed processing (e.g., Kafka transactions, Flink checkpoints). In practice, at-least-once delivery plus idempotent sinks gives effectively-once results.

### Q37. Explain Kafka partitions, consumer groups, and offsets.
A topic is split into partitions (unit of parallelism and ordering). Within a consumer group, each partition is read by one consumer. Offsets track consumer position and are committed to enable resume/replay.

### Q38. What are windowing types in stream processing?
**Tumbling** (fixed, non-overlapping), **sliding** (fixed, overlapping), **session** (gap-based on inactivity). Combine with watermarks to handle late data.

---

## 8. Data Quality, Governance & Observability

### Q39. How do you ensure data quality in pipelines?
Validate at each layer: schema checks, null/uniqueness/range/referential checks, row-count and freshness reconciliation, anomaly detection. Tools: Great Expectations, Deequ, dbt tests, Soda. Fail fast or quarantine bad records, and alert with clear ownership.

### Q40. What is data lineage and why does it matter?
Tracking where data came from and how it was transformed. It supports impact analysis, debugging, compliance and trust (e.g., OpenLineage, DataHub, AWS Glue/Lake Formation lineage).

### Q41. What are data contracts?
Formal agreements between producers and consumers on schema, semantics, SLAs and quality. They prevent breaking changes and shift data quality left to the source.

### Q42. How do you monitor pipelines?
Track job success/failure, runtime, data freshness, volume anomalies and cost; centralize logs/metrics (CloudWatch, Datadog, Prometheus/Grafana); set SLA alerts and runbooks; add data observability tools for silent failures.

---

## 9. DevOps / DataOps

### Q43. How do you do CI/CD for data pipelines?
Version control everything; run linting and unit tests on transforms; integration tests on sample data; deploy via GitHub Actions or similar to dev, then staging, then prod with approvals; use Terraform for infrastructure; promote immutable artifacts; support rollback.

### Q44. Why use Infrastructure as Code (Terraform) for data platforms?
Reproducible, reviewable, version-controlled environments; consistent dev/stage/prod; drift detection; safer changes via plan/apply; reusable modules.

### Q45. How do you control cloud data costs (FinOps)?
Right-size and auto-scale compute, use spot capacity where safe, lifecycle policies and storage tiering, partition and compact data, prune scans, turn off idle resources, tag resources and set budgets/alerts.

---

## 10. Modern Trends (Frequently Asked Now)

### Q46. What role does dbt play in modern stacks?
dbt handles the **T** in ELT: SQL-based, version-controlled transformations with testing, documentation and lineage, run inside the warehouse/lakehouse.

### Q47. How is AI/LLM changing data engineering?
Pipelines now feed ML/LLM workloads: embeddings and vector stores for RAG, unstructured data ingestion (documents, text), feature stores, and AI-assisted code/SQL generation. Core skills (quality, governance, scalable pipelines) matter more, not less.

### Q48. What is a feature store?
A central system to define, compute, store and serve ML features consistently for both training (offline) and inference (online), avoiding training/serving skew.

### Q49. What is the Modern Data Stack / Data Mesh?
**Modern Data Stack**: cloud-native, modular tools (ingestion, warehouse/lakehouse, dbt, orchestration, BI). **Data Mesh**: decentralized, domain-owned data products with federated governance and self-serve platform.

### Q50. How would you design a pipeline from scratch? (System design)
A solid answer covers: requirements (volume, latency, SLAs) → sources and ingestion (CDC/batch/stream) → storage layers (raw/standardized/curated) → processing engine → orchestration → data quality → serving layer → security/governance → monitoring and alerting → cost and scalability → failure handling and backfill strategy.

---

## Bonus: Behavioral / Scenario Questions

- Describe a time a pipeline failed in production. How did you detect, fix and prevent it?
- How do you handle conflicting requirements from stakeholders?
- Tell me about the biggest performance improvement you delivered.
- How do you decide between building and buying a tool?
- How do you document and hand off pipelines to other teams?

---

*Tip: for each answer, prepare a concrete example from your own projects: scale, tools, problem, your decision, and the measurable outcome.*
