# CDC Lakehouse Data Engineering Knowledge Base & Mother Document

> **Project Reference**: AWS CDC Lakehouse Pipeline for Enterprise Retail & E-Commerce  
> **Core Stack**: AWS Glue (PySpark), Apache Hudi, Amazon S3, Amazon Redshift Spectrum, Amazon Athena, Managed Workflows for Apache Airflow (MWAA), Terraform, GitHub Actions  
> **Purpose**: Single source of truth (Mother Document) consolidating end-to-end architecture, technical deep dives, performance tuning, failure handling, and scenario-based interview Q&A.

---

## 📑 Table of Contents

1. [Professional Profile & Project Overview](#1-professional-profile--project-overview)
2. [End-to-End Lakehouse Architecture & Data Flow](#2-end-to-end-lakehouse-architecture--data-flow)
3. [CDC Processing, Apache Hudi & Lakehouse Mechanics](#3-cdc-processing-apache-hudi--lakehouse-mechanics)
4. [Distributed Compute Engine: AWS Glue & PySpark](#4-distributed-compute-engine-aws-glue--pyspark)
5. [Analytical Serving Layer: Redshift Spectrum & Athena](#5-analytical-serving-layer-redshift-spectrum--athena)
6. [Airflow Orchestration, Concurrency & Scheduling](#6-airflow-orchestration-concurrency--scheduling)
7. [Data Quality, Schema Evolution & Governance](#7-data-quality-schema-evolution--governance)
8. [Performance Tuning, Partitioning & Scaling](#8-performance-tuning-partitioning--scaling)
9. [Production Incident Response, Debugging & Resiliency](#9-production-incident-response-debugging--resiliency)
10. [Open Project-Specific Action Items](#10-open-project-specific-action-items)

---

## 1. Professional Profile & Project Overview

### Q: Please introduce yourself.
**A:**  
"Hi, I'm Arunava. I've been with Zensar Technologies for about four years now. I started out in Data Engineering early on, building a strong, hands-on foundation in the AWS data ecosystem—working heavily with services like AWS Glue, Lambda, S3, Redshift, Athena, and SQS, using Managed Airflow (MWAA) to orchestrate complex pipelines. Alongside data engineering, I am deeply involved in DevOps and Infrastructure as Code (IaC), writing Terraform to provision cloud infrastructure and setting up automated CI/CD pipelines with GitHub Actions. What I enjoy most is that intersection—not just writing Spark transformation logic, but owning how data systems are deployed, monitored, secured, and maintained end-to-end in production."

---

### Q: Can you summarize the AWS services and tools you have used over time?
**A:**  
* **Storage & Data Lake**: Amazon S3 (Landing, Raw, Standardized, Curated tiers), Apache Hudi (ACID storage format, upserts/deletes).
* **Compute & Processing**: AWS Glue (PySpark ETL jobs, DynamicFrames, Glue Crawlers, Glue Data Catalog), AWS Lambda (event-driven micro-batching, validation triggers), Amazon EC2.
* **Data Warehousing & Querying**: Amazon Redshift (MPP analytical warehouse), Amazon Redshift Spectrum (external querying over S3 data lake), Amazon Athena (serverless ad-hoc Presto querying for data audits).
* **Orchestration & Event Messaging**: Managed Workflows for Apache Airflow (MWAA), Amazon EventBridge, Amazon SNS (alerts), Amazon SQS / Dead Letter Queues (error routing).
* **Security & Governance**: AWS IAM (least-privilege roles, service policies), AWS KMS (SSE-KMS encryption), AWS Lake Formation (column/table-level access controls), CloudTrail, CloudWatch.
* **DevOps & CI/CD**: Terraform (modular IaC), GitHub & GitHub Actions (automated linting, testing, deployment).

---

### Q: What was your individual contribution to the CDC Pipeline project?
**A:**  
"My role was focused on execution and end-to-end pipeline delivery:
1. **Ingestion & Validation**: Built validation scripts in AWS Glue to verify file arrival, batch integrity, and structural schema conformance in the Landing zone.
2. **Standardization & CDC Engine**: Implemented PySpark jobs using Apache Hudi to process upstream Change Data Capture (CDC) feeds (handling SCD Type-1 Upserts and hard Deletes via `EmptyHoodieRecordPayload`).
3. **Data Cataloging & Query Optimization**: Synchronized table metadata with AWS Glue Data Catalog and created external Redshift Spectrum tables with Materialized Views in Redshift for downstream consumers.
4. **Airflow DAG Orchestration**: Designed MWAA DAGs enforcing task dependencies, sensor checks, retries, and concurrency limits across 15–20 simultaneous pipelines.
5. **DevOps & IaC**: Provisioned AWS resources using Terraform and automated deployment workflows through GitHub Actions."

---

### Q: How many team members were involved and how was work divided?
**A:**  
"We had a 14-person engineering team:
* **Leadership**: 1 Engineering Manager and 1 Technical Lead.
* **Architecture**: 3 Data Architects who defined the enterprise data models, security frameworks, and high-level ingestion strategy.
* **Engineering Squad**: 9 Data Engineers (a mix of senior and mid-level engineers, including myself).  
We divided pipeline ownership by business domain. I owned the end-to-end pipelines for the core **Inventory and Items** domains."

---

### Q: How many pipelines and tables did you manage?
**A:**  
"The overall project ingested data from over 17 upstream interfaces encompassing roughly 57 core tables. Within our domain breakdown, I actively owned and maintained the pipelines for high-volume retail tables:
* `Item_master` (core product attributes, hierarchy, SKU metadata)
* `Item_location` (store/warehouse item mappings, localized pricing)
* `promotion_items` (active discount campaigns, marketing flags)
* `inventory_on_hand` (real-time stock levels, warehouse availability)  
These inventory and promotional tables generated the highest volume of CDC traffic and required strict state management and deduplication."

---

### Q: What was your pipeline success rate?
**A:**  
"Our pipeline success rate was comfortably above 95%. The architecture was resilient because we designed all jobs for idempotency. Since we used Apache Hudi for upserts and partition-level overwrites, any failure caused by temporary network throttling or malformed upstream payloads could be safely retried or backfilled without generating duplicate records."

---

## 2. End-to-End Lakehouse Architecture & Data Flow

### Q: Explain your data flow end-to-end.
**A:**  
"The project serves as the centralized data platform for retail operations, enabling Data Science, Marketing, Purchasing, and Warehouse Operations to project future sales and optimize stock replenishment. Data flows through a 4-tier Lakehouse architecture on AWS:

```
[Upstream Systems: RMS, FPP, CAM, APEX, AMOS]
                     │
                     ▼ (Daily File Drop / CDC Feeds via AIT Integration Team)
        ┌─────────────────────────┐
        │   1. S3 Landing Layer   │ ──► AWS Glue Validation (Batch ID, structural checks)
        └────────────┬────────────┘     Queryable via Athena for audit
                     │
                     ▼ (PySpark Glue ETL)
        ┌─────────────────────────┐
        │     2. S3 Raw Layer     │ ──► Append audit columns (ingest_dt, primary_keys)
        └────────────┬────────────┘     Convert format, retain raw payload
                     │
                     ▼ (PySpark + Apache Hudi Job)
        ┌─────────────────────────┐
        │  3. S3 Standardized     │ ──► SCD Type-1 Upserts (op != 'd') & Deletes (op == 'd')
        │       Layer (Hudi)      │     Latest event resolution via update_ts_dms
        └────────────┬────────────┘     Glue Data Catalog metadata sync
                     │
                     ▼ (Redshift Spectrum External Queries)
        ┌─────────────────────────┐
        │   4. Curated Layer      │ ──► Materialized Views, Business Aggregations
        │      (Redshift)         │ ──► Consumed by Data Science & PowerBI/QuickSight
        └─────────────────────────┘
```

1. **Landing Layer**: Upstream feeds (flat files and database change feeds from systems like RMS, CAM, FPP) are delivered into S3 by a dedicated integration team (AIT). AWS Glue executes structural validation (file arrival checks, batch IDs, row delimiter integrity). Schema is registered in the Glue Data Catalog for ad-hoc audit queries via Amazon Athena.
2. **Raw Layer**: A PySpark Glue job reads validated files from Landing, retains the raw payload, appends technical audit columns (e.g., `ingest_dt`, generated composite `primary_key`), converts the data into an optimized format, and writes it to the Raw S3 bucket.
3. **Standardized Layer (Hudi Lakehouse)**: AWS Glue uses Apache Hudi to perform SCD Type-1 upserts and deletes directly on S3. It evaluates CDC operation flags (`op`), resolves out-of-order changes using `update_ts_dms`, removes obsolete `before` columns, writes standardized columnar data, and updates Glue Catalog partition metadata.
4. **Curated Layer (Serving Layer)**: Amazon Redshift Spectrum references the standardized Hudi tables directly in S3. Business transformations and aggregations are materialized into Redshift tables and Materialized Views for downstream analytics by Data Science and Operations."

---

### Q: What is the difference between Bronze, Silver, Gold layers and how do they map to your 4 layers?
**A:**  
* **Landing / Raw (Bronze Layer)**: Append-only raw data ingestion. Captures the raw source payload as-is, preserving historical record arrival with basic metadata (`ingest_dt`, source file name).
* **Standardized Layer (Silver Layer)**: Cleaned, deduplicated, and conformant data. Schema is enforced, types are cast, CDC events are reconciled (upserts/deletes), and records represent the current cleaned state.
* **Curated Layer (Gold Layer)**: Business-level aggregated data models, star schemas, dimensional tables, and Materialized Views optimized for BI reporting, ML models, and executive dashboards.

---

### Q: What was the frequency of data arrival and how did you orchestrate processing?
**A:**  
"The upstream integration team (AIT) dropped raw data files and CDC feeds into our Landing S3 bucket on a daily schedule. However, processing an entire day's drop in one monolithic batch risked exhausting cluster memory and bottlenecking resources.  
To optimize compute utilization, we configured Airflow DAGs to partition and process the daily drop in **smaller, hourly chunks (7 to 10 GB per run)**. This distributed the processing load evenly across the day and ensured SLA compliance."

---

### Q: What data volumes did the pipeline handle?
**A:**  
"During peak batch runs, the pipeline ingested and processed between **7 and 10 GBs of data per hour**. Across 17 upstream interfaces and 57 tables, this involved millions of CDC mutations that required rapid primary-key lookups, deduplication, and file compaction in Hudi within each 1-hour window."

---

## 3. CDC Processing, Apache Hudi & Lakehouse Mechanics

### Q: Explain your CDC payload structure and how you process inserts, updates, and deletes in Apache Hudi.
**A:**  
"Our CDC payload generated by the upstream change stream (e.g., AWS DMS / Debezium) contains four key elements:
1. `before`: Snapshot of the row before the mutation (used for audit).
2. `after`: Snapshot of the row after the mutation.
3. `op`: Operation flag (`'i'` for Insert, `'u'` for Update, `'d'` for Delete).
4. `update_ts_dms`: Upstream source transaction timestamp.

**Processing Logic inside the AWS Glue Hudi Job:**
1. **Schema Standardization**: We drop the `before` struct and flatten/rename the `after` fields to match our standardized business data schema.
2. **Inserts and Updates (`op != 'd'`)**: We filter for all non-delete records and route them to Hudi's `upsert` write operation. Hudi matches incoming records against existing S3 data using `hoodie.datasource.write.recordkey.field`. When duplicate keys exist, Hudi compares `hoodie.datasource.write.precombine.field` (configured to `update_ts_dms`) to ensure only the latest state is written.
3. **Hard Deletes (`op == 'd'`)**: We filter for records where `op == 'd'`, set the Hudi payload class to `org.apache.hudi.common.model.EmptyHoodieRecordPayload`, and execute the write. Hudi purges the matching primary keys from the Parquet data files during the commit."

```python
# Sample Hudi Write Configuration in AWS Glue PySpark
hudi_options = {
    'hoodie.table.name': 'item_master_standardized',
    'hoodie.datasource.write.recordkey.field': 'item_id',
    'hoodie.datasource.write.precombine.field': 'update_ts_dms',
    'hoodie.datasource.write.partitionpath.field': 'region',
    'hoodie.datasource.write.table.type': 'COPY_ON_WRITE',
    'hoodie.datasource.write.operation': 'upsert',
    'hoodie.datasource.hive_sync.enable': 'true',
    'hoodie.datasource.hive_sync.database': 'standardized_db',
    'hoodie.datasource.hive_sync.table': 'item_master',
    'hoodie.datasource.hive_sync.use_jdbc': 'false',
    'hoodie.datasource.hive_sync.mode': 'hms'
}

# Delete payload configuration for hard deletes
hudi_delete_options = {
    **hudi_options,
    'hoodie.datasource.write.payload.class': 'org.apache.hudi.common.model.EmptyHoodieRecordPayload'
}
```

---

### Q: How does the MERGE / Upsert operation work under the hood in a Lakehouse?
**A:**  
"Under the hood, a Lakehouse format (Hudi / Delta Lake / Iceberg) executes a `MERGE` via a two-phase process:
1. **Join & Index Lookup**: The engine joins incoming candidate keys against the target table's index / metadata to identify which specific Parquet data files contain existing records that match the join condition (`ON target.id = source.id`).
2. **Rewrite / Delta Log Commit**:
   * *Copy-on-Write (CoW)*: For every file containing updated or deleted rows, Hudi reads the existing unchanged rows, merges them in memory with the new updates, and writes out a brand-new Parquet file version. The table metadata log is updated atomically to point to the new files.
   * *Merge-on-Read (MoR)*: Inserts/updates are written to columnar log files (Avro/delta logs) without immediately rewriting base Parquet files. Background compaction asynchronously merges base and log files."

---

### Q: What benefits did you get by choosing Apache Hudi / Delta Lake over plain Parquet?
**A:**  
| Capability | Plain Parquet on S3 | Apache Hudi / Lakehouse |
| :--- | :--- | :--- |
| **Record Updates (SCD-1)** | Requires full partition or table overwrite | Point updates via file-level Upsert index |
| **Hard Deletes (GDPR/CDC)** | Expensive full-table rewrite | Native delete payload (`EmptyHoodieRecordPayload`) |
| **ACID Guarantees** | None (readers see partial writes) | Snapshot isolation and atomic commits via timeline log |
| **Concurrency** | Write-write collisions cause corruption | Optimistic Concurrency Control (OCC) |
| **Time Travel & Rollback** | Requires manual snapshot directories | Query historical commits via timestamp/commit instant |
| **Catalog Sync** | Requires manual `MSCK REPAIR` or crawler | Automated instant sync to AWS Glue Data Catalog |

---

### Q: How do you handle late-arriving CDC data?
**A:**  
"My approach follows one fundamental principle: **Never confuse Event Time with Processing Time.**
1. **Event Time Partitioning**: We extract the source transaction timestamp (`update_ts_dms`) and use that to dictate logical partition paths (e.g., `year/month/day`), rather than using the pipeline execution timestamp (`current_date()`).
2. **Dynamic Partition Routing**: PySpark is configured with `spark.sql.sources.partitionOverwriteMode=dynamic`. If a batch running on Thursday contains delayed CDC events from Tuesday, Spark dynamically routes those records to the Tuesday S3 partition without overwriting or destroying other Tuesday records.
3. **Idempotent Hudi Upsert**: Hudi uses `update_ts_dms` as the precombine key. If an older record arrives after a newer record has already been committed, Hudi recognizes that `incoming.update_ts_dms < existing.update_ts_dms` and safely ignores the stale update."

---

### Q: How do you manage Schema Evolution in your CDC pipeline?
**A:**  
"We manage schema evolution across four defensive layers:
1. **Early Detection & Gatekeeper**: We validate incoming payloads against the AWS Glue Schema Registry baseline. If a breaking change occurs (e.g., a data type changes from integer to string or a primary key column is dropped), the record is quarantined to an S3 Dead Letter Queue (DLQ) and an alert is triggered.
2. **Handling Safe Additive Changes**: When an upstream system adds a new optional column, Spark is configured with `mergeSchema=true`. Hudi automatically updates the Glue Catalog table schema. Downstream queries reading historical data simply return `NULL` for the new column.
3. **Handling Upstream Column Drops**: If an upstream table drops a column, we maintain the column in the Lakehouse schema and populate it with `NULL` for incoming rows. This prevents downstream BI queries and ML features from failing while giving teams time to refactor.
4. **Lakehouse Metadata Evolution**: Because Apache Hudi maintains schema definitions in its metadata transaction logs, schema modifications do not require rewriting petabytes of historical Parquet files."

---

## 4. Distributed Compute Engine: AWS Glue & PySpark

### Q: What is AWS Glue and how does it differ from Amazon EMR?
**A:**  
* **AWS Glue**: Serverless, managed Apache Spark environment. AWS provisions, tunes, and terminates compute nodes automatically. You pay per Data Processing Unit (DPU) per second. Best for automated, scheduled batch ETL pipelines and metadata management.
* **Amazon EMR**: Managed cluster platform (EC2-based or EKS-based) where you have full control over Hadoop, Spark, Flink, and Hive configurations, master/core/task node instance types, and spot pricing. Best for long-running big data clusters, highly customized Spark configurations, or petabyte-scale persistent processing.

---

### Q: What is the difference between AWS Glue DynamicFrame and Spark DataFrame?
**A:**  
* **DynamicFrame**: A Glue-native abstraction designed to handle messy, semi-structured, and inconsistent data. It does not enforce a rigid schema up front; each record contains its own schema. It supports native Glue transforms (`ResolveChoice`, `Unbox`, `Relationalize`).
* **DataFrame**: Standard Apache Spark distributed dataset with a strictly defined, uniform schema. It benefits from the Spark Catalyst optimizer and Tungsten execution engine.  
* *Best Practice*: Use DynamicFrames during Landing/Raw ingestion for schema flexibility, then immediately convert to DataFrames (`dynamic_frame.toDF()`) for heavy joins, aggregations, and Hudi writes.

---

### Q: What AWS Glue worker types did you use?
**A:**  
* **Standard**: 1 DPU (4 vCPUs, 16 GB memory) with 50 GB disk. (Legacy)
* **G.1X**: 1 DPU (4 vCPUs, 16 GB memory) with 64 GB NVMe SSD. Ideal for memory-intensive batch jobs with moderate data volumes.
* **G.2X**: 2 DPUs (8 vCPUs, 32 GB memory) with 128 GB NVMe SSD. Ideal for heavy transforms, large-scale shuffles, and Apache Hudi upsert indexing over large datasets.  
* *Project Configuration*: We used **G.2X workers** for our Standardized Hudi upsert jobs to handle the in-memory primary-key indexing overhead, and **G.1X workers** for lightweight Landing-to-Raw validation jobs. _[Open — confirm exact DPU allocation per job for your specific cluster]_.

---

### Q: How do you troubleshoot and fix Out-Of-Memory (OOM) errors in Spark / AWS Glue?
**A:**  
"Spark OOM errors are diagnosed by distinguishing between **Driver OOM** and **Executor OOM**:

1. **Driver OOM (`java.lang.OutOfMemoryError: Java heap space`)**:
   * *Root Causes*: Calling `.collect()` or `.toPandas()` on large datasets, broadcasting a table that exceeds driver memory, or excessive schema inference.
   * *Fixes*: Replace `.collect()` with `.take(n)` or write directly to S3; increase `spark.driver.memory`; increase `spark.driver.maxResultSize`.
2. **Executor OOM / Container Killed by YARN**:
   * *Root Causes*: Severe data skew causing one partition to swell; insufficient executor memory during wide joins/aggregations; high memory consumption during Spark shuffle operations.
   * *Fixes*:
     * Optimize memory split: increase `spark.executor.memory` and `spark.memory.fraction` (default 0.6).
     * Increase shuffle partitions: set `spark.sql.shuffle.partitions` (e.g., from default 200 to 1000+ depending on data size).
     * Use Kryo serialization: set `spark.serializer=org.apache.spark.serializer.KryoSerializer`.
     * Clean up cache: call `df.unpersist()` after cached DataFrames are no longer required."

---

### Q: What is Data Skew and how did you handle it in your project?
**A:**  
"Data Skew occurs when data is unevenly distributed across partitions based on a join or grouping key (e.g., a few popular promotional items or central warehouse location IDs represent 80% of all transaction records). In Spark, 99 tasks finish in seconds, while 1 task processing the skewed key runs for hours or crashes with an OOM.

**Strategies to Mitigate Data Skew:**
1. **Salting the Skewed Key**: Add a random integer prefix (e.g., `concat(key, '_', floor(rand() * 10))`) to the skewed key in the large table, replicate the smaller table 10 times with corresponding salt keys, join on the salted key, and aggregate back.
2. **Broadcast Joins**: For joining large transaction tables with small reference tables (under 100MB), force a broadcast join (`broadcast(small_df)`). This eliminates shuffle stages entirely.
3. **Adaptive Query Execution (AQE)**: Enable `spark.sql.adaptive.enabled=true` and `spark.sql.adaptive.skewJoin.enabled=true`. Spark automatically detects skewed partitions at runtime and splits them into smaller sub-partitions."

---

### Q: How do you optimize a slow AWS Glue / Spark job?
**A:**  
1. **Predicate Pushdown & Partition Pruning**: Push filters directly to the S3 reader (`push_down_predicate` in Glue DynamicFrames or `.filter()` before transformations) so only relevant folders are scanned.
2. **Avoid Wide Shuffles**: Replace `groupByKey()` with `reduceByKey()` or `aggregateByKey()` to leverage map-side combining before data is shuffled across the network.
3. **Convert DynamicFrame to DataFrame**: Take advantage of Catalyst optimizer optimizations for relational queries.
4. **Tune Parallelism**: Align `spark.sql.shuffle.partitions` with available executor cores ($2\times \text{to } 3\times \text{total cores}$).
5. **Enable Glue Job Bookmarks**: Prevent reprocessing unchanged data across incremental runs.
6. **JVM Garbage Collection Tuning**: Use `-XX:+UseG1GC` to avoid long GC pause freezes on large executor heaps."

---

### Q: How do you inspect and use the Spark UI to debug performance?
**A:**  
* **Jobs & Stages Tab**: Check the DAG visualization to identify which stage took the most time and look for unexpected shuffle boundaries.
* **Tasks Summary (Event Timeline)**: Compare the Min, Median, 75th percentile, and Max task durations. A huge gap between Median and Max indicates **Data Skew**.
* **Storage Tab**: Check the fraction of cached DataFrames stored in memory vs spilled to disk (`Disk Spill` indicates memory pressure).
* **Executors Tab**: Identify task failures, GC time percentage, and memory usage per executor node."

---

## 5. Analytical Serving Layer: Redshift Spectrum & Athena

### Q: What is Amazon Redshift Spectrum and when do you use it?
**A:**  
"Amazon Redshift Spectrum is a feature of Amazon Redshift that allows you to run standard SQL queries directly against exabytes of structured and semi-structured data stored in Amazon S3, without loading the data into Redshift local storage.

**When to Use Spectrum:**
* Querying massive historical/cold data in S3 while keeping only hot, recent data (e.g., last 30–90 days) in local Redshift cluster storage.
* Eliminating ETL data-loading wait times for large datasets.
* Achieving significant cost savings (reduces expensive Redshift cluster storage by 40%+)."

---

### Q: How does Amazon Redshift differ from Redshift Spectrum and Amazon Athena?
**A:**  
| Feature | Amazon Redshift | Redshift Spectrum | Amazon Athena |
| :--- | :--- | :--- | :--- |
| **Architecture** | Provisioned MPP Data Warehouse | Feature of Redshift querying S3 | Serverless interactive query engine (Presto) |
| **Storage Location** | Local Redshift Managed Storage (RMS) | Amazon S3 object storage | Amazon S3 object storage |
| **Pricing Model** | Node hours (provisioned) or RPU (Serverless) | $5 per TB scanned + Redshift cluster cost | $5 per TB scanned (pure pay-per-query) |
| **Use Case** | High-concurrency BI dashboards, complex sub-second analytics | Seamless SQL joins between S3 data lake and Redshift tables | Ad-hoc data lake exploration, schema audit, validation checks |

---

### Q: How do you optimize query performance in Amazon Redshift?
**A:**  
1. **Distribution Styles (`DISTSTYLE`)**:
   * `KEY`: Distribute rows based on high-cardinality join columns (e.g., `item_id`) so joining tables reside on the same compute slices, avoiding network broadcast.
   * `ALL`: Replicate small lookup/dimension tables to all compute nodes.
   * `AUTO / EVEN`: For tables without clear join keys.
2. **Sort Keys (`SORTKEY`)**:
   * Define `COMPOUND` sort keys on frequently filtered columns (e.g., `transaction_date`, `store_id`) to enable Zone Map block skipping.
3. **Table Maintenance**: Run `VACUUM` to reclaim disk space from deleted rows and resort tables; run `ANALYZE` to update query planner statistics.
4. **Concurrency Scaling & WLM**: Configure Workload Management (WLM) queues to prevent long-running ad-hoc queries from blocking operational reporting."

---

### Q: How did you design Materialized Views in Redshift for your retail project?
**A:**  
"We created Materialized Views in Redshift that pre-compute complex aggregations over the external Redshift Spectrum tables (e.g., daily sales velocity, inventory turn rate by store and SKU). These views are refreshed automatically on a schedule (`REFRESH MATERIALIZED VIEW`), allowing BI dashboards and Data Science models to query pre-computed results in milliseconds without rescanning raw S3 files."

---

## 6. Airflow Orchestration, Concurrency & Scheduling

### Q: How did you manage concurrency and DAG execution in MWAA?
**A:**  
"We ran 15 to 20 concurrent pipelines in MWAA. To guarantee data integrity and SLA adherence, we implemented strict orchestration rules:
1. **Parallelism for Independent Interfaces**: Ingestion pipelines for independent domains (e.g., promotions vs product catalog) executed in parallel to maximize throughput.
2. **Sequential Queueing for Dependent Tables**: If two jobs targeted the same underlying Apache Hudi table (e.g., continuous CDC updates and a bulk batch backfill), they were queued sequentially using Airflow Pool limits or DAG trigger dependencies (`TriggerDagRunOperator`) to prevent file-locking collisions and write conflicts.
3. **Pool Allocation**: Critical business pipelines were allocated to high-priority Airflow worker pools with dedicated slot limits."

---

### Q: What would you do if a scheduled pipeline job did not trigger as expected?
**A:**  
"I follow a systematic 6-step troubleshooting workflow:
1. **Verify Schedule & Timezones**: Check if the DAG schedule interval and `start_date` are correctly aligned in UTC. Remember that Airflow executes a DAG run only at the *end* of its data interval (e.g., a daily job for 2026-08-30 triggers at 2026-08-31 00:00:00).
2. **Check Upstream Task Dependencies & Sensors**: Inspect the DAG Graph View to see if an upstream task or `ExternalTaskSensor` / `S3KeySensor` is stuck waiting for a file that arrived late or with an unexpected filename pattern.
3. **Inspect Airflow Scheduler Health**: Review scheduler logs in CloudWatch to check for scheduler heartbeat failures, database connection pool exhaustion, or syntax parsing errors in DAG definitions.
4. **Examine Resource & Slot Limits**: Verify whether Airflow worker pools or global concurrency limits (`max_active_runs`, `max_active_tasks_per_dag`) have been saturated.
5. **Check DAG Pause State**: Confirm that the DAG was not unintentionally left in a `paused` state in the Airflow UI.
6. **Trigger Manual Backfill**: If the schedule was missed due to scheduler downtime, trigger a manual DAG run with the appropriate execution date context."

---

### Q: How do you handle a pipeline run that exceeds its allocated execution window?
**A:**  
1. **Locate Bottleneck Task**: Check Airflow Gantt charts and task instance logs to identify which specific task exceeded its duration baseline.
2. **Correlate with Input Data Volume**: Compare the current batch data volume against historical averages (e.g., did an upstream marketing campaign trigger 5x normal transaction volume?).
3. **Inspect Spark / Glue Resource Metrics**: Check CloudWatch metrics (`Glue Active Workers`, `CPUUtilization`, `MemoryUtilization`) and the Spark UI to detect executor throttling or disk spill.
4. **Enforce SLAs & Timeouts**: Configure `execution_timeout` on Airflow tasks to terminate runaway processes and trigger `on_failure_callback` alerts to on-call engineers."

---

### Q: What is your approach for managing and monitoring pipeline dependencies?
**A:**  
* **Cross-DAG Dependencies**: Use `ExternalTaskSensor` or `TriggerDagRunOperator` with execution date matching.
* **S3 Data Ingestion Triggers**: Use `S3KeySensor` or event-driven triggers via Amazon EventBridge and AWS Lambda to trigger Airflow DAGs immediately upon file arrival in S3.
* **Lineage & Metadata Tracking**: Tag Airflow tasks and register table lineage in the Glue Data Catalog or Apache Atlas."

---

## 7. Data Quality, Schema Evolution & Governance

### Q: How do you ensure Data Quality in PySpark pipelines?
**A:**  
"We built a metadata-driven Data Quality framework in PySpark:
1. **Metadata Rule Repository**: Validation rules are defined in a centralized table or configuration file (`rule_id`, `column_name`, `rule_type`, `threshold`, `action`):
   * Null checks (`column IS NOT NULL`)
   * Range / Value checks (`price > 0`, `discount BETWEEN 0 AND 100`)
   * Uniqueness checks (`COUNT(DISTINCT id) == COUNT(id)`)
   * Referential integrity checks against master lookup datasets.
2. **Dynamic Rule Application**: The PySpark engine reads rules dynamically and applies validation expressions using `great_expectations` or custom PySpark DataFrame filters.
3. **Error Routing to Dead Letter Queue (DLQ)**: Validated records proceed to the Standardized Layer; records failing non-critical checks are tagged with failure codes and written to an S3 error bucket / DLQ for investigation.
4. **Data Quality Summary Reporting**: Every pipeline run writes a DQ summary record (`batch_id`, `total_records`, `passed_records`, `quarantined_records`, `execution_time`) for auditing."

---

### Q: How do you address duplicate records in a data pipeline?
**A:**  
1. **Identify Source**: Determine whether duplicates originated at the source (duplicate CDC events), due to producer retries over the network, or from faulty join logic in ETL transformations.
2. **Staging / Raw Deduplication**: Use Spark window functions (`ROW_NUMBER() OVER (PARTITION BY primary_key ORDER BY update_ts_dms DESC)`) or `df.dropDuplicates(['primary_key'])`.
3. **Lakehouse Target Deduplication**: Rely on Apache Hudi / Delta Lake `upsert` and `MERGE` logic, which enforces unique primary keys based on the precombine timestamp.
4. **Automated Reconciliation**: Schedule daily validation jobs comparing source and target record counts and checksum hashes."

---

### Q: How is data secured in Amazon S3 and Redshift?
**A:**  
* **Encryption at Rest**: Enforce Server-Side Encryption with AWS KMS customer-managed keys (`SSE-KMS`) on all S3 buckets and Redshift clusters.
* **Encryption in Transit**: Enforce TLS/HTTPS via S3 bucket policies (`aws:SecureTransport: true`).
* **IAM Least Privilege**: Restrict pipeline execution roles to specific bucket prefixes and Glue databases; use IAM Roles for EC2/Glue/MWAA rather than long-term API access keys.
* **Network Isolation**: Deploy Glue jobs and Redshift inside private VPC subnets with VPC Endpoints (Gateway endpoints for S3) to prevent traffic from traversing the public internet.
* **Fine-Grained Governance**: Use AWS Lake Formation to enforce table-level, column-level, and row-level access permissions."

---

## 8. Performance Tuning, Partitioning & Scaling

### Q: How do you choose an optimal data partitioning strategy?
**A:**  
1. **Understand Query Access Patterns**: Partition by columns that are consistently filtered in SQL `WHERE` clauses (e.g., `transaction_date`, `region`, `store_id`).
2. **Avoid Over-Partitioning (The Small File Problem)**: Ensure partitions contain sufficient data. The target file size per partition should be **128 MB to 256 MB**. Partitioning a 1 GB dataset by minute or second generates thousands of tiny KB-sized files, overwhelming S3 metadata and Spark driver query planning.
3. **Partitioning Methods**:
   * *Range / Date Partitioning*: Best for append-heavy time-series transactions (`year=YYYY/month=MM/day=DD`).
   * *List / Category Partitioning*: Best for low-cardinality geographic or domain attributes (`region=US_EAST`).
   * *Hash Partitioning*: Distribute data evenly across buckets to prevent hotspots."

---

### Q: How do you manage the Small File Problem in S3 and Lakehouse tables?
**A:**  
1. **Compaction via Hudi / Delta**: Run periodic Lakehouse compaction jobs that merge small delta commit files into optimized base Parquet files (128MB–256MB).
2. **PySpark Coalesce vs Repartition**:
   * Use `df.coalesce(n)` to decrease partition counts before writing to S3 without triggering a full network shuffle.
   * Use `df.repartition(n, 'column')` when data needs to be redistributed evenly across keys.
3. **Glue Auto-Splitting & S3 Max Record Counts**: Configure `hoodie.parquet.max.file.size` and `hoodie.parquet.small.file.limit` to ensure Hudi groups incoming writes into appropriately sized files."

---

### Q: How would you scale the AWS pipeline for a 5x–10x increase in data volume?
**A:**  
1. **Compute Scaling**:
   * Upgrade Glue worker types from `G.1X` to `G.2X` or scale the number of allocated DPUs.
   * Enable Glue Autoscaling (`--enable-auto-scaling=true`) so the driver dynamically adds workers during heavy shuffle stages and scales down during lightweight stages.
2. **Spark Parallelism & Memory**:
   * Increase `spark.sql.shuffle.partitions` proportionally to match higher partition counts.
   * Tune shuffle buffer sizes and configure Kryo serialization.
3. **Lakehouse Tuning**:
   * Switch from Copy-on-Write (CoW) to **Merge-on-Read (MoR)** table types in Hudi if write latency becomes a bottleneck under high-velocity streaming CDC.
4. **Redshift Concurrency Scaling**:
   * Enable Concurrency Scaling in Redshift to automatically spin up transient clusters during peak analytical query loads."

---

## 9. Production Incident Response, Debugging & Resiliency

### Q: What is your universal framework for handling production pipeline failures?
**A:**  
"I follow a structured 6-step incident response framework:
1. **Clarify & Scope**: Determine the blast radius immediately. Which domain tables are impacted? Is it a single upstream interface or all 17 interfaces? Are downstream BI dashboards delayed?
2. **Detect & Triage**: Inspect CloudWatch alarms, Airflow task failure notifications, and SNS alerts to capture exact failure timestamps and task IDs.
3. **Contain & Mitigate**: Stop dependent downstream jobs to prevent corrupted or partial data from propagating into the Curated layer. If an issue was caused by a bad code release, roll back to the previous stable Git commit.
4. **Diagnose & Isolate**: Inspect Airflow logs, AWS Glue CloudWatch logs, and Spark UI error traces. Pinpoint whether the root cause is a malformed source file, schema mismatch, infrastructure throttling, or resource exhaustion (OOM).
5. **Fix & Verify**: Apply the code fix or quarantine the invalid source records into a DLQ. Trigger a test backfill in a staging environment, then rerun the failed Airflow DAG in production. Verify row counts and checksums.
6. **Prevent & Post-Mortem**: Document root causes, refine data quality validation rules, adjust retry policies, and update runbooks."

---

### Q: What exact steps do you take if a pipeline fails at midnight?
**A:**  
1. **Acknowledge Alert**: Check the automated SNS/Slack alert containing the failed DAG name, task ID, and CloudWatch log URL.
2. **Review Task Logs**:
   * Check Airflow task logs for immediate error messages (e.g., `GlueJobRunException`, `SensorTimeout`).
   * Navigate to AWS CloudWatch `/aws-glue/jobs/error` to read the PySpark stack trace.
3. **Verify Data Arrival & Schema**: Check the S3 Landing bucket to verify whether the source file arrived on schedule and matches expected structural delimiters and column headers.
4. **Classify Failure**:
   * *Transient Error (API throttling, network timeout)*: Trigger an immediate retry.
   * *Data Quality / Schema Drift*: Isolate the bad file to an S3 quarantine bucket and allow the pipeline to proceed with clean batches.
   * *Cluster OOM / Infrastructure Crash*: Increase Glue worker allocation or tune shuffle partitions before restarting.
5. **Clear & Rerun**: Clear the failed task instance in Airflow. Airflow automatically reruns the failed task and resumes downstream dependencies.
6. **Notify Stakeholders**: Post an update in the engineering operations channel regarding resolution status and updated SLA delivery times."

---

### Q: What approach do you follow if data ingestion from a third-party API or source database fails?
**A:**  
1. **Review HTTP Status / Error Codes**:
   * `429 (Too Many Requests)`: Back off ingestion rate, reduce batch request sizes, implement exponential backoff with random jitter.
   * `401 / 403 (Authentication / Authorization)`: Verify API keys, AWS Secrets Manager credentials, and token expiration.
   * `500 / 503 (Server Unavailable)`: Check third-party provider status pages, verify network connectivity and NAT Gateway routes.
2. **Fallback & Resiliency**: Serve or retain last-known-good cached data for non-critical downstream consumers while retrying.
3. **Dead Letter Queue Routing**: For persistent payload failures, log the full request payload, error trace, and timestamp to an S3 DLQ for manual inspection."

---

### Q: How do you reconcile data drift between source databases and the S3 Lakehouse?
**A:**  
1. **Automated Reconciliation Jobs**: Run scheduled reconciliation queries comparing primary-key record counts and checksum hashes grouped by partition buckets between source relational databases and target Hudi tables.
2. **Audit Windows**: Compare data with a small time buffer (e.g., $t - 1 \text{ hour}$) to account for active in-flight CDC propagation lag.
3. **Corrective Resynchronization**: If discrepancy is detected, generate an automated corrective batch extracting the specific out-of-sync keys from the source and applying an idempotent Hudi upsert to the Lakehouse."

---

## 10. Open Project-Specific Action Items

The following items should be verified against your specific project deployment parameters prior to interviews:

1. **Exact AWS Glue Cluster Sizing**:
   * _[Open — verify exact DPU count used in production, e.g., 10 DPUs G.2X for Standardized Hudi jobs]_.
2. **Baseline Job Runtimes**:
   * _[Open — confirm exact average runtime per hourly batch, e.g., 35–45 minutes]_.
3. **Resume Metric Justifications**:
   * _[Open — verify exact narrative for "25% query optimization" (achieved via Redshift Spectrum partition pruning & Hudi compaction) and "45% automation" (achieved via MWAA DAG orchestration & Terraform CI/CD)]_.
4. **Table Schema Details**:
   * _[Open — verify primary keys and partitioning keys for `Item_master` (`item_id`, partitioned by `region`) and `inventory_on_hand` (`item_id`, `store_id`, partitioned by `snapshot_date`)]_.

---
*End of CDC Lakehouse Data Engineering Mother Document.*

