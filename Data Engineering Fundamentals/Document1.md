# Data Engineering Fundamentals: Questions & Answers

A structured reference covering core concepts, architectures, tools, and industry-relevant extras. Duplicate questions from the original list are merged. Questions marked **(Added)** are extra topics commonly asked in interviews and used in industry.

---

## Table of Contents

1. [Foundations](#1-foundations)
2. [Databases & Transactions](#2-databases--transactions)
3. [Data Modeling](#3-data-modeling)
4. [Storage Architectures](#4-storage-architectures)
5. [File Formats](#5-file-formats)
6. [Open Table Formats & Lakehouse Tech](#6-open-table-formats--lakehouse-tech)
7. [Processing Patterns & Architectures](#7-processing-patterns--architectures)
8. [Pipeline Design Concepts](#8-pipeline-design-concepts)
9. [Big Data & Orchestration Tools](#9-big-data--orchestration-tools)
10. [Cloud Data Engineering](#10-cloud-data-engineering)
11. [DevOps for Data: CI/CD & DataOps](#11-devops-for-data-cicd--dataops)
12. [Data Reporting](#12-data-reporting)
13. [Additional Industry Questions (Added)](#13-additional-industry-questions-added)

---

## Question Index

Total questions: **112**

**[1. Foundations](#1-foundations)**

- [Q1. What is Data Engineering?](#q1-what-is-data-engineering)
- [Q2. What is the Data Engineering Lifecycle?](#q2-what-is-the-data-engineering-lifecycle)
- [Q3. What is Data Generation?](#q3-what-is-data-generation)
- [Q4. What is Data Transformation?](#q4-what-is-data-transformation)
- [Q5. What is Data Serving?](#q5-what-is-data-serving)
- [Q6. What are Data Upstream and Data Downstream?](#q6-what-are-data-upstream-and-data-downstream)
- [Q7. What is a Pipeline?](#q7-what-is-a-pipeline)

**[2. Databases & Transactions](#2-databases--transactions)**

- [Q8. What is a Database?](#q8-what-is-a-database)
- [Q9. What are CRUD operations?](#q9-what-are-crud-operations)
- [Q10. What are ACID properties?](#q10-what-are-acid-properties)
- [Q11. What is Atomicity?](#q11-what-is-atomicity)
- [Q12. What is a Write-Ahead Log (WAL)?](#q12-what-is-a-write-ahead-log-wal)
- [Q13. What is a WAL Buffer?](#q13-what-is-a-wal-buffer)
- [Q14. What is OLTP?](#q14-what-is-oltp)
- [Q15. What is OLAP?](#q15-what-is-olap)
- [Q16. OLTP vs OLAP](#q16-oltp-vs-olap)
- [Q17. What is Normalization?](#q17-what-is-normalization)
- [Q18. What is Database Modeling?](#q18-what-is-database-modeling)
- [Q19. What is Data Modeling?](#q19-what-is-data-modeling)

**[3. Data Modeling](#3-data-modeling)**

- [Q20. What is Dimensional Data Modeling?](#q20-what-is-dimensional-data-modeling)
- [Q21. What is a Star Schema?](#q21-what-is-a-star-schema)
- [Q22. What is a Snowflake Schema?](#q22-what-is-a-snowflake-schema)
- [Q23. Star vs Snowflake](#q23-star-vs-snowflake)
- [Q24. What are Slowly Changing Dimensions (SCD)?](#q24-what-are-slowly-changing-dimensions-scd)
- [Q25. What is SCD Type 2?](#q25-what-is-scd-type-2)
- [Q26. What is SCD Type 3?](#q26-what-is-scd-type-3)
- [Q27. What is a Surrogate Key? (Added)](#q27-what-is-a-surrogate-key-added)
- [Q28. What are the types of Fact Tables? (Added)](#q28-what-are-the-types-of-fact-tables-added)
- [Q29. What is Data Vault? (Added)](#q29-what-is-data-vault-added)

**[4. Storage Architectures](#4-storage-architectures)**

- [Q30. What is a Data Warehouse?](#q30-what-is-a-data-warehouse)
- [Q31. What are Data Warehouse Layers?](#q31-what-are-data-warehouse-layers)
- [Q32. What is a Data Lake?](#q32-what-is-a-data-lake)
- [Q33. Data Lake vs Data Warehouse](#q33-data-lake-vs-data-warehouse)
- [Q34. What is a Data Lakehouse?](#q34-what-is-a-data-lakehouse)
- [Q35. What is a Data Mart?](#q35-what-is-a-data-mart)
- [Q36. What is Data Mesh?](#q36-what-is-data-mesh)
- [Q37. What is Data Fabric?](#q37-what-is-data-fabric)

**[5. File Formats](#5-file-formats)**

- [Q38. Row-Based vs Column-Based File Formats](#q38-row-based-vs-column-based-file-formats)
- [Q39. What is CSV?](#q39-what-is-csv)
- [Q40. What is JSON?](#q40-what-is-json)
- [Q41. What is Avro?](#q41-what-is-avro)
- [Q42. What is Parquet?](#q42-what-is-parquet)
- [Q43. What is ORC? (Added)](#q43-what-is-orc-added)
- [Q44. Which format should I choose? (Added)](#q44-which-format-should-i-choose-added)

**[6. Open Table Formats & Lakehouse Tech](#6-open-table-formats--lakehouse-tech)**

- [Q45. What is an Open Table Format?](#q45-what-is-an-open-table-format)
- [Q46. What is Delta Lake?](#q46-what-is-delta-lake)
- [Q47. What is a Transaction Log?](#q47-what-is-a-transaction-log)
- [Q48. Delta vs Iceberg vs Hudi (Added)](#q48-delta-vs-iceberg-vs-hudi-added)
- [Q49. What is Delta Live Tables (DLT)?](#q49-what-is-delta-live-tables-dlt)
- [Q50. What is Lakebase?](#q50-what-is-lakebase)

**[7. Processing Patterns & Architectures](#7-processing-patterns--architectures)**

- [Q51. What is ETL (Extract, Transform, Load)?](#q51-what-is-etl-extract-transform-load)
- [Q52. What is ELT (Extract, Load, Transform)?](#q52-what-is-elt-extract-load-transform)
- [Q53. ETL vs ELT](#q53-etl-vs-elt)
- [Q54. What is Batch Processing?](#q54-what-is-batch-processing)
- [Q55. What is Streaming Data Processing?](#q55-what-is-streaming-data-processing)
- [Q56. What is Lambda Architecture?](#q56-what-is-lambda-architecture)
- [Q57. What is Kappa Architecture?](#q57-what-is-kappa-architecture)
- [Q58. Lambda vs Kappa](#q58-lambda-vs-kappa)
- [Q59. What is Medallion Architecture?](#q59-what-is-medallion-architecture)
- [Q60. What is the Bronze Stage?](#q60-what-is-the-bronze-stage)
- [Q61. What is the Silver Stage?](#q61-what-is-the-silver-stage)
- [Q62. What is the Gold Stage?](#q62-what-is-the-gold-stage)

**[8. Pipeline Design Concepts](#8-pipeline-design-concepts)**

- [Q63. What is Incremental (Data) Loading?](#q63-what-is-incremental-data-loading)
- [Q64. What is Back-date Refresh (Backfill)?](#q64-what-is-back-date-refresh-backfill)
- [Q65. What is Idempotency?](#q65-what-is-idempotency)
- [Q66. What is "Exactly Once"?](#q66-what-is-exactly-once)
- [Q67. What are Watermarks?](#q67-what-are-watermarks)
- [Q68. What is Upsert?](#q68-what-is-upsert)
- [Q69. What is CDC (Change Data Capture)?](#q69-what-is-cdc-change-data-capture)

**[9. Big Data & Orchestration Tools](#9-big-data--orchestration-tools)**

- [Q70. What is Big Data Engineering?](#q70-what-is-big-data-engineering)
- [Q71. What is Distributed Computing?](#q71-what-is-distributed-computing)
- [Q72. What is Apache Spark?](#q72-what-is-apache-spark)
- [Q73. Spark Architecture](#q73-spark-architecture)
- [Q74. What is Apache Airflow?](#q74-what-is-apache-airflow)
- [Q75. What is Apache Kafka?](#q75-what-is-apache-kafka)
- [Q76. What is Apache Hive?](#q76-what-is-apache-hive)
- [Q77. What is Trino?](#q77-what-is-trino)
- [Q78. What is dbt?](#q78-what-is-dbt)
- [Q79. What is Apache Flink? (Added)](#q79-what-is-apache-flink-added)
- [Q80. Airflow vs Kafka vs Spark (Added)](#q80-airflow-vs-kafka-vs-spark-added)

**[10. Cloud Data Engineering](#10-cloud-data-engineering)**

- [Q81. What is Cloud Computing?](#q81-what-is-cloud-computing)
- [Q82. What is Cloud Data Engineering?](#q82-what-is-cloud-data-engineering)
- [Q83. Cloud data services map](#q83-cloud-data-services-map)
- [Q84. Typical AWS data pipeline (Added)](#q84-typical-aws-data-pipeline-added)

**[11. DevOps for Data: CI/CD & DataOps](#11-devops-for-data-cicd--dataops)**

- [Q85. What is CI/CD?](#q85-what-is-cicd)
- [Q86. What is DataOps?](#q86-what-is-dataops)
- [Q87. What is Infrastructure as Code (IaC)? (Added)](#q87-what-is-infrastructure-as-code-iac-added)

**[12. Data Reporting](#12-data-reporting)**

- [Q88. What is Data Reporting?](#q88-what-is-data-reporting)

**[13. Additional Industry Questions (Added)](#13-additional-industry-questions-added)**

- [Q89. What is Partitioning, and how is it different from Bucketing?](#q89-what-is-partitioning-and-how-is-it-different-from-bucketing)
- [Q90. What is the Small Files Problem?](#q90-what-is-the-small-files-problem)
- [Q91. What is Data Skew and how do you handle it?](#q91-what-is-data-skew-and-how-do-you-handle-it)
- [Q92. What is a Shuffle in Spark?](#q92-what-is-a-shuffle-in-spark)
- [Q93. Broadcast Join vs Sort-Merge Join](#q93-broadcast-join-vs-sort-merge-join)
- [Q94. What is Data Quality and what are its dimensions?](#q94-what-is-data-quality-and-what-are-its-dimensions)
- [Q95. What is Data Lineage?](#q95-what-is-data-lineage)
- [Q96. What is a Data Catalog and Metadata Management?](#q96-what-is-a-data-catalog-and-metadata-management)
- [Q97. What is Data Governance?](#q97-what-is-data-governance)
- [Q98. What is Data Observability?](#q98-what-is-data-observability)
- [Q99. What is Schema Evolution vs Schema Enforcement?](#q99-what-is-schema-evolution-vs-schema-enforcement)
- [Q100. What is a Data Contract?](#q100-what-is-a-data-contract)
- [Q101. What is Time Travel?](#q101-what-is-time-travel)
- [Q102. What is the CAP Theorem?](#q102-what-is-the-cap-theorem)
- [Q103. What are Indexes and when are they useful?](#q103-what-are-indexes-and-when-are-they-useful)
- [Q104. Common SQL window functions asked in interviews](#q104-common-sql-window-functions-asked-in-interviews)
- [Q105. What is a Dead Letter Queue (DLQ)?](#q105-what-is-a-dead-letter-queue-dlq)
- [Q106. What is Push vs Pull ingestion?](#q106-what-is-push-vs-pull-ingestion)
- [Q107. Orchestration vs Choreography](#q107-orchestration-vs-choreography)
- [Q108. What is Master Data Management (MDM)?](#q108-what-is-master-data-management-mdm)
- [Q109. What is Reverse ETL?](#q109-what-is-reverse-etl)
- [Q110. What is a Feature Store?](#q110-what-is-a-feature-store)
- [Q111. How do you design a CDC pipeline into a lakehouse? (Added: common system design question)](#q111-how-do-you-design-a-cdc-pipeline-into-a-lakehouse-added-common-system-design-question)
- [Q112. How do you optimise a slow pipeline? (Added)](#q112-how-do-you-optimise-a-slow-pipeline-added)

---

## 1. Foundations

### Q1. What is Data Engineering?
Data Engineering is the discipline of designing, building, and maintaining the systems that **collect, store, transform, and serve data** so that analysts, data scientists, and applications can use it reliably. Data engineers build pipelines, manage storage platforms, ensure data quality, and make data available at scale and on time.

**In short:** raw data in, trustworthy and usable data out.

### Q2. What is the Data Engineering Lifecycle?
The lifecycle (popularised by *Fundamentals of Data Engineering*, Reis & Housley) has five stages, supported by cross-cutting "undercurrents":

| Stage | Purpose |
|---|---|
| **Generation** | Source systems create data (apps, IoT, logs, APIs) |
| **Ingestion** | Move data from sources into the platform (batch or streaming) |
| **Storage** | Persist data (warehouse, lake, lakehouse, object storage) |
| **Transformation** | Clean, join, aggregate, and model data into useful shapes |
| **Serving** | Deliver data to BI, ML, reverse ETL, and applications |

**Undercurrents:** security, data management/governance, DataOps, data architecture, orchestration, and software engineering.

### Q3. What is Data Generation?
Data generation is the **source stage** where data is produced. Sources include:
- Transactional databases (MySQL, PostgreSQL, Oracle)
- Application logs and clickstreams
- SaaS tools (Salesforce, Stripe)
- IoT sensors and devices
- Third-party APIs, files, and message queues

Key considerations: schema, volume, velocity, change frequency, and whether the data engineer controls the source (often they do not).

### Q4. What is Data Transformation?
Transformation converts raw data into a clean, structured, business-ready form. Typical operations:
- Cleaning (nulls, duplicates, type casting)
- Standardisation (formats, units, timezones)
- Joining and enriching from multiple sources
- Aggregation and derivation of metrics
- Applying business rules and data modeling (facts and dimensions)

Tools: Spark, SQL, dbt, AWS Glue, pandas, Flink.

### Q5. What is Data Serving?
Serving is the final stage where processed data is **delivered to consumers**:
- **Analytics / BI:** dashboards via Tableau, Power BI, QuickSight
- **ML:** feature stores and training datasets
- **Operational analytics:** low-latency APIs and serving databases
- **Reverse ETL:** pushing warehouse data back into SaaS tools (CRM, marketing)
- **Data sharing:** Snowflake shares, Delta Sharing

### Q6. What are Data Upstream and Data Downstream?
These describe position relative to a given system in the data flow.
- **Upstream:** systems that produce or feed data *into* your pipeline (source DBs, APIs, producers).
- **Downstream:** systems that *consume* your output (dashboards, ML models, other pipelines).

**Why it matters:** a schema change upstream can break everything downstream. This is why data contracts, lineage, and communication between teams are important.

### Q7. What is a Pipeline?
A data pipeline is an **automated sequence of steps** that moves data from source to destination, often with transformation along the way. It includes ingestion, processing, validation, loading, scheduling, monitoring, and alerting. Pipelines can be batch, streaming, or hybrid.

---

## 2. Databases & Transactions

### Q8. What is a Database?
A database is an **organised collection of data** stored and accessed electronically, managed by a Database Management System (DBMS). Types include:
- **Relational (RDBMS):** PostgreSQL, MySQL, Oracle
- **NoSQL:** document (MongoDB), key-value (DynamoDB, Redis), wide-column (Cassandra), graph (Neo4j)
- **Analytical/columnar:** Redshift, Snowflake, BigQuery, ClickHouse

### Q9. What are CRUD operations?
The four basic data operations:

| Operation | SQL | HTTP analogue |
|---|---|---|
| **C**reate | `INSERT` | POST |
| **R**ead | `SELECT` | GET |
| **U**pdate | `UPDATE` | PUT/PATCH |
| **D**elete | `DELETE` | DELETE |

### Q10. What are ACID properties?
Guarantees that make database transactions reliable:

- **Atomicity:** a transaction is all-or-nothing.
- **Consistency:** a transaction moves the database from one valid state to another, respecting constraints.
- **Isolation:** concurrent transactions do not interfere; results are as if run serially (levels: read uncommitted, read committed, repeatable read, serializable).
- **Durability:** once committed, data survives crashes (typically via the WAL and disk flush).

### Q11. What is Atomicity?
Atomicity means a transaction is **indivisible**: either every operation succeeds and is committed, or none are applied (rollback).

**Example:** transferring money. Debit from A and credit to B must both happen. If the credit fails, the debit is rolled back.

### Q12. What is a Write-Ahead Log (WAL)?
WAL is a durability and recovery technique: **changes are written to an append-only log on disk before they are applied to the actual data files.**

Benefits:
- **Crash recovery:** replay committed entries, undo uncommitted ones.
- **Performance:** sequential log writes are faster than random data-page writes.
- **Replication and CDC:** the log can be streamed to replicas or read by CDC tools (e.g. PostgreSQL logical decoding, MySQL binlog).

### Q13. What is a WAL Buffer?
The WAL buffer is an **in-memory area** where log records are first collected before being flushed to the WAL file on disk. Flushing happens on transaction commit, when the buffer fills, or periodically. It batches many small writes into fewer disk writes. In PostgreSQL it is controlled by `wal_buffers`.

### Q14. What is OLTP?
**Online Transaction Processing:** systems built for many small, fast, concurrent read/write transactions (orders, payments, bookings).
- Row-oriented storage, normalised schema
- Low latency, high concurrency, strong ACID guarantees
- Examples: PostgreSQL, MySQL, Oracle, SQL Server

### Q15. What is OLAP?
**Online Analytical Processing:** systems built for complex queries over large historical datasets (aggregations, trends, reporting).
- Column-oriented storage, denormalised/dimensional schema
- Read-heavy, scan-heavy, batch loaded
- Examples: Redshift, Snowflake, BigQuery, ClickHouse

### Q16. OLTP vs OLAP

| Aspect | OLTP | OLAP |
|---|---|---|
| Purpose | Run the business | Analyse the business |
| Workload | Many small transactions | Few large analytical queries |
| Storage | Row-based | Column-based |
| Schema | Normalised (3NF) | Star/snowflake, denormalised |
| Data | Current | Historical, integrated |
| Latency | Milliseconds | Seconds to minutes |
| Users | Applications, customers | Analysts, data scientists |

### Q17. What is Normalization?
Normalization organises relational tables to **reduce redundancy and avoid update anomalies**.
- **1NF:** atomic values, no repeating groups
- **2NF:** 1NF plus no partial dependency on part of a composite key
- **3NF:** 2NF plus no transitive dependencies (non-key columns depend only on the key)
- **BCNF:** every determinant is a candidate key

OLTP favours normalisation; OLAP often deliberately denormalises for read speed.

### Q18. What is Database Modeling?
Database modeling is designing the **structure of a database**: tables, columns, keys, relationships, and constraints. It progresses through three levels:
1. **Conceptual:** entities and relationships (ER diagram), business view
2. **Logical:** attributes, keys, normalisation, independent of DBMS
3. **Physical:** data types, indexes, partitions, and storage for a specific DBMS

### Q19. What is Data Modeling?
Data modeling is the broader practice of **defining how data is structured, related, and stored** so it supports business needs. It covers transactional modeling (ER/normalised) and analytical modeling (dimensional, Data Vault, One Big Table). Good models improve consistency, performance, and understanding.

---

## 3. Data Modeling

### Q20. What is Dimensional Data Modeling?
A modeling technique (Ralph Kimball) optimised for **analytics and reporting**. Data is split into:
- **Fact tables:** measurable business events (sales amount, quantity) with foreign keys to dimensions
- **Dimension tables:** descriptive context (customer, product, date, store)

Design steps (Kimball): choose the business process, declare the **grain**, identify dimensions, identify facts.

### Q21. What is a Star Schema?
A central **fact table** directly joined to **denormalised dimension tables**, forming a star shape.
- Fewer joins, fast queries, simple for BI users
- Some redundancy in dimensions

```
            dim_date
               |
dim_product -- fact_sales -- dim_customer
               |
            dim_store
```

### Q22. What is a Snowflake Schema?
A variation where dimensions are **normalised into sub-dimensions** (e.g. product → category → department).
- Less storage redundancy, easier dimension maintenance
- More joins, slower and more complex queries

### Q23. Star vs Snowflake

| Aspect | Star | Snowflake |
|---|---|---|
| Dimensions | Denormalised | Normalised |
| Joins | Fewer | More |
| Query speed | Faster | Slower |
| Storage | More | Less |
| Simplicity | Simple | More complex |

### Q24. What are Slowly Changing Dimensions (SCD)?
Dimension attributes change over time (customer address, employee role). SCD defines **how to handle those changes** in the warehouse.

| Type | Approach | History kept? |
|---|---|---|
| **Type 0** | Never change | N/A |
| **Type 1** | Overwrite old value | No |
| **Type 2** | Add a new row per change | Full history |
| **Type 3** | Add a "previous value" column | Limited (one prior value) |
| **Type 4** | Separate history table | Yes |
| **Type 6** | Hybrid of 1 + 2 + 3 | Yes |

### Q25. What is SCD Type 2?
Each change **inserts a new row** and closes out the old one, preserving full history. Typical columns: surrogate key, natural key, `effective_start_date`, `effective_end_date`, `is_current`.

| sk | cust_id | city | start_date | end_date | is_current |
|---|---|---|---|---|---|
| 1 | C100 | Kolkata | 2022-01-01 | 2024-06-30 | N |
| 2 | C100 | Pune | 2024-07-01 | 9999-12-31 | Y |

**Pros:** complete history, point-in-time reporting. **Cons:** table growth, more complex loads (needs a MERGE).

### Q26. What is SCD Type 3?
Adds a **column to hold the previous value** alongside the current one (e.g. `current_city`, `previous_city`). Only limited history (usually one prior value) is retained. Good when only the most recent change matters.

### Q27. What is a Surrogate Key? (Added)
A system-generated, meaningless unique identifier (usually an integer or hash) used as the dimension primary key instead of the business (natural) key. It is essential for SCD Type 2, insulates the warehouse from source key changes, and speeds up joins.

### Q28. What are the types of Fact Tables? (Added)
- **Transaction:** one row per event (every sale)
- **Periodic snapshot:** one row per entity per period (daily account balance)
- **Accumulating snapshot:** one row per process lifecycle, updated as milestones complete (order placed → shipped → delivered)
- **Factless:** records events with no numeric measure (student attended class)

### Q29. What is Data Vault? (Added)
A modeling method built for scalable, auditable, historised enterprise warehouses using three core structures: **Hubs** (business keys), **Links** (relationships), and **Satellites** (descriptive attributes with history). It is flexible to source changes and highly traceable, at the cost of complexity.

---

## 4. Storage Architectures

### Q30. What is a Data Warehouse?
A centralised, structured repository of **integrated, historical, subject-oriented, non-volatile** data optimised for analytics (Bill Inmon's definition). Uses schema-on-write: data is cleaned and modeled before loading.

Examples: Amazon Redshift, Snowflake, Google BigQuery, Azure Synapse.

### Q31. What are Data Warehouse Layers?
A typical layered design:

1. **Staging / Landing:** raw copy of source data, minimal change
2. **Integration / Core (Enterprise layer):** cleaned, conformed, deduplicated, historised
3. **Presentation / Semantic / Mart layer:** dimensional models, aggregates, business-friendly views for BI

Modern lake/lakehouse equivalents: Bronze → Silver → Gold (Medallion). In your own pipelines these are often named landing → raw → standardised → curated.

### Q32. What is a Data Lake?
A centralised repository storing **raw data of any format** (structured, semi-structured, unstructured) at low cost, usually on object storage (S3, ADLS, GCS). Uses **schema-on-read**: structure is applied when data is queried.

Risks: without governance, a lake becomes a "data swamp".

### Q33. Data Lake vs Data Warehouse

| Aspect | Data Lake | Data Warehouse |
|---|---|---|
| Data type | Any (raw) | Structured, processed |
| Schema | Schema-on-read | Schema-on-write |
| Cost | Cheap object storage | Higher |
| Users | Data scientists, engineers | Analysts, BI users |
| Flexibility | High | Lower |
| ACID / governance | Weak by default | Strong |
| Best for | ML, exploration, raw archive | Reporting, BI |

### Q34. What is a Data Lakehouse?
An architecture that **combines lake storage with warehouse capabilities**: cheap open-format storage plus ACID transactions, schema enforcement, indexing, and SQL performance, enabled by open table formats (Delta Lake, Iceberg, Hudi). One copy of data supports BI, ML, and streaming. Examples: Databricks, AWS (S3 + Iceberg/Athena/Redshift Spectrum), Snowflake with Iceberg.

### Q35. What is a Data Mart?
A **subset of a data warehouse** focused on a single business function or department (sales, finance, HR). Smaller, faster, and easier to govern. It can be dependent (built from the warehouse) or independent (built from sources directly).

### Q36. What is Data Mesh?
A **decentralised, organisational and architectural approach** (Zhamak Dehghani) built on four principles:
1. **Domain ownership:** business domains own their data
2. **Data as a product:** data is treated with product thinking (SLAs, docs, quality)
3. **Self-serve data platform:** shared infrastructure enabling domain teams
4. **Federated computational governance:** global standards, locally enforced

### Q37. What is Data Fabric?
A **technology-driven, metadata-centric architecture** that provides a unified, integrated layer over distributed data across clouds and on-prem. It uses active metadata, knowledge graphs, and automation to discover, integrate, govern, and deliver data.

**Mesh vs Fabric:** Mesh is primarily an organisational/people model (decentralised ownership); Fabric is primarily a technology/automation approach (centralised intelligence over metadata). They can be complementary.

---

## 5. File Formats

### Q38. Row-Based vs Column-Based File Formats

| Aspect | Row-based | Column-based |
|---|---|---|
| Layout | Values of a row stored together | Values of a column stored together |
| Best for | Write-heavy, full-row reads, streaming | Analytical reads of few columns |
| Compression | Lower | Excellent (similar values adjacent) |
| Query speed (analytics) | Slower | Much faster (column pruning, predicate pushdown) |
| Examples | CSV, JSON, Avro | Parquet, ORC |

### Q39. What is CSV?
**Comma-Separated Values:** plain-text, row-based, human-readable tabular format.
- Pros: universal, simple, easy to inspect
- Cons: no schema or data types, no nested data, delimiter/quoting issues, poor compression, slow at scale

### Q40. What is JSON?
**JavaScript Object Notation:** text-based, row-oriented, **semi-structured** format with key-value pairs and nesting. Ubiquitous for APIs and event data. Cons: verbose, no enforced schema, slow to scan at scale. `JSON Lines` (one object per line) is preferred for big data.

### Q41. What is Avro?
A **row-based binary** format from Apache with the **schema embedded** in the file (defined in JSON).
- Excellent **schema evolution** support (add/remove fields with defaults)
- Compact and fast to write
- Standard for **Kafka** messages (with Schema Registry) and streaming ingestion

### Q42. What is Parquet?
A **columnar binary** format (Apache) built for analytics.
- Column pruning and predicate pushdown via row-group statistics
- High compression and encoding (dictionary, RLE)
- Supports nested data and schema
- The default for Spark, Athena, Redshift Spectrum, and most lakehouse tables

### Q43. What is ORC? (Added)
**Optimized Row Columnar:** another columnar format, originating in the Hive ecosystem, with built-in indexes and good compression. Common in Hive-based stacks; Parquet is more widely adopted elsewhere.

### Q44. Which format should I choose? (Added)
- **Streaming / ingestion / Kafka:** Avro
- **Analytics / lake storage:** Parquet (inside Delta/Iceberg/Hudi)
- **Interchange with humans / small exports:** CSV
- **APIs / semi-structured events:** JSON, then convert to Parquet on ingest

---

## 6. Open Table Formats & Lakehouse Tech

### Q45. What is an Open Table Format?
A metadata layer on top of files (usually Parquet) in object storage that makes a collection of files behave like a **database table**. It provides:
- ACID transactions
- Schema evolution and enforcement
- Time travel (query previous versions)
- Efficient upserts, deletes, and merges
- Partition evolution and file-level statistics

Major formats: **Delta Lake**, **Apache Iceberg**, **Apache Hudi**.

### Q46. What is Delta Lake?
An open-source storage layer (created by Databricks) that adds a **transaction log** to Parquet files on object storage, delivering ACID transactions, scalable metadata, time travel, schema enforcement/evolution, `MERGE`/`UPDATE`/`DELETE`, and unified batch + streaming.

### Q47. What is a Transaction Log?
In general: a sequential record of all changes made to a database or table, used for recovery, auditing, and replication.

In **Delta Lake**: the `_delta_log/` directory holds ordered JSON commit files (plus periodic Parquet checkpoints) recording every add/remove file action. The log is the **single source of truth** for table state, enabling atomic commits, optimistic concurrency, and time travel.

### Q48. Delta vs Iceberg vs Hudi (Added)

| Aspect | Delta Lake | Apache Iceberg | Apache Hudi |
|---|---|---|---|
| Origin | Databricks | Netflix | Uber |
| Metadata | JSON log + checkpoints | Manifest lists / manifests / snapshots | Timeline + metadata table |
| Strength | Databricks/Spark integration | Engine neutrality, partition evolution | Upserts and incremental pulls, CDC-friendly |
| Table types | Single | Single | Copy-on-Write and Merge-on-Read |

### Q49. What is Delta Live Tables (DLT)?
A Databricks **declarative framework for building reliable data pipelines**. You define *what* the tables should contain (SQL or Python); DLT manages orchestration, dependencies, incremental processing, retries, autoscaling, and **data quality expectations** (e.g. drop or fail rows violating rules). Databricks has since been evolving it into *Lakeflow Spark Declarative Pipelines*, so check current docs for naming.

### Q50. What is Lakebase?
Databricks' **managed, serverless PostgreSQL-compatible OLTP database** integrated with the lakehouse (based on technology from the Neon acquisition). It lets applications run transactional workloads on Postgres while syncing data with lakehouse tables, bridging OLTP and OLAP without separate pipelines.

---

## 7. Processing Patterns & Architectures

### Q51. What is ETL (Extract, Transform, Load)?
1. **Extract** data from sources
2. **Transform** it in a separate processing engine (clean, join, aggregate)
3. **Load** the final result into the target (usually a warehouse)

Suited to: strict schemas, sensitive data masked before loading, limited target compute. Tools: Informatica, Talend, AWS Glue, SSIS.

### Q52. What is ELT (Extract, Load, Transform)?
1. **Extract** from sources
2. **Load** raw data directly into the target
3. **Transform** inside the target using its compute (SQL, dbt)

Suited to: cloud warehouses/lakehouses with scalable compute, keeping raw data for reprocessing.

### Q53. ETL vs ELT

| Aspect | ETL | ELT |
|---|---|---|
| Transform location | Separate engine | Inside target platform |
| Raw data retained | Often not | Yes |
| Flexibility | Lower | Higher |
| Best with | On-prem, legacy | Cloud warehouse / lakehouse |
| Typical tools | Informatica, Glue, SSIS | dbt, Snowflake, BigQuery, Databricks SQL |

### Q54. What is Batch Processing?
Processing **bounded, accumulated data** at scheduled intervals (hourly, daily). High throughput, simpler, cost-efficient, but with latency. Tools: Spark, Glue, Airflow-scheduled SQL.

### Q55. What is Streaming Data Processing?
Processing **unbounded data continuously, event by event or in micro-batches**, with low latency (seconds or less). Use cases: fraud detection, real-time dashboards, IoT, personalization. Tools: Kafka, Kafka Streams, Flink, Spark Structured Streaming, Kinesis.

### Q56. What is Lambda Architecture?
Proposed by Nathan Marz; it runs **two parallel paths** over the same data:
- **Batch layer:** accurate, complete reprocessing of all historical data
- **Speed layer:** low-latency, approximate real-time views
- **Serving layer:** merges both for queries

**Downside:** two codebases to maintain and keep consistent.

### Q57. What is Kappa Architecture?
Proposed by Jay Kreps; a **streaming-only** simplification of Lambda. All data flows through a single stream-processing pipeline (e.g. Kafka + Flink). Historical reprocessing is done by **replaying the log** through the same code.

**Pros:** one codebase. **Cons:** requires long retention of the event log; heavy reprocessing can be costly.

### Q58. Lambda vs Kappa

| Aspect | Lambda | Kappa |
|---|---|---|
| Paths | Batch + speed | Streaming only |
| Code | Two codebases | One |
| Reprocessing | Batch layer | Replay the stream |
| Complexity | Higher | Lower |

### Q59. What is Medallion Architecture?
A lakehouse design pattern (popularised by Databricks) that organises data into **progressive quality layers**:

- **Bronze:** raw, as-ingested data. Append-only, minimal change, full history and lineage (often with ingestion metadata).
- **Silver:** cleaned, deduplicated, validated, conformed data. Schema enforced, joined/enriched; the "enterprise view".
- **Gold:** curated, aggregated, business-level data modeled for consumption (BI, ML, reporting; star schemas, KPIs).

Benefits: clear data quality progression, easy reprocessing from Bronze, separation of concerns, and better governance.

### Q60. What is the Bronze Stage?
The ingestion layer: raw data stored exactly as received (CSV, JSON, CDC events), usually append-only, plus metadata such as load time and source file. It acts as the replayable source of truth.

### Q61. What is the Silver Stage?
The refined layer: data is cleansed, typed, deduplicated, validated, and joined. Business keys are conformed, and CDC changes are often merged here (e.g. SCD1/SCD2 logic).

### Q62. What is the Gold Stage?
The consumption layer: aggregated, denormalised, business-ready tables (fact/dimension models, KPI tables, feature tables) optimised for reporting and analytics.

---

## 8. Pipeline Design Concepts

### Q63. What is Incremental (Data) Loading?
Loading **only new or changed records** since the last run rather than reloading the entire dataset (full load). Methods:
- **Timestamp / watermark column** (`updated_at > last_run`)
- **Auto-increment ID** high-water mark
- **CDC** from database logs
- **Hash/diff comparison** of rows
- **Partition-based** (load only today's partition)

Benefits: faster, cheaper, less load on sources. Challenges: handling deletes, late data, and missed updates.

### Q64. What is Back-date Refresh (Backfill)?
Re-running or reloading data for **past time periods**, typically because of late-arriving data, a bug fix, new logic, or onboarding a new table. Pipelines should be parameterised by date (e.g. Airflow `catchup`, `logical_date`) and idempotent so backfills do not create duplicates.

### Q65. What is Idempotency?
An operation is idempotent if **running it multiple times produces the same result as running it once**. This is critical for safe retries and backfills.

Techniques:
- `MERGE`/upsert instead of blind `INSERT`
- Overwrite a specific partition (`INSERT OVERWRITE`)
- Delete-then-insert within a transaction for the batch window
- Deterministic outputs and unique keys

### Q66. What is "Exactly Once"?
Delivery/processing semantics:

| Semantic | Meaning | Risk |
|---|---|---|
| **At-most-once** | Message processed 0 or 1 time | Data loss |
| **At-least-once** | Processed 1 or more times | Duplicates |
| **Exactly-once** | Effect applied exactly one time | Hardest to achieve |

Exactly-once is usually achieved as **at-least-once delivery + idempotent/transactional processing** (Kafka idempotent producers and transactions, Flink checkpointing, Delta/Iceberg atomic commits).

### Q67. What are Watermarks?
In streaming, a watermark is a **threshold on event time** stating "no events older than this are expected." It lets the engine decide when a time window is complete, how long to wait for **late data**, and when to drop old state.

In Spark: `.withWatermark("event_time", "10 minutes")`.

*(In batch incremental loading, "watermark" also refers to the stored high-water mark, such as the last processed timestamp.)*

### Q68. What is Upsert?
**Update + Insert:** if the key exists, update the row; otherwise insert it. Implemented as `MERGE INTO` (Delta, Iceberg, Redshift, Snowflake, SQL Server), `INSERT ... ON CONFLICT` (PostgreSQL), or `ON DUPLICATE KEY UPDATE` (MySQL).

```sql
MERGE INTO target t
USING source s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET t.name = s.name, t.updated_at = s.updated_at
WHEN NOT MATCHED THEN INSERT (id, name, updated_at) VALUES (s.id, s.name, s.updated_at);
```

### Q69. What is CDC (Change Data Capture)?
CDC identifies and captures **inserts, updates, and deletes** from a source database and delivers them downstream, usually in near real time.

Approaches:
- **Log-based** (reads WAL/binlog/redo log): low overhead, captures deletes. *Preferred.* Tools: Debezium, AWS DMS, Oracle GoldenGate, Fivetran.
- **Trigger-based:** DB triggers write to audit tables; adds source overhead.
- **Query-based:** poll with timestamps/version columns; misses hard deletes.

CDC events usually carry an operation flag (`I/U/D`), before/after images, and a commit timestamp or sequence number.

---

## 9. Big Data & Orchestration Tools

### Q70. What is Big Data Engineering?
Engineering practice for data that exceeds the capacity of traditional single-node systems, characterised by the **V's**: Volume, Velocity, Variety, Veracity, Value. It uses distributed storage (HDFS, S3) and distributed compute (Spark, Flink, Hive) to process data at scale.

### Q71. What is Distributed Computing?
Splitting a workload across **multiple machines (nodes)** that work in parallel and coordinate to act as one system. Gains: scalability, throughput, and fault tolerance. Challenges: network overhead, data skew, consistency, partial failures.

### Q72. What is Apache Spark?
An open-source, **in-memory distributed data processing engine** supporting batch, SQL, streaming, machine learning (MLlib), and graph workloads through APIs in Python (PySpark), Scala, Java, SQL, and R. Much faster than MapReduce due to in-memory computation and DAG optimisation.

### Q73. Spark Architecture
- **Driver:** runs the main program, creates the `SparkContext/SparkSession`, builds the DAG, schedules tasks, and coordinates executors.
- **Cluster Manager:** allocates resources (Standalone, YARN, Kubernetes, Mesos).
- **Executors:** worker JVM processes that run tasks and cache data.
- **Tasks / Stages / Jobs:** an **action** triggers a **job**, split into **stages** at shuffle boundaries, each made of parallel **tasks** (one per partition).

Key concepts:
- **RDD → DataFrame → Dataset** abstractions
- **Lazy evaluation:** transformations build a plan; actions execute it
- **Catalyst optimizer** and **Tungsten** engine optimise queries
- **Narrow vs wide transformations:** wide ones (`groupBy`, `join`) cause a **shuffle**

### Q74. What is Apache Airflow?
An open-source **workflow orchestration platform** where pipelines are defined as **DAGs (Directed Acyclic Graphs) in Python**.
- Components: Scheduler, Webserver/UI, Executor, Workers, Metadata DB
- Concepts: DAG, Task, Operator, Sensor, Hook, XCom, Connection, Pool, Variable
- Features: scheduling, retries, dependencies, backfills, alerting, rich integrations (AWS, Spark, dbt)

Airflow **orchestrates**; it should not do heavy data processing itself.

### Q75. What is Apache Kafka?
A distributed, fault-tolerant **event streaming platform** (publish-subscribe log).
- **Topic:** named stream, split into **partitions** (unit of parallelism and ordering)
- **Producer / Consumer / Consumer Group**
- **Broker** and cluster, replication for durability
- **Offsets** track consumer progress; messages are retained for a configurable period
- Ecosystem: Kafka Connect (integration), Kafka Streams, ksqlDB, Schema Registry

Use cases: event-driven systems, CDC pipelines, log aggregation, real-time analytics.

### Q76. What is Apache Hive?
A **SQL-on-Hadoop data warehouse layer** that lets you query large datasets in distributed storage using HiveQL, translated to MapReduce, Tez, or Spark jobs. The **Hive Metastore** (table schemas, partitions, locations) is still the de facto metadata catalog used by Spark, Trino, Presto, Athena, and Glue. Hive supports external tables, partitioning, and bucketing.

### Q77. What is Trino?
A distributed **SQL query engine** (fork of Presto) for fast, interactive analytics across many sources (S3, Hive, Iceberg, Delta, MySQL, Kafka, etc.) without moving data. It separates compute from storage, runs MPP-style in memory, and supports **federated queries** (joins across systems). AWS Athena is built on Presto/Trino.

### Q78. What is dbt?
**Data Build Tool:** a transformation framework for the **"T" in ELT**. Analysts and engineers write modular **SQL `SELECT` statements (models)**, and dbt compiles, orders (via `ref()` dependencies), and runs them in the warehouse.

Features: Jinja templating, incremental models, snapshots (SCD2), tests (unique, not null, relationships), documentation, lineage graph, version control and CI integration.

### Q79. What is Apache Flink? (Added)
A distributed stream-processing framework with true event-at-a-time processing, event-time semantics, stateful computation, and exactly-once guarantees via checkpointing. Often chosen over Spark Structured Streaming for very low latency and complex stateful logic.

### Q80. Airflow vs Kafka vs Spark (Added)

| Tool | Role |
|---|---|
| Airflow | Orchestrates *when* and *in what order* jobs run |
| Spark | Processes data (compute) |
| Kafka | Transports/streams events (messaging) |
| Hive/Trino | Query layers and metadata |

---

## 10. Cloud Data Engineering

### Q81. What is Cloud Computing?
On-demand delivery of computing resources (servers, storage, databases, networking, analytics) over the internet with **pay-as-you-go** pricing.

- **Service models:** IaaS (EC2), PaaS (Glue, Elastic Beanstalk), SaaS (Salesforce), plus serverless (Lambda)
- **Deployment models:** public, private, hybrid, multi-cloud
- **Benefits:** elasticity, scalability, no upfront hardware cost, global reach, managed services
- Major providers: AWS, Microsoft Azure, Google Cloud

### Q82. What is Cloud Data Engineering?
Building data platforms and pipelines using **cloud-native, managed services** rather than self-managed hardware. Emphasises decoupled storage and compute, serverless processing, elasticity, and cost optimisation (FinOps).

### Q83. Cloud data services map

| Need | AWS | Azure | GCP |
|---|---|---|---|
| Object storage | S3 | ADLS Gen2 / Blob | Cloud Storage |
| ETL / Spark | Glue, EMR | Data Factory, Synapse, Databricks | Dataflow, Dataproc |
| Warehouse | Redshift | Synapse | BigQuery |
| Query on lake | Athena, Redshift Spectrum | Synapse Serverless | BigQuery external |
| Streaming | Kinesis, MSK | Event Hubs | Pub/Sub |
| Serverless compute | Lambda | Functions | Cloud Functions |
| Orchestration | MWAA, Step Functions | Data Factory | Cloud Composer |
| Messaging | SNS, SQS | Service Bus | Pub/Sub |
| Catalog | Glue Data Catalog | Purview | Dataplex |

### Q84. Typical AWS data pipeline (Added)
Source DB → **DMS** (CDC) → **S3** landing (Parquet/Avro) → **Glue/Spark** transforms (raw → standardised → curated) → **Hudi/Iceberg/Delta** tables → **Glue Catalog** → **Athena / Redshift Spectrum** → BI. Orchestrated by **Airflow (MWAA)**, with **SNS/SQS** for event notifications and alerts, and **Lambda** for lightweight triggers.

---

## 11. DevOps for Data: CI/CD & DataOps

### Q85. What is CI/CD?
- **Continuous Integration:** developers frequently merge code to a shared repo; each change triggers automated build, lint, and tests.
- **Continuous Delivery/Deployment:** validated changes are automatically released (delivery = ready to deploy with manual approval; deployment = fully automatic).

In data engineering: unit tests for transformations, data-quality tests, DAG validation, IaC (Terraform) plans, and automated deploys of Glue jobs, Airflow DAGs, or dbt projects across dev → test → prod. Tools: GitHub Actions, GitLab CI, Jenkins, Azure DevOps.

### Q86. What is DataOps?
A methodology applying **DevOps, Agile, and lean principles to data analytics**. Goals: faster delivery, higher quality, and reliable, collaborative data workflows.

Practices:
- Version control for code, SQL, and configs
- CI/CD for pipelines
- Automated data testing and quality checks
- Monitoring, observability, and alerting
- Environment management and Infrastructure as Code
- Collaboration between engineers, analysts, and business

### Q87. What is Infrastructure as Code (IaC)? (Added)
Defining and provisioning infrastructure through version-controlled declarative code (Terraform, CloudFormation, Pulumi). Benefits: reproducibility, review, drift detection, and consistent environments.

---

## 12. Data Reporting

### Q88. What is Data Reporting?
The process of **collecting, organising, and presenting data in a readable form** (tables, charts, dashboards) to support decisions. Types:
- **Operational reports:** day-to-day (orders today)
- **Analytical / BI dashboards:** trends, KPIs, drill-downs
- **Ad hoc reports:** one-off exploratory questions
- **Regulatory/compliance reports:** mandated filings

Tools: Power BI, Tableau, Looker, Amazon QuickSight, Superset, Metabase. A **semantic layer** keeps metric definitions consistent ("single source of truth").

---

## 13. Additional Industry Questions (Added)

### Q89. What is Partitioning, and how is it different from Bucketing?
- **Partitioning:** physically splits data into directories by column values (e.g. `dt=2026-10-03`). Enables **partition pruning**. Use low-to-medium cardinality columns (date, region).
- **Bucketing:** hashes a column into a fixed number of files. Helps joins and sampling on high-cardinality keys, and avoids shuffles.

Over-partitioning creates the **small files problem**.

### Q90. What is the Small Files Problem?
Too many tiny files hurt performance (metadata overhead, slow listing, excessive tasks). Fix with **compaction** (`OPTIMIZE` in Delta, `rewrite_data_files` in Iceberg, Hudi clustering/compaction), sensible partitioning, and larger write batches (target ~128 MB to 1 GB files).

### Q91. What is Data Skew and how do you handle it?
Uneven distribution of data across partitions so a few tasks do most of the work. Mitigations: **salting** keys, **broadcast joins** for small tables, repartitioning, Spark **AQE** (adaptive query execution) skew-join handling, and filtering null/hot keys.

### Q92. What is a Shuffle in Spark?
Redistribution of data across the cluster (disk and network I/O) required by wide transformations (`groupBy`, `join`, `distinct`, `repartition`). It is the most expensive Spark operation; minimise it using broadcast joins, early filtering, and partition-aware design.

### Q93. Broadcast Join vs Sort-Merge Join
- **Broadcast hash join:** a small table is sent to all executors; no shuffle of the large table. Fast.
- **Sort-merge join:** both sides are shuffled and sorted on the key; scales to large tables.

### Q94. What is Data Quality and what are its dimensions?
Fitness of data for its intended use. Dimensions: **accuracy, completeness, consistency, timeliness, uniqueness, validity**. Implement via checks at each layer (null counts, uniqueness, referential integrity, freshness, volume anomalies). Tools: Great Expectations, Deequ, Soda, dbt tests, DLT expectations.

### Q95. What is Data Lineage?
A map of where data came from, how it was transformed, and where it flows. Supports impact analysis ("what breaks if I change this?"), debugging, audits, and compliance. Tools: OpenLineage, Marquez, DataHub, Collibra, Unity Catalog.

### Q96. What is a Data Catalog and Metadata Management?
A searchable inventory of data assets with **technical** (schema, location), **business** (definitions, owners), and **operational** (freshness, usage) metadata. Examples: AWS Glue Data Catalog, Unity Catalog, DataHub, Alation, Collibra.

### Q97. What is Data Governance?
Policies, roles, and processes ensuring data is **secure, high quality, compliant, and well managed**: ownership/stewardship, access control (RBAC/ABAC), masking, encryption, retention, classification, and regulatory compliance (GDPR, India's DPDP Act, HIPAA).

### Q98. What is Data Observability?
Continuous monitoring of data health across five pillars: **freshness, volume, schema, distribution, and lineage**. Detects broken pipelines and silent data issues before consumers do. Tools: Monte Carlo, Datadog, Elementary, Soda.

### Q99. What is Schema Evolution vs Schema Enforcement?
- **Schema enforcement (validation):** rejects writes that do not match the table schema.
- **Schema evolution:** controlled changes (add column, widen type) without rewriting data. Supported by Avro, Parquet, Delta, Iceberg, and Hudi.

### Q100. What is a Data Contract?
A formal, versioned agreement between data producers and consumers covering schema, semantics, quality expectations, SLAs, and ownership. It prevents upstream changes from silently breaking downstream consumers.

### Q101. What is Time Travel?
Querying a table as it existed at an earlier version or timestamp (Delta, Iceberg, Hudi, Snowflake). Useful for audits, rollback, reproducibility, and debugging.

### Q102. What is the CAP Theorem?
A distributed system can guarantee only two of three during a network partition: **Consistency, Availability, Partition tolerance**. Because partitions are unavoidable, the real trade-off is **CP vs AP**. Example: HBase/MongoDB lean CP; Cassandra/DynamoDB lean AP (tunable).

### Q103. What are Indexes and when are they useful?
Data structures (B-tree, hash, bitmap) that speed up lookups at the cost of extra storage and slower writes. Common in OLTP. Columnar warehouses rely instead on sort keys/clustering, zone maps, and partition pruning.

### Q104. Common SQL window functions asked in interviews
`ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()/LEAD()`, `SUM() OVER (PARTITION BY ... ORDER BY ...)`. Frequently used for deduplication:

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY id ORDER BY updated_at DESC) AS rn
  FROM raw_table
) WHERE rn = 1;
```

### Q105. What is a Dead Letter Queue (DLQ)?
A holding queue/location for messages or records that failed processing after retries (bad schema, poison messages). It prevents one bad record from blocking the pipeline and enables later inspection and reprocessing. Examples: SQS DLQ, Kafka DLQ topic.

### Q106. What is Push vs Pull ingestion?
- **Push:** the source sends data to the platform (webhooks, producers to Kafka).
- **Pull:** the platform fetches data on a schedule (API polling, JDBC extracts).

### Q107. Orchestration vs Choreography
- **Orchestration:** a central controller (Airflow, Step Functions) defines and triggers the workflow.
- **Choreography:** services react to events independently (SNS/SQS/Kafka event-driven), with no central coordinator.

### Q108. What is Master Data Management (MDM)?
Creating a single, trusted "golden record" for core business entities (customer, product, supplier) across systems through matching, deduplication, and survivorship rules.

### Q109. What is Reverse ETL?
Syncing curated warehouse/lakehouse data **back into operational tools** (CRM, ad platforms, support systems) so business teams can act on it. Tools: Census, Hightouch.

### Q110. What is a Feature Store?
A central repository for managing, sharing, and serving ML features consistently for both training (offline) and inference (online), avoiding training-serving skew.

### Q111. How do you design a CDC pipeline into a lakehouse? (Added: common system design question)
1. Capture changes from source logs (DMS/Debezium) into a **landing** zone (Parquet/Avro with op flag and commit timestamp).
2. Ingest to **raw/bronze** append-only for replay and audit.
3. In **standardised/silver**, deduplicate by key using the latest commit timestamp, then **merge** (upsert/delete) into a Hudi/Delta/Iceberg table (SCD1 for current state, SCD2 for history).
4. Build **curated/gold** marts and register in the catalog for Athena/Redshift Spectrum.
5. Orchestrate with Airflow; ensure idempotency, data-quality checks, late-data handling, schema-drift alerts, and monitoring.

### Q112. How do you optimise a slow pipeline? (Added)
- Measure first: Spark UI, query plans, stage/task times
- Reduce data early: column pruning, predicate pushdown, partition pruning
- Use columnar formats and compaction
- Fix skew and minimise shuffles; use broadcast joins
- Right-size executors, memory, and parallelism; enable AQE
- Cache only what is reused; avoid unnecessary actions
- Process incrementally instead of full reloads

---

## Quick Revision Cheat Sheet

| Concept | One-liner |
|---|---|
| Data Engineering | Build systems that move and shape data for use |
| OLTP / OLAP | Run the business / analyse the business |
| ETL / ELT | Transform before load / load then transform |
| Star schema | Fact table + denormalised dimensions |
| SCD2 | New row per change, full history |
| Lake / Warehouse / Lakehouse | Raw cheap / curated fast / both with ACID |
| Parquet / Avro | Columnar for analytics / row for streaming |
| Delta / Iceberg / Hudi | ACID table layers over Parquet |
| Medallion | Bronze (raw) → Silver (clean) → Gold (business) |
| Lambda / Kappa | Batch + speed / stream only |
| Idempotency | Re-running gives the same result |
| Watermark | Late-data threshold in streaming |
| CDC | Capture inserts/updates/deletes from source logs |
| Upsert | Update if exists, insert if not |
| DataOps | DevOps principles for data pipelines |

---

*End of document.*
