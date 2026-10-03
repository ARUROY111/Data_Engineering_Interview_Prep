# Data Engineer Scenario-Based Interview Guide (137 Questions with Answers)

> **Topic**: Real-world problem solving: data pipelines, Apache Spark, lakehouse (Delta / Hudi / Iceberg), streaming, AWS data services, SQL, Airflow, and distributed systems.
> **Coverage**: Q1 to Q100 (core scenarios, corrected and updated), Q101 to Q137 (additional day-to-day scenarios), plus an answer framework.

---

## Quick Navigation

- [Questions 1 – 20: Spark Tuning, Shuffling, Ingestion & Lakehouse Architecture](#questions-1--20)
- [Questions 21 – 40: Streaming, Aggregations, Failures, SCD & Lineage](#questions-21--40)
- [Questions 41 – 60: Scalability, Security, Joins, Transformations & Reusability](#questions-41--60)
- [Questions 61 – 80: High-Throughput Ingestion, Medallion Architecture & Optimization](#questions-61--80)
- [Questions 81 – 100: Kafka Streaming, Multi-Tenancy, Data Quality & End-to-End Projects](#questions-81--100)
- [Questions 101 – 137: Additional Real-World Scenarios](#questions-101--137)
- [How to Structure Scenario Answers](#how-to-structure-scenario-answers)

---

## Questions 1 – 20

### Q1. Your Spark job is running very slow. How would you identify the bottleneck?
**A:**  
1. **Spark UI Analysis (Port 4040)**:
   * **Stages & Jobs Tab**: Identify stages with disproportionately long runtimes or high task counts.
   * **Event Timeline & Task Metrics**: Compare the Min, Median, 75th percentile, and Max task durations. A large variance between Median and Max signals **Data Skew**.
   * **Shuffle Read / Write**: Check total shuffle volume. High shuffle read/write indicates expensive wide transformations (`groupByKey`, sort-merge joins).
   * **Spill (Memory) and Spill (Disk)**: Identifies executors running out of RAM during shuffles/aggregations and spilling to disk.
   * **Executors Tab**: Check GC (Garbage Collection) time. If GC time above 10-15% of task time, the JVM is thrashing memory.
2. **Driver vs Executor Logs**: Look for serialization bottlenecks, large unbroadcasted variables, or network timeout warnings in CloudWatch/YARN.
3. **Storage I/O**: Check if reading millions of small files from S3 or if partition pruning is missing.

---

### Q2. A dataset contains millions of duplicate records. How would you remove them efficiently?
**A:**  
1. **Keep the latest record per key (deterministic)**:
   ```python
   from pyspark.sql.window import Window
   from pyspark.sql.functions import col, row_number

   window_spec = Window.partitionBy("entity_id").orderBy(col("event_timestamp").desc(), col("ingest_ts").desc())
   deduped_df = df.withColumn("rn", row_number().over(window_spec)).filter(col("rn") == 1).drop("rn")
   ```
   Add a tiebreaker column so the result is the same on every run.
2. **Exact duplicates (any copy is fine)**:
   ```python
   deduped_df = df.dropDuplicates()                 # whole-row duplicates
   deduped_df = df.dropDuplicates(["entity_id"])    # keeps an ARBITRARY row per key
   ```
   Use the key-subset form only when it does not matter which copy survives.
3. **Storage-layer deduplication (Lakehouse upsert)**: Dedupe the source batch first, then `MERGE INTO target USING staging ON target.id = staging.id` in Delta Lake / Apache Hudi / Iceberg.
4. **Efficiency**: Filter and select columns before deduplicating, and dedupe as early as possible so later stages shuffle less data.
---

### Q3. You have a large dataset and need to join it with a small lookup table. What approach would you use?
**A:**  
* **Approach**: **Broadcast Hash Join**.
* **Mechanism**: Spark sends the entire small lookup table to every executor's memory.
* **Benefit**: Eliminates the shuffle of the large table, turning a sort-merge join into a local hash lookup per partition.
* **Sizing**: The default `spark.sql.autoBroadcastJoinThreshold` is 10 MB. You can raise it (commonly up to a few hundred MB, depending on executor memory), force it with a hint, or let AQE convert the join at runtime when it discovers one side is small after filtering.
* **Code**:
  ```python
  from pyspark.sql.functions import broadcast
  result_df = large_df.join(broadcast(small_lookup_df), "lookup_id", "left")
  ```
---

### Q4. Your Spark job is causing excessive shuffling. How would you optimize it?
**A:**  
1. **Filter & Project Early**: Drop unused columns (`df.select(...)`) and filter rows before joins and aggregations.
2. **Leverage Broadcast Joins**: Broadcast small dimension tables with `broadcast(df)`.
3. **Use partial aggregation**: DataFrame/SQL aggregations (`groupBy().agg()`) already combine map-side. In RDD code, use `reduceByKey` / `aggregateByKey` instead of `groupByKey`.
4. **Reuse partitioning**: If a dataset is joined or grouped repeatedly on the same key, repartition once on that key (or bucket the table) and reuse it. A repartition is itself a shuffle, so it only pays off when reused.
5. **Enable Adaptive Query Execution (AQE)**: `spark.sql.adaptive.enabled=true` (default in Spark 3.2+) coalesces shuffle partitions, switches join strategies, and handles skew at runtime.
6. **Remove unnecessary wide operations**: Avoid repeated `distinct`, `orderBy`, and `repartition` calls that are not needed.
---

### Q5. A streaming pipeline is producing late-arriving data. How would you handle it?
**A:**  
1. **Event Time vs Processing Time**: Base partition paths and windowed aggregations on the payload's event timestamp, never the pipeline's ingestion clock.
2. **Watermarking in Spark Structured Streaming**:
   ```python
   streaming_df.withWatermark("event_time", "2 hours") \
       .groupBy(window("event_time", "10 minutes"), "user_id") \
       .count()
   ```
   The watermark tells Spark how long to keep state; events older than the watermark are **dropped from the aggregation**. Spark does not send them to a DLQ by itself.
3. **Do not lose late events**: Write every raw event to the bronze layer *before* aggregating (or split late rows in `foreachBatch` by comparing event time to the current watermark and write them to a late-data table).
4. **Correct the history**: Reprocess affected windows/partitions from bronze, or `MERGE INTO` the Delta/Hudi/Iceberg aggregate table so late data updates historical results without duplicates.
---

### Q6. Your data pipeline needs to process 1TB of data daily. How would you design it?
**A:**  
* **Architecture**:
  * **Ingestion**: Raw files land in Amazon S3 (Landing Zone) partitioned by `/year/month/day/hour/`.
  * **Compute**: AWS Glue / Amazon EMR running Apache Spark with autoscaling enabled and `G.2X` workers.
  * **Storage Format**: Convert raw files to Parquet with Snappy compression and write to Delta Lake / Apache Iceberg in 128MB–256MB file sizes.
  * **Tuning**: Set `spark.sql.shuffle.partitions = 1000-2000`, enable Kryo serialization, and use broadcast joins for dimensions.
  * **Orchestration**: Orchestrate hourly or incremental micro-batches using Apache Airflow (MWAA) with retries and SLA monitoring.
  * **Serving**: External query layer using Redshift Spectrum and Athena.

---

### Q7. A table contains skewed data causing uneven partitions. How would you handle data skew?
**A:**  
1. **Key Salting**: Add a random salt integer prefix (`concat(key, '_', floor(rand() * 10))`) to the skewed key on the large table, explode the lookup table 10 times with corresponding salt keys, join on salted keys, and aggregate back.
2. **Adaptive Query Execution (AQE)**:
   ```python
   spark.conf.set("spark.sql.adaptive.enabled", "true")
   spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
   ```
3. **Broadcast Join**: If one of the joining tables is small, broadcast it to bypass partition hashing entirely.
4. **Isolate Skewed Keys**: Filter out nulls/frequent keys, process them in a separate isolated pipeline path, and union the results back.

---

### Q8. A business team requires near real-time dashboards. What architecture would you design?
**A:**  
* **Ingestion**: Amazon Kinesis Data Streams / Amazon MSK (Kafka) captures live operational events.
* **Streaming Engine**: Amazon Managed Service for Apache Flink (formerly Amazon Managed Service for Apache Flink) or Spark Structured Streaming computes 1-minute windowed aggregations.
* **Serving Layer**: Stream aggregated metrics into **Amazon Redshift** (streaming ingestion with auto-refreshing materialized views), **Amazon OpenSearch**, **ClickHouse**, or **DynamoDB**. For time-series metrics use Amazon Timestream for InfluxDB (Timestream for LiveAnalytics is closed to new customers).
* **Visualization**: **Amazon QuickSight** (direct query or SPICE with frequent refresh) or Grafana.
---

### Q9. Your pipeline needs to support schema evolution. How would you design for that?
**A:**  
1. **Registry Enforcement**: Use AWS Glue Schema Registry or Confluent Schema Registry with `BACKWARD` or `FULL` compatibility rules.
2. **Storage Layer Schema Evolution**: Use **Delta Lake / Apache Iceberg** which support out-of-the-box metadata schema evolution (`.option("mergeSchema", "true")`).
3. **Safe Ingestion Logic**:
   * New optional columns are automatically added to the catalog; historical queries return `NULL`.
   * Dropped columns are retained in the data lake schema and null-padded.
   * Incompatible type changes are quarantined to an S3 Dead Letter Queue (DLQ).

---

### Q10. How would you design a CDC (Change Data Capture) pipeline?
**A:**  
1. **Source Capture**: AWS DMS or Debezium tails the relational database transaction log (binlog/WAL), extracting row-level mutations (`INSERT`, `UPDATE`, `DELETE`) with metadata (`op`, `timestamp`, `primary_key`).
2. **Staging**: DMS buffers delta change feeds into an S3 raw bucket.
3. **Lakehouse MERGE**: A scheduled PySpark Glue job reads the delta stream and executes a `MERGE INTO` statement against target Apache Hudi / Delta Lake tables on S3, applying updates and hard deletes.
4. **Catalog Sync**: Metadata changes are automatically synced to the AWS Glue Data Catalog for Athena/Redshift querying.

---

### Q11. Your Spark job fails due to memory issues. What steps would you take?
**A:**  
1. **Identify Driver vs Executor Failure**:
   * *Driver OOM*: Caused by `.collect()`, large broadcasts, or metadata overhead → Increase `spark.driver.memory` and write directly to S3.
   * *Executor OOM / YARN Container Killed*: Caused by data skew, large shuffles, or memory leak → Increase `spark.executor.memory` and `spark.executor.memoryOverhead`.
2. **Tune Partitioning**: Increase `spark.sql.shuffle.partitions` (e.g., from 200 to 1000+) to reduce partition data size per executor task.
3. **Garbage Collection**: Enable G1GC garbage collector (`-XX:+UseG1GC`) to prevent GC thrashing.
4. **Unpersist Caches**: Call `df.unpersist()` on intermediate DataFrames that are no longer needed.

---

### Q12. A dataset contains nested JSON structures. How would you flatten it?
**A:**  
* **Use PySpark built-in functions (`from_json`, `explode_outer`, struct field access)**:
  ```python
  from pyspark.sql.functions import col, explode_outer

  # Explode array of objects into rows. explode_outer keeps rows whose array is null or empty.
  exploded_df = df.withColumn("item", explode_outer(col("order_items")))

  # Flatten struct attributes
  flat_df = exploded_df.select(
      col("order_id"),
      col("customer_id"),
      col("item.item_id").alias("item_id"),
      col("item.price").alias("price"),
      col("item.quantity").alias("quantity")
  )
  ```
* Plain `explode` silently drops rows with null or empty arrays, so use `explode_outer` unless you want that.
* If the JSON arrives as a string column, parse it first with `from_json(col, schema)`.
---

### Q13. Your ETL job is producing too many small files. How would you fix it?
**A:**  
1. **Before Writing in Spark**:
   * Use `df.coalesce(N)` to reduce output partitions without a full shuffle.
   * Use `df.repartition(N)` (or `repartition(N, "partition_col")`) when data must be redistributed evenly.
   * Let AQE coalesce small shuffle partitions.
2. **Table-format maintenance**:
   * Delta Lake: `OPTIMIZE` (bin-packing) and optimized writes.
   * Iceberg: `rewrite_data_files` procedure (or the Glue Data Catalog automatic compaction).
   * Hudi: small-file handling at write time (`hoodie.parquet.small.file.limit`, `hoodie.parquet.max.file.size`), **clustering** to rewrite small files, and **compaction** for Merge-on-Read tables (merges log files into base files).
3. **Target size**: 128 MB to 256 MB per file, e.g. `hoodie.parquet.max.file.size = 268435456`.
4. **Fix the source**: Batch upstream writers (e.g. increase Firehose buffer size/interval) so tiny files are not created in the first place.
---

### Q14. A join operation is causing performance issues. What optimization strategies would you use?
**A:**  
1. **Broadcast Join**: For small-to-large table joins (under roughly 100 MB (tunable, default threshold 10 MB)), force a broadcast join (`broadcast(small_df)`).
2. **Bucket & Sort on Join Keys**: Pre-bucket both datasets by the join key (`df.write.bucketBy(num_buckets, "join_key")`), enabling bucket-to-bucket joins without runtime shuffles.
3. **Filter Early**: Filter nulls and irrelevant rows before joining.
4. **Mitigate Skew**: Use key salting if specific join keys contain disproportionate data volume.
5. **Sort-Merge Join Tuning**: Ensure adequate shuffle partitions to avoid disk spilling.

---

### Q15. How would you design a data lake architecture?
**A:**  
* **Storage Hierarchy (Medallion Architecture)**:
  * **Landing Zone (Transient)**: Raw file arrival from upstream sources.
  * **Bronze (Raw)**: Append-only immutable historical store preserving exact source payloads with arrival metadata (`ingest_timestamp`).
  * **Silver (Standardized/Conformed)**: Cleaned, deduplicated, and typed data with ACID Lakehouse tables (Delta Lake / Hudi / Iceberg).
  * **Gold (Curated/Business Layer)**: Dimensional models (star/snowflake schema), pre-aggregated metrics, and Materialized Views in Redshift/Snowflake.
* **Security & Governance**: Centralized access via AWS Lake Formation, encryption via AWS KMS (`SSE-KMS`), and cataloging in AWS Glue Data Catalog.

---

### Q16. A pipeline processes streaming clickstream data. How would you store and analyze it?
**A:**  
1. **Ingestion**: Amazon Kinesis Data Streams / Kafka captures JSON clickstream events.
2. **Streaming Ingestion**: Amazon Data Firehose or Spark Structured Streaming converts streams into columnar Parquet and commits them into an S3 Bronze lake partitioned by `date/hour`.
3. **Real-Time Aggregations**: Spark Structured Streaming or Flink computes tumbling window metrics (active sessions, CTR per minute) and writes to Redis / DynamoDB.
4. **Historical Analytics**: S3 Parquet partitions are queried directly using Amazon Athena and Redshift Spectrum.
---

### Q17. A dataset contains inconsistent schema across files. How would you handle it?
**A:**  
1. **AWS Glue DynamicFrames**: Use `ResolveChoice` to handle ambiguity (e.g., casting mixed integer/string fields into strings).
2. **Schema Merging in Spark**:
   ```python
   df = spark.read.option("mergeSchema", "true").parquet("s3://bucket/path/")
   ```
3. **Pre-Ingestion Validation**: Read with a permissive mode (`PERMISSIVE`) and route malformed records to a `_corrupt_record` column for Dead Letter Queue quarantine.

---

### Q18. Your Spark job repeatedly processes the same data. How would you optimize using caching?
**A:**  
* **Use Caching with Optimal Storage Level**:
  ```python
  from pyspark import StorageLevel

  # Cache serialized data to reduce JVM heap footprint
  df_transformed.persist(StorageLevel.MEMORY_AND_DISK_SER)

  # Trigger evaluation action
  df_transformed.count()

  # Perform multiple downstream computations...

  # Always release memory when finished
  df_transformed.unpersist()
  ```
* **Best Practice**: Cache only intermediate DataFrames that are branched into multiple downstream actions or used in iterative algorithms.

---

### Q19. A table has billions of rows and queries are slow. How would you optimize query performance?
**A:**  
1. **Partition Pruning**: Ensure queries filter on physical partition columns (e.g., `date`).
2. **Columnar Storage with Compression**: Store as Parquet/ORC so engines scan only referenced columns.
3. **Clustering / Z-Ordering** (syntax depends on the format):
   * Delta Lake: `OPTIMIZE tbl ZORDER BY (col)`
   * Iceberg: `CALL system.rewrite_data_files(table => 'db.tbl', strategy => 'sort', sort_order => 'zorder(col1,col2)')`
   * Hudi: clustering with sort / space-filling-curve strategies
4. **Data Warehouse Keys**: In Amazon Redshift, choose a `DISTKEY` on the common join column and a `SORTKEY` on common filter columns (or use `AUTO` and let Redshift decide).
5. **Materialized Views**: Precompute heavy aggregations.
6. **Read the plan**: Use `EXPLAIN` / Spark UI to confirm pruning and pushdown actually happen.
---

### Q20. How would you design a partitioning strategy for a large dataset?
**A:**  
1. **Partition on low-to-moderate cardinality columns that appear in most filters**: usually a date (`year/month/day`) and sometimes a category such as `region`.
2. **Target at least ~128 MB per partition** (ideally 128 MB to 256 MB files): every partition directory should hold substantial data.
3. **Avoid Over-Partitioning**: Do not partition by minute-level timestamps or high-cardinality IDs (`user_id`), which creates millions of tiny files and slow catalog listings.
4. **High-cardinality filter columns** (user_id, device_id) are handled with bucketing, sort order / Z-order / clustering, or table-format indexes, not partitions.
5. **Dynamic partition overwrite**: `spark.sql.sources.partitionOverwriteMode=dynamic` replaces only the partitions present in the DataFrame, leaving other partitions untouched. Every touched partition is replaced **completely**, so the DataFrame must hold the full contents of those partitions.
---

## Questions 21 – 40

### Q21. How would you ensure fault tolerance in a distributed data pipeline?
**A:**  
1. **Pipeline Idempotency**: Design writes using Lakehouse `MERGE INTO` or full-partition overwrites so re-executing failed batches never creates duplicate rows.
2. **Checkpointing & State Recovery**: Use Spark Structured Streaming checkpoints in S3 to persist offset and state progress.
3. **Automated Retries**: Configure retries with exponential backoff in Apache Airflow (`retries=3`, `retry_delay=timedelta(minutes=5)`, `retry_exponential_backoff=True`).
4. **Decoupled Architecture**: Use durable message brokers (Kafka/Kinesis/SQS) that persist messages during consumer downtime.
5. **Dead Letter Queues (DLQ)**: Isolate bad records into quarantine storage without halting the main pipeline.
---

### Q22. Your Spark job produces incorrect aggregations. How would you debug it?
**A:**  
1. **Check Input Null Handling**: Nulls in join keys or grouping columns often skew aggregates (`SUM`, `COUNT`, `AVG`).
2. **Inspect Join Cardinality**: Check for unintentional Many-to-Many joins multiplying row counts.
3. **Examine Window Specifications**: Verify that `ORDER BY` inside window functions is not converting simple aggregations into cumulative running totals.
4. **Floating Point Precision**: Use `DecimalType` instead of `DoubleType` for financial calculations.
5. **Isolate Sub-DataFrames**: Print intermediate schema and sample outputs (`df.show(5)`) at each transformation stage.

---

### Q23. A streaming pipeline needs exactly-once processing. How would you implement it?
**A:**  
"True exactly-once delivery over a network is not achievable; what we achieve is an **exactly-once processing effect**:
1. **Replayable Source**: Kafka/Kinesis retaining offsets so data can be re-read after failure.
2. **Stateful Engine with Checkpointing**: Spark Structured Streaming records offsets in its checkpoint offset log and commit log; Apache Flink uses distributed checkpoints (with two-phase-commit sinks where supported).
3. **Idempotent or Transactional Sink**:
   * Delta Lake / Hudi / Iceberg commits are atomic. In `foreachBatch`, use a deterministic `MERGE INTO` on a unique event ID (or Delta's `txnAppId`/`txnVersion` idempotent-write options) so a replayed batch has no extra effect.
   * Relational database upserts with `UNIQUE` constraints on message IDs."
---

### Q24. Your ETL pipeline requires data validation rules. How would you implement them?
**A:**  
1. **Metadata-Driven Rule Framework**: Define validation rules in a configuration table (e.g., `not_null`, `range_check`, `regex_match`).
2. **PySpark Rule Engine** (NULL-safe, so no row is lost):
   ```python
   from pyspark.sql import functions as F

   rule     = (F.col("price") > 0) & F.col("customer_id").isNotNull()
   is_valid = F.coalesce(rule, F.lit(False))      # NULL result counts as invalid

   valid_df   = df.filter(is_valid)
   invalid_df = df.filter(~is_valid)              # exact complement of valid_df

   invalid_df.write.parquet("s3://lake/dlq/reason=validation_failure/")
   # Reconciliation check: valid_df.count() + invalid_df.count() == df.count()
   ```
   Filtering with two separate conditions can drop rows whose columns are NULL (three-valued logic), which is why the complement form is used.
3. **Integration with Tools**: **Great Expectations** or **AWS Glue Data Quality** for automated pre/post-load assertions.
---

### Q25. A data pipeline fails due to schema mismatch. How would you handle it?
**A:**  
1. **Immediate Triage**: Isolate the failing incoming batch to an S3 quarantine directory to unblock downstream pipelines.
2. **Permissive Schema Parsing**: Use Spark JSON/CSV `mode='PERMISSIVE'` with `columnNameOfCorruptRecord` to capture malformed rows.
3. **Schema Registry Validation**: Enforce schema compatibility in Glue Schema Registry.
4. **Patch & Backfill**: If the change was an intended upstream addition, update the Glue Data Catalog with `mergeSchema=true` and rerun the batch.

---

### Q26. Your Spark job is generating huge shuffle files. How would you reduce them?
**A:**  
1. **Filter and Select Columns Early**: Drop unnecessary columns before joins and aggregations to reduce byte volume.
2. **Aggregate before shuffling**: DataFrame aggregations do this automatically; in RDD code use `reduceByKey` instead of `groupByKey`.
3. **Tune Shuffle Partition Count**: Set `spark.sql.shuffle.partitions` appropriately (about 2 to 4 tasks per core, and partitions of roughly 100 to 200 MB), or rely on AQE coalescing.
4. **Use Broadcast Joins**: Eliminate shuffles for small lookup tables.
5. **Compression**: Shuffle compression is on by default (`spark.shuffle.compress=true`, lz4). `zstd` (`spark.io.compression.codec=zstd`) gives a better ratio at some CPU cost.
6. **Fix skew**: One huge key produces one huge shuffle block; use AQE skew handling or salting.
---

### Q27. You need to merge incremental data into a data warehouse table. How would you design it?
**A:**  
1. **Staging Table Load**: Load incremental delta files into a transient staging table in Amazon Redshift / Snowflake (for Redshift use `COPY` from S3).
2. **Option A, native MERGE** (supported in Redshift and Snowflake):
   ```sql
   MERGE INTO target_table
   USING staging_table s
   ON target_table.id = s.id
   WHEN MATCHED THEN UPDATE SET col1 = s.col1, col2 = s.col2
   WHEN NOT MATCHED THEN INSERT (id, col1, col2) VALUES (s.id, s.col1, s.col2);
   ```
3. **Option B, delete + insert in one transaction**:
   ```sql
   BEGIN TRANSACTION;
   DELETE FROM target_table
   USING staging_table
   WHERE target_table.id = staging_table.id;

   INSERT INTO target_table
   SELECT * FROM staging_table;
   END TRANSACTION;
   ```
4. **Lakehouse Alternative**: In Delta Lake / Hudi / Iceberg, run `MERGE INTO target USING staging ON target.id = staging.id WHEN MATCHED THEN UPDATE SET * WHEN NOT MATCHED THEN INSERT *`.
5. Deduplicate the staging data by key first so the merge is deterministic.
---

### Q28. Your pipeline requires historical tracking of changes. How would you implement SCD Type 2?
**A:**  
* **Schema Design**: Include `surrogate_key`, `start_date`, `end_date`, `is_current_flag`, and optionally a `row_hash` of the tracked attributes.
* **PySpark / SQL Implementation**:
  1. New records: insert with `start_date = current_date`, `end_date = '9999-12-31'`, `is_current = 'Y'`.
  2. Changed records (row hash differs): update the active row with `end_date = current_date` and `is_current = 'N'`, then insert a new active row with the new values.
  3. In one atomic `MERGE`, stage changed rows twice: once with the real key (to expire the old row) and once with a NULL merge key (so it falls into the insert branch).
* Use the event's effective date, not the load date, if history must be point-in-time accurate.
---

### Q29. A batch job runs for 6 hours. How would you reduce the runtime?
**A:**  
1. **Switch from Full Refresh to Incremental Processing**: Use CDC or watermark timestamps to process only new/updated records.
2. **Eliminate Data Skew**: Use key salting and AQE skew join optimizations.
3. **Optimize Joins**: Convert small table joins to Broadcast Hash Joins.
4. **Partition Pruning**: Pushdown predicates so Spark reads only relevant partitions from S3.
5. **Scale Cluster Resources**: Enable Glue Autoscaling or increase executor cores/memory.

---

### Q30. How would you monitor and alert failures in data pipelines?
**A:**  
* **Metrics & Alarms**: Configure Amazon CloudWatch alarms on pipeline failure metrics, task duration anomalies, and consumer lag.
* **Orchestrator Callbacks**: Configure Apache Airflow `on_failure_callback` to send structured alerts to Slack and PagerDuty containing DAG name, failed task ID, execution date, and direct links to CloudWatch logs.
* **Data Quality Alerts**: Trigger alerts if record quarantine counts in the Dead Letter Queue exceed defined percentage thresholds.

---

### Q31. A business team needs hourly data refresh. How would you schedule pipelines?
**A:**  
1. **Airflow (MWAA) Scheduling**: Set `schedule="@hourly"` (cron `0 * * * *`; `schedule_interval` is the legacy argument name) with `catchup=False`.
2. **Event-Driven Triggers**: Use Amazon EventBridge + AWS Lambda to trigger the Airflow DAG (via the MWAA API) as soon as the hourly file lands in S3.
3. **Partition Alignment**: Organize output S3 paths by `/year=YYYY/month=MM/day=DD/hour=HH/` to enable fast incremental loads into Redshift.
4. Make each run idempotent and parameterized by the data interval, so reruns and delays are safe.
---

### Q32. Your Spark cluster resources are underutilized. How would you improve utilization?
**A:**  
1. **Check task count vs cores in the Spark UI**: If a stage has fewer tasks than cores, cores sit idle.
2. **Adjust Partition Count**: For DataFrame/SQL jobs raise `spark.sql.shuffle.partitions` or `repartition` before the heavy stage, and check input split size (`spark.sql.files.maxPartitionBytes`). Aim for 2 to 4 tasks per core. (`spark.default.parallelism` only affects RDD operations.)
3. **Enable Dynamic Resource Allocation**: `spark.dynamicAllocation.enabled=true` so executors scale with load (Glue: use Auto Scaling).
4. **Right-Size Worker Types**: Switch worker/instance types when CPU is saturated while memory sits idle, or the reverse.
5. **Fix skew**: One long task keeps the whole stage (and cluster) waiting.
---

### Q33. A dataset contains corrupt records. How would you detect and handle them?
**A:**  
1. **Spark Read Mode `PERMISSIVE`**:
   ```python
   df = spark.read.option("mode", "PERMISSIVE") \
       .option("columnNameOfCorruptRecord", "_corrupt_record") \
       .json("s3://bucket/path/")
   ```
2. **Isolate and Quarantine**:
   ```python
   corrupt_df = df.filter(col("_corrupt_record").isNotNull())
   clean_df = df.filter(col("_corrupt_record").isNull()).drop("_corrupt_record")

   corrupt_df.write.parquet("s3://lake/quarantine/")
   ```
3. **Trigger Alert**: Dispatch an SNS notification if corrupt row count exceeds threshold.

---

### Q34. You need to process logs from multiple sources. How would you design ingestion?
**A:**  
1. **Collection**: Deploy Fluent Bit / Amazon CloudWatch Agent to forward server logs to **Amazon Data Firehose** (formerly Amazon Data Firehose).
2. **Partitioning on Ingestion**: Firehose buffers, compresses (GZIP/Snappy), and writes objects to S3. Use **dynamic partitioning** (or one delivery stream per source) to get prefixes like `/source_name/YYYY/MM/DD/HH/`.
3. **ETL Standardization**: Scheduled Glue PySpark jobs parse JSON/syslog patterns, extract structured fields, and write Parquet for Athena, or index into Amazon OpenSearch for search and dashboards.
---

### Q35. Your Spark job fails intermittently. How would you troubleshoot?
**A:**  
1. **Correlate with Input Data Volatility**: Check if intermittent failures coincide with large batch size spikes or sudden data skew.
2. **Check for Spot Instance Termination**: If running on Amazon EMR with Spot instances, check YARN logs for node decommission events.
3. **Inspect Network / Throttling**: Look for AWS S3 `503 Slow Down` throttling errors or database connection timeouts.
4. **Review Spark UI Task Logs**: Identify if specific executor nodes are failing due to transient JVM GC pause timeouts (`Heartbeat to driver timed out`).

---

### Q36. How would you design a metadata-driven ETL framework?
**A:**  
1. **Control Database / Config Store**: Store pipeline definitions in DynamoDB/PostgreSQL (`pipeline_id`, `source_path`, `target_table`, `schema_def`, `primary_keys`, `watermark_col`, `dq_rules`).
2. **Generic PySpark Execution Engine**: A standardized Spark script accepts `pipeline_id` as a parameter, dynamically fetches config, applies transformations, executes validations, and writes output.
3. **Audit & Lineage**: Automatically logs job execution stats (`rows_read`, `rows_written`, `duration`, `status`) to an audit table upon completion.

---

### Q37. A large dataset must be queried interactively. What storage format would you choose?
**A:**  
* **Format**: **Apache Iceberg (or Delta Lake) on Parquet, with sort/Z-order clustering on the common filter columns**.
* **Rationale**:
  * Columnar layout gives strong compression and column projection.
  * Partitioning plus min/max file statistics (and Z-order/sort clustering) let engines such as Athena, Trino/Starburst, and Spark skip most non-matching files.
  * Snapshot isolation lets readers run while writers commit.
* Z-order syntax differs by format (Delta `OPTIMIZE ... ZORDER BY`, Iceberg `rewrite_data_files` with a `zorder` sort order); Athena's Iceberg `OPTIMIZE` only bin-packs files.
---

### Q38. Your pipeline must support both batch and streaming. How would you design it?
**A:**  
* **Kappa Architecture (Unified Engine)**:
  * Ingest all data into **Apache Kafka / Amazon Kinesis** as the single source of truth.
  * Use **Apache Spark Structured Streaming** or **Apache Flink** to process both real-time stream micro-batches and historical backfills using identical transformation logic.
  * Persist output into an ACID Lakehouse format (Delta Lake / Hudi) on S3, providing unified serving for both streaming analytics and batch BI queries.

---

### Q39. A job frequently fails due to executor loss. What would you investigate?
**A:**  
1. **Executor Memory Overruns**: Check whether executor memory + overhead exceeded the container limit. YARN reports "Container killed by YARN for exceeding memory limits" (usually exit code 143); a kernel/Kubernetes OOM kill shows exit code 137. Read the diagnostic message, not only the code.
2. **Long GC Pauses**: A full GC can freeze an executor so it misses heartbeats (`Heartbeat timed out`, default network timeout 120 s). Tune G1GC and memory.
3. **Spot Instance Reclamation**: Use on-demand nodes for the driver and critical executors, and enable decommissioning/graceful shutdown.
4. **Disk Full during Shuffles**: Shuffle spill exhausting executor local disk.
---

### Q40. Your team needs to track data lineage. How would you implement it?
**A:**  
1. **OpenLineage**: Add the OpenLineage Spark listener (and the Airflow OpenLineage provider) so each job run emits inputs, outputs, schemas, and run metadata to a backend such as Marquez, DataHub, or Apache Atlas.
2. **AWS-native option**: Amazon DataZone / the SageMaker catalog can capture lineage (including OpenLineage events). The Glue Data Catalog and Lake Formation handle metadata and permissions but do not draw lineage graphs.
3. **Airflow**: `inlets`/`outlets` (Datasets, renamed Assets in Airflow 3) describe which tasks produce and consume data and drive data-aware scheduling.
4. **Column-level lineage** needs a tool that parses SQL/Spark plans (OpenLineage facets, DataHub, dbt docs).
---

## Questions 41 – 60

### Q41. How would you build a scalable data ingestion framework?
**A:**  
* **Decoupled Architecture**: Ingestion decoupled from transformation via Amazon S3 / Kafka buffer.
* **Configuration-Driven**: Metadata configs define source connections, ingestion frequency, file formats, and target paths.
* **Autoscaling Ingestion Compute**: Serverless ingestion workers (AWS Lambda for micro-batches, AWS Glue / EMR Autoscaling for bulk batch).
* **Automated Data Quality & DLQ**: Real-time schema validation rejecting bad payloads to dead letter storage without breaking ingestion streams.

---

### Q42. Your dataset contains PII data. How would you secure it?
**A:**  
1. **Encryption**: AWS KMS customer-managed keys at rest (`SSE-KMS`) and TLS 1.2+ in transit.
2. **Tokenization / Keyed Hashing**: Replace sensitive values with tokens from a vault or token service, use format-preserving encryption, or use a keyed hash (HMAC-SHA256) with the key in AWS Secrets Manager/KMS. A plain unsalted `SHA-256(ssn)` is reversible by brute force because the input space is small.
   ```python
   import hmac, hashlib
   from pyspark.sql import functions as F

   KEY = get_secret("pii-hmac-key").encode()   # from Secrets Manager, never hardcoded

   @F.udf("string")
   def tokenize(v):
       return hmac.new(KEY, v.encode(), hashlib.sha256).hexdigest() if v else None
   ```
3. **Granular Access Control**: **AWS Lake Formation** column-level permissions and row filters, with least-privilege IAM roles.
4. **Discovery**: **Amazon Macie** to find and classify sensitive data in S3.
5. **Governance**: Retention limits, audit logging (CloudTrail), and a process for deletion requests.
---

### Q43. A Spark job needs to process millions of small JSON files. How would you optimize it?
**A:**  
1. **Group files on read (AWS Glue)**:
   ```python
   dyf = glueContext.create_dynamic_frame.from_options(
       connection_type="s3",
       connection_options={"paths": ["s3://bucket/raw/"], "recurse": True,
                           "groupFiles": "inPartition", "groupSize": "134217728"},  # 128 MB
       format="json")
   ```
2. **Plain Spark**: Tune `spark.sql.files.maxPartitionBytes` and `spark.sql.files.openCostInBytes` so many small files pack into each task, and **provide an explicit schema** so Spark does not scan millions of files for schema inference.
3. **Compact once**: Run a Glue job or EMR S3DistCp (`--groupBy`, `--targetSize`) to merge tiny files into larger Parquet files, and process the compacted data from then on.
4. **Fix the producer**: Batch writes (e.g. larger Firehose buffer) so small files stop accumulating.
5. (Databricks-only: Auto Loader `cloudFiles` handles this natively; it is not available in Glue/EMR.)
---

### Q44. How would you implement idempotent pipelines?
**A:**  
1. **Deterministic Keys & Lakehouse Upserts**: `MERGE INTO target USING staging ON target.id = staging.id`, with the staging data deduplicated by key.
2. **Full-Partition Dynamic Overwrite**:
   ```python
   spark.conf.set("spark.sql.sources.partitionOverwriteMode", "dynamic")
   df.write.mode("overwrite").partitionBy("date").parquet("s3://bucket/path/")
   ```
   Safe only when `df` contains the complete data for every partition it touches.
3. **Batch Tracking Table**: Record batch/event IDs in a transactional table to skip batches already applied.
4. **Parameterize by data interval**, not by "now", so reruns produce the same output.
---

### Q45. Your data warehouse queries are slow. How would you optimize them?
**A:**  
1. **Review Distribution & Sort Keys (Redshift)**: Join tables on a shared `DISTKEY`, use `DISTSTYLE ALL` for small dimensions, and sort on filtered columns (or `AUTO`, which Redshift tunes over time).
2. **Statistics and Sorting**: Redshift runs auto-analyze and auto-vacuum, but after large loads check `SVV_TABLE_INFO` (`stats_off`, `unsorted`, `skew_rows`) and run `ANALYZE` / `VACUUM` where needed.
3. **Workload Management**: Auto WLM or separate queues for ETL vs reporting, with Concurrency Scaling for peaks.
4. **Materialized Views**: Precompute heavy aggregations for repeated dashboard queries.
5. **Read the plan**: Look for `DS_BCAST_INNER` / `DS_DIST_BOTH` redistribution steps and fix with better distribution keys.
---

### Q46. A dataset arrives late from upstream systems. How would you handle dependencies?
**A:**  
1. **Sensors with Timeouts**: Use `S3KeySensor` / `ExternalTaskSensor` in `mode="reschedule"` (or deferrable operators) with sensible timeouts, so they do not hold worker slots.
2. **Event-Driven Triggers**: Replace fixed cron timing with EventBridge rules that trigger the pipeline when the upstream file arrives.
3. **SLA Monitoring & Fallbacks**: On Airflow 2.x use `sla_miss_callback`; in Airflow 3 (where SLAs were removed) use task timeouts plus CloudWatch or external monitors. Alert on-call, and if the business allows, run downstream jobs on the last-known-good data past the cutoff and mark the output as stale.
---

### Q47. Your pipeline needs automatic retries. How would you implement it?
**A:**  
* **Orchestrator Level**: In Airflow DAG default arguments:
  ```python
  default_args = {
      'retries': 3,
      'retry_delay': timedelta(minutes=5),
      'retry_exponential_backoff': True,
      'max_retry_delay': timedelta(minutes=30)
  }
  ```
* **Application Level**: Wrap external API and database connection calls in retry decorators with exponential backoff and randomized jitter (e.g., `tenacity` library in Python).

---

### Q48. A Spark job reads data from S3 slowly. What optimizations would you try?
**A:**  
1. **Partition Pruning**: Filter on partition columns so only the needed prefixes are read.
2. **Columnar Formats**: Parquet/ORC with predicate pushdown and column pruning instead of CSV/JSON.
3. **File layout**: Fewer, larger files (128 MB to 256 MB), and tune `spark.sql.files.maxPartitionBytes`.
4. **Avoid S3 throttling (`503 Slow Down`)**: S3 scales automatically per prefix (about 5,500 GET and 3,500 PUT requests per second per prefix); spread keys across prefixes, use fewer larger files, and retry with exponential backoff.
5. **Committer and connector**: Use the EMRFS S3-optimized committer (EMR/Glue) or S3A committers, and tune connection-pool settings.
6. **S3 Gateway VPC Endpoint**: Keeps traffic private and avoids NAT gateway data charges (a cost/security benefit rather than a speed-up).
---

### Q49. Your pipeline requires audit logging. How would you implement it?
**A:**  
* **Audit Metadata Table**: Maintain an audit log database (`pipeline_name`, `batch_id`, `execution_date`, `records_read`, `records_written`, `quarantined_records`, `start_time`, `end_time`, `status`).
* **PySpark Audit Wrapper**: Record row counts before and after transformations and write execution metrics to the audit table in a `finally:` block.
* **Structured JSON Application Logs**: Emit logs in JSON containing `trace_id` and `batch_id` to Amazon CloudWatch.

---

### Q50. A dataset must support time-travel queries. How would you design it?
**A:**  
* **Use a Modern Table Format**: **Delta Lake**, **Apache Iceberg**, or Hudi.
* **Query Historical Snapshots**:
  ```sql
  -- Query by timestamp
  SELECT * FROM item_inventory TIMESTAMP AS OF '2026-08-30 00:00:00';

  -- Query by commit/snapshot version
  SELECT * FROM item_inventory VERSION AS OF 42;
  ```
  (Athena Iceberg syntax: `FOR TIMESTAMP AS OF` / `FOR VERSION AS OF`.)
* **Manage Retention**: The time-travel window is bounded by retention settings. Delta: `delta.logRetentionDuration` (default 30 days) and `delta.deletedFileRetentionDuration` (default 7 days, enforced by `VACUUM`). Iceberg: `expire_snapshots`. Balance the window against storage cost.
---

### Q51. Your pipeline needs schema validation before processing. How would you implement it?
**A:**  
1. **Pre-Processing Gatekeeper**: Compare column names and data types, not the whole `StructType`, because `StructType` equality also compares nullability and metadata (files are usually read as all-nullable, which causes false mismatches):
   ```python
   def sig(schema):
       return {(f.name.lower(), f.dataType.simpleString()) for f in schema.fields}

   missing = sig(expected_schema) - sig(df.schema)
   extra   = sig(df.schema) - sig(expected_schema)

   if missing:                         # breaking change: quarantine and alert
       df.write.parquet("s3://lake/schema_mismatch_quarantine/")
       raise ValueError(f"Missing or changed columns: {missing}")
   if extra:                           # additive change: allow and log
       logger.warning("New columns detected: %s", extra)
   ```
2. **AWS Glue Schema Registry**: Enforce compatibility checks (BACKWARD/FULL) for streaming schemas.
---

### Q52. You need to process IoT streaming data. What architecture would you design?
**A:**  
1. **Ingestion**: **AWS IoT Core** receives MQTT messages and an IoT rule forwards them to **Amazon Kinesis Data Streams** (or MSK).
2. **Real-Time Processing**: **Amazon Managed Service for Apache Flink** or Spark Structured Streaming cleans data, deduplicates by device ID and message ID, and calculates rolling 5-minute anomaly metrics.
3. **Hot Path Storage**: Anomalies and latest device state go to **DynamoDB**, or to **Timestream for InfluxDB** / OpenSearch for time-series queries and alerting.
4. **Cold Path Storage**: Raw telemetry flows through **Amazon Data Firehose** to S3 Parquet partitions for long-term analytics and ML training.
---

### Q53. Your Spark job processes skewed keys during joins. What techniques can help?
**A:**  
* **Key Salting**: Append random integer suffixes (`0-N`) to the skewed key on the larger table, explode the smaller table N times with matching suffixes, join on salted keys, and aggregate.
* **Broadcast Join**: If the joining table is small, broadcast it to bypass partition hashing.
* **AQE Skew Join Optimization**: Enable `spark.sql.adaptive.skewJoin.enabled=true`.
* **Separate Skewed Keys**: Split the DataFrame into skewed and non-skewed subsets, process separately, and union.

---

### Q54. A dataset needs to be deduplicated using latest timestamp. How would you implement it?
**A:**  
* **PySpark Window Row-Numbering**:
  ```python
  from pyspark.sql.window import Window
  from pyspark.sql.functions import col, row_number

  window = Window.partitionBy("id").orderBy(col("updated_at").desc())
  deduped = df.withColumn("rank", row_number().over(window)).filter("rank = 1").drop("rank")
  ```
* **Lakehouse Upsert (Delta)**: Delta `MERGE` has no precombine option, and it raises an error if several source rows match one target row. So dedupe the source first, then merge and ignore stale updates:
  ```python
  (DeltaTable.forPath(spark, path).alias("t")
     .merge(deduped.alias("s"), "t.id = s.id")
     .whenMatchedUpdateAll(condition="s.updated_at > t.updated_at")
     .whenNotMatchedInsertAll()
     .execute())
  ```
* **Hudi**: Set the ordering field (formerly called the precombine field; `hoodie.table.ordering.fields` in Hudi 1.x, `hoodie.datasource.write.precombine.field` in older versions) to `updated_at`, so the record with the latest value wins.
---

### Q55. A data pipeline must be highly available. What design principles would you use?
**A:**  
1. **Multi-AZ & Serverless Compute**: Deploy orchestrators (MWAA) and compute (AWS Glue / EMR) across multiple Availability Zones.
2. **Idempotency & Replayability**: Ensure all writes are idempotent so pipelines can be restarted from any failure point.
3. **Decoupled Buffering**: Ingest into durable distributed streams (Kafka/Kinesis) to survive downstream downtime.
4. **Automated Health Probes & Failover**: Configure automated health checks and failovers for databases and pipelines.

---

### Q56. A table must be optimized for analytical queries. How would you design the schema?
**A:**  
1. **Star Schema Dimensional Modeling**: A central fact table (numeric measures and foreign keys) surrounded by denormalized dimension tables.
2. **Columnar Format**: Parquet with Snappy or Zstd compression.
3. **Partitioning & Clustering**: Partition by date (low cardinality) and sort/Z-order or cluster by high-frequency filter columns.
4. **Surrogate Keys**: Integer surrogate keys for dimensions to keep joins compact and to support SCD2.
5. **Grain first**: Define the fact table grain explicitly, and keep measures additive where possible.
---

### Q57. A Spark job runs slowly due to wide transformations. How would you optimize it?
**A:**  
1. **Reduce data before the shuffle**: Filter rows and prune columns first. DataFrame aggregations combine map-side automatically; in RDD code use `reduceByKey`/`aggregateByKey` instead of `groupByKey`.
2. **Broadcast Hash Joins**: Avoid shuffles for small lookup tables.
3. **Tune Shuffle Parallelism**: Set `spark.sql.shuffle.partitions` to match data size and cluster cores (2 to 4 tasks per core) or rely on AQE.
4. **Handle skew**: AQE skew join, salting, or isolating hot keys.
5. **Avoid repeated wide operations**: Cache or persist a reused intermediate result, and drop redundant `distinct`/`sort` steps.
---

### Q58. You must track slowly changing dimensions. How would you implement SCD Type 1 vs Type 2?
**A:**  
* **SCD Type 1 (Overwrite)**: Overwrite existing attribute values directly using `MERGE INTO target USING staging ON target.id = staging.id WHEN MATCHED THEN UPDATE SET target.attr = staging.attr`. No historical tracking.
* **SCD Type 2 (History Tracking)**: Maintain active and historical rows with `start_date`, `end_date`, and `is_current_flag`. Expire existing matching active records and insert new active versions.

---

### Q59. How would you build a reusable ETL framework?
**A:**  
* **Modular Codebase**: Decouple logic into distinct modules: `extractors`, `transformers`, `validators`, `loaders`.
* **Configuration-Driven Execution**: Pass YAML/JSON config files containing source paths, transformation rules, target schemas, and validation criteria.
* **Standardized Logging & Error Handling**: Implement generic try-catch wrappers that emit structured metrics to centralized logging systems.

---

### Q60. A pipeline processes 10 million records per minute. What architecture would support it?
**A:**  
* **Sizing first**: 10 million per minute is about 167,000 records per second. On Kinesis (1 MB/s or 1,000 records/s per shard on writes) that needs roughly 170+ shards or on-demand mode; on Kafka, size partitions by throughput and consumer parallelism.
* **Ingestion**: **Apache Kafka (Amazon MSK)** or **Amazon Kinesis** with provisioned capacity and a well-distributed partition key.
* **Processing**: **Apache Flink** or **Spark Structured Streaming** on EMR/Managed Flink with checkpointing.
* **Storage**: Micro-batches committed into **Apache Iceberg / Delta Lake** on S3 as partitioned Parquet, with scheduled compaction to control small files.
* **Serving**: Real-time aggregates in **Redis / ClickHouse / DynamoDB** for sub-second dashboards.
---

## Questions 61 – 80

### Q61. Your job requires joining multiple large datasets. How would you optimize joins?
**A:**  
1. **Pre-Bucketing & Sorting**: Bucket both datasets on the join key (`df.write.bucketBy(...)`) so Spark performs sort-merge joins without shuffling.
2. **Adaptive Query Execution**: Enable AQE to coalesce partitions and resolve data skew dynamically.
3. **Filter Early**: Apply aggressive pushdown predicates before join operations.
4. **Avoid Multiple Shuffles**: Chain joins on the same partitioning key sequentially to reuse partition layouts.

---

### Q62. A dataset requires incremental processing only. How would you design it?
**A:**  
* **Use the table format's incremental read**:
  * Delta: `spark.read.format("delta").option("readChangeFeed","true").option("startingVersion", n)` (Change Data Feed must be enabled on the table).
  * Iceberg: incremental or changelog reads between two snapshot IDs.
  * Hudi: incremental query (`hoodie.datasource.query.type=incremental` with a begin instant time).
* **Plain files or databases (high-watermark)**: Filter `last_updated > last_successful_watermark` (use `>=` with a small overlap window and idempotent merge to catch late rows), and update the control table only after the batch commits successfully.
* **Streaming**: Structured Streaming file source with a checkpoint also gives incremental file processing. (Databricks Auto Loader is the Databricks-specific equivalent.)
---

### Q63. A Spark job must handle late-arriving data. How would you manage it?
**A:**  
1. **Event-Time Partitioning**: Use the source event timestamp to decide the target partition, not the arrival time.
2. **Lakehouse MERGE (preferred)**: Upsert late rows into the Delta/Hudi/Iceberg table by primary key; only the affected files are rewritten and existing rows are preserved.
3. **If using plain Parquet**: Read the affected partition, union the late rows, deduplicate, and then overwrite that partition. Do **not** write only the late rows with dynamic partition overwrite, because that replaces the whole partition with just those rows.
4. **Downstream aggregates**: Recompute only the impacted dates/windows, and track an "arrival lag" metric to tune how far back reprocessing must look.
---

### Q64. Your cluster experiences frequent executor failures. How would you debug it?
**A:**  
1. **Inspect Failure Messages in YARN / Spark UI**: "Container killed by YARN for exceeding memory limits" (exit code often 143), kernel OOM kills (exit 137), or node loss/decommission messages for spot reclaim. Use the diagnostic text, not just the exit code.
2. **Check GC Logs**: Long pauses can make the driver mark executors dead (`Heartbeat timed out`).
3. **Check Local Disk**: Verify shuffle spill did not fill executor local storage (EBS/instance store).
4. **Check data**: Skewed partitions or huge records cause single executors to blow up.
---

### Q65. A pipeline requires high throughput and low latency. What technologies would you use?
**A:**  
* **Ingestion**: Apache Kafka (MSK) for high-throughput distributed pub-sub.
* **Processing**: **Apache Flink** (event-at-a-time streaming with millisecond latency).
* **Storage / Serving**: **Amazon DynamoDB** or **ClickHouse** for low-latency point lookups and aggregations.

---

### Q66. Your pipeline needs to maintain historical snapshots. How would you implement it?
**A:**  
* **Modern Table Format Snapshots**: Use **Delta Lake** or **Apache Iceberg**, which automatically maintain immutable historical snapshots in their metadata transaction logs.
* **Partitioned Snapshot Backups**: Periodically write full partition snapshots to `/snapshots/snapshot_date=YYYY-MM-DD/` on S3.

---

### Q67. A job must detect anomalies in incoming data. How would you design it?
**A:**  
1. **Statistical Detection**: Compute the rolling mean and standard deviation over the **previous** N points or window (excluding the current record), and flag deviations in both directions:
   ```python
   from pyspark.sql import functions as F
   from pyspark.sql.window import Window

   w = Window.partitionBy("sensor_id").orderBy("event_ts").rowsBetween(-100, -1)
   scored = (df.withColumn("mean_val", F.avg("value").over(w))
               .withColumn("std_val",  F.stddev("value").over(w))
               .withColumn("is_anomaly",
                           F.abs(F.col("value") - F.col("mean_val")) > 3 * F.col("std_val")))
   ```
   For skewed or non-normal data use median absolute deviation (MAD) or IQR instead of mean/std.
2. **Also monitor pipeline-level anomalies**: row-count drops/spikes, null-rate jumps, and freshness.
3. **Routing**: Send anomalies to an SNS alert topic and a quarantine bucket for review.
---

### Q68. Your pipeline reads data from multiple APIs. How would you handle ingestion?
**A:**  
1. **Asynchronous Concurrent Ingestion**: Use Python `asyncio` / `aiohttp` or AWS Lambda workers in parallel to fetch data concurrently.
2. **Rate Limit Handling**: Implement token bucket rate limiting and exponential backoff with jitter.
3. **Staging**: Stage raw API JSON responses directly into S3 Landing before triggering Spark batch ETL.

---

### Q69. Your Spark job runs fine locally but fails in production. How would you debug it?
**A:**  
1. **Data Volume Discrepancy**: Production data volume is orders of magnitude larger, revealing memory bottlenecks, OOMs, and data skew not visible on small local test data.
2. **Environment & Dependencies**: Check for missing JARs, Spark version incompatibilities, or IAM permission boundaries.
3. **Resource Sizing**: Verify executor memory fractions and shuffle partition settings.

---

### Q70. How would you design a centralized logging system for data pipelines?
**A:**  
* **Structured Logs**: Emit JSON logs containing `timestamp`, `pipeline_id`, `task_id`, `batch_id`, `severity`, and `message`.
* **Aggregation**: Forward logs via CloudWatch Agent or FluentBit into **Amazon CloudWatch** or **Amazon OpenSearch**.
* **Dashboards & Alerts**: Create dashboards tracking error rates and configure metric filters to trigger PagerDuty alerts on critical exceptions.

---

### Q71. A dataset must support machine learning workloads. How would you design storage?
**A:**  
* **Format**: Parquet (Snappy or Zstd) for fast columnar scans from Spark, pandas, and PyTorch data loaders.
* **Feature Store Integration**: **Amazon SageMaker Feature Store** (managed low-latency online store plus an offline store in S3), or an equivalent such as Feast.
* **Versioning**: Delta/Iceberg time travel or dataset snapshots to reproduce exact training datasets.
* **Point-in-time correctness**: Join features using event timestamps to avoid label leakage.
---

### Q72. You need to handle millions of events per second. What architecture would you build?
**A:**  
* **Ingestion**: **Apache Kafka (Amazon MSK)** with many partitions across high-throughput brokers (or Kinesis in on-demand/large provisioned mode), with a well-distributed key.
* **Compute**: **Apache Flink** (Amazon Managed Service for Apache Flink or on EMR) for stateful, low-latency stream processing.
* **Sink**: **Apache Iceberg** tables on S3 with tuned commit intervals and scheduled compaction; real-time metrics to ClickHouse, OpenSearch, or Timestream for InfluxDB.
* **Operations**: Monitor consumer lag, backpressure, checkpoint duration, and hot partitions.
---

### Q73. Your Spark job frequently runs out of memory. What tuning steps would you take?
**A:**  
1. **Find the cause first**: Check for skew (a few huge tasks), huge broadcasts, `collect()` on large data, and oversized partitions.
2. Increase `spark.sql.shuffle.partitions` (or rely on AQE) to shrink per-task data.
3. Increase `spark.executor.memory` and `spark.executor.memoryOverhead`. For PySpark and UDF-heavy jobs the Python workers live in the overhead memory, so also consider `spark.executor.pyspark.memory`.
4. Resolve skew with AQE skew handling or key salting.
5. Adjust `spark.memory.fraction` / `spark.memory.storageFraction` only if the Spark UI shows storage or execution memory pressure.
6. Use Kryo serialization for RDD-heavy jobs, and unpersist caches you no longer need.
---

### Q74. A dataset contains nested arrays and structs. How would you flatten them?
**A:**  
* **Recursive flattening helper**:
  ```python
  from pyspark.sql import functions as F
  from pyspark.sql.types import StructType, ArrayType

  def flatten(df):
      while True:
          complex_cols = [(f.name, f.dataType) for f in df.schema.fields
                          if isinstance(f.dataType, (StructType, ArrayType))]
          if not complex_cols:
              return df
          name, dtype = complex_cols[0]
          if isinstance(dtype, StructType):
              expanded = [F.col(f"{name}.{c}").alias(f"{name}_{c}") for c in dtype.names]
              df = df.select("*", *expanded).drop(name)
          else:
              df = df.withColumn(name, F.explode_outer(name))
          # explode_outer keeps rows with null or empty arrays
  ```
* **Targeted version** for a known schema:
  ```python
  df_exploded = df.withColumn("phone_record", F.explode_outer("contact_numbers"))
  df_flat = df_exploded.select(
      "user_id", "user_name",
      F.col("phone_record.type").alias("phone_type"),
      F.col("phone_record.number").alias("phone_number"))
  ```
* Exploding multiple arrays multiplies rows (a cartesian product per record), so explode one at a time and aggregate where needed.
---

### Q75. A data lake requires multiple processing layers. How would you design medallion architecture?
**A:**  
* **Bronze (Raw Zone)**: Immutable, append-only raw data as received from source systems with metadata timestamps.
* **Silver (Standardized Zone)**: Conformed, typed, deduplicated, and cleansed Lakehouse tables (Delta Lake/Hudi) with schema enforcement.
* **Gold (Curated Zone)**: Aggregated business data marts, star schemas, and Materialized Views ready for BI reporting and analytics.

---

### Q76. How would you implement incremental ingestion from relational databases?
**A:**  
* **Method 1: Log-Based CDC (Recommended)**: Use AWS DMS or Debezium to stream row-level change logs into S3/Kafka.
* **Method 2: High-Watermark Query**:
  ```sql
  SELECT * FROM source_table WHERE last_updated > :previous_watermark_timestamp;
  ```
  *Save new max timestamp into the control table upon successful commit.*

---

### Q77. Your ETL pipeline must support backfills. How would you design it?
**A:**  
1. **Parameterized Execution Dates**: Jobs accept explicit `start_date`/`end_date` (or the Airflow data interval) instead of using `current_date()`.
2. **Idempotent Writes**: Lakehouse `MERGE INTO`, or full-partition overwrite, so reruns replace data cleanly.
3. **Run in chunks** (by day or month), with limits on concurrency (`max_active_runs`, pools) so production is not starved.
4. **Airflow**: Use the backfill feature (Airflow 2.x CLI: `airflow dags backfill -s 2026-01-01 -e 2026-01-31 pipeline_dag`; Airflow 3 provides backfills in the UI/API and CLI). `catchup=True` is the scheduler-driven alternative.
5. Validate each chunk (counts/checksums) before moving on.
---

### Q78. A dataset contains inconsistent timestamps. How would you standardize them?
**A:**  
* **Parse every known format, then standardize to UTC**:
  ```python
  from pyspark.sql import functions as F

  parsed = F.coalesce(
      F.to_timestamp("raw_date", "yyyy-MM-dd HH:mm:ss"),
      F.to_timestamp("raw_date", "dd/MM/yyyy HH:mm"),
      F.to_timestamp("raw_date", "MM-dd-yyyy"),
  )
  standardized_df = (df.withColumn("local_ts", parsed)
                       # source timezone comes from data or config, not a hard-coded guess
                       .withColumn("clean_timestamp_utc",
                                   F.to_utc_timestamp("local_ts", F.col("source_tz"))))
  ```
* `to_utc_timestamp(ts, tz)` treats `ts` as local time in `tz`, so supply the right source zone (use zone names such as `Asia/Kolkata`, never fixed offsets).
* Route rows that match no format (`parsed IS NULL`) to a quarantine table.
* Keep the original raw value for audit.
---

### Q79. A pipeline needs automated schema detection. How would you implement it?
**A:**  
* **AWS Glue Crawlers**: Schedule crawlers on S3 prefixes to infer schemas and register/update tables and partitions in the Glue Data Catalog.
* **Spark inference**: Infer from a sample (`samplingRatio`), then store the schema and enforce it on later reads instead of re-inferring.
* **Registry and contracts**: Combine with Glue Schema Registry (streaming) and compare detected schema with the registered one; alert or quarantine on breaking changes.
* **Databricks only**: Auto Loader with `cloudFiles.schemaLocation` infers and tracks evolving schemas.
---

### Q80. A job requires sorting billions of records. What strategies would you use?
**A:**  
1. **Range Partitioning**: Use `df.repartitionByRange(num_partitions, "sort_key")` to distribute sorted ranges across executors.
2. **External Sorting in Spark**: Spark automatically uses Tungsten memory-optimized external sort-merge, spilling sorted runs to NVMe disk if memory is exceeded.
3. **Avoid Total Sort if Not Needed**: Use `df.sortWithinPartitions("sort_key")` to sort data locally within partitions without a full global shuffle.

---

## Questions 81 – 100

### Q81. Your Spark job reads from Kafka. How would you ensure fault tolerance?
**A:**  
1. **Enable Checkpointing**: Configure `checkpointLocation` on Amazon S3 in Spark Structured Streaming to commit processed Kafka topic offsets.
2. **Idempotent Sinks**: Write to Lakehouse formats (Delta Lake / Hudi) supporting atomic commits.
3. **Fail-Safe Processing**: If the Spark job crashes, restarting it causes Spark to read the last committed offset from the S3 checkpoint and resume without data loss.

---

### Q82. A pipeline must handle duplicate streaming events. How would you deduplicate them?
**A:**  
* **Streaming Deduplication with Watermarking**:
  ```python
  streaming_df.withWatermark("event_timestamp", "1 hour") \
      .dropDuplicates(["event_id", "event_timestamp"])
  ```
  The deduplication key must match your business definition of a duplicate. Including `event_timestamp` is required for the watermark to clean up state, and it works only when duplicates carry the **same** event timestamp. If a retry can change the timestamp, dedupe on `event_id` alone and accept a larger state (or bound it with a TTL).
* **Storage Deduplication**: Merge micro-batches into Delta/Hudi/Iceberg by primary key in `foreachBatch`, after deduplicating inside the batch.
---

### Q83. A dataset contains extremely large partitions. How would you rebalance them?
**A:**  
1. **`df.repartition(N)`**: Round-robin shuffle into N even partitions (target about 128 MB each).
2. **Repartition by a higher-cardinality or composite key** (for example `customer_id` plus date) when downstream operations need co-location. If one key value is itself huge, a composite key containing it will not help.
3. **Salting**: Add a salt to the hot key to spread it across several tasks, then aggregate back.
4. **AQE**: Enable it so Spark coalesces small and splits skewed shuffle partitions at runtime; `spark.sql.adaptive.advisoryPartitionSizeInBytes` (default 64 MB) sets the target size.
5. For file output, use `maxRecordsPerFile` or repartition before write to cap file size.
---

### Q84. Your data team needs governance and access control. How would you implement it?
**A:**  
* **AWS Lake Formation**: Centralize access permissions to Glue Data Catalog databases, tables, columns, and rows.
* **IAM Least Privilege Roles**: Assign distinct IAM roles for Data Analysts (read-only curated layer), Data Scientists (read-only silver/gold), and Data Engineers (write/admin access).
* **Audit Tracking**: Enable **AWS CloudTrail** and Lake Formation audit logs to monitor all data access requests.

---

### Q85. A pipeline processes financial transactions. How would you ensure data accuracy?
**A:**  
1. **Reconciliation**: Compare control totals (count and sum per batch and per day) with the source ledger, and verify debits equal credits for double-entry data.
2. **Data Types**: Use exact `DecimalType` sized for the currency (not float/double). Some FX and crypto data needs more than 2 decimal places.
3. **Validation Gatekeeper**: Reject and quarantine missing timestamps, invalid accounts, and invalid currencies. Model transaction type and sign explicitly (refunds, reversals, and credits are legitimate negative or opposite-direction entries), and validate against that.
4. **Idempotency**: Uniqueness constraints are informational only in Redshift/Snowflake and do not exist on S3 tables, so enforce uniqueness in the pipeline: dedupe by transaction ID and use an idempotent `MERGE`.
5. **Audit trail**: Immutable raw data, lineage, and logs of every correction.
---

### Q86. Your ETL job requires dynamic configuration. How would you design it?
**A:**  
* Store configurations in **AWS Systems Manager Parameter Store** or **Amazon DynamoDB**.
* The PySpark script fetches config parameters dynamically at startup, avoiding hardcoded SQL expressions, S3 bucket names, or table paths.

---

### Q87. A dataset must support multi-tenant access. How would you design security?
**A:**  
1. **Isolation model by scale**:
   * Few tenants: separate S3 prefixes or tables per tenant, and a Lake Formation data filter (row filter) per tenant granted to that tenant's role.
   * Many tenants: Redshift row-level security policies keyed on the session user/role, or IAM attribute-based access control (`aws:PrincipalTag`) on S3 access points / bucket policies.
   * Strict isolation or compliance needs: separate accounts or encryption keys per tenant.
2. **Partitioning**: Do not partition by `tenant_id` when there are thousands of tenants (small-file explosion); use clustering or bucketing instead.
3. **Encryption**: Tenant-specific KMS keys when contractually required.
4. **Audit**: CloudTrail and Lake Formation access logs, plus regular tests that one tenant cannot read another's data.
---

### Q88. A Spark job must join structured and semi-structured data. How would you do it?
**A:**  
1. Ingest structured table as standard DataFrame.
2. Ingest semi-structured JSON using `schema_of_json()` or parse structs into relational columns using `from_json()`.
3. Join on common relational keys using Broadcast Hash Join if the dimension is small.

---

### Q89. A data warehouse requires partition pruning. How would you implement it?
**A:**  
* **External tables (Athena, Redshift Spectrum, Iceberg)**: Partition on the filter column (e.g., `event_date`), filter directly on that raw column (no functions wrapped around it): `WHERE event_date BETWEEN '2026-08-01' AND '2026-08-30'`.
* **Native Redshift tables** have no partitions; pruning happens through **sort keys** and zone maps, so choose sort keys on the common range-filter columns.
* **Verify** with `EXPLAIN` (or the scan statistics/bytes scanned) that non-matching partitions or blocks are skipped.
---

### Q90. Your pipeline needs automatic scaling. What technologies would you use?
**A:**  
* **AWS Glue Autoscaling**: Set `--enable-auto-scaling=true` to allow Glue to dynamically add workers during compute-intensive shuffles and remove workers when idle.
* **Amazon EMR Managed Scaling**: Automatically scales cluster core and task EC2 nodes based on YARN memory and CPU metrics.

---

### Q91. How would you design a high-performance feature store for ML?
**A:**  
* **Dual-Store Architecture**:
  * **Offline Store (source of truth)**: S3 with Delta/Iceberg/Parquet storing point-in-time feature history for training.
  * **Online Store (low latency)**: DynamoDB, Redis, or SageMaker Feature Store's online store serving real-time lookups (single-digit to low double-digit milliseconds).
  * **Materialization**: Feature pipelines compute once, write to the offline store, and materialize the latest values to the online store, which avoids drift between the two.
* **Point-in-time joins** for training sets to prevent leakage, and the same transformation code for training and serving.
---

### Q92. Your Spark job must process compressed files. What considerations are needed?
**A:**  
* **Splittability**:
  * GZIP: not splittable (one large file is processed by one task).
  * BZIP2: splittable, but slow and CPU-heavy.
  * Raw Snappy, LZ4, and Zstd files: not splittable on their own.
  * Inside **Parquet/ORC/Avro**, Snappy/Zstd/GZIP compression is applied per block or page, so the file remains splittable.
* **Best Practice**: Store data as Parquet with Snappy (speed) or Zstd (ratio). If you receive large GZIP files, split them into many moderate files or convert them to Parquet once at ingestion.
---

### Q93. A dataset must support real-time fraud detection. What architecture would you design?
**A:**  
1. Ingest transactions via **Amazon Kinesis Data Streams**.
2. Process with **Apache Flink / Spark Streaming** to calculate real-time window metrics (e.g., number of transactions from same card across different cities in 5 minutes).
3. Query feature store (**DynamoDB**) and execute ML fraud model inference via AWS Lambda / SageMaker endpoint.
4. Flag fraudulent transactions to Amazon SNS topic within sub-second SLAs.

---

### Q94. Your data pipeline requires strong data quality checks. How would you implement them?
**A:**  
1. **Pre-Ingestion Validation**: Schema validation and null checks.
2. **In-Flight DQ Assertions**: Great Expectations / AWS Glue Data Quality executing business assertion rules.
3. **Post-Load Reconciliation**: Row count, checksum, and statistical distribution comparisons against source systems.
4. **Automated Quarantine**: Route invalid records to DLQ buckets with alerting.

---

### Q95. A dataset must support fast search queries. What storage solutions would you consider?
**A:**  
* **Amazon OpenSearch Service**: For full-text search, multi-field filtering, and log analytics.
* **Amazon Athena + Parquet**: For large-scale ad-hoc analytical SQL search queries on S3.
* **Amazon DynamoDB with Global Secondary Indexes (GSIs)**: For sub-millisecond point search lookups on specific keys.

---

### Q96. Your pipeline must maintain audit history for compliance. How would you implement it?
**A:**  
* **S3 Versioning & Object Lock (WORM)**: Object Lock in *compliance* mode cannot be bypassed (governance mode can be bypassed by privileged users), so choose the mode your regulation requires.
* **Lakehouse Commit History**: Delta/Iceberg/Hudi snapshots give an audit trail, but `VACUUM` and `expire_snapshots` remove old history, so align their retention with the compliance period.
* **Pipeline Audit Log Table**: Record every batch (source, counts, status, code version).
* **Access auditing**: CloudTrail data events and Lake Formation logs.
---

### Q97. A streaming pipeline must recover from failures automatically. How would you design it?
**A:**  
1. **Retention longer than your worst-case outage**: Kafka defaults to 7 days; Kinesis defaults to 24 hours (extendable up to 365 days).
2. **Checkpointing**: Spark Structured Streaming / Flink checkpoints on durable storage (S3).
3. **Auto-restart**: Run on Kubernetes, EMR, or Managed Flink with restart policies and alarms on repeated failures.
4. **Idempotent Destination Writes**: Replaying an uncommitted micro-batch must not duplicate data (atomic table-format commits or deterministic MERGE).
5. **Lag alarms**: Alert on consumer lag/iterator age so slow recovery is noticed.
---

### Q98. Your Spark job must read from multiple partitions simultaneously. How would you optimize it?
**A:**  
* **Speed up partition discovery**: Keep `spark.sql.sources.parallelPartitionDiscovery.threshold` at a suitable value (default 32) so large listings run in parallel.
* **Avoid S3 listing altogether**: Use Glue Catalog partition indexes or Athena partition projection, and table formats (Iceberg/Delta/Hudi) that read file lists from metadata instead of S3 `LIST` calls.
* **Prune**: Filter on partition columns so only needed partitions are listed and read.
* **Keep the partition count manageable** (see the partitioning strategy in Q20).
---

### Q99. A pipeline must process historical and real-time data together. How would you design it?
**A:**  
* **Lambda vs Kappa Architecture**:
  * Implement **Kappa Architecture**: Stream all real-time events through Kafka into an S3 Lakehouse table.
  * Historical backfills and real-time events write to the same Delta Lake / Iceberg table using unified PySpark code, allowing queries to seamlessly join real-time and historical partitions.

---

### Q100. Explain an end-to-end data engineering project you built and the challenges you solved.
**A:**  
Use a real project of your own. Structure it as **context, architecture, challenges, result**, and only quote numbers you can defend. Example structure (replace the bracketed parts with your own facts):

"I built a CDC-based Lakehouse pipeline on AWS for [business area].
* **Architecture**: AWS DMS captures changes from [source database] into S3. AWS Glue (PySpark) jobs apply `INSERT`/`UPDATE`/`DELETE` records as SCD Type 1 upserts and deletes into Apache Hudi tables on S3, orchestrated by Apache Airflow. Curated data is queried through Athena and Redshift Spectrum.
* **Challenges and solutions**:
  1. *Small files*: [what I observed], solved with [compaction/clustering and write-size tuning].
  2. *Data skew*: [which keys], solved with [salting/AQE].
  3. *Duplicates and ordering*: solved with idempotent upserts using an ordering field on the change timestamp.
  4. *Cost/performance*: [what I changed] which gave [measured improvement and how I measured it].
* **Result**: [data volume, latency/SLA, reliability, and business impact]."

Be ready for follow-ups: how do you handle late or out-of-order CDC events, schema changes, reprocessing, and monitoring?

---

## Questions 101 – 137

### Q101. Your Glue job reprocesses the same files every run. Why, and how do you fix it?
**Answer:**
- **Cause:** job bookmarks are off, or they are on but not wired correctly. Bookmarks need a `transformation_ctx` on each source, `job.init(args["JOB_NAME"], args)` at the start and `job.commit()` at the end. They do **not** apply when you read with plain `spark.read` (only DynamicFrame reads).
- For S3, bookmarks track files by last-modified time. Files overwritten in place get a new timestamp and are re-read.
- JDBC sources need bookmark keys that are sorted and monotonically increasing.
- **Fixes:** enable `--job-bookmark-option job-bookmark-enable`, add `transformation_ctx`, write new files with new names, and make the load **idempotent anyway** (MERGE/partition overwrite) so a bookmark reset or retry is harmless. Reset deliberately with `aws glue reset-job-bookmark --job-name <job>` for a backfill.

### Q102. DMS delivers CDC files to S3. How do you apply them correctly (ordering, duplicates, deletes)?
**Answer:**
1. Configure the DMS S3 target to include the operation column (`Op`: I/U/D) and a **timestamp column** (`TimestampColumnName`). Set `IncludeOpForFullLoad` so full-load rows are marked as inserts.
2. Full load files land first, then CDC files. Process both with the same logic.
3. **Deduplicate per key:** keep the latest change per primary key within the batch (`row_number` over key, order by change timestamp desc, then a tiebreaker such as file order or transaction ID).
4. **Apply with MERGE:** `WHEN MATCHED AND Op='D' THEN DELETE` (or soft delete), `WHEN MATCHED THEN UPDATE`, `WHEN NOT MATCHED AND Op != 'D' THEN INSERT`.
5. DMS can emit duplicates after a task restart, so the merge must be idempotent. Reject stale events by comparing the change timestamp with the target's last-applied timestamp.
6. Watch for large objects (LOB settings), DDL changes, and keep the DMS task's latency metrics under alarm.

### Q103. Hudi Copy-on-Write or Merge-on-Read? How do you choose?
**Answer:**
| | COW | MOR |
|---|---|---|
| Write | Rewrites base Parquet files on update (higher write cost) | Appends updates to log files (fast writes) |
| Read | Fast, plain Parquet | Snapshot queries merge logs on read (slower), read-optimized queries skip logs (stale) |
| Maintenance | None for merging | Needs compaction |
| Fits | Read-heavy, batch updates, simple consumers | Frequent small updates, near-real-time ingestion |

Check engine support before choosing. Athena supports both; Redshift Spectrum support has historically centered on COW, so verify the current matrix for MOR. For SCD1 on a nightly CDC load with Spectrum consumers, COW is the simple, safe choice.

### Q104. Redshift Spectrum queries on S3 are slow. What do you check?
**Answer:**
- **Scanned bytes vs returned bytes** (`SVL_S3QUERY_SUMMARY`): high scan means partition pruning or columnar pruning is not working.
- Is the table **partitioned** on the filtered column, and does the query filter on the raw partition column (no functions wrapped around it)?
- File format and size: Parquet/ORC, files at least ~128 MB, not thousands of KB files. Compact if needed.
- Catalog partition count: too many partitions slows planning. Use partition indexes or projection where available.
- Statistics: set table row-count properties for external tables so the planner joins sensibly.
- Join large external tables to small local dimension tables, not external-to-external.
- Keep hot, frequently joined data in local Redshift tables. Use Spectrum for cold or large history.

### Q105. Redshift queries got slow after a big load. What do you check?
**Answer:**
1. `SVV_TABLE_INFO`: `stats_off` (stale statistics), `unsorted` (percent unsorted region), `skew_rows` (distribution skew), `tbl_rows`.
2. Run `ANALYZE` and `VACUUM` (sort) where auto maintenance has not caught up.
3. Check **queueing**: WLM/concurrency-scaling metrics, long queries blocking short ones.
4. Look at the plan (`EXPLAIN`) for broadcast/redistribution steps (`DS_BCAST_INNER`, `DS_DIST_BOTH`). Fix with a better `DISTKEY`/`DISTSTYLE`.
5. Check for disk-based (spilled) steps and for a skewed `DISTKEY`.
6. Compare with the last good run of the query (query history) to see what changed.

### Q106. A Redshift `COPY` from S3 fails or loads partial data. How do you troubleshoot?
**Answer:**
- Read the error tables: `SYS_LOAD_ERROR_DETAIL` (or `STL_LOAD_ERRORS` on provisioned clusters) show file, line, column, and reason.
- Common causes: delimiter/quote issues, type mismatch, column count mismatch, string longer than the column, bad date format, IAM role missing S3/KMS permissions.
- Use `MAXERROR` carefully, and prefer loading to a staging table to validate before merging.
- For speed: split input into multiple files (a multiple of the number of slices), compress, use a manifest to control exactly which files load.
- Make the load idempotent: load into staging, then delete+insert or MERGE, inside one transaction.

### Q107. Upstream renames a column or changes a column type without telling you. How do you handle it?
**Answer:**
- **Detect:** schema validation at ingestion, schema registry compatibility checks, or a data contract (Q117). Fail or quarantine the batch instead of silently writing nulls.
- **Rename:** Parquet reads columns by name, so a rename looks like a dropped column plus a new one. Iceberg tracks columns by ID, so renames are metadata-only. Delta needs column mapping enabled. Hudi has limited rename support.
- **Type change:** widening (int to long, float to double) is usually safe. Narrowing or int to string is breaking.
- **Safe migration:** add a new column, populate both for a period, backfill history, switch consumers, then drop the old column.
- Keep raw bronze untouched so you can replay after fixing the mapping.

### Q108. The source sends a full snapshot every day. How do you detect inserts, updates, and deletes?
**Answer:**
```python
curr = spark.table("silver.customers").filter("is_deleted = false")
inserts = snapshot.join(curr, "id", "left_anti")
deletes = curr.join(snapshot, "id", "left_anti")
# updates: same id, different row hash
h = lambda d: d.withColumn("row_hash", F.sha2(F.concat_ws("||", *cols), 256))
updates = h(snapshot).alias("s").join(h(curr).alias("c"), "id") \
            .filter("s.row_hash <> c.row_hash")
```
Apply with MERGE. Mark deletes as soft deletes. **Safeguard:** if the snapshot's row count drops by more than a set threshold (or is empty), stop and alert. A partial or failed extract would otherwise delete everything.

### Q109. You must backfill one year of history without hurting production. How?
**Answer:**
- Run in **chunks** (by month or day partition), not one huge job. Parameterize start/end dates.
- Use **idempotent writes** (partition overwrite with full partition contents, or MERGE) so a failed chunk can be rerun.
- Isolate resources: separate Airflow pool and `max_active_runs`, a separate Glue job/cluster, and throttle reads on the source database or API.
- Write to a **shadow table/location**, validate counts and checksums per chunk, then swap or merge into the live table.
- Pause or coordinate with the daily incremental job so they do not write the same partitions concurrently.
- Estimate cost and runtime from one sample chunk first.

### Q110. Your Glue bill doubled. How do you cut cost without hurting SLAs?
**Answer:**
1. Find the expensive jobs (DPU-hours per job in Cost Explorer/CloudWatch).
2. Right-size: worker type and count. Enable **Auto Scaling**. Do not over-provision a job that idles.
3. Use the **Flex execution class** for non-urgent jobs.
4. Process incrementally (bookmarks/watermarks) instead of full refresh.
5. Fix small files and skew, which burn DPU hours. Use Parquet, partition pruning, and predicate pushdown.
6. Use a recent Glue version (performance improvements) and avoid unnecessary `count()`/`collect()` actions.
7. Move trivial transforms to Athena/Redshift SQL or a Python shell/Lambda job.
8. Add cost alarms and per-job budgets via tags.

### Q111. How do you prove the target matches the source after a migration or pipeline run?
**Answer:**
- **Layered reconciliation:** (1) row counts per partition/day, (2) sum/min/max of numeric and date columns, (3) null counts of key columns, (4) distinct key counts, (5) row-level hash comparison for a sample or the full set (hash of concatenated columns, compared with an anti-join or aggregated per bucket).
- Compare like with like: freeze the source with a snapshot or a cutoff timestamp, and allow a tolerance window for in-flight CDC.
- Automate it as a task after every load, and write results (source count, target count, difference, status) to an audit table with alerts on mismatch.
- Investigate differences by narrowing: partition, then key range, then row.

### Q112. The pipeline is green, but the business says the dashboard numbers are wrong. What do you do?
**Answer:**
1. **Clarify:** which metric, which dates, what is expected, since when. Get a concrete failing example.
2. **Trace backward** with lineage: dashboard query, gold table, silver, bronze, source. Compare counts and values at each hop to find where it diverges.
3. **Common causes:** join fan-out (duplicates), late data not included, timezone/date boundary, a filter or dedupe rule change, a stale or partial upstream load, schema change producing nulls, a double-counted backfill.
4. Use **time travel** (Delta/Iceberg) to compare today's table with yesterday's.
5. **Fix and repair:** correct the logic, backfill affected partitions idempotently, and tell stakeholders what was wrong and the corrected date range.
6. **Prevent:** add the missing test (reconciliation, uniqueness, freshness, volume anomaly) so the same failure turns the pipeline red next time.

### Q113. A Lambda function must process 5 GB files, but Lambda has limits. What are your options?
**Answer:**
- Lambda limits: 15-minute timeout, up to 10 GB memory, and limited ephemeral `/tmp` storage. So do not load whole files.
- **Stream** the object in ranges/chunks (S3 range GETs), process incrementally, write outputs in parts.
- **Fan out:** an orchestrator splits the file by byte ranges or lines, and Step Functions Distributed Map runs many Lambdas in parallel.
- For heavy transforms, hand off to **Glue, EMR Serverless, or ECS/Fargate** and use Lambda only for triggering.
- Always make the function idempotent, since S3 events can be delivered more than once.

### Q114. Two jobs write to the same lakehouse table at the same time. What happens and how do you design for it?
**Answer:**
- Table formats use **optimistic concurrency**: a writer commits only if no conflicting commit happened since it started; otherwise it fails or retries. Writes to different partitions or files usually do not conflict. Overlapping updates to the same files do.
- **Hudi:** enable OCC with a lock provider (DynamoDB, Hive Metastore, or ZooKeeper).
- **Delta on S3:** multi-cluster writes historically needed a DynamoDB-backed log store. Newer versions can use S3 conditional writes, so verify your version.
- **Iceberg:** the catalog performs an atomic commit (Glue Catalog supports this), with retries on conflict.
- **Design choices:** separate writers by partition, serialize conflicting jobs in Airflow (pools, `max_active_runs=1`), retry on commit conflict, and keep transactions small.

### Q115. A user asks you to delete their data (GDPR "right to erasure"). How do you do it in a data lake?
**Answer:**
1. Locate all copies: bronze, silver, gold, derived tables, extracts, caches, and backups (lineage helps).
2. Delete rows with a table-format `DELETE`/MERGE (Delta, Iceberg, Hudi), keyed by a user identifier.
3. Deletes are logical until old files are removed: run `VACUUM` (Delta), `expire_snapshots` + `remove_orphan_files` (Iceberg), or cleaning/compaction (Hudi). Time travel will otherwise still expose the data.
4. Raw immutable layers: rewrite the affected files, or use **crypto-shredding** (encrypt per-user data with a per-user key and delete the key).
5. Run deletions in batches, log the request and completion for audit, and define a deadline-based SLA.

### Q116. Another AWS account needs read access to your curated tables. How do you share securely?
**Answer:**
- **Lake Formation cross-account sharing:** grant table or column permissions to the other account (via AWS RAM). The consumer creates a resource link in its own catalog and queries via Athena/Redshift/EMR with its own IAM roles.
- Alternative: S3 bucket policy plus Glue catalog resource policy (more manual and harder to audit).
- For warehouse data: Redshift **data sharing** avoids copying.
- Encrypt with KMS and grant the consumer account key access. Share only the curated layer, with column-level restriction for PII, and log access via CloudTrail.

### Q117. Upstream teams keep breaking your pipelines with unannounced changes. What do you propose?
**Answer:** **Data contracts.**
- A versioned agreement between producer and consumer covering schema (types, nullability), semantics (units, allowed values), freshness/volume SLAs, ownership, and the change process.
- Enforce in CI: producers' schema changes are checked against registered contracts (schema registry compatibility). Breaking changes require a new version and a deprecation window.
- Enforce at runtime: ingestion validates against the contract, quarantines violations, and alerts the owning team.
- Give every dataset an owner and an on-call route.

### Q118. You must load 200 tables with the same logic in Airflow. How do you build the DAG?
**Answer:**
- Do not copy-paste tasks. Use a **metadata/config table or YAML** and **dynamic task mapping** (`.expand()`, Airflow 2.3+) or a DAG factory.
- Keep each task **idempotent** and parameterized by table and date.
- Control load with pools and `max_active_tasks`. Group tables by priority or domain (TaskGroups).
- Avoid heavy code at DAG top level (parse time!). Do not call databases or APIs when the file is parsed. Read config cheaply or in a task.
- Pass small values via XCom. Put large data in S3 and pass the path.
- Use deferrable sensors/operators for long waits.

### Q119. Airflow tasks stay in "queued" for a long time. What do you check?
**Answer:**
1. **Capacity limits:** `parallelism`, `max_active_tasks_per_dag`, `max_active_runs`, pool slots exhausted (including slots held by sensors in `poke` mode).
2. **Workers:** Celery/Kubernetes workers down, at `worker_concurrency`, or autoscaling lagging. On MWAA, check environment class and min/max workers.
3. **Scheduler health:** heartbeat, parse time (slow DAG files), too many DAGs.
4. Queue mismatch: task assigned to a queue no worker listens to.
5. Fixes: sensors in `reschedule` mode or deferrable, right-size pools, scale workers, simplify top-level DAG code.

### Q120. A join returns far fewer rows than expected. What do you check?
**Answer:**
- **Key mismatch:** type difference (string vs int, implicit casts), leading/trailing spaces, case, zero-padding, hidden characters. Fix with explicit casts, `trim`, `lower`.
- **Nulls:** `NULL = NULL` is not true. Use null-safe equality (`<=>` / `eqNullSafe`) if appropriate.
- **Join type:** inner instead of left.
- **Filters in the wrong place:** a `WHERE` on the right table of a left join turns it into an inner join. Put it in the `ON` clause.
- Diagnose with a `left_anti` join to list unmatched keys, and compare distinct key counts on both sides.

### Q121. The same Spark job gives different results on different runs. Why?
**Answer:** Non-determinism sources:
- `row_number()`/`dropDuplicates`/`first()` on ties without a tiebreaker column.
- `rand()` without a seed, `monotonically_increasing_id()` assumptions, `collect_list` order.
- Reading a folder that is changing while the job runs (new files arriving).
- Non-deterministic UDFs, floating-point summation order, time-dependent functions (`current_timestamp`).
- **Fix:** add deterministic ordering and tiebreakers, set seeds, read a fixed snapshot (table version or explicit file list), and use `Decimal` for money.

### Q122. A Python UDF makes the job 10x slower. What do you do?
**Answer:**
- Python UDFs move data JVM to Python row by row (serialization overhead) and block Catalyst optimizations.
- **Preference order:** built-in Spark SQL functions, then SQL expressions (`F.expr`, `when/otherwise`), then **pandas UDFs** (Arrow-based, vectorized), then a Scala/Java UDF, and a plain Python UDF last.
- Filter and prune columns before the UDF, and avoid calling it on skewed or duplicate values (compute on distinct values and join back).

### Q123. A broadcast join fails with a timeout or driver OOM. What now?
**Answer:**
- A broadcast table is collected to the driver and shipped to executors. If it is bigger than expected after decompression, it can exhaust memory or exceed `spark.sql.broadcastTimeout`.
- Check the real in-memory size (Parquet compresses heavily), filter/select the small table first, raise driver memory and the timeout only if justified, or disable broadcast for that join (`spark.sql.autoBroadcastJoinThreshold=-1` or a `MERGE`/`SHUFFLE_HASH` hint).
- Prefer letting AQE choose based on runtime statistics.

### Q124. A pandas script in a Glue Python shell job runs out of memory. What are your options?
**Answer:**
- Process in **chunks** (`chunksize`), select only needed columns (`usecols`), set compact dtypes (`category`, `int32`), and write Parquet in parts.
- Switch to a lazy or out-of-core engine: **Polars** (lazy), **DuckDB**, or **Dask**.
- If data is truly large or growing, move to Glue Spark / EMR Serverless.
- Avoid copies (`inplace`-style chains create copies), delete intermediates, and read Parquet instead of CSV.

### Q125. SQL: assign session IDs to clickstream events using a 30-minute inactivity gap.
**Answer:**
```sql
WITH ordered AS (
  SELECT user_id, event_ts,
         LAG(event_ts) OVER (PARTITION BY user_id ORDER BY event_ts) AS prev_ts
  FROM events
),
flagged AS (
  SELECT user_id, event_ts,
         CASE WHEN prev_ts IS NULL
                OR DATEDIFF(minute, prev_ts, event_ts) > 30
              THEN 1 ELSE 0 END AS new_session
  FROM ordered
)
SELECT user_id, event_ts,
       SUM(new_session) OVER (PARTITION BY user_id ORDER BY event_ts
                              ROWS UNBOUNDED PRECEDING) AS session_id
FROM flagged;
```
`DATEDIFF(minute, ...)` is Redshift/Snowflake syntax; in Spark use `(unix_timestamp(event_ts) - unix_timestamp(prev_ts)) / 60`.

### Q126. SQL: find users with 3 or more consecutive login days (gaps and islands).
**Answer:**
```sql
WITH d AS (
  SELECT DISTINCT user_id, CAST(login_ts AS DATE) AS login_date FROM logins
),
g AS (
  SELECT user_id, login_date,
         login_date - CAST(ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS INT) AS grp
  FROM d
)
SELECT user_id, MIN(login_date) AS streak_start, MAX(login_date) AS streak_end, COUNT(*) AS days
FROM g
GROUP BY user_id, grp
HAVING COUNT(*) >= 3;
```
Idea: within a consecutive run, `date - row_number` is constant. Adjust the date arithmetic to your engine (`DATEADD`, `date_sub`).

### Q127. A fact row arrives before its dimension row (late-arriving dimension). How do you handle it, and which SCD2 version does a late fact join to?
**Answer:**
- **Late dimension:** insert an **inferred member** (placeholder row with a surrogate key, natural key, and `is_inferred = true`). Load the fact against it, then update that dimension row (Type 1 on the inferred flag) when the real record arrives, so facts do not need reloading.
- **Which SCD2 version:** join on the natural key and the event date falling within the version's range, `fact.event_date >= dim.start_date AND fact.event_date < dim.end_date`. This gives point-in-time-correct results, including for late facts.
- If a late dimension change backdates history, you may need to restate dimension version ranges and affected facts.

### Q128. Kinesis shows `ProvisionedThroughputExceeded` and iterator age keeps growing. What do you do?
**Answer:**
- **Write side:** a **hot shard** from a skewed partition key. Fix the key (add a random suffix or use a higher-cardinality key). Respect limits per shard (1 MB/s or 1,000 records/s writes). Add shards, use on-demand mode, batch with `PutRecords`, and retry with backoff.
- **Read side:** consumers share 2 MB/s per shard. Use **enhanced fan-out** for dedicated throughput per consumer, scale consumers to shard count, and speed up the processing logic.
- Monitor `GetRecords.IteratorAgeMilliseconds` (consumer lag) and alarm on it.

### Q129. Your Spark Structured Streaming job lags behind the Kafka topic. How do you fix it?
**Answer:**
- Measure: per-batch input rate vs processing rate, batch duration, and offsets behind latest.
- Increase parallelism: topic partitions and Spark cores must match. Tune `maxOffsetsPerTrigger` so batches are neither tiny nor enormous.
- Reduce per-batch work: avoid skewed keys, expensive UDFs, and tiny-file sinks. Tune shuffle partitions for streaming (state-store partitions are fixed once the checkpoint exists).
- Scale executors. Increase the trigger interval if latency permits (fewer, bigger batches).
- Check the sink (S3 commit time, database upserts) since it is often the bottleneck.

### Q130. Streaming state keeps growing until the job runs out of memory. Why?
**Answer:**
- Stateful operations (aggregations, dedup, stream-stream joins) keep state until a **watermark** allows cleanup. With no watermark, state is kept forever.
- Add `withWatermark` on the event-time column, bound join conditions with a time range, include the watermark column in dedup keys.
- Use the **RocksDB state store** for large state (Spark 3.2+), and monitor state size per batch.
- Consider changing the design: shorter windows, pre-aggregate upstream, or move state to an external store.

### Q131. Streaming into Iceberg/Delta creates thousands of tiny files. What do you do?
**Answer:**
- Trade-off: shorter commit/trigger intervals mean lower latency but more small files. Raise the trigger interval to what the business really needs.
- Schedule **compaction**: Iceberg `rewrite_data_files` (or Athena `OPTIMIZE ... REWRITE DATA USING BIN_PACK`, or Glue Data Catalog's automatic compaction optimizer), Delta `OPTIMIZE`, Hudi clustering/compaction.
- Also expire old snapshots and remove orphan files, otherwise metadata and storage grow.
- Partition coarsely (daily/hourly), not by high-cardinality keys.

### Q132. Design disaster recovery for your data platform. What do you decide first?
**Answer:**
- First define **RPO** (acceptable data loss) and **RTO** (acceptable downtime) per tier. Not everything needs the same level.
- **S3:** versioning plus cross-region replication for critical buckets. Object Lock for immutable raw data.
- **Warehouse:** Redshift automated and cross-region snapshots.
- **Catalog and metadata:** export Glue Data Catalog definitions, store DAGs, Glue scripts, and configs in Git.
- **Infrastructure as code** (Terraform) so the stack can be recreated in another region.
- **Streams:** replay from retained logs. Document and **rehearse** the runbook, since an untested DR plan is a guess.

### Q133. How would you build CI/CD for data pipelines with GitHub Actions and Terraform?
**Answer:**
- **On pull request:** lint (ruff/flake8, black), unit tests (`pytest` with a small local SparkSession and fixtures), Airflow DAG integrity test (load `DagBag`, assert no import errors), `terraform fmt`, `validate`, and `plan` (posted to the PR).
- **On merge to main:** `terraform apply` (reviewed or approved), upload Glue scripts and packaged dependencies to S3, sync DAGs to the MWAA bucket.
- **Environments:** dev, then staging, then prod, with the same code and different variables. Smoke test after deployment.
- **Security:** authenticate to AWS using GitHub **OIDC** and an IAM role (no long-lived keys). Keep secrets in Secrets Manager.
- **State:** remote Terraform state in S3 with locking (a DynamoDB table or the newer native S3 lockfile).
- Version everything and support rollback by redeploying a previous tag.

### Q134. Glue, EMR, EMR Serverless, or Redshift SQL for a transformation? How do you choose?
**Answer:**
| Option | Choose when |
|---|---|
| **Glue (Spark)** | Serverless, event/schedule-driven jobs, quick to operate, moderate size, native catalog integration |
| **EMR on EC2/EKS** | Fine control over cluster and Spark/Hive/Presto versions, long-running or very large workloads, custom libraries, cost tuning with spot |
| **EMR Serverless** | Spark without managing clusters, bursty workloads, more tuning freedom than Glue |
| **Redshift SQL (ELT)** | Data already in the warehouse, SQL-friendly transformations, strong set-based joins and aggregations |
| **Athena/dbt** | Lightweight SQL transformations on S3 |

Decide by data volume, team skills, latency, cost model, and operational burden, not by habit.

### Q135. At 3 AM, the pipeline missed its SLA. Walk through your response.
**Answer:**
1. **Acknowledge and triage:** what failed, since when, which downstream datasets and consumers are affected (blast radius).
2. **Stop the damage:** pause downstream jobs that would publish wrong data, and notify stakeholders with an ETA.
3. **Diagnose:** logs, Spark UI/CloudWatch, recent deployments or schema changes, upstream arrival time, resource limits.
4. **Fix or work around:** rerun (the pipeline should be idempotent), scale resources, or temporarily skip a non-critical stage.
5. **Verify:** reconciliation and quality checks before reopening downstream jobs.
6. **Communicate** the resolution, and afterward write a blameless **post-mortem** with root cause, impact, and preventive actions (alerts, tests, runbook updates).

### Q136. How do you handle timezones and daylight saving time in pipelines?
**Answer:**
- Store and process timestamps in **UTC**. Keep the original timezone/offset as a separate column if it matters for the business.
- Convert to local time only at the presentation layer, using proper zone names (`America/New_York`), never fixed offsets.
- DST creates **ambiguous** (clocks repeat) and **non-existent** (clocks skip) local times, so avoid parsing naive local timestamps when possible, or define a rule.
- Define "business day" boundaries explicitly (in which timezone), since date partitions and daily aggregates depend on it.
- Test around DST transition dates.

### Q137. How do you monitor data freshness and detect missing or abnormal data automatically?
**Answer:**
- **Freshness:** expected arrival time per dataset vs latest partition/max event timestamp. Alert on breach.
- **Volume:** compare row counts with the same weekday in previous weeks (z-score or percent bands), and alert on both drops and spikes.
- **Completeness and validity:** null rates, uniqueness of keys, accepted-values checks, referential integrity (Great Expectations, Glue Data Quality, dbt tests).
- **Distribution drift:** key metric ranges and category proportions.
- Record all results in a metrics table, show them on a dashboard, and route alerts by dataset owner with severity levels. Avoid alert noise by tuning thresholds.

---

## How to Structure Scenario Answers

Use this structure in interviews so answers sound designed, not memorized:

1. **Clarify** (30 seconds): data volume, latency need, source and sink, failure tolerance, team and cost constraints.
2. **State the design** in one or two sentences, then go through ingestion, processing, storage, serving, orchestration.
3. **Name the trade-offs:** why this option and not the obvious alternative (for example MOR vs COW, Glue vs EMR).
4. **Cover failure modes:** retries, idempotency, late and duplicate data, schema change, backfills.
5. **Cover operations:** monitoring, alerting, data quality, cost, security.
6. **Tie it to your experience:** one sentence about a similar problem you solved, with real numbers you can defend.

**Phrases that signal seniority:** "I'd make it idempotent first", "what's the SLA and the cost of being wrong?", "I'd verify with reconciliation", "that depends on the table format and engine version".
