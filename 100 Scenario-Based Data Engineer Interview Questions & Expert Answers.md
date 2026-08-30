# 100 Scenario-Based Data Engineer Interview Questions & Expert Answers

> **Author / Reference**: Vinesh Diddi – Data Engineer  
> **Topic**: Real-World Problem Solving, Data Pipelines, Apache Spark, Lakehouse, Streaming, SQL, and Distributed Systems.

---

## 📑 Quick Navigation

- [Questions 1 – 20: Spark Tuning, Shuffling, Ingestion & Lakehouse Architecture](#questions-1--20)
- [Questions 21 – 40: Streaming, Aggregations, Failures, SCD & Linage](#questions-21--40)
- [Questions 41 – 60: Scalability, Security, Joins, Transformations & Reusability](#questions-41--60)
- [Questions 61 – 80: High-Throughput Ingestion, Medallion Architecture & Optimization](#questions-61--80)
- [Questions 81 – 100: Kafka Streaming, Multi-Tenancy, Data Quality & End-to-End Projects](#questions-81--100)

---

## Questions 1 – 20

### Q1. Your Spark job is running very slow. How would you identify the bottleneck?
**A:**  
1. **Spark UI Analysis (Port 4040)**:
   * **Stages & Jobs Tab**: Identify stages with disproportionately long runtimes or high task counts.
   * **Event Timeline & Task Metrics**: Compare the Min, Median, 75th percentile, and Max task durations. A large variance between Median and Max signals **Data Skew**.
   * **Shuffle Read / Write**: Check total shuffle volume. High shuffle read/write indicates expensive wide transformations (`groupByKey`, sort-merge joins).
   * **Spill (Memory) and Spill (Disk)**: Identifies executors running out of RAM during shuffles/aggregations and spilling to disk.
   * **Executors Tab**: Check GC (Garbage Collection) time. If GC time $> 10-15\%$ of task time, the JVM is thrashing memory.
2. **Driver vs Executor Logs**: Look for serialization bottlenecks, large unbroadcasted variables, or network timeout warnings in CloudWatch/YARN.
3. **Storage I/O**: Check if reading millions of small files from S3 or if partition pruning is missing.

---

### Q2. A dataset contains millions of duplicate records. How would you remove them efficiently?
**A:**  
1. **PySpark Window Functions (Keeping Latest State)**:
   ```python
   from pyspark.sql.window import Window
   from pyspark.sql.functions import col, row_number

   window_spec = Window.partitionBy("entity_id").orderBy(col("event_timestamp").desc())
   deduped_df = df.withColumn("rn", row_number().over(window_spec)).filter(col("rn") == 1).drop("rn")
   ```
2. **Built-in `dropDuplicates`**:
   ```python
   # Only shuffles on the distinct keys, utilizing map-side combining
   deduped_df = df.dropDuplicates(["entity_id"])
   ```
3. **Storage Layer Deduplication (Lakehouse Upsert)**:
   * Load data into Delta Lake / Apache Hudi / Iceberg using `MERGE INTO target USING staging ON target.id = staging.id`.

---

### Q3. You have a large dataset and need to join it with a small lookup table. What approach would you use?
**A:**  
* **Approach**: **Broadcast Hash Join**.
* **Mechanism**: Spark sends the entire small lookup table ($<100\text{ MB}$) to every executor node's memory.
* **Benefit**: Completely eliminates the expensive wide shuffle stage and network data transfer, turning an $O(N \log N)$ shuffle sort-merge join into a local $O(N)$ hash lookup.
* **Code**:
  ```python
  from pyspark.sql.functions import broadcast
  result_df = large_df.join(broadcast(small_lookup_df), "lookup_id", "left")
  ```

---

### Q4. Your Spark job is causing excessive shuffling. How would you optimize it?
**A:**  
1. **Replace `groupByKey()` with `reduceByKey()` / `aggregateByKey()`**: Enables map-side combining before network transfer.
2. **Leverage Broadcast Joins**: Broadcast small dimension tables ($< 100\text{ MB}$) using `broadcast(df)`.
3. **Pre-partition on Join/Grouping Keys**: If the dataset is reused across multiple operations, use `df.repartition("join_key")` or persist with partitioning.
4. **Filter & Project Early**: Drop unused columns (`df.select(...)`) and filter rows before join/grouping operations.
5. **Enable Adaptive Query Execution (AQE)**: Set `spark.sql.adaptive.enabled=true` to automatically coalesce shuffle partitions at runtime.

---

### Q5. A streaming pipeline is producing late-arriving data. How would you handle it?
**A:**  
1. **Event Time vs Processing Time**: Always base partition paths and windowed aggregations on the payload's source event timestamp, never the pipeline's ingestion clock.
2. **Watermarking in Spark Structured Streaming**:
   ```python
   streaming_df.withWatermark("event_time", "2 hours") \
       .groupBy(window("event_time", "10 minutes"), "user_id") \
       .count()
   ```
   *Watermark defines the threshold beyond which late data is dropped or routed to a side DLQ.*
3. **Dynamic Partition Routing & Lakehouse Upserts**: Route late events dynamically into historical S3 date partitions using Delta Lake/Hudi `MERGE INTO`, safely updating historical aggregations without duplicates.

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
* **Ingestion**: Amazon Kinesis Data Streams / Apache Kafka captures live operational events.
* **Streaming Engine**: Apache Flink (Kinesis Data Analytics) or Spark Structured Streaming processes 1-minute windowed aggregations.
* **Serving Layer**: Stream aggregated metrics directly into **Amazon Redshift** (with Auto-refreshing Materialized Views) or **Amazon Timestream / ClickHouse / DynamoDB**.
* **Visualization**: **Amazon QuickSight** or Grafana configured with direct-query mode or SPICE fast-refresh.

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
   * *Driver OOM*: Caused by `.collect()`, large broadcasts, or metadata overhead $\rightarrow$ Increase `spark.driver.memory` and write directly to S3.
   * *Executor OOM / YARN Container Killed*: Caused by data skew, large shuffles, or memory leak $\rightarrow$ Increase `spark.executor.memory` and `spark.executor.memoryOverhead`.
2. **Tune Partitioning**: Increase `spark.sql.shuffle.partitions` (e.g., from 200 to 1000+) to reduce partition data size per executor task.
3. **Garbage Collection**: Enable G1GC garbage collector (`-XX:+UseG1GC`) to prevent GC thrashing.
4. **Unpersist Caches**: Call `df.unpersist()` on intermediate DataFrames that are no longer needed.

---

### Q12. A dataset contains nested JSON structures. How would you flatten it?
**A:**  
* **Use PySpark built-in functions (`explode`, `from_json`, `col.*`)**:
  ```python
  from pyspark.sql.functions import col, explode

  # Explode array of objects into individual rows
  exploded_df = df.withColumn("item", explode(col("order_items")))

  # Flatten struct attributes
  flat_df = exploded_df.select(
      col("order_id"),
      col("customer_id"),
      col("item.item_id").alias("item_id"),
      col("item.price").alias("price"),
      col("item.quantity").alias("quantity")
  )
  ```

---

### Q13. Your ETL job is producing too many small files. How would you fix it?
**A:**  
1. **Before Writing in Spark**:
   * Use `df.coalesce(N)` to reduce the number of output partitions without triggering a full network shuffle.
   * Use `df.repartition(N)` if data redistribution across keys is required.
2. **Lakehouse Auto-Compaction**: Configure Delta Lake `OPTIMIZE` or Apache Hudi automated compaction jobs to periodically merge small delta files into optimal 128MB–256MB Parquet files.
3. **Glue Max File Sizing**: Set `hoodie.parquet.max.file.size = 268435456` (256MB).

---

### Q14. A join operation is causing performance issues. What optimization strategies would you use?
**A:**  
1. **Broadcast Join**: For small-to-large table joins ($<100\text{ MB}$), force a broadcast join (`broadcast(small_df)`).
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
2. **Streaming Ingestion**: Kinesis Firehose or Spark Structured Streaming converts streams into columnar Parquet and commits them into an S3 Bronze lake partitioned by `date/hour`.
3. **Real-Time Aggregations**: Spark Structured Streaming computes tumbling window metrics (active sessions, CTR per minute) and streams to Redis / DynamoDB.
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
1. **Partition Pruning**: Ensure queries filter on physical partition columns (e.g., `date`, `region`).
2. **Columnar Storage with Snappy Compression**: Store as Parquet/ORC so query engines scan only referenced columns.
3. **Z-Ordering / Clustering**: Use Delta Lake / Iceberg `OPTIMIZE ZORDER BY (frequently_queried_column)` to collocate multidimensional data.
4. **Data Warehouse Indexing**: In Amazon Redshift, configure optimal `DISTKEY` (join key) and `COMPOUND SORTKEY` (filter keys).
5. **Materialized Views**: Precompute heavy aggregations.

---

### Q20. How would you design a partitioning strategy for a large dataset?
**A:**  
1. **Select High-Cardinality Query Filter Keys**: Partition by date (`year=YYYY/month=MM/day=DD`) or category (`region=US`).
2. **Target File Sizes of 128MB to 256MB**: Ensure each partition directory contains substantial data volume.
3. **Avoid Over-Partitioning**: Avoid partitioning by timestamp to the minute or high-cardinality IDs (`user_id`), which produces millions of tiny KB files (Small File Problem).
4. **Implement Dynamic Partition Routing**: Use dynamic partition overwrite mode in PySpark to write into target partition folders without wiping historical partitions.

---

## Questions 21 – 40

### Q21. How would you ensure fault tolerance in a distributed data pipeline?
**A:**  
1. **Pipeline Idempotency**: Design write operations using Lakehouse `MERGE INTO` or dynamic partition overwrites so re-executing failed batches never creates duplicate rows.
2. **Checkpointing & State Recovery**: Use Spark Structured Streaming checkpoints in S3 to persist offset progress.
3. **Automated Retries**: Configure exponential backoff retries in Apache Airflow (`retries=3`, `retry_delay=timedelta(minutes=5)`).
4. **Decoupled Architecture**: Use durable distributed message brokers (Kafka/SQS) that persist messages during consumer downtime.
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
"While exactly-once network delivery is impossible, an **Exactly-Once Processing Effect** is achieved via:
1. **Replay-able Source**: Kafka/Kinesis retaining message offsets.
2. **Stateful Streaming Engine**: Spark Structured Streaming / Apache Flink with checkpointing and Write-Ahead Logs (WAL).
3. **Idempotent Sink**:
   * Delta Lake / Apache Hudi utilizing unique transaction IDs in `MERGE INTO`.
   * Relational database upserts with `UNIQUE` constraints on message IDs."

---

### Q24. Your ETL pipeline requires data validation rules. How would you implement them?
**A:**  
1. **Metadata-Driven Rule Framework**: Define validation rules in a configuration table (e.g., `not_null`, `range_check`, `regex_match`).
2. **Dynamic PySpark Rule Engine**:
   ```python
   # Dynamic check for validation rules
   valid_df = df.filter("price > 0 AND customer_id IS NOT NULL")
   invalid_df = df.filter("price <= 0 OR customer_id IS NULL")

   # Route invalid rows to DLQ
   invalid_df.write.parquet("s3://lake/dlq/reason=validation_failure/")
   ```
3. **Integration with Tools**: Implement **Great Expectations** or **AWS Glue Data Quality** for automated pre/post-load assertion checks.

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
2. **Replace `groupByKey` with `reduceByKey`**: Applies map-side combining to aggregate data prior to network transmission.
3. **Tune Shuffle Partition Count**: Set `spark.sql.shuffle.partitions` appropriately to match cluster core counts.
4. **Use Broadcast Joins**: Eliminate shuffles entirely for small lookup tables.
5. **Enable Shuffle File Compression**: Set `spark.shuffle.compress=true` and `spark.io.compression.codec=lz4`.

---

### Q27. You need to merge incremental data into a data warehouse table. How would you design it?
**A:**  
1. **Staging Table Load**: Ingest incremental delta files into a transient staging table in Amazon Redshift / Snowflake.
2. **Atomic Merge Transaction**:
   ```sql
   BEGIN TRANSACTION;
   DELETE FROM target_table
   USING staging_table
   WHERE target_table.id = staging_table.id;

   INSERT INTO target_table
   SELECT * FROM staging_table;
   END TRANSACTION;
   ```
3. **Lakehouse Alternative**: In Delta Lake / Hudi, execute standard `MERGE INTO target USING staging ON target.id = staging.id WHEN MATCHED THEN UPDATE WHEN NOT MATCHED THEN INSERT`.

---

### Q28. Your pipeline requires historical tracking of changes. How would you implement SCD Type 2?
**A:**  
* **Schema Design**: Include audit columns: `start_date`, `end_date`, `is_current_flag`, `surrogate_key`.
* **PySpark / SQL Implementation**:
  1. Identify new records $\rightarrow$ Insert with `start_date = current_date`, `end_date = '9999-12-31'`, `is_current = 'Y'`.
  2. Identify changed records $\rightarrow$ Update existing active row setting `end_date = current_date` and `is_current = 'N'`, then insert a new row with updated values, `start_date = current_date`, `end_date = '9999-12-31'`, and `is_current = 'Y'`.
  3. Execute via a single atomic Lakehouse / Warehouse `MERGE` statement.

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
1. **Airflow MWAA Scheduling**: Set cron expression `@hourly` (`0 * * * *`) with `catchup=False`.
2. **S3 Event-Driven Triggers**: Use Amazon EventBridge + AWS Lambda to trigger the Airflow DAG immediately upon hourly file delivery in S3.
3. **Partition Alignment**: Organize output S3 paths by `/year=YYYY/month=MM/day=DD/hour=HH/` to enable rapid incremental loads into Redshift.

---

### Q32. Your Spark cluster resources are underutilized. How would you improve utilization?
**A:**  
1. **Enable Dynamic Resource Allocation**: Set `spark.dynamicAllocation.enabled=true` so executors scale up during compute-heavy stages and terminate during idle periods.
2. **Adjust Partition Count**: If partition count is too low ($< \text{available cores}$), cores sit idle. Increase `spark.default.parallelism` to $2\times\text{--}3\times$ total cores.
3. **Right-Size Worker Types**: Switch from memory-heavy to compute-optimized worker types if CPU is saturated while memory sits empty.

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
1. **Collection**: Deploy FluentBit / Amazon CloudWatch Agents to forward server logs into **Amazon Kinesis Data Firehose**.
2. **Partitioning on Ingestion**: Firehose automatically buffers, compresses (GZIP/Snappy), and writes raw log objects to S3 partitioned by source type and timestamp (`/source_name/YYYY/MM/DD/HH/`).
3. **ETL Standardization**: Scheduled Glue PySpark jobs parse JSON/syslog regex patterns, extract structured dimensions, and load into an Amazon OpenSearch / Athena data lake.

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
* **Format**: **Apache Iceberg / Delta Lake on Parquet with Z-Order Clustering**.
* **Rationale**:
  * Columnar layout provides extreme compression and column projection.
  * Z-Ordering / Min-Max file statistics enable query engines (Athena, Starburst, Presto) to skip 90%+ of non-matching data files.
  * Snapshot isolation ensures high-concurrency read queries never conflict with ongoing writes.

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
1. **Executor Memory Overruns (OOM / YARN Killed)**: Check if executor memory + overhead exceeded container limits (`Exit code 137` / `Container killed by YARN for exceeding memory limits`).
2. **Long GC Pauses**: JVM full GC freezes the executor, causing it to miss heartbeat intervals to the driver (`Heartbeat timed out after 120s`). Fix: Tune G1GC.
3. **Spot Instance Rebalancing**: Cloud provider reclaiming spot instances. Fix: Use on-demand nodes for core drivers/executors.
4. **Disk Full during Shuffles**: Massive intermediate shuffle spills exhausting executor local disk space.

---

### Q40. Your team needs to track data lineage. How would you implement it?
**A:**  
1. **OpenLineage / Apache Atlas**: Integrate Spark jobs with the **OpenLineage Spark listener**, which automatically captures dataset inputs, outputs, schemas, and job run metadata.
2. **AWS Native Lineage**: Use **AWS Lake Formation** and **AWS Glue Data Catalog** lineage graphs.
3. **Airflow Lineage**: Utilize Airflow's built-in `inlets` and `outlets` dataset tracking to visualize DAG-to-DAG dependencies.

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
1. **Encryption**: Enforce AWS KMS customer-managed key encryption at rest (`SSE-KMS`) and TLS 1.2+ in transit.
2. **Tokenization / Hashing**: Mask or hash sensitive columns (`SHA-256(ssn)`) in the raw/silver layer before exposing data to analytics.
3. **Granular Access Control**: Use **AWS Lake Formation** to enforce column-level security and row-level filtering based on IAM roles.
4. **Automated Discovery**: Run **Amazon Macie** to discover and classify sensitive customer data in S3.

---

### Q43. A Spark job needs to process millions of small JSON files. How would you optimize it?
**A:**  
1. **Combine Files on Read**:
   ```python
   # Set max partition split size in Spark
   spark.conf.set("spark.sql.files.maxPartitionBytes", "134217728") # 128MB
   df = spark.read.json("s3://bucket/path/*.json")
   ```
2. **Pre-Compaction via Glue**: Run an AWS Glue job or Amazon EMR S3DistCp to merge tiny JSON files into larger bundled files.
3. **Direct Lakehouse Ingestion**: Ingest raw files directly into Delta Lake using Auto Loader (`cloudFiles`), which automatically handles millions of small files efficiently.

---

### Q44. How would you implement idempotent pipelines?
**A:**  
1. **Deterministic Primary Keys & Lakehouse Upserts**: Use `MERGE INTO target USING staging ON target.id = staging.id`.
2. **Dynamic Partition Overwrite**:
   ```python
   spark.conf.set("spark.sql.sources.partitionOverwriteMode", "dynamic")
   df.write.mode("overwrite").partitionBy("date").parquet("s3://bucket/path/")
   ```
3. **Deduplication Key Table**: Record batch/event IDs in a transactional database table to prevent re-processing identical execution batches.

---

### Q45. Your data warehouse queries are slow. How would you optimize them?
**A:**  
1. **Review Distribution & Sort Keys (Redshift)**: Ensure joining tables share identical `KEY` distribution, small tables use `ALL`, and filtered columns are defined as `SORTKEY`.
2. **Run Maintenance**: Execute `VACUUM` and `ANALYZE` to reclaim space and update query planner statistics.
3. **Enable Concurrency Scaling & WLM**: Configure separate Workload Management queues for ETL vs reporting queries.
4. **Create Materialized Views**: Precompute heavy aggregations for repetitive dashboard queries.

---

### Q46. A dataset arrives late from upstream systems. How would you handle dependencies?
**A:**  
1. **Sensors with Timeouts**: Use Airflow `ExternalTaskSensor` or `S3KeySensor` configured with `mode='reschedule'` and reasonable timeouts to avoid consuming worker slots while waiting.
2. **Event-Driven Triggers**: Replace static cron schedules with Amazon EventBridge rules that trigger pipelines dynamically upon upstream file arrival.
3. **SLA Monitoring & Fallbacks**: Configure `sla_miss_callback` to alert on-call engineers and execute downstream jobs with last-known-good datasets if data arrives past cutoff windows.

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
1. **Partition Pruning**: Ensure queries contain `WHERE` clauses filtering on partition prefix paths.
2. **Increase Split Sizing**: Set `spark.sql.files.maxPartitionBytes=134217728` (128MB).
3. **Avoid S3 Rate Throttling**: If hitting `503 Slow Down`, distribute file prefixes or increase S3 request limits.
4. **Switch to Columnar Formats**: Replace CSV/JSON with Parquet/ORC to read only required columns.
5. **Use AWS S3 Gateway VPC Endpoint**: Ensures traffic travels across dedicated AWS network backbones instead of traversing NAT Gateways.

---

### Q49. Your pipeline requires audit logging. How would you implement it?
**A:**  
* **Audit Metadata Table**: Maintain an audit log database (`pipeline_name`, `batch_id`, `execution_date`, `records_read`, `records_written`, `quarantined_records`, `start_time`, `end_time`, `status`).
* **PySpark Audit Wrapper**: Record row counts before and after transformations and write execution metrics to the audit table in a `finally:` block.
* **Structured JSON Application Logs**: Emit logs in JSON containing `trace_id` and `batch_id` to Amazon CloudWatch.

---

### Q50. A dataset must support time-travel queries. How would you design it?
**A:**  
* **Implement Modern Table Format**: Use **Delta Lake** or **Apache Iceberg**.
* **Query Historical Snapshots**:
  ```sql
  -- Query by timestamp
  SELECT * FROM item_inventory TIMESTAMP AS OF '2026-08-30 00:00:00';

  -- Query by commit version
  SELECT * FROM item_inventory VERSION AS OF 42;
  ```
* **Manage Retention**: Configure log retention (`vacuum` threshold) to balance historical time-travel window requirements against storage costs.

---

### Q51. Your pipeline needs schema validation before processing. How would you implement it?
**A:**  
1. **Pre-Processing Gatekeeper**: Compare incoming DataFrame schema against an expected `StructType` definition:
   ```python
   expected_schema = StructType([...])
   if df.schema != expected_schema:
       # Quarantine and alert
       df.write.parquet("s3://lake/schema_mismatch_quarantine/")
       raise ValueError("Schema mismatch detected")
   ```
2. **AWS Glue Schema Registry**: Enforce strict compatibility checks during schema deserialization.

---

### Q52. You need to process IoT streaming data. What architecture would you design?
**A:**  
1. **Ingestion**: **AWS IoT Core** ingests MQTT messages and forwards to **Amazon Kinesis Data Streams**.
2. **Real-Time Processing**: **Apache Flink / Spark Streaming** cleans data, deduplicates by device ID, and calculates rolling 5-minute anomaly metrics.
3. **Hot Path Storage**: Low-latency anomalies written to **Amazon DynamoDB / Timestream** for immediate operational alerting.
4. **Cold Path Storage**: Raw telemetry batched via **Kinesis Firehose** into S3 Parquet partitions for long-term ML training.

---

### Q53. Your Spark job processes skewed keys during joins. What techniques can help?
**A:**  
* **Key Salting**: Append random integer suffixes (`0-N`) to the skewed key on the larger table, explode the smaller table $N$ times with matching suffixes, join on salted keys, and aggregate.
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
* **Lakehouse Upsert**: Configure `precombineField="updated_at"` in Apache Hudi / Delta Lake `MERGE INTO`.

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
1. **Star Schema Dimensional Modeling**: Normalize into central Fact tables containing numerical metrics and foreign keys, surrounded by denormalized Dimension tables.
2. **Columnar Format**: Store data in Parquet with Snappy compression.
3. **Partitioning & Clustering**: Partition by date and Z-order/cluster by high-frequency query filter columns.
4. **Surrogate Keys**: Use integer surrogate keys for dimensions to optimize join performance.

---

### Q57. A Spark job runs slowly due to wide transformations. How would you optimize it?
**A:**  
1. **Map-Side Aggregations**: Replace `groupByKey` with `reduceByKey` or `aggregateByKey`.
2. **Broadcast Hash Joins**: Eliminate wide join shuffles for lookup tables under 100MB.
3. **Filter Early**: Eliminate unneeded rows and columns before entering the wide transformation.
4. **Tune Shuffle Parallelism**: Align `spark.sql.shuffle.partitions` with cluster CPU cores.

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
* **Ingestion**: **Apache Kafka (Amazon MSK)** or **Amazon Kinesis** with pre-provisioned partition/shard capacity.
* **Processing**: **Apache Flink** or **Spark Structured Streaming** running on Amazon EMR with high memory/compute node clusters.
* **Storage**: Micro-batches committed into **Apache Iceberg / Delta Lake** on S3 with partitioned Parquet storage.
* **Caching / Serving**: Real-time aggregations written to **Redis / ClickHouse** for sub-second dashboard access.

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
* **Delta Lake Change Data Feed (CDF) / Auto Loader**:
  ```python
  df = spark.readStream.format("cloudFiles") \
      .option("cloudFiles.format", "parquet") \
      .load("s3://lake/bronze/")
  ```
* **Watermark Filtering**: Filter source datasets where `last_updated_timestamp > last_successful_watermark` and update the control table upon commit.

---

### Q63. A Spark job must handle late-arriving data. How would you manage it?
**A:**  
1. **Event-Time Partition Routing**: Extract source event timestamps and dynamically write records into the corresponding historical partition path (`spark.sql.sources.partitionOverwriteMode=dynamic`).
2. **Lakehouse MERGE**: Use Delta Lake / Hudi upsert logic to update historical partitions safely without overwriting non-matching historical data.

---

### Q64. Your cluster experiences frequent executor failures. How would you debug it?
**A:**  
1. **Inspect Failure Codes in YARN / Spark UI**: Look for `Exit code 137` (Container memory limit exceeded) or `Exit code 143` (SIGTERM / Spot preemption).
2. **Check GC Logs**: Review if long garbage collection pauses caused the driver to mark executors dead (`Heartbeat timed out`).
3. **Check Local Disk Saturation**: Verify if shuffle spill filled up executor local EBS storage.

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
1. **Statistical Anomaly Detection**: In PySpark / Flink, compute rolling mean and standard deviation over a sliding window:
   ```python
   # Flag records where value exceeds 3 standard deviations
   anomaly_df = df.filter(col("value") > (col("mean_val") + 3 * col("stddev_val")))
   ```
2. **Routing**: Route identified anomalies to an Amazon SNS alert topic and quarantine S3 bucket.

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
* **Format**: Parquet with Snappy compression for fast batch scans in Python/PyTorch.
* **Feature Store Integration**: Integrate with **Amazon SageMaker Feature Store** (online low-latency store in DynamoDB, offline historical store in S3).
* **Versioning**: Use Delta Lake / Iceberg time-travel to reproduce exact ML training datasets for model compliance.

---

### Q72. You need to handle millions of events per second. What architecture would you build?
**A:**  
* **Ingestion**: **Apache Kafka (Amazon MSK)** scaled with hundreds of partitions across high-throughput broker nodes.
* **Compute**: **Apache Flink** on EMR running event-driven stream transformations.
* **Sink**: Write to **Apache Iceberg** tables on S3 with optimized commit intervals and write real-time metrics to **Amazon Timestream / ClickHouse**.

---

### Q73. Your Spark job frequently runs out of memory. What tuning steps would you take?
**A:**  
1. Increase `spark.executor.memory` and `spark.executor.memoryOverhead`.
2. Increase `spark.sql.shuffle.partitions` to reduce data volume per shuffle task.
3. Tune memory split: increase `spark.memory.fraction` (default 0.6).
4. Resolve data skew via key salting.
5. Use Kryo serialization (`spark.serializer=org.apache.spark.serializer.KryoSerializer`).

---

### Q74. A dataset contains nested arrays and structs. How would you flatten them?
**A:**  
* **PySpark Recursive / Column Flattening**:
  ```python
  from pyspark.sql.functions import col, explode

  # Explode array column
  df_exploded = df.withColumn("phone_record", explode(col("contact_numbers")))

  # Extract nested struct fields
  df_flat = df_exploded.select(
      col("user_id"),
      col("user_name"),
      col("phone_record.type").alias("phone_type"),
      col("phone_record.number").alias("phone_number")
  )
  ```

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
1. **Parameterized Execution Dates**: Design all Spark and Airflow jobs to accept explicit date ranges (`start_date`, `end_date`) instead of using hardcoded `current_date()`.
2. **Idempotent Writes**: Use dynamic partition overwrite or Lakehouse `MERGE INTO` so backfilled data overwrites target partitions cleanly.
3. **Airflow Catchup**: Enable Airflow backfilling via CLI:
   `airflow dags backfill -s 2026-01-01 -e 2026-01-31 pipeline_dag`.

---

### Q78. A dataset contains inconsistent timestamps. How would you standardize them?
**A:**  
* **Standardize to UTC**:
  ```python
  from pyspark.sql.functions import to_timestamp, to_utc_timestamp, col

  # Parse multiple date formats and convert to UTC
  standardized_df = df.withColumn(
      "clean_timestamp",
      to_utc_timestamp(to_timestamp(col("raw_date"), "yyyy-MM-dd HH:mm:ss"), "America/New_York")
  )
  ```

---

### Q79. A pipeline needs automated schema detection. How would you implement it?
**A:**  
* **AWS Glue Crawlers**: Schedule Glue Crawlers to scan S3 prefixes periodically, infer schemas, and register/update partitions in the AWS Glue Data Catalog.
* **Delta Lake Auto Loader**: Use Spark `cloudFiles` with `cloudFiles.schemaLocation` to automatically infer and track evolving schemas on S3.

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
* **Storage Deduplication**: Merge streaming micro-batches into Delta Lake / Hudi using primary key matching.

---

### Q83. A dataset contains extremely large partitions. How would you rebalance them?
**A:**  
1. **Repartitioning by Multi-Column Key**: Repartition on a composite key (e.g., `region` + `date`) to break giant single-key partitions into smaller balanced partitions.
2. **`df.repartition(N)`**: Force a uniform shuffle across $N$ partitions.
3. **Adaptive Query Execution (AQE)**: Set `spark.sql.adaptive.advisoryPartitionSizeInBytes=134217728` (128MB) to let Spark dynamically rebalance partitions.

---

### Q84. Your data team needs governance and access control. How would you implement it?
**A:**  
* **AWS Lake Formation**: Centralize access permissions to Glue Data Catalog databases, tables, columns, and rows.
* **IAM Least Privilege Roles**: Assign distinct IAM roles for Data Analysts (read-only curated layer), Data Scientists (read-only silver/gold), and Data Engineers (write/admin access).
* **Audit Tracking**: Enable **AWS CloudTrail** and Lake Formation audit logs to monitor all data access requests.

---

### Q85. A pipeline processes financial transactions. How would you ensure data accuracy?
**A:**  
1. **Double-Entry Reconciliation**: Verify that debits equal credits at every batch boundary.
2. **Data Types**: Use exact `DecimalType(18, 2)` instead of floating-point numbers.
3. **Strict Validation Gatekeeper**: Reject and quarantine any transaction with missing timestamps, invalid accounts, or negative amounts.
4. **Idempotent Upserts**: Ensure transaction IDs have unique constraints to prevent double-charging.

---

### Q86. Your ETL job requires dynamic configuration. How would you design it?
**A:**  
* Store configurations in **AWS Systems Manager Parameter Store** or **Amazon DynamoDB**.
* The PySpark script fetches config parameters dynamically at startup, avoiding hardcoded SQL expressions, S3 bucket names, or table paths.

---

### Q87. A dataset must support multi-tenant access. How would you design security?
**A:**  
1. **Partition by Tenant ID**: Store data in isolated S3 prefix paths (`s3://lake/tenant_id=XYZ/`).
2. **Row-Level Security**: Configure Lake Formation Row-Level Filters mapping tenant IAM session tags to matching `tenant_id` rows.
3. **Dedicated Encryption Keys**: Use tenant-specific AWS KMS keys if contractually required.

---

### Q88. A Spark job must join structured and semi-structured data. How would you do it?
**A:**  
1. Ingest structured table as standard DataFrame.
2. Ingest semi-structured JSON using `schema_of_json()` or parse structs into relational columns using `from_json()`.
3. Join on common relational keys using Broadcast Hash Join if the dimension is small.

---

### Q89. A data warehouse requires partition pruning. How would you implement it?
**A:**  
* Ensure table is partitioned on filter columns (e.g., `date`).
* Structure queries to filter explicitly on the physical partition key: `WHERE event_date BETWEEN '2026-08-01' AND '2026-08-30'`.
* Verify in query execution plans (`EXPLAIN`) that non-matching partitions are skipped.

---

### Q90. Your pipeline needs automatic scaling. What technologies would you use?
**A:**  
* **AWS Glue Autoscaling**: Set `--enable-auto-scaling=true` to allow Glue to dynamically add workers during compute-intensive shuffles and remove workers when idle.
* **Amazon EMR Managed Scaling**: Automatically scales cluster core and task EC2 nodes based on YARN memory and CPU metrics.

---

### Q91. How would you design a high-performance feature store for ML?
**A:**  
* **Dual-Store Architecture**:
  * **Online Store (Low-Latency)**: Amazon DynamoDB or Redis serving real-time feature lookups ($<10\text{ ms}$) for live inference.
  * **Offline Store (Scale & Historical)**: Amazon S3 with Delta Lake / Parquet storing point-in-time feature snapshots for ML model training.
  * **Unified Ingestion**: Feature pipelines write simultaneously to both online and offline stores.

---

### Q92. Your Spark job must process compressed files. What considerations are needed?
**A:**  
* **Splittability**: GZIP and standard BZIP2 files are non-splittable by default (a single multi-gigabyte GZIP file is processed by only ONE core).
* **Best Practice**: Use **Snappy compressed Parquet** or **bzip2 with splittable indexing**, ensuring Spark splits large compressed files across multiple executor tasks in parallel.

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
* **S3 Object Versioning & S3 Object Lock (WORM - Write Once Read Many)**: Guarantees raw data files cannot be overwritten or deleted.
* **Lakehouse Commit Logs**: Delta Lake / Iceberg commit history preserving exact snapshot histories for regulatory audits.
* **Pipeline Audit Log Table**: Record every ETL execution batch metadata.

---

### Q97. A streaming pipeline must recover from failures automatically. How would you design it?
**A:**  
1. **Persistent Message Retention**: Retain 7 days of raw stream data in Kafka/Kinesis.
2. **S3 Checkpointing**: Enable Spark Structured Streaming checkpoints.
3. **Supervisor Auto-Restart**: Run streaming jobs on Kubernetes / EMR with auto-restart policies on pod failure.
4. **Idempotent Destination Writes**: Ensure replaying uncommitted micro-batches overwrites partial writes cleanly.

---

### Q98. Your Spark job must read from multiple partitions simultaneously. How would you optimize it?
**A:**  
* Optimize partition discovery: enable `spark.sql.sources.parallelPartitionDiscovery.enabled=true` and `spark.sql.sources.parallelPartitionDiscovery.threshold=32`.
* Increase parallel S3 listing threads to speed up multi-partition directory scans.

---

### Q99. A pipeline must process historical and real-time data together. How would you design it?
**A:**  
* **Lambda vs Kappa Architecture**:
  * Implement **Kappa Architecture**: Stream all real-time events through Kafka into an S3 Lakehouse table.
  * Historical backfills and real-time events write to the same Delta Lake / Iceberg table using unified PySpark code, allowing queries to seamlessly join real-time and historical partitions.

---

### Q100. Explain an end-to-end data engineering project you built and the challenges you solved.
**A:**  
"I designed and built an enterprise-scale Change Data Capture (CDC) Lakehouse platform on AWS:
* **Architecture**: AWS DMS captures transactional database mutations from PostgreSQL and stages them in Amazon S3. AWS Glue (PySpark) jobs process CDC feeds (`INSERT`, `UPDATE`, `DELETE`) and execute SCD Type-1 upserts and hard deletes into Apache Hudi / Delta Lake on S3. Amazon Redshift Spectrum and Athena query curated data directly for business intelligence dashboards. Orchestrated end-to-end using Apache Airflow (MWAA).
* **Key Challenges Solved**:
  1. *Small File Problem*: Resolved by implementing automated Lakehouse compaction jobs and tuning Spark partition write sizes (128MB–256MB).
  2. *Data Skew*: Resolved join bottlenecks on high-volume keys using key salting and Spark Adaptive Query Execution (AQE).
  3. *Zero Duplicate Ingestion*: Enforced strict pipeline idempotency using Lakehouse primary-key upserts with event-time precombining.
  4. *Cost Optimization*: Storing historical data in S3 queried via Redshift Spectrum cut data warehouse cluster storage costs by 40%."

---
*End of 100 Scenario-Based Data Engineer Interview Questions & Answers.*
