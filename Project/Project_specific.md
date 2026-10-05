# Data Engineering Interview Preparation — Project & Scenario-Based Q&A

## Arunava Roy | 3–5 Years Data Engineering Interview Level

> **Purpose:** This is a polished interview-ready version of the project questions from the source document.
> The answers are written in a **3–5 years Data Engineer voice**: practical, hands-on, structured, and confident without sounding like a senior architect.
>
> **Answering pattern:** Direct answer → approach → project context → validation → result/trade-off.
>
> **Important:** Do not memorize every sentence. Memorize the structure and key points, then answer naturally.

> **What changed in this version:** the bank is de-duplicated (overlapping questions are merged into one master answer each), the gaps are filled (Hudi internals, Airflow, Redshift, SNS/SQS/Lambda/EventBridge, Terraform and CI/CD, Glue and Spark specifics), and the 30 requested questions are all covered. **Your experience and project-explanation answers are unchanged.**
>
> **⚠ Verify markers:** where an answer uses a plausible scenario or ballpark number built from your pipeline architecture (not from your records), it is flagged **⚠ Verify before interview**. Replace those details with what really happened. A full checklist is at the end.

---

# Coverage of the 30 Requested Questions

1. How do you design a pipeline for billions of records? → **Q15**
2. How do you handle late-arriving data? → **Q54**
3. How do you manage schema evolution? → **Q57**
4. How do you design incremental loads? → **Q56**
5. How do you handle duplicate records? → **Q60**
6. How do you debug a failed production pipeline? → **Q16**
7. How do you optimize a slow Spark job? → **Q24**
8. How do you handle data skew? → **Q28**
9. When will you use broadcast join? → **Q29**
10. How do you design CDC processing? → **Q39**
11. How do you manage small file problems? → **Q36**
12. How do you choose partition columns? → **Q35**
13. How do you improve SQL query performance? → **Q68**
14. How do you ensure data quality? → **Q59**
15. How do you design retry and failure handling? → **Q17**
16. How to make your pipeline idempotent? → **Q18**
17. How do you tackle schema changes? → **Q57**
18. How to check and tackle data quality issues? → **Q59**
19. What production issues have you handled and how? → **Q90**
20. What was your biggest issue and what did you learn? → **Q91**
21. How do you tackle underutilized clusters? → **Q32**
22. How do you handle OOM / memory pressure issues? → **Q31**
23. How to detect data drift and tackle it? → **Q58**
24. How do you check and handle external API failures? → **Q21**
25. How to check and handle file corruption? → **Q20**
26. How to tackle data duplication throughout your zones? → **Q60**
27. How do you plan for disaster recovery? → **Q22**
28. How do you prioritize deliverables across multiple tasks? → **Q92**
29. What enhancements would you suggest for your pipeline? → **Q93**
30. What is zero-ETL and how can it apply to your project? → **Q88**

---

# 1. Project Introduction & Ownership

## Q1. Tell me about yourself.

### How to tackle
Keep this to around 60–90 seconds: experience, Data Engineering focus, AWS/Spark/Airflow, DevOps exposure, actual ownership, target role.

### Model answer
> “Hi, I’m Arunava. I’ve been working with Zensar Technologies for around four years, with most of my hands-on experience in Data Engineering.
>
> I started in the Data Engineering track and worked mainly with the AWS data ecosystem, including S3, Glue, Lambda, Redshift and Athena, along with Airflow for orchestration. I also got exposure to CDC-based data processing and Spark-based transformations.
>
> Alongside Data Engineering, I worked with DevOps tools such as Terraform and GitHub Actions, mainly around infrastructure provisioning and CI/CD.
>
> In my current project, I work on building and maintaining data pipelines where data comes from multiple upstream systems, lands in S3, goes through Raw, Standardized and Curated layers, and is finally consumed by downstream teams.
>
> My strongest interest is in the Data Engineering side, particularly building reliable pipelines, optimizing Spark workloads, and working with AWS data services.”

### Follow-ups / traps
- How much did you personally implement?
- Which AWS service did you use most?
- Why are you moving toward Data Engineering?
- How much Spark did you actually use?

---

## Q2. Which AWS services have you worked with?

### How to tackle
Group services by purpose instead of listing them randomly.

### Model answer
> “I’ve worked across storage, processing, orchestration, messaging and deployment.
>
> For storage and analytics, I’ve mainly worked with S3, Redshift and Athena. For ETL and distributed processing, I’ve worked with AWS Glue and Spark. For orchestration, we use Managed Airflow.
>
> For event-driven communication, I’ve worked with services such as Lambda, EventBridge, SNS and SQS. On the infrastructure side, I have exposure to IAM, Terraform and GitHub Actions for CI/CD.
>
> So my primary Data Engineering stack has been S3, Glue, Spark, Airflow, Athena and Redshift, with the other AWS services supporting orchestration, integration and deployment.”

### Follow-ups / traps
- Why Glue instead of EMR?
- Why Athena instead of Redshift?
- Where did Lambda fit?
- Why use SQS?
- How did Terraform help?

---

## Q3. What was your individual contribution to the project?

### How to tackle
Separate team architecture from your own ownership.

### Model answer
> “The high-level architecture was defined by our architects and leads, but my responsibility was more on the implementation and end-to-end delivery side.
>
> If a business domain or table needed to be onboarded, I worked on the Glue processing logic, CDC handling, Standardized-layer transformations, Airflow orchestration and production deployment.
>
> I was particularly involved with the Inventory and Items domain, including tables such as Item_master, Item_location, promotion_items and inventory_on_hand.
>
> I also worked on troubleshooting failed pipelines, Spark optimization, data-quality issues and deployment-related activities using Terraform and GitHub Actions.
>
> So I would describe my role as taking a defined requirement or architecture and owning the implementation through testing, deployment and production support.”

### Follow-ups / traps
- Which pipeline did you personally build?
- What problem did you personally solve?
- What was designed by the architect versus implemented by you?

---

# 2. End-to-End Architecture

## Q4. Explain your data flow end-to-end.

### How to tackle
Use: **Upstream → Landing → Raw → Standardized → Curated → Consumers**.

### Model answer
> “My project is in the retail and e-commerce domain. The objective is to prepare reliable data that can be consumed by teams such as Data Science, Operations, Marketing and Purchasing.
>
> The pipeline is divided mainly into four layers.
>
> **First is Landing.** An integration team receives data from upstream systems and delivers files and CDC feeds into S3. At this stage, we perform checks such as file arrival, filename or batch validation and basic structural validation. We also register metadata in the Glue Data Catalog, and the data can be queried through Athena.
>
> **Second is Raw.** We process the validated Landing data and add technical metadata such as ingestion timestamps and required primary-key information. We retain the source information while converting it into a format suitable for downstream processing and write it to the Raw S3 layer.
>
> **Third is Standardized.** This is where most of the CDC processing happens. We use AWS Glue with Spark and Apache Hudi. We handle inserts, updates and deletes, deduplicate records and use the business key and update timestamp to determine the latest state. This gives us an SCD Type-1 style current-state representation.
>
> **Finally, Curated.** We expose the Standardized data through Redshift Spectrum and apply business-specific transformations. The final datasets are exposed through Materialized Views for downstream consumers.
>
> The overall flow is automated and orchestrated using Managed Airflow.”

### Follow-ups / traps
- Why four layers?
- Why Hudi in Standardized?
- Why Spectrum in Curated?
- Why not put business logic in Raw?
- What happens when a source record is deleted?
- How do you rerun a failed batch?

---

## Q5. Why did you separate Raw, Standardized and Curated?

### Model answer
> “The main reason is separation of responsibilities and easier recovery.
>
> In Raw, we preserve the ingested data with technical metadata and avoid applying heavy business logic.
>
> In Standardized, we clean, deduplicate and apply CDC logic so that we have a reliable standardized representation.
>
> Curated is consumer-oriented. This is where we apply business transformations and expose data in a form that is easier for downstream teams to consume.
>
> This separation also helps troubleshooting. If a curated output is wrong, I can trace the issue back through the Standardized, Raw and Landing layers instead of having everything mixed together.”

---

## Q6. What was the data volume?

### Model answer
> “During our batch processing, the volume was around 7 to 10 GB per run, with the project processing data across multiple upstream interfaces and many tables.
>
> The challenge was not only the raw volume. We also had around 17 upstream interfaces and roughly 57 core tables, with multiple pipelines running concurrently and CDC updates that needed to be processed within the required processing window.
>
> So the important part was managing concurrency, incremental processing, partitioning and Spark performance rather than simply processing one large file.”

### Follow-ups / traps
- How many records?
- What was the largest table?
- What was the execution time?
- What happened during peak volume?

---

## Q7. How frequently did data arrive?

### Model answer
> “The upstream integration team delivered the raw data to our Landing S3 area on a daily basis.
>
> After the daily drop arrived, our processing was divided into smaller hourly chunks. So I would distinguish between **upstream delivery frequency**, which was daily, and **downstream processing frequency**, which was organized into hourly processing windows.
>
> This helped us manage the processing load and state across multiple tables instead of trying to process the entire daily drop as one large workload.”

### Follow-up: Why not process everything once daily?
> “Breaking the processing into smaller units gave us better control over workload, failures and concurrency. A failure in one processing unit was easier to isolate and recover than restarting one very large workload.”

---

# 3. Pipeline Scale, Concurrency & Sizing

## Q8. How many pipelines/jobs were running concurrently?

### Model answer
> “The project had around 15 to 20 pipelines running concurrently depending on the workload.
>
> Independent interfaces could run in parallel to meet the SLA. However, when pipelines were dependent on each other, or when multiple jobs could write to the same Hudi table, we controlled the execution order and avoided unsafe concurrent writes.
>
> Airflow was used to manage these dependencies and scheduling.”

---

## Q9. What was your team size?

### Model answer
> “The overall team was around 14 people. We had an Engineering Manager, Tech Lead, three Data Architects and around nine Data Engineers.
>
> The architects focused on the high-level design and data model, while the engineering team handled implementation, testing, deployment and production support.
>
> My ownership was primarily within the Data Engineering implementation team, particularly around Inventory and Items-related pipelines.”

---

## Q10. What was the pipeline success rate?

### How to tackle
Give a defensible range, explain what the failures were, and say how you measured it. Never quote a precise figure you cannot back up.

### Model answer
> “Most hourly runs completed on the first attempt, roughly 97–98%. I looked at it through the Airflow run history rather than a single dashboard number.
>
> The failures were mostly source-related rather than logic bugs: late or missing files, malformed records, and the occasional transient AWS error such as throttling.
>
> The recovery pattern was always the same: find the failed stage, decide whether it was transient or data-related, fix or retry, and rerun only the affected table or window. Because our Standardized writes were key-based upserts, a rerun did not create duplicate business records.
>
> I'd rather give you an honest range than a precise number I can't verify from memory.”

### Follow-ups / traps
- How did you measure it?
- What was the most common failure category?
- Avoid claiming Hudi guarantees duplicate-free retries on its own. Idempotency comes from the record key, ordering field and overall design.

### ⚠ Verify before interview
- Replace "97–98%" with a real figure from Airflow (DAG runs filtered by state over the last 30–90 days).

---

## Q11. What was your cluster size/configuration?

### How to tackle
Show you size by workload, not by habit. Give a ballpark and the reasoning behind it.

### Model answer
> “It depended on the table. Most standard tables ran on Glue G.1X workers with a small count, roughly 5 to 10. Heavier tables such as inventory_on_hand got more capacity, around 10 to 20 workers or G.2X, and small reference tables ran on the minimum.
>
> I sized by input volume and then checked Spark UI and CloudWatch metrics for executor utilization before changing anything. If executors were idle or the job had few tasks, adding workers would not help.
>
> The configuration was driven per table, so I could tune a heavy table without touching the others.”

### Follow-ups / traps
- Why G.1X vs G.2X?
- How did you know the job was under- or over-provisioned?

### ⚠ Verify before interview
- Check the real worker type and count in the Glue job definitions (or Terraform variables) for 2–3 of your tables and replace the ranges.

---

## Q12. What was the average execution time?

### How to tackle
Give typical vs heaviest, tie it to the hourly window, and say how you detect regressions.

### Model answer
> “Most table jobs finished in roughly 3 to 10 minutes. The heaviest ones, like inventory_on_hand at peak, ran closer to 15 to 25 minutes.
>
> Because we processed in hourly windows, the goal was for all concurrent pipelines to finish within the hour with a buffer for retries. I tracked task duration trends in Airflow and compared each run against a normal baseline.
>
> When a run took noticeably longer, I checked input volume, file counts, skew and shuffle before touching resources.”

### ⚠ Verify before interview
- Pull the actual min/typical/max task durations from the Airflow Gantt or task duration view.

---

## Q13. Your volume is only 7–10 GB per run. Why use Glue/Spark and Hudi?

### How to tackle
Be honest that volume alone does not justify Spark, then give the real reasons: table count, CDC logic, concurrency, growth and standardization.

### Model answer
> “You're right that 7 to 10 GB on its own could run on a single node. The reasons were not just size.
>
> We had around 17 interfaces and 57 tables with 15 to 20 pipelines running concurrently, CDC merge logic with inserts, updates and deletes, and a requirement for catalog integration with Athena and Redshift Spectrum. Glue gave us a managed, uniform framework for all of them without running our own cluster.
>
> Hudi gave us record-level upserts and deletes on S3, which plain Parquet does not.
>
> There was also headroom: volumes grow, and the same framework scales without a redesign. For the very small reference tables, we kept the worker count at the minimum so we weren't over-paying.”

### Follow-ups / traps
- Would you use Lambda or a single-node job for tiny tables?
- At what point would you move to EMR?

---

## Q14. Data volume increases 5x (or 10x). What are your steps, and can the project scale?

### How to tackle
Merged answer for 5x and 10x. Do not promise untested scalability.

### Model answer
> “First I'd find out whether the increase is expected or abnormal, such as a duplicate delivery or a backfill.
>
> Then I compare input size, file count, partition distribution and runtime to the baseline. I check whether pruning still works and whether the increase introduced shuffle, skew or small-file problems, and I review joins to see which operation became the bottleneck.
>
> Only after optimizing the data flow do I increase capacity, and I validate both SLA and cost, since 5x data shouldn't automatically mean 5x cost.
>
> For 10x, I wouldn't claim the current configuration just works. The architecture is distributed, so it is a good foundation, but I'd run a capacity test, vary worker configuration, and watch where the bottleneck moves: shuffle, driver, S3 request rates, concurrency limits or Hudi file-group counts. Then I'd tune based on measurements.”

---

## Q15. How do you design a pipeline for billions of records?

### How to tackle
Four pillars: distributed processing, storage layout, avoiding data movement, and operations.

### Model answer
> “I'd focus on four things.
>
> **Process incrementally.** Reprocessing everything each time doesn't scale, so I'd load only new or changed data and use a table format such as Hudi for record-level upserts if the data changes.
>
> **Storage layout.** Parquet with compression, partitioning by how the data is queried (usually date), and output files in the range of roughly 128 MB to 1 GB. I'd run compaction or clustering so billions of rows don't become millions of tiny files.
>
> **Minimize data movement.** Read only needed columns and partitions, filter early, broadcast truly small sides, handle skew explicitly and avoid wide transformations I don't need. Pre-aggregate where consumers only need summaries.
>
> **Operate it safely.** Idempotent writes, retries, checkpoints or intermediate persisted stages for long pipelines, monitoring of runtime and data quality, and cost tracking.
>
> I'd validate the design by testing on a representative sample and a scaled-up run, rather than assuming linear scaling. And I'd say plainly that our own volume is 7 to 10 GB per run, so this is design knowledge applied to our patterns rather than something I ran at billions.”

### Follow-ups / traps
- How would you handle hot keys at this scale?
- How would you size the cluster?

---

# 4. Production Troubleshooting, Failure Handling & Recovery

## Q16. How do you debug a failed production pipeline?

### How to tackle
**Detect → Locate → Classify → Mitigate → Recover → Validate → Prevent.** This is the master answer for failed pipelines, failed Glue jobs and "fails at midnight" scenarios.

### Model answer
> “First I establish the symptom: failure, slowness, missing data, duplicates or wrong output. Then I locate the layer, whether Landing, Raw, Standardized or Curated.
>
> In Airflow I find the failed task and its logs, take the Glue job run ID from there, and go to CloudWatch for the driver and error logs. For Spark failures I open Spark UI or the history server to see the failing stage and task.
>
> Then I classify the failure by its signature. A missing-key error usually means the file hasn't arrived. An analysis exception about a column means schema drift. An out-of-memory error means skew or sizing. An access denied error is IAM. A concurrent-runs exception is a Glue job limit. A lock or conflict error from Hudi is concurrent writers.
>
> If it is transient, I let the configured retry handle it or clear the task. If it is bad data, I isolate the batch, alert the source team and fix or reprocess. I rerun only the affected table or window, never the whole DAG blindly.
>
> After recovery I validate counts, key metrics and downstream availability, then write the root cause and add a check or alert so it doesn't repeat.
>
> If consumers are waiting, I communicate status early. Mitigation and communication come before the perfect fix.”

### Follow-ups / traps
- What's the first thing you check for a Glue failure?
- How do you decide between clearing a task and rerunning the DAG?
- Do not change configuration before understanding the cause.

---

## Q17. How do you design retry and failure handling?

### How to tackle
Separate transient from permanent failures, then describe the layers: task retries, isolation, quarantine, alerting and safe downstream behavior.

### Model answer
> “I start by classifying failures. Transient ones, such as timeouts, throttling or temporary service errors, deserve retries. Permanent ones, such as bad data, schema breaks or logic errors, do not, because retrying identical input just wastes time.
>
> For transient failures I use Airflow retries with a delay and exponential backoff, a sensible retry cap and an execution timeout so a stuck task cannot hang the window. I avoid stacking retries at multiple levels. If both the Glue job and Airflow retry three times, a single problem becomes nine runs.
>
> For permanent failures I isolate the bad batch or records into a quarantine location with the reason, alert the owning team through SNS or the alerting channel, and fix the cause.
>
> Failures must be isolated per table or interface, so one bad table doesn't block 56 others. Dependent downstream tasks must not run on incomplete data.
>
> All of this only works if the processing is idempotent, so a retry cannot create duplicates.”

### Follow-ups / traps
- What happens after retries are exhausted?
- How do you stop retries from hiding a real problem? (Track retry counts as a metric.)

---

## Q18. How do you make your pipeline idempotent?

### How to tackle
Define it first: running the same input again produces the same target state, with no duplicates and no losses. Then list concrete techniques per layer. This also covers rerunning after a partial failure.

### Model answer
> “Idempotency means a rerun or retry leaves the target in the same state as one successful run.
>
> The main techniques I use:
>
> - **Key-based upserts instead of appends.** In Standardized, Hudi upserts on a record key and an ordering field, so replaying a batch updates the same records.
> - **Deterministic logic.** Nothing depends on the current time or random values. In Airflow I use the data interval, not `now()`.
> - **Overwrite by window.** For partition-based loads, I overwrite the specific partition rather than append to it.
> - **A batch ledger.** An audit record of batch ID, counts and status, so I can skip or safely reprocess a batch.
> - **Watermarks advance only after success**, never before.
> - **Idempotent side effects.** Notifications and downstream triggers are deduplicated, and queue consumers handle at-least-once delivery.
>
> For partial output, I first check whether the target is safe to rerun. With Hudi, a failed write leaves an inflight commit on the timeline that is rolled back, so the table does not expose half a commit. For append-only targets I clean up the partial output before the rerun.
>
> I verify idempotency by running the same batch twice and comparing counts and checksums.”

### Follow-ups / traps
- Is upsert alone enough? No. It needs a correct record key and ordering field.
- How do you make a Redshift load idempotent? (Delete and insert in one transaction, or MERGE.)

---

## Q19. The pipeline fails intermittently but succeeds when manually rerun. How do you investigate?

### Model answer
> “I would compare the failed and successful runs rather than assuming the issue is random.
>
> I would look at input availability, execution timing, external dependencies, network/API errors, resource utilization and Spark task failures.
>
> If the failures correlate with higher workload or concurrent jobs, I would investigate resource contention.
>
> If they correlate with a particular input file, I would inspect the data.
>
> I would also check whether retries are masking the underlying problem.
>
> The goal is to identify the condition that makes the failure intermittent and then add the appropriate protection or monitoring.”

---

## Q20. How do you check and handle file corruption and malformed files?

### How to tackle
Detection at the door, quarantine, no downstream contamination, and a clear threshold for partial corruption.

### Model answer
> “Corruption shows up as truncated or zero-byte uploads, wrong delimiters or encoding, bad column counts, broken Parquet footers or checksum mismatches.
>
> I detect it at Landing, before it reaches Raw. Checks include file size, a manifest or control file with row count and checksum from the sender, header and column-count validation, and a test read of the file structure. Upstream should write the data file first and a marker or manifest last, because S3 has no atomic rename and we do not want to process a half-uploaded file.
>
> In Spark I read with an explicit schema and a permissive mode that captures bad rows in a corrupt-record column. I avoid silently ignoring corrupt files, because that hides data loss.
>
> If the whole file is bad, I quarantine it with the reason, alert the source team, request a resend and do not advance the watermark. If only a tiny percentage of rows is bad and within an agreed threshold, I route those rows to a quarantine location and continue. Above the threshold, I fail the file.
>
> Nothing downstream proceeds as if the file were valid.”

### Follow-ups / traps
- How do you tell a late file from a partially uploaded file?
- Why not just drop malformed rows?

---

## Q21. How do you check and handle external API failures?

### How to tackle
Retries with backoff and jitter, timeouts, rate limits, idempotent requests, checkpointing and a fallback.

### Model answer
> “I start by understanding the failure type: timeouts, rate limiting (429), server errors (5xx), authentication expiry or a changed response schema.
>
> For transient errors I retry with exponential backoff and jitter, a cap on attempts and strict timeouts. For rate limits I throttle and honor the retry-after header. For auth I refresh tokens and keep credentials in Secrets Manager.
>
> I make requests idempotent where possible, and for paginated calls I checkpoint the last successful page so a retry resumes instead of restarting.
>
> I avoid calling an API from thousands of Spark tasks at once. I would either fetch in a controlled, throttled step or batch the calls.
>
> I validate the response schema and record counts, because an API can return HTTP 200 with incomplete or changed data.
>
> If the API stays down past the retry budget, the task fails with a clear reason and alert, downstream does not consume partial data, and where the business accepts it, we may use the last known good data with a freshness flag.”

### Follow-ups / traps
- What is a circuit breaker and when would you use one?
- How do you avoid double-processing after a retry?

---

## Q22. How do you plan for disaster recovery (including accidental S3 deletion)?

### How to tackle
Start with RPO and RTO, then protect data, metadata, code and infrastructure separately. End with a replay-based recovery path.

### Model answer
> “I start with the business RPO and RTO, because that decides how much protection is worth paying for.
>
> **Data:** S3 versioning and lifecycle rules, cross-region replication for critical layers, and Object Lock where immutability is required. For an accidental delete with versioning, the delete usually just adds a delete marker, so I remove the marker or restore the previous version, after identifying the exact objects and delete time.
>
> **Metadata:** Glue Catalog definitions and Redshift objects recreated from code (Terraform and DDL scripts in Git), plus Redshift automated snapshots with cross-region copy.
>
> **Code and orchestration:** DAGs, Glue scripts and Terraform are all in Git, so the environment can be rebuilt in another region.
>
> **Replay path:** Landing and Raw are our immutable record. If Standardized or Curated is damaged, I rebuild them from Raw rather than depending on a backup of every layer.
>
> One caution: for a Hudi table, the data files and the `.hoodie` timeline must stay consistent. Restoring only some files can corrupt the table. Asynchronous object replication isn't transactionally consistent, so replay from Raw is the safer recovery for lake tables.
>
> I would also run a recovery drill, because an untested plan is only an assumption.”

### Follow-ups / traps
- Do not assume S3 recovery is always possible. It depends on versioning, backups and retention.
- What is the difference between backup and replication?

---

## Q23. How do you monitor pipelines, and why is monitoring job failure alone not enough?

### How to tackle
Merged answer: technical health plus data health.

### Model answer
> “We use Airflow for task-level visibility, Glue job logs and metrics for ETL, and CloudWatch for AWS-level monitoring, with alerts through SNS. Glue job state changes can also be routed through EventBridge to alert on failures or timeouts.
>
> But a job can succeed technically and still produce bad data. It might process zero records because the file never arrived, or 50% fewer than normal, or produce duplicates or schema surprises.
>
> So I monitor technical health (failures, retries, duration trends against baseline, queue or backlog) and data health (file arrival, input and output counts, volume anomalies, freshness, schema changes, duplicate rates and DQ rule results).
>
> For a no-data case specifically, the pipeline checks that the expected file or partition arrived and compares counts with historical expectations, alerting on zero or unusually low volume.
>
> Alerts should be actionable: clear owner, clear message, link to runbook, and tuned to avoid noise.”

### Follow-ups / traps
- How would you detect a pipeline that is silently getting slower over weeks?

---

# 5. Spark Optimization & Glue

## Q24. How do you optimize a slow Spark/Glue job?

### How to tackle
This is the master answer. It also covers "a job suddenly takes 5x longer", "a join got slow after volume grew" and "what optimization techniques have you used". Order: **Baseline → Read less → Shuffle less → Skew less → Compute efficiently → Validate.**

### Model answer
> “I don't start by increasing workers. I first find where the time is going.
>
> I compare the run to a known good baseline and check whether input volume, file count or partition distribution changed.
>
> Then I look at what is being read: partition pruning, columnar format, and whether we read columns or rows we don't need. Next I check the S3 layout for too many small files.
>
> In Spark UI I find the expensive stages and look at shuffle read/write, task duration spread, spill and failed tasks. A few very slow tasks point to skew. Large shuffle points to joins and aggregations that could be reduced by filtering earlier, selecting fewer columns or broadcasting a small side.
>
> If the job suddenly got 5x slower, I ask what changed: input size, a join strategy that flipped from broadcast to sort-merge, pruning that stopped working, or a new skewed key.
>
> Only after the data flow is efficient do I tune resources or Spark settings, such as worker type and count, shuffle partitions and AQE.
>
> Finally I compare runtime, resource use and cost against the baseline to prove the change helped.”

### Techniques I actually reach for
- Column pruning, early filters, partition pruning and predicate pushdown
- Parquet with compression, sensible file sizes
- Broadcast joins for genuinely small sides, AQE for skew and partition coalescing
- Incremental processing instead of full reloads
- Avoiding repeated expensive actions and Python UDFs where built-in functions work
- Tuning workers only after evidence

### Follow-ups / traps
- Never start with "increase workers".
- What would you check if only one stage is slow?

---

## Q25. Explain the Spark UI and what you look for.

### Model answer
> “I mainly use Spark UI to understand where the job is spending time.
>
> I start with jobs and stages, then look for stages with unusually high duration.
>
> In stage details, I check task duration, input/output, shuffle read and shuffle write.
>
> If one or a few tasks are significantly slower than the others, that can indicate data skew.
>
> I also check executor memory, failed tasks and whether a join or aggregation is causing a large shuffle.
>
> The goal is to use the UI to identify the bottleneck rather than guessing whether the problem is CPU, memory, shuffle or data distribution.”

---

## Q26. Explain the DAG generated by your Spark pipeline.

### Model answer
> “When I write Spark transformations, Spark builds a logical execution plan rather than immediately processing the data.
>
> Transformations are lazy. When an action such as write or count is triggered, Spark creates the execution DAG.
>
> The DAG is divided into stages around shuffle boundaries.
>
> Narrow transformations such as filter and select can usually remain within the same stage because data does not need to move between partitions.
>
> Operations such as groupBy and many joins introduce shuffle boundaries and therefore create additional stages.
>
> Within stages, Spark executes tasks against partitions of the data.”

---

## Q27. What is the difference between narrow and wide transformations?

### Model answer
> “A narrow transformation does not require data from multiple parent partitions to be combined. Examples are filter, select and map.
>
> A wide transformation requires data to move between partitions, which is called shuffle. Examples include groupBy, join and repartition.
>
> Wide transformations are generally more expensive because they involve network and disk I/O and can become bottlenecks at scale.”

---

## Q28. How do you handle data skew?

### How to tackle
This is the master skew answer. It also covers "repartition fixes skew?", "one customer owns 60% of transactions" and "one task takes 40 minutes while others take 2". Rule: **find the hot key first.**

### Model answer
> “Skew means data is unevenly distributed so some tasks get far more work than others.
>
> The symptom is one or a few tasks running much longer than the rest in a stage, with a much larger shuffle read. I confirm it in Spark UI by comparing task duration and size, and then profile the key to find the hot value. I also rule out resource problems before blaming skew.
>
> The fix depends on the situation:
>
> - **One side small:** broadcast it so there is no shuffle on the large side. This works if the customer lookup is small, even when one customer has 60% of the transactions.
> - **Both sides large:** use AQE skew-join handling, which splits oversized partitions, or salt the hot key by adding a random suffix on the large side and exploding the small side to match.
> - **Aggregation skew:** do a two-stage aggregation, partially aggregating with a salted key and then combining.
> - **Bad default values:** sometimes the hot key is a default like a null or a dummy location, and the right fix is to filter or handle it separately.
>
> A common mistake is `repartition(customer_id)`. That hashes the same key to the same partition, so the hot key stays in one partition and nothing improves.
>
> In our project, the pattern I watched for was a few high-volume locations or placeholder values dominating joins on the inventory tables. I would confirm in Spark UI before changing anything.”

### Follow-ups / traps
- Why doesn't repartition by the same key fix skew?
- Do you need salting if AQE skew join is enabled?

### ⚠ Verify before interview
- If you hit skew in a real table, replace the inventory example with it.

---

## Q29. When will you use a broadcast join, and how small is "small"?

### How to tackle
Master answer for broadcast joins, the "2 TB table joined to 50 MB table" question, and "how small?". Give the actual default number, then explain why you still verify.

### Model answer
> “I use a broadcast join when one side is small enough to send to every executor, because then Spark skips shuffling the large side.
>
> The default `spark.sql.autoBroadcastJoinThreshold` is 10 MB, and Spark only auto-broadcasts when its size estimate is under that. For a 50 MB lookup joined to a 2 TB table, I would first confirm the real size, since file size on disk and in-memory size differ, then either use an explicit broadcast hint or raise the threshold, while making sure executor memory can hold it.
>
> I don't decide on row count alone. I check the physical plan or Spark UI to confirm it is actually a broadcast hash join, and watch for memory pressure or driver issues. The broadcast data is collected through the driver first.
>
> I avoid broadcasting when the 'small' table is borderline large, since failures there are worse than a slower sort-merge join. Also, if the large side is already filtered by pruning, the join size can change, so I check after filters.
>
> With AQE on, Spark can also switch a sort-merge join to broadcast at runtime when the real size after filtering turns out small.”

### Follow-ups / traps
- Name other join strategies (sort-merge, shuffle hash, broadcast nested loop, cartesian).
- What happens if you broadcast something too large?

---

## Q30. Repartition vs coalesce, and why can too many or too few partitions hurt?

### How to tackle
Merged answer for repartition cost, coalesce use and partition-count sizing.

### Model answer
> “`repartition()` triggers a full shuffle, so it can redistribute data evenly or by a column but costs network, disk and time. `coalesce()` only reduces the number of partitions by merging existing ones without a full shuffle, so it is cheaper, though it can leave partitions uneven.
>
> Too few partitions limit parallelism: a few huge tasks, long runtimes and higher memory risk. Too many create scheduling overhead and, on write, many small files.
>
> A practical guide is aiming for partitions of roughly 100 to 200 MB of input, and ensuring the number of tasks is a reasonable multiple of the cores available. `spark.sql.shuffle.partitions` defaults to 200, which is often wrong for both tiny and very large jobs, and AQE can coalesce small shuffle partitions automatically.
>
> I use repartition before a write when I need balanced output files, and coalesce when I only need fewer output files from already-balanced data. I'd never repartition just by habit.”

### Follow-ups / traps
- Can repartition solve skew? (No, not by the same key.)

---

## Q31. How do you handle OOM and memory pressure issues?

### How to tackle
Separate driver OOM from executor OOM, find the cause in logs and Spark UI, fix the data shape first, and resize last.

### Model answer
> “First I identify which side failed. Driver OOM usually comes from `collect()`, `toPandas()`, broadcasting something large, or listing and tracking a huge number of files and partitions. Executor OOM usually comes from oversized or skewed partitions, huge groups in an aggregation, wide rows, exploding arrays, large caches or heavy Python UDFs. 'Container killed for exceeding memory limits' points to off-heap/overhead memory.
>
> In Spark UI I look at which stage failed, the partition size and task spread, spill to disk, and executor memory use.
>
> Fixes, in order of preference:
>
> - Reduce partition size by increasing shuffle partitions or repartitioning appropriately.
> - Fix skew, which is a common hidden cause.
> - Select fewer columns and filter earlier.
> - Remove `collect`/`toPandas`, and lower or disable broadcast for big tables.
> - Replace Python UDFs with built-in functions.
> - Persist with a disk-backed storage level rather than memory-only, and unpersist when done.
> - Compact small files that bloat driver metadata.
>
> In Glue, I usually can't tune executor memory freely, since it's set by the worker type. So if the data shape is fine and the job is genuinely memory-bound, I move to a larger worker type such as G.2X. That is the last step, not the first.
>
> I confirm the fix by rerunning with the same data and checking spill and peak memory.”

### Follow-ups / traps
- Increasing workers doesn't fix a skewed task. Why?

---

## Q32. How do you tackle underutilized clusters and autoscaling?

### How to tackle
Master answer for underutilized Glue/Spark capacity and the autoscaling concept. Measure first, right-size second, automate third.

### Model answer
> “Autoscaling means adjusting compute to demand instead of holding fixed capacity, to meet the SLA without paying for idle resources. It can't fix an inefficient join or skew though, so I check the cause of low utilization first.
>
> To detect underutilization I look at Glue CloudWatch metrics comparing executors allocated against executors actually needed, CPU and memory use, and Spark UI's executor tab for idle executors. I also check task counts: if a job has fewer tasks than available cores, extra workers just sit idle.
>
> Typical causes are over-provisioned workers for small tables, too few partitions, a serial section such as a single large write or driver-side work, and long waits on external systems.
>
> Fixes: right-size workers per table class (small, medium, large), enable Glue auto scaling so it scales up only when needed, group tiny tables into fewer jobs, fix partition counts, and use Flex execution for non-urgent jobs.
>
> After changes I compare cost per run and runtime to make sure the SLA still holds.”

### Follow-ups / traps
- Is cheaper always better? A lower-cost config that misses the hourly window is a failure.

---

## Q33. The business says "finish in under an hour" or you have one optimization before a deadline. What do you do?

### How to tackle
Merged answer for the SLA question and the "only one optimization" question. Evidence over fashion.

### Model answer
> “I'd establish the baseline and the expected input volume, then find the bottleneck with Glue metrics and Spark UI instead of picking a technique that sounds advanced.
>
> Then I choose the highest-impact, lowest-risk change. If most time is spent shuffling because we read unnecessary history, fixing partition pruning beats adding workers. If the data flow is already efficient and the job is CPU- or memory-bound, scaling workers is the right call.
>
> I check file sizes and pruning, then shuffle and skew, then joins and late filters, then output partitioning. Only afterward do I touch worker capacity.
>
> I test the change quickly on a representative run, compare runtime, reliability and cost, and monitor the next production run. If I can't get under the target safely, I tell stakeholders early with options and trade-offs.”

---

## Q34. What Glue-specific features and settings should I know?

### How to tackle
Show practical Glue knowledge, not just "Glue runs Spark".

### Model answer
> “The ones I use most:
>
> - **Worker types.** G.1X is 1 DPU (4 vCPU, 16 GB) and G.2X is 2 DPU (8 vCPU, 32 GB), with larger ones for heavy jobs. Memory-heavy jobs go to larger workers.
> - **Glue version.** It determines the Spark version and bundled library support. For Hudi, I enable it through the `--datalake-formats hudi` job parameter.
> - **Auto scaling and Flex.** Auto scaling adjusts workers to demand, and Flex execution is cheaper for non-urgent jobs.
> - **Job bookmarks.** They track processed data for supported sources, but with CDC feeds and Hudi I rely on my own watermarks and batch ledger, since bookmarks can be limiting.
> - **DynamicFrame vs DataFrame.** DynamicFrames tolerate messy schemas through choice types, but I convert to DataFrames for most transformations.
> - **Max concurrent runs.** The default for a job is 1, so running the same job for multiple windows in parallel needs this raised. Account-level DPU and concurrency quotas also matter with 15 to 20 pipelines.
> - **Observability.** Spark UI, continuous logging and job metrics enabled through job parameters.”

### Follow-ups / traps
- What happens if the same Glue job is triggered twice at once with default settings? (A concurrent-runs error.)

---

# 6. Partitioning, S3 & File Layout

## Q35. How do you choose partition columns (and what did you follow)?

### How to tackle
Merged answer for "how do you choose" and "what strategy did you follow". Use query patterns, cardinality, distribution and partition size.

### Model answer
> “I look at four things: query and processing patterns, cardinality, data distribution, and the resulting partition size.
>
> A good partition column lets frequent queries skip most data. Date is the common choice for large, growing datasets, since consumers usually filter by a date range. Very high-cardinality columns like customer or item ID are dangerous because they create huge numbers of partitions and small files.
>
> I aim for partitions that are big enough to be worthwhile, typically hundreds of MB to a few GB, and I validate actual partition and file sizes after writing.
>
> For Hudi tables there is an extra rule: the partition path must be stable for a record. If the partition column can change on update, a non-global index can leave the old copy in the old partition and create a duplicate, so I'd either choose an immutable column or use a global index.
>
> For our data, the choice followed how each table is consumed and processed: time-based for large event-like and transaction-like tables, and little or no partitioning for small reference tables such as item master, where partitioning would only create overhead.”

### Follow-ups / traps
- Why not partition by item or location?
- What is over-partitioning?

### ⚠ Verify before interview
- Confirm your real partition columns for 2–3 tables.

---

## Q36. How do you manage the small file problem (and why is `coalesce(1)` dangerous)?

### How to tackle
Find the cause, then fix at write time and with periodic compaction.

### Model answer
> “First I find why they appear: too many partitions, frequent small incremental writes, or too much output parallelism.
>
> At write time I control the number of output partitions. `coalesce(n)` reduces partitions without a full shuffle, and `repartition(n)` gives balanced files with a shuffle. For lakehouse tables I use the table's own services: Hudi sizes files at write time using its small-file limit and max file size settings, and compaction or clustering rewrites small files into bigger ones.
>
> I avoid `coalesce(1)` on large data: one task must process the entire output, which creates a bottleneck and can run out of memory.
>
> Small files hurt because of S3 listing and open overhead, task scheduling overhead and slower Athena and Spectrum queries, so I check file counts and sizes as part of monitoring.”

### Follow-ups / traps
- How do you choose the target file size?

---

## Q37. The job reads 2 TB but only needs yesterday's data. What would you investigate?

### Model answer
> “I would investigate partition pruning first.
>
> If the S3 data is partitioned by date and the query or Spark read includes the partition filter correctly, the engine should avoid scanning unrelated partitions.
>
> I would verify the physical read and actual input size rather than assuming pruning is happening.
>
> I would also check whether the data is stored in a columnar format and whether the job is reading unnecessary columns.”

---

## Q38. How would you load a huge CSV file?

### How to tackle
Use distributed reading, an explicit schema, columnar output, and watch for compression gotchas.

### Model answer
> “I'd avoid treating it as one in-memory object and read it with Spark so the work runs in parallel.
>
> I'd supply an explicit schema instead of inference, since inference scans the data, read only needed columns, filter early, and write Parquet partitioned appropriately without creating tiny files.
>
> Two details matter. A gzip-compressed CSV is not splittable, so a single large `.csv.gz` gets processed by one task. I'd ask for splittable delivery (many files or bzip2/ungzipped chunks) or convert once and parallelize afterwards. Also, multi-line quoted fields can prevent splitting.
>
> For bad rows I choose the read mode deliberately and capture corrupt records for review rather than silently dropping them.”

---

# 7. CDC, Hudi & SCD

## Q39. How do you design CDC processing?

### How to tackle
Master CDC answer (also covers "explain your CDC/Hudi flow" and "what is the most important thing in CDC"). Core model: **Key + Operation + Ordering + Idempotency.**

### Model answer
> “A CDC payload tells you which record changed, what the operation was (insert, update or delete) and when it changed. Correct CDC processing means the target eventually matches the true latest source state.
>
> In our Standardized layer, the Glue job maps the incoming payload to the standardized schema, then deduplicates by business key using the update timestamp so only the latest event per key is kept in the batch. Inserts and updates go through the Hudi upsert path with a configured record key and ordering field, and deletes are processed separately using Hudi's delete mechanism against the same record key. After the write, table metadata is available through the Glue Catalog.
>
> The most important design points for me are a reliable record key, a reliable ordering field, clear delete semantics, and handling of duplicate events, out-of-order events, late events and reruns. If the source doesn't provide reliable ordering, I'd raise that as a data-contract issue.
>
> Validation after each run is counts and duplicate-key checks, plus sampling affected keys.”

### Follow-ups / traps
- CDC does not automatically mean streaming. Ours is orchestrated in batches.
- What if the delete arrives before the insert?

---

## Q40. Explain SCD Type 1 in your project.

### Model answer
> “SCD Type 1 means we maintain the latest value of a record and do not preserve the previous version as historical rows.
>
> For example, if a customer's city changes from Kolkata to Bangalore, the target record is updated to Bangalore rather than keeping both versions as historical records.
>
> In our Standardized layer, the Hudi upsert pattern supports this kind of current-state representation.”

---

## Q41. If Standardized is SCD Type 1, where does history live? What about SCD Type 2?

### How to tackle
Show you understand what SCD1 loses and where history is kept.

### Model answer
> “In Standardized we keep the latest state only, which is SCD Type 1. History is not lost though: Raw keeps every CDC record with ingestion timestamps, so we can audit changes and replay them. Hudi also offers incremental queries and, within the retention of its timeline, time-travel reads.
>
> If the business needs history in a consumable form, I'd use SCD Type 2, normally in Curated. Each version gets an effective-from date, an effective-to date and a current flag, often with a hash of tracked columns to detect real changes. When a tracked attribute changes, I close the existing current row and insert a new row. Facts then join to the version valid at event time.
>
> The trade-offs are more storage, more complex joins and more complex late-arriving-change handling. So I only use Type 2 for attributes where history actually matters, like price or category on items.”

### Follow-ups / traps
- Why not store everything as SCD2?

---

## Q42. How did you process deletes?

### Model answer
> “We identify CDC records where the operation is delete and process them separately from inserts and updates.
>
> In Hudi, deletes can be represented using the appropriate delete payload or configuration. The important part is that the record key matches the existing target record so that the correct record is removed.
>
> After processing, I would validate counts and sample affected keys to make sure the delete was applied correctly.”

### Follow-up: What if the target record does not exist?
> “I would treat that as an idempotent delete or investigate whether the event arrived out of order, depending on the business and CDC semantics. I would not blindly recreate the record.”

---

## Q43. What if the same CDC event arrives twice, or events arrive out of order?

### How to tackle
Distinguish dedupe within a batch from protection against stale events arriving in a later batch.

### Model answer
> “For duplicates: within a batch, I keep one row per key based on the ordering field. A duplicate event with the same key and ordering value then maps to the same record, so replaying it doesn't create a second business record. I still validate with duplicate-key checks and counts, because idempotency belongs to the whole pipeline design, not the word 'upsert'.
>
> For out-of-order events, the ordering field is critical. We must not apply whichever event happens to arrive last. The precombine field resolves conflicts among records in the same incoming batch. The harder case is a stale event arriving in a later batch than a newer one. Whether it overwrites the stored record depends on the payload class or merge mode: a payload that compares the ordering value with the stored record keeps the newer state, while one that simply takes the incoming record would overwrite it. I'd confirm this behavior for our Hudi version rather than assume it.
>
> If the source can't give a reliable timestamp or sequence number, I'd raise it with the source team, because correct reconciliation becomes very difficult.”

### Follow-ups / traps
- Which ordering field do you use and who provides it?
- How would you test the out-of-order case?

### ⚠ Verify before interview
- Check your Hudi version and configured payload class or merge mode.

---

## Q44. What is Hudi and how does it help with upserts? Does it prevent duplicates automatically?

### How to tackle
Merged answer for "how does Hudi help" and the "you said Hudi handles duplicates automatically" trap.

### Model answer
> “Hudi is a table format that adds record-level operations on top of data in S3, using Parquet as the base file format. It keeps a timeline of commits, an index that maps record keys to file groups, and table services for compaction, clustering and cleaning.
>
> In our CDC pipeline it let us upsert and delete by key instead of appending every change as a new row.
>
> On duplicates, I'd phrase it carefully: Hudi provides the mechanism for key-based upserts, but correctness depends on the record key, the precombine or ordering field, the write operation type, the index and partition-path design. A wrong key can overwrite distinct records or leave duplicates. So idempotency is a property of the overall pipeline design.”

### Follow-ups / traps
- Does Hudi replace Parquet? No, it manages Parquet files with a table layer.

---

## Q45. What benefit do modern table formats provide compared with plain Parquet?

### Model answer
> “Plain Parquet is a file format. A table format such as Hudi adds table-level capabilities on top of files.
>
> For our use case, the important advantage was supporting record-level operations such as upserts and deletes and maintaining the metadata needed for those operations.
>
> That is particularly useful for CDC workloads where data is not simply append-only.
>
> I would not say Hudi replaces Parquet—Hudi commonly uses Parquet as the underlying file format while adding table-management capabilities.”

---

## Q46. How does MERGE work conceptually?

### Model answer
> “A MERGE compares incoming records with the target using a matching key.
>
> If a matching target record exists, it can be updated. If there is no match, the incoming record can be inserted.
>
> Depending on the implementation, deletes can also be handled.
>
> Conceptually, this is useful for CDC because one operation can reconcile inserts and updates against the current target state.”

---

## Q47. Copy-on-Write vs Merge-on-Read: which did you use and why?

### How to tackle
Explain both, give the trade-off, then say which fits your read/write pattern.

### Model answer
> “In Copy-on-Write, an update rewrites the affected Parquet base file, so reads are simple and fast but writes cost more. In Merge-on-Read, updates go to row-based log files that are merged with the base file at read time, or compacted later. That gives faster, lighter writes but more read-time work, and it needs compaction.
>
> MoR also offers a read-optimized query that reads only base files, which is faster but can be stale until compaction.
>
> For our hourly batch pattern, where downstream consumers read through Athena and Redshift Spectrum and simplicity matters, Copy-on-Write was the better fit. MoR makes more sense for frequent, write-heavy or near-real-time updates where write latency matters more than read simplicity.
>
> The choice also depends on engine support: query engines differ on what they support for MoR snapshot reads, so I'd check support for Athena and Spectrum before choosing it.”

### Follow-ups / traps
- What is write amplification?
- When would you change to MoR?

### ⚠ Verify before interview
- Confirm your actual table type. If you used MoR, swap the conclusion and mention compaction scheduling.

---

## Q48. How do you configure record key, precombine field, partition path and index?

### How to tackle
Four settings, each with a failure mode.

### Model answer
> “**Record key** identifies a record uniquely. For a table like inventory_on_hand, that is a composite key such as item and location, not just item, otherwise different rows collapse into one. Composite keys use the complex key generator.
>
> **Precombine field** decides which record wins when several have the same key in a batch. We use the source update timestamp or a sequence number.
>
> **Partition path** decides the physical partition. It should be stable for a record. If it can change on update, a non-global index can leave the old row behind and create duplicates across partitions. Small reference tables are often non-partitioned.
>
> **Index** locates existing records during upserts. Bloom-style indexes are common defaults, and global or bucket indexes exist for other needs. The choice affects upsert performance and cross-partition uniqueness.
>
> I validate all four with profiling before onboarding a table: is the key actually unique in the source, is the ordering field reliable, and is the partition column immutable.”

### Follow-ups / traps
- What happens if the record key is not unique?
- What if the precombine field is null?

---

## Q49. What Hudi write operations are there, and how do deletes work?

### How to tackle
Know the main operation types and when each is used.

### Model answer
> “The common ones are **upsert** (the default, update if the key exists, else insert), **insert** (skips the existing-record lookup, faster but doesn't dedupe), **bulk_insert** (for large initial loads, with sorting options for file layout), **delete** (removes by record key) and **insert_overwrite** or **delete_partition** for partition-level changes.
>
> Our incremental CDC applies upserts for inserts and updates. For the initial historical load of a table I'd use bulk_insert, then switch to upsert for ongoing changes.
>
> Deletes can be hard deletes, where records matching the keys are removed by the delete operation, or soft deletes, where the record is kept with a delete flag or nulled values. We process CDC delete events separately and apply them as hard deletes by record key, then validate counts and sample keys.
>
> If a delete arrives for a key that doesn't exist, I treat it as an idempotent no-op or investigate ordering, and I don't recreate the record.”

### Follow-ups / traps
- When would you prefer soft deletes?

---

## Q50. How do you handle Hudi compaction, clustering and cleaning?

### How to tackle
Table services keep the table healthy. Know what each does and why it matters.

### Model answer
> “**Cleaning** removes older file versions that are no longer needed, controlled by how many commits are retained. It keeps storage under control but also limits how far back you can query, and long-running readers need enough retention.
>
> **Compaction** applies to Merge-on-Read: it merges log files into base files so reads stay fast. It can run inline or asynchronously.
>
> **Clustering** rewrites small files into larger ones and can sort data by query columns, which helps the small-file problem and query performance.
>
> Hudi also does file sizing at write time by routing new records to small files, based on the small-file limit and max file size settings.
>
> The operational point is to monitor file counts and sizes per table and schedule these services so they don't compete with the main ingestion window or conflict with concurrent writers.”

### Follow-ups / traps
- What happens if cleaning is too aggressive?

---

## Q51. What if two pipelines try to update the same Hudi table?

### How to tackle
Default is a single writer. Multi-writer needs explicit configuration.

### Model answer
> “Hudi assumes a single writer by default, so uncontrolled concurrent writes to the same table are unsafe.
>
> Multi-writer is possible with optimistic concurrency control, which needs a lock provider such as DynamoDB, Hive metastore or ZooKeeper. Conflicts are detected when two writers touch the same file groups, and one of them fails and must retry. It adds complexity and retry handling.
>
> In practice, I avoid the situation by design: one pipeline owns one table, and when two inputs feed the same table I serialize them with Airflow dependencies or a pool with a single slot for that table. Independent tables run in parallel, which is where the 15 to 20 concurrent pipelines come from.
>
> Table services like compaction and clustering also need coordination if they run alongside ingestion.”

### Follow-ups / traps
- What error would you expect on a conflict, and how do you recover?

---

## Q52. What happens to a Hudi table if a write fails midway? Can you roll back?

### How to tackle
Timeline, inflight commits, rollback, savepoints.

### Model answer
> “Hudi tracks every write as an instant on a timeline, moving from requested to inflight to completed. If a write fails, the inflight commit is never completed, so readers don't see its partial data, and a later write or rollback cleans it up. That is what makes reruns safe on Hudi tables.
>
> For deliberate recovery, a savepoint marks a commit so it is protected from cleaning, and a restore can roll the table back to it. Hudi also supports incremental queries from a commit time and time-travel reads within the retention window.
>
> For serious corruption, my fallback is to rebuild the table from Raw, since that is our replay source.”

### Follow-ups / traps
- Why is manually deleting files from a Hudi table dangerous?

---

## Q53. How are Hudi tables queried through Athena and Redshift Spectrum?

### How to tackle
Know the query types and the compatibility caveat.

### Model answer
> “Hudi supports snapshot queries (latest state), read-optimized queries (Merge-on-Read base files only) and incremental queries (changes since a commit).
>
> For access from AWS engines, the table is synced to the Glue Data Catalog so Athena and Spectrum can see it. Athena supports Hudi tables directly. For Spectrum, support has had restrictions, historically strongest for Copy-on-Write tables, so I check the current documentation for the table type and Hudi version in use. Partition sync matters as well: if new partitions aren't in the catalog, consumers won't see the data.
>
> This is one reason table type is chosen with the consumption engines in mind.”

### ⚠ Verify before interview
- Confirm how your Curated layer reads the Standardized Hudi tables and what restrictions apply for your version.

---

# 8. Late Data, Incremental Loads & Schema Changes

## Q54. How do you handle late-arriving data?

### How to tackle
Separate event time from processing time. Then cover placement, reprocessing and downstream refresh.

### Model answer
> “First I distinguish event time from processing time. If an event happened Tuesday but arrived Thursday, it is still a Tuesday event, so I use the source business timestamp, not the load date, for logical placement.
>
> In an incremental design, I need a safe way to revisit history. I use a lookback or overlap window so late records within the window are picked up, and a separate backfill for anything older.
>
> In our CDC-style processing, Hudi upserts help because the record key and ordering field let a late update modify the existing record instead of appending a duplicate. A late-arriving update to an old record needs its partition to be stable, which is another reason I avoid mutable partition columns.
>
> Downstream, I check aggregates and materialized views: a late change can alter numbers that were already published, so I refresh the affected windows and, if needed, tell consumers.”

### Follow-ups / traps
- How late is too late? Agree a cutoff with the business.

---

## Q55. A source file is delayed. How should the pipeline behave?

### How to tackle
Merged answer for "source is delayed" and "daily file arrives six hours late".

### Model answer
> “First I separate a delayed source from a real pipeline failure. The orchestration layer should know whether the expected input arrived. In Airflow I use a sensor on the expected file or manifest, in reschedule or deferrable mode so it doesn't hold a worker slot, with a timeout based on the agreed SLA.
>
> If the file is mandatory, downstream tasks must wait or fail with a clear 'upstream delay' reason and alert. They must not consume incomplete data just to turn the DAG green.
>
> When the file arrives, the affected window is triggered or backfilled. If it belongs to an earlier business date, it must be processed by event time and update the affected history, with Curated refreshed accordingly.
>
> I'd also notify consumers if their SLA is at risk.”

---

## Q56. How do you design incremental loads (and what if the update timestamp is unreliable)?

### How to tackle
Merged answer. Reliable key → persisted state → upsert → rerun-safe → overlap window if timestamps are untrustworthy.

### Model answer
> “I start by identifying a reliable incremental marker: a last-modified timestamp, a CDC sequence number or a high-watermark field.
>
> I persist the last successfully processed state, for example in an audit or control table, and select only new or changed records past it. The target is updated with an upsert or merge. I record batch ID, counts and status for auditability, and I only advance the watermark after the write succeeds, so a failed run is simply repeated.
>
> If the source timestamp isn't fully reliable, I don't trust an exact boundary. I look for a stronger ordering like a sequence number, and if only timestamps exist I re-read a small overlap window before the last watermark and deduplicate by business key and ordering field. The cost is reading slightly more data; the benefit is not missing late updates.
>
> I also keep a full-refresh or replay path from Raw for repairs, and treat the first historical load differently from ongoing increments.”

### Follow-ups / traps
- Where do you store the watermark and how do you protect it from corruption?
- How are deletes captured in a timestamp-based incremental load? (They are not, without CDC or soft-delete flags.)

---

## Q57. How do you manage schema evolution and schema changes?

### How to tackle
Master answer for schema evolution, "source adds a column", "int to string", "schema merge is enough?" and "job succeeds but the column is missing". Model: **Detect → Classify → Validate → Evolve → Test.**

### Model answer
> “I classify changes first. Additive, nullable columns are usually backward-compatible. Dropped columns, renames and type changes (especially narrowing, like string to int) are breaking.
>
> **Detect.** At Landing we compare the incoming structure with the expected schema or contract and alert on differences instead of discovering them downstream.
>
> **Raw.** Preserve what arrived, including the new column, so nothing is lost.
>
> **Standardized.** Schema evolution is explicit. Spark transformations often select a fixed list of columns, so a new column discovered in the Glue Catalog won't appear in the output until the code selects it. Catalog metadata, DataFrame schema and the Hudi table schema are separate concerns. Hudi handles additive columns and type promotions, and renames or drops need extra care.
>
> **Curated.** Redshift Spectrum picks up catalog changes, but materialized views have a fixed definition, so exposing a new column means altering or recreating the view.
>
> For breaking changes, like an integer becoming a string, I don't blindly cast just to make the job pass, because that can hide a source defect. I confirm intent with the source team, quarantine incompatible input if needed, and make a controlled change.
>
> If the job succeeded but a new column is missing downstream, I compare the source schema, catalog schema, DataFrame schema and target schema to see where it was dropped, which is nearly always a fixed projection.
>
> Then I test with both old-shape and new-shape files and deploy.”

### Follow-ups / traps
- Is `mergeSchema` enough? No.
- How do you handle a column rename?

---

## Q58. How do you detect data drift and tackle it?

### How to tackle
Separate schema drift (structure) from data drift (content or distribution), then baseline, detect, triage, act.

### Model answer
> “Schema drift is a change in structure. Data drift is when the content changes while the structure stays the same: null rates rise, value distributions shift, new category values appear, units change, volumes jump or collapse, or freshness slips.
>
> To detect it, I build baselines per important column and table: row counts, null percentage, distinct counts, min, max, mean and percentiles, and the set of category values. Each run is compared to a rolling baseline with thresholds, using percentage change or a z-score, and seasonal context where relevant. Tools like Glue Data Quality, Deequ or Great Expectations help; I can also write the profiling in Spark and store results in a metrics table.
>
> When drift fires, I triage: is it an expected business change or an upstream defect? A spike in promotion_items during a sale is expected. Negative quantities in inventory_on_hand or a sudden jump in nulls in item_master is a defect.
>
> For defects I flag or quarantine the affected data depending on severity, contact the source team, fix or reprocess, and add a rule. For expected changes I update thresholds, with seasonality in mind, and tell downstream consumers.”

### Follow-ups / traps
- How do you avoid alert fatigue from noisy thresholds?

---

# 9. Data Quality & Duplicates

## Q59. How do you ensure data quality, and how do you check and tackle quality issues?

### How to tackle
Master DQ answer. Rules → layer-specific checks → actions → reconciliation → fix and prevent.

### Model answer
> “I make data quality explicit instead of trusting that the job succeeded. I think in dimensions: completeness, validity, uniqueness, consistency, timeliness and accuracy.
>
> Rules are metadata-driven so the framework applies them consistently, not hand-coded per job. Checks are layer-specific: file-level checks at Landing (arrival, name, structure, counts), key not-null and counts at Raw, uniqueness, type validity and referential checks in Standardized, and business rules plus reconciliation in Curated.
>
> Each rule has a severity. Critical failures block promotion to the next layer. Row-level failures go to a quarantine location with the failure reason, and warnings alert without blocking. A summary table tracks total, passed and failed records per batch and per rule.
>
> I reconcile counts and key sums across layers and, where possible, against the source, to catch silent loss or duplication.
>
> When an issue is found, I triage: how many records, since when, which downstream consumers are affected. I find the root cause (source, logic or schema), fix it, reprocess the affected window from the correct layer, validate, and add or tighten the rule so it's caught earlier next time.
>
> Examples in our domain: items in item_location without a matching item_master, duplicate keys in item_master, or negative or null quantities in inventory_on_hand.”

### Follow-ups / traps
- Who owns a DQ failure, the pipeline team or the source team?
- How do you prevent DQ checks from becoming too slow?

---

## Q60. How do you handle duplicate records, including across zones?

### How to tackle
Master answer for duplicates, duplicate output and "duplication throughout your zones". Define duplicate first, then detect, fix and prevent per layer.

### Model answer
> “First I define 'duplicate' from the business side. Two identical-looking rows might be valid separate events.
>
> Common causes are the source resending data, repeated CDC events, overlap windows, a wrong or non-unique record key, join fan-out, non-idempotent retries, a Hudi partition path that changes on update, and concurrent writers.
>
> Per zone:
>
> - **Landing:** prevent file-level duplicates with a batch or file ledger and checksum so the same file isn't processed twice.
> - **Raw:** keep everything as delivered, with ingestion metadata and batch ID, so we can always trace and replay. Raw is allowed to contain duplicates; it is not the cleaned layer.
> - **Standardized:** deduplicate by business key and ordering field, using `row_number()` over the key ordered by the latest timestamp for batch data, or Hudi upserts for CDC. Check key uniqueness after every write.
> - **Curated:** make sure joins don't multiply rows, by checking cardinality on the join keys. A uniqueness test on the output grain catches it.
>
> To detect, I use group-by-key-having-count-greater-than-one queries and count reconciliation between layers.
>
> If duplicates are found, I trace which layer introduced them, fix that cause, then rebuild the affected partitions from the layer above and validate. I avoid blanket `DISTINCT` since it can hide a real logic or key problem.”

### Follow-ups / traps
- How can Hudi produce duplicates? (Wrong key, changing partition path with a non-global index, concurrent writers.)

---

## Q61. A downstream dashboard shows incorrect numbers. How do you investigate?

### Model answer
> “I would start from the dashboard metric and trace it backwards.
>
> First I would verify whether the curated output itself is incorrect.
>
> If it is, I would compare it with the Standardized layer, then Raw and finally the source input.
>
> I would check for duplicated records, missing partitions, incorrect joins, CDC processing errors, late-arriving data or an incorrect business transformation.
>
> I would also compare record counts and key aggregates at each layer.
>
> This layered approach helps identify exactly where the numbers diverged instead of debugging the entire pipeline blindly.”

---

# 10. Airflow Orchestration

## Q62. How did you design your Airflow DAGs for 17 interfaces and 57 tables?

### How to tackle
Config-driven DAGs, clear layer dependencies, small idempotent tasks, isolation between tables.

### Model answer
> “We didn't hand-write 57 separate pipelines. The DAGs are driven by configuration per interface and table (names, keys, layer settings, DQ rules), so onboarding a table is mostly adding configuration rather than new code. DAGs are generated from that config, or tasks are expanded dynamically.
>
> A typical flow per table: wait for the file or batch to arrive, run Landing validation, run the Glue job for Raw, then the Glue CDC job for Standardized, then refresh the Curated objects in Redshift and run checks.
>
> Tasks are small and idempotent, so a failed step can be cleared and rerun alone. Dependencies are explicit: a table can't move to Curated until its Standardized load and checks have passed, and tables sharing a target are serialized.
>
> Independent interfaces run in parallel, with isolation so a failure in one doesn't stop unrelated ones. Failure and retry settings, timeouts and alerts are set in a shared default configuration.
>
> I keep DAG files light, since heavy top-level code slows the scheduler's parsing.”

### Follow-ups / traps
- One DAG per table or per interface? What are the trade-offs?
- How do you handle cross-DAG dependencies? (Sensors, dataset-aware scheduling or triggers.)

### ⚠ Verify before interview
- Confirm whether your DAGs are per interface or per table and whether they are generated from config.

---

## Q63. How do you control concurrency in Airflow (pools, parallelism)?

### How to tackle
Know the layers of limits and why they exist.

### Model answer
> “There are several levels. Environment-level parallelism limits the total running tasks. Per-DAG limits control concurrent tasks and concurrent DAG runs, and per-task limits control concurrent instances of one task. Pools cap concurrency for a shared resource across DAGs.
>
> With 15 to 20 pipelines in parallel, I use pools to protect shared resources: for example a pool limiting concurrent Glue job submissions so we stay inside Glue and account quotas, and a single-slot pool, or a max active runs of one, to serialize writers to the same Hudi table.
>
> On managed Airflow the worker capacity itself scales within configured minimum and maximum workers, so I also keep an eye on whether tasks are queued for lack of slots.
>
> The Glue side matters too: the job's maximum concurrent runs setting must allow the number of parallel runs we trigger, otherwise Glue rejects them.”

### Follow-ups / traps
- What causes tasks to sit in 'queued'?

---

## Q64. How do you handle file arrival and sensors in Airflow?

### How to tackle
Sensor modes and worker-slot cost.

### Model answer
> “For file arrival I use an S3 sensor on the expected file or a manifest marker, with a poke interval and a timeout aligned to the SLA.
>
> Mode matters. A sensor in poke mode holds a worker slot the whole time it waits, which wastes capacity when many sensors wait in parallel. Reschedule mode frees the slot between checks, and deferrable operators hand the waiting to a lightweight trigger process, which is better still on shared infrastructure.
>
> Timeout behavior is deliberate: when the timeout hits, the task fails with a clear upstream-delay reason and alerts, rather than letting downstream continue.
>
> I prefer a marker or manifest file written last by the sender, so the sensor doesn't trigger on a half-uploaded file.”

### Follow-ups / traps
- Why not trigger the DAG from an S3 event instead? (Both work. Events reduce polling, but sensors keep the dependency logic in Airflow.)

---

## Q65. How do you configure retries, timeouts, trigger rules and alerts?

### How to tackle
Show you know the settings and the pitfalls.

### Model answer
> “For tasks that call transient-prone services I set retries with a delay and exponential backoff, plus an execution timeout so nothing runs forever. For tasks that fail on bad data I keep retries low or zero, since repeating won't help.
>
> Trigger rules control what runs after failures. The default requires all upstream tasks to succeed, and I use all-done for cleanup or notification tasks that must run regardless, and the none-failed variants after branching.
>
> For alerting I attach failure callbacks that publish to SNS or the team channel with the DAG, task, run date and log link. For duration or SLA problems, depending on the Airflow version, I use SLA features or alerts on execution time against baseline.
>
> I avoid double-retry multiplication: if Glue retries itself and Airflow retries too, attempts multiply, so I choose one level to own retries.”

### Follow-ups / traps
- What happens to downstream tasks when one upstream fails?

---

## Q66. How do you make DAGs idempotent and handle backfills?

### How to tackle
Logical date, no side effects on rerun, controlled backfills.

### Model answer
> “Every task takes its window from the DAG run's data interval, never from the current time, so rerunning a past interval processes the same data. Targets are written with upserts or partition overwrites, so reruns don't duplicate.
>
> For recovery I clear the failed task, or tasks in a date range, rather than rerunning the whole DAG. For historical backfills I limit concurrency, run in order where state matters, and check capacity so a backfill doesn't starve the live windows.
>
> I turn catchup off by default so a pause doesn't trigger a flood of runs, and use explicit backfills when I want them.
>
> I keep state in tables like the batch ledger, and use XCom only for small metadata, not data.”

### Follow-ups / traps
- What are the risks of catchup on a table with shared writers?

---

## Q67. What should I know about running on managed Airflow (MWAA)?

### How to tackle
Practical operations knowledge.

### Model answer
> “DAGs, plugins and requirements are stored in an S3 bucket the environment syncs from, so deployment is a CI/CD step that uploads there. Python dependencies come through a requirements file, which is why dependency changes can need an environment update and testing.
>
> Capacity depends on the environment class and the worker minimum and maximum. Scheduler health, parse time, queued tasks and worker utilization are exposed as CloudWatch metrics and logs, which I watch.
>
> Access to AWS services goes through the environment's execution role, so least-privilege IAM matters, and secrets live in Secrets Manager rather than in DAG code.
>
> A recurring operational issue is DAG parse time: heavy imports or top-level API calls slow the scheduler for everyone.”

### ⚠ Verify before interview
- Confirm your environment class, worker limits and how DAGs are deployed.

---

# 11. SQL, Athena, Redshift & Spectrum

## Q68. How do you improve SQL query performance?

### How to tackle
Merged answer (includes the general SQL approach). Plan first.

### Model answer
> “I start with the execution plan to find the real bottleneck instead of guessing.
>
> Then I reduce data scanned: selective filters applied early, only the columns needed, and partition or sort-key pruning where the engine supports it. I check join conditions and cardinality so I don't multiply rows, and join order and distribution so large data isn't moved unnecessarily.
>
> I avoid functions on filtered columns that block pruning or index use, avoid unnecessary `DISTINCT`, sorts and correlated subqueries, and use window functions or pre-aggregation where they replace expensive self-joins. Intermediate results can be materialized when they're reused.
>
> I check statistics and engine-specific features: indexes in a transactional database, sort and distribution keys and ANALYZE in Redshift, partitions and file layout in Athena.
>
> Finally I benchmark before and after, comparing time and data scanned, rather than assuming the rewrite helped.”

### Follow-ups / traps
- Do not say 'always use EXISTS instead of IN'. Test with the optimizer.
- Write a query to find the latest record per key. (Use `row_number()` over the key ordered by timestamp descending.)

---

## Q69. A query suddenly becomes slow. What do you check?

### Model answer
> “I first compare the slow query with a previously healthy execution plan.
>
> I check whether data volume changed, statistics are stale, a new join is multiplying rows, an index or partition is no longer being used, or the execution plan changed.
>
> I also check whether the underlying table has recently had large inserts, updates or deletes.
>
> After identifying the cause, I make the smallest targeted change and benchmark it.”

---

## Q70. Your query result has 10 million rows instead of 5 million after a join. What do you check?

### Model answer
> “I would immediately check join cardinality.
>
> If one side has multiple records for the join key, a one-to-many or many-to-many relationship can multiply rows.
>
> I would profile the join keys on both sides, identify duplicate keys and confirm whether the relationship is expected.
>
> I would not simply add `DISTINCT` because that could hide the underlying data or join logic problem.”

---

## Q71. Why would you use Athena instead of Redshift?

### Model answer
> “Athena is useful when I need serverless SQL directly against data in S3, especially for ad-hoc analysis or exploration.
>
> Redshift is more appropriate when I need a warehouse environment for repeated analytical workloads, more controlled performance and warehouse-specific capabilities.
>
> In our architecture, Athena was useful for querying and validating S3 data, while Redshift/Spectrum supported the curated consumption layer.
>
> The decision depends on workload, performance, concurrency, cost and data location rather than saying one service is always better.”

---

## Q72. How do you optimize Athena queries?

### Model answer
> “The biggest principle is to reduce the amount of data Athena scans.
>
> I use columnar formats such as Parquet, partition the data appropriately and make sure queries include partition filters where possible.
>
> I avoid `SELECT *` when only a few columns are required.
>
> I also pay attention to small files because excessive file counts can hurt performance.
>
> Finally, I check the amount of data scanned and compare it before and after the optimization.”

---

## Q73. How do you design Redshift tables: distribution and sort keys?

### How to tackle
Colocate joins and enable range pruning.

### Model answer
> “Distribution decides how rows spread across nodes. KEY distribution on a frequent join column colocates matching rows so joins avoid redistribution, ALL copies a small dimension to every node, EVEN spreads rows evenly when there's no clear join key, and AUTO lets Redshift choose and adjust.
>
> Sort keys decide physical order, so range filters on a sorted column such as a date skip blocks through zone maps. I sort on the most common filter column.
>
> I validate using EXPLAIN. Steps showing large data broadcast or redistribution signal a poor distribution choice for that join. I also keep skew in mind: a distribution key with a few dominant values piles data onto one slice.
>
> For our Curated layer, much of the data is read through Spectrum and materialized views, so these choices matter most for the tables and views that are materialized locally.”

### Follow-ups / traps
- Compound vs interleaved sort keys?

---

## Q74. How do you maintain Redshift performance: VACUUM, ANALYZE, WLM?

### How to tackle
Housekeeping and workload management.

### Model answer
> “VACUUM reclaims space from deletes and updates and re-sorts rows, and ANALYZE refreshes statistics the planner uses. Redshift does both automatically in the background in many cases, but after big loads or deletes I check that tables aren't left unsorted or with stale stats, since stale stats produce bad plans.
>
> For workload management, automatic WLM and concurrency scaling help when many consumers query at once, and I'd separate heavy refresh work from interactive dashboard queries so they don't compete.
>
> For diagnosing slow queries I use EXPLAIN and the system views to look for large scans, nested loop joins and heavy redistribution, then fix keys, filters or the SQL.”

---

## Q75. How do materialized view refreshes work in your Curated layer?

### How to tackle
Staleness is the key concept. Refresh must be ordered after upstream data is ready.

### Model answer
> “A materialized view stores precomputed results, which makes consumer queries fast, but it is only as fresh as its last refresh.
>
> A refresh can be incremental or a full recompute. Whether incremental refresh is possible depends on the query shape and the engine version, so I check the refresh history and status views to see which type actually ran. If a view over external data ends up doing a full recompute, that affects runtime and cost, and I'd consider restructuring it or materializing into a local table.
>
> In our pipeline the refresh is an explicit task in Airflow that runs only after the Standardized load, its catalog or partition sync, and the quality checks have passed. Otherwise the view would refresh against incomplete data.
>
> I also validate counts between the Standardized source and the view after the refresh.”

### Follow-ups / traps
- What happens to the view if a column is added upstream?
- View vs materialized view vs table?

### ⚠ Verify before interview
- Check how your MVs refresh (auto or scheduled by Airflow) and whether they're incremental.

---

## Q76. Why use Redshift Spectrum, and how do you tune it?

### How to tackle
Replaces the earlier short answer. Use cases, pruning, cost model, caveats.

### Model answer
> “Spectrum lets Redshift query data that stays in S3 through external tables from the Glue Catalog, so we don't have to load every dataset into Redshift storage. That suits large lake datasets, and lets us join lake data with local warehouse tables.
>
> In our project, the Standardized data in S3 is referenced via Spectrum in Curated, where we apply business transformations and expose consumer-friendly outputs through materialized views.
>
> Tuning mostly means reducing what Spectrum scans: Parquet, partition columns used in filters so pruning works, fewer and larger files, and selecting only needed columns. It is billed by data scanned, and heavy repeated queries are better served by materializing results locally.
>
> Caveats: new partitions must be visible in the catalog, and table-format support (for example Hudi) has restrictions that depend on table type and version, so compatibility must be checked.”

### Follow-ups / traps
- Athena vs Spectrum for the same S3 data?

---

# 12. AWS Architecture, Events, Deployment & Cost

## Q77. Explain an architecture involving S3, Lambda, Glue, EMR and Redshift.

### Model answer
> “A possible architecture would be:
>
> S3 acts as the data lake and landing zone.
>
> Lambda can handle lightweight event-driven processing, such as detecting a new file and triggering a workflow.
>
> Glue can handle managed ETL and Spark-based transformations.
>
> EMR can be used when we need more control over the Spark/Hadoop environment or specialized cluster-level configuration.
>
> Redshift can serve as the analytical warehouse for data that needs warehouse-oriented querying.
>
> The exact choice between Glue and EMR depends on workload, customization, operational control and cost.”

### Trap
Do not say Lambda should perform heavy ETL on large datasets.

---

## Q78. When would you choose Glue over EMR?

### Model answer
> “I would choose Glue when I want managed Spark ETL with less cluster administration and the workload fits Glue's execution model.
>
> EMR becomes more attractive when I need deeper control over the cluster, custom frameworks, specialized configurations or workloads that benefit from that level of control.
>
> For our project, Glue fit well because our ETL workloads were Spark-based and we wanted a managed AWS service.”

---

## Q79. Walk me through an event-driven file-arrival flow with S3, SNS, SQS, Lambda and Airflow.

### How to tackle
Pattern: **S3 event → SNS (fan-out) → SQS (buffer) → Lambda (light work) → Airflow trigger.**

### Model answer
> “When the integration team drops a file into the Landing bucket, S3 emits an event notification. It goes to an SNS topic so several consumers can subscribe, such as the processing path and an audit or monitoring path.
>
> The processing subscriber is an SQS queue, which buffers bursts when many files land at once and gives retries and a dead-letter queue. A Lambda consumes the queue and does lightweight work: validate the filename and batch convention, record the arrival in the audit table, and trigger the Airflow DAG for that batch, using the managed Airflow API or CLI token mechanism.
>
> Heavy validation and transformation stay in Glue, not Lambda.
>
> Failures are handled with retries, then the DLQ, and an alert, so no file is silently lost.”

### Follow-ups / traps
- What if the same S3 event is delivered twice? (The consumer must be idempotent.)
- Why queue between SNS and Lambda?

### ⚠ Verify before interview
- Confirm which of these services your project actually uses for file arrival and where each sits. Trim the answer to match reality.

---

## Q80. Where did Lambda fit, and what are its limits?

### How to tackle
Replaces the earlier generic Lambda answer. Light, event-driven work only.

### Model answer
> “Lambda handled lightweight, event-driven tasks: reacting to a file arrival, checking the filename and batch convention, writing an audit record, sending a notification or starting a workflow.
>
> I would not use it for large-scale transformation. It has a maximum runtime of 15 minutes, limited memory (up to 10 GB) and ephemeral storage, and no distributed processing, so several gigabytes of data belongs in Glue and Spark.
>
> Operational points: configure memory and timeout sensibly, use reserved concurrency to protect downstream systems, make handlers idempotent since events can be delivered more than once, and for SQS triggers use partial batch failure reporting so one bad message doesn't retry the whole batch.”

### Follow-ups / traps
- How do you handle a Lambda that times out repeatedly?

---

## Q81. SNS vs SQS, and why use SQS?

### How to tackle
Answers the earlier "Why use SQS?" follow-up.

### Model answer
> “SNS is publish/subscribe: one message fans out to many subscribers, pushed immediately. SQS is a queue: messages wait until a consumer processes them.
>
> SNS plus SQS together is a common pattern: SNS fans out, and each consumer gets its own durable queue.
>
> I'd use SQS to decouple producer from consumer, absorb bursts without overloading Lambda or downstream, get automatic retries through the visibility timeout, and isolate poison messages in a dead-letter queue after a set number of attempts.
>
> Details I'd mention: standard queues deliver at least once and don't guarantee ordering, so consumers must be idempotent; FIFO queues give ordering and deduplication within a message group at lower throughput; the visibility timeout must exceed processing time or messages are processed twice; and long polling reduces empty receives.”

### Follow-ups / traps
- How do you redrive messages from a DLQ?

---

## Q82. Where does EventBridge fit, and why Airflow for scheduling?

### How to tackle
Scheduling and event routing, with a clear boundary versus Airflow.

### Model answer
> “EventBridge can run scheduled rules and route events between AWS services. A useful pattern is routing Glue job state-change events to SNS to alert on failures or timeouts, and triggering lightweight automation on a schedule or an event.
>
> For the pipelines themselves I prefer Airflow for scheduling because it handles dependencies, retries, backfills, and visibility across tasks. EventBridge doesn't model those dependency graphs. So EventBridge is the glue for events and alerts; Airflow owns the workflow.”

---

## Q83. How did you manage batch and streaming pipelines?

### Model answer
> “Our main processing model was batch-oriented, with upstream data delivered to S3 and then processed in scheduled chunks.
>
> Some upstream feeds represented database changes through CDC, so although the data represented changes, the overall processing model was still orchestrated in batches rather than being a continuously running streaming application.
>
> I would distinguish CDC from streaming: CDC describes the change data being captured, while streaming describes the way data is continuously transported and processed.”

### Important distinction
**CDC does not automatically mean streaming.**

---

## Q84. Explain Bronze, Silver and Gold layers.

### Model answer
> “They are a common way of describing a layered data architecture.
>
> Bronze generally represents raw or minimally processed data.
>
> Silver represents cleaned, standardized and validated data.
>
> Gold represents business-ready, curated data used by consumers.
>
> In our project, the terminology was closer to Landing, Raw, Standardized and Curated, but the concept is similar: preserve raw information, standardize it, and then create consumer-oriented outputs.”

---

## Q85. How did Terraform help in your project?

### How to tackle
Answers the earlier "How did Terraform help?" follow-up with practical detail.

### Model answer
> “Terraform let us define infrastructure as code, so environments are repeatable, reviewable and version-controlled. We used it for resources like S3 buckets, Glue jobs and their parameters, IAM roles and policies, and messaging resources, with modules to avoid copy-paste across environments.
>
> Environment differences go in variable files, remote state is stored centrally with locking, and a plan is reviewed in the pull request before anything is applied. That catches accidental changes and permission drift.
>
> The practical benefits: adding a new table's Glue job is a configuration change, environments stay consistent, and recovery or rebuild in another region is possible.”

### Follow-ups / traps
- How do you handle someone changing a resource manually in the console? (Drift detection with plan, then import or revert.)

### ⚠ Verify before interview
- Replace the resource list with what your Terraform actually manages.

---

## Q86. Walk me through your CI/CD with GitHub Actions.

### How to tackle
PR checks → deploy to dev → promote to prod with approval.

### Model answer
> “On a pull request, the workflow runs linting and unit tests for the transformation code (using a local Spark session on small test data), and Terraform format, validate and plan, with the plan visible for review.
>
> On merge, it deploys to the lower environment: the Glue scripts go to S3, DAG files go to the managed Airflow bucket, and Terraform applies infrastructure changes. Promotion to production uses a protected environment with manual approval.
>
> AWS access uses short-lived role-based credentials rather than stored keys, and secrets stay in Secrets Manager. Rollback is redeploying the previous tagged version.
>
> I also run a smoke run on a sample after deployment, so we know the pipeline works before the real window.”

### ⚠ Verify before interview
- Match the stages to what your repository really does.

---

## Q87. Do you have deployment experience?

### Model answer
> “Yes. I have exposure to both application and infrastructure deployment.
>
> For infrastructure, I worked with Terraform so resources could be provisioned consistently rather than manually.
>
> For CI/CD, GitHub Actions was used to automate parts of the deployment process.
>
> My Data Engineering contribution included making sure Glue/Airflow-related changes could move through the development and deployment process in a controlled way.”

### Follow-up: What is the advantage of Infrastructure as Code?
> “It makes infrastructure repeatable, version-controlled and easier to review and reproduce across environments.”

---

## Q88. What is zero-ETL and how could it apply to your project?

### How to tackle
Define it accurately, give AWS examples, then give a balanced, hybrid project view.

### Model answer
> “Zero-ETL means managed integrations that replicate data from a source into an analytics target without you building and running extraction pipelines. On AWS, examples are Aurora, RDS and DynamoDB replicating into Redshift, application sources through managed integrations, and auto-copy and streaming ingestion into Redshift. It doesn't literally mean zero transformation. Transformations still happen in the warehouse.
>
> For our project, upstream data arrives from an integration team as files and CDC feeds into S3. If some source systems were Aurora or RDS databases that are supported, zero-ETL into Redshift could give near-real-time replicas for operational reporting, with far less pipeline code to maintain.
>
> But I wouldn't replace the whole design. Our layered lake gives us immutable Raw for replay, validation and quarantine at Landing, Hudi for governed CDC state, and a catalog usable by Athena and Spectrum. Zero-ETL has limited source support, less control over transformation and validation, and lands data in Redshift rather than our lake layers.
>
> So I'd propose a hybrid: use zero-ETL where the source is supported and consumers need low latency, keep the lake pipeline where we need governance, history and replay, and run data quality checks on the replicated tables too.”

### Follow-ups / traps
- What limits does zero-ETL have? (Supported sources, transformation control, schema changes, regional and quota constraints.)
- Does it remove the need for data modeling and quality? No.

---

## Q89. Did you achieve cost savings? How would you measure them?

### How to tackle
Define the measurement, then give one concrete example you can defend.

### Model answer
> “I measure cost by comparing a baseline and the optimized workload on both cost and performance: Glue worker-hours or DPU-hours per run, runtime, S3 storage and request volume, Athena data scanned, and Redshift usage. A change that cuts runtime but needs much more compute isn't automatically a saving, so I track cost per successful run alongside SLA and reliability.
>
> An example from our pipelines: several small reference tables ran on larger worker counts than needed. After checking executor utilization, we reduced them to the minimum and, for lighter jobs, enabled auto scaling. Runtime barely changed, and the worker-hours dropped noticeably.
>
> The second lever was reading less: fixing partition filters and column selection reduced scan and shuffle, which cut both runtime and cost.”

### ⚠ Verify before interview
- Replace the example with a real change from your project, with a before and after number if you can find one.

---

# 13. Project Stories, Prioritization & Improvements

## Q90. What production issues have you handled, and how?

### How to tackle
Pick 4–5 and use one structure each: **Symptom → Cause → Fix → Prevention.** Keep each to 30–45 seconds. These are plausible scenarios built from your pipeline. Replace or adjust them with what really happened.

### Model answer
> “Here are a few representative ones.
>
> **1. Late source file blocking a window.** The hourly run for an interface sat waiting and then failed with no data. The cause was the upstream delivery running late. We moved to a sensor in reschedule mode with an SLA-based timeout and a clear upstream-delay alert, and the window was backfilled once the file arrived. After that, the team knew within minutes whether it was a source delay or our failure.
>
> **2. A new column that never reached Curated.** The source added a column to a CDC feed. The Glue job succeeded, but the column was missing downstream because the transformation selects fixed columns, and the materialized view had a fixed definition. We updated the projection and recreated the view, added schema-diff detection at Landing, and now test with old and new file shapes.
>
> **3. Slow job from skew.** A join on the inventory tables had one task running far longer than the rest because a few placeholder or high-volume locations dominated. We confirmed in Spark UI, handled the placeholder values separately and enabled AQE skew handling, which brought the stage back inside the window.
>
> **4. Concurrent writers on one Hudi table.** Two runs hit the same target and one failed with a write conflict. We serialized them with an Airflow pool and dependency, and made sure each table has a single owner pipeline.
>
> **5. Small files hurting query speed.** Spectrum queries on a Standardized table slowed down as small files accumulated. We tuned file sizing and ran clustering, which cut file counts and improved the queries.
>
> In each case I validated counts after the fix and added a check or alert so we'd find it earlier next time.”

### ⚠ Verify before interview
- These are built from your architecture. Keep only those that match real incidents and swap in real details, and never claim one you can't discuss in depth.

---

## Q91. What was the biggest issue you faced and what did you learn?

### How to tackle
Pick one real, substantial incident with a clear arc: impact, investigation, fix, lesson. This also covers "what was a major challenge in your project". Below is a plausible scenario using your inventory tables. Replace it with your real one, keeping the structure.

### Model answer
> “The most significant issue was a record key problem on an inventory table.
>
> **Symptom.** A consumer noticed inventory totals in Curated were lower than the source. The jobs had all succeeded, so nothing had alerted.
>
> **Investigation.** I compared counts across layers: Raw matched the source, Standardized was lower. That narrowed it to the Hudi write. The table's record key was only the item, but inventory is held per item and location, so upserts for different locations of the same item were overwriting each other. Rows were collapsing, not duplicating.
>
> **Fix.** We changed the record key to a composite of item and location, using the complex key generator. Because Raw keeps every CDC record, we rebuilt the Standardized table from Raw with a bulk load and then resumed incremental upserts. We refreshed Curated and reconciled totals to the source before telling consumers it was fixed.
>
> **Prevention.** Before onboarding any table, we now profile the proposed key against the source to prove uniqueness. We added per-layer count reconciliation and a key-uniqueness check so a silent collapse is caught immediately. A second engineer reviews key and ordering-field configuration.
>
> **Lesson.** A successful job doesn't mean correct data. Key design is a correctness decision, not a configuration detail, and having an immutable Raw layer is what made recovery clean.”

### Follow-ups / traps
- How did you tell the business and manage the impact window?
- What would you do differently?

### ⚠ Verify before interview
- This is a plausible scenario, not a record of events. Replace it with your real biggest incident and keep the same structure.

---

## Q92. How do you prioritize deliverables when you have multiple tasks?

### How to tackle
Give a clear order of priority, show you communicate trade-offs, and give a short example.

### Model answer
> “I prioritize by business impact and urgency, with a fixed order of precedence.
>
> First, production incidents and anything threatening an SLA, because consumers are affected now. Second, data correctness issues, since wrong data is worse than late data. Third, committed deliverables with deadlines. Fourth, improvements and tech debt.
>
> I clarify the real deadline and impact with the requester instead of assuming, break large tasks into smaller pieces and flag dependencies and blockers early.
>
> When priorities conflict, I raise it with my lead with the trade-off stated clearly, for example: 'If I take the new table onboarding today, the reconciliation fix slips to Thursday. Which do you prefer?' That keeps decisions with the people who own them.
>
> A practical example: if a production failure lands in the middle of onboarding a new table, I pause onboarding, stabilize production, tell stakeholders the new ETA, and resume afterwards.”

### Follow-ups / traps
- What if two stakeholders both say theirs is urgent?
- How do you avoid constant context switching?

---

## Q93. What enhancements would you suggest for your pipeline?

### How to tackle
Merged with "what would you improve". Group by theme, pick 3–4 with a reason.

### Model answer
> “I'd group them into four areas.
>
> **Observability.** Data-level monitoring dashboards (input and output counts, latency, schema changes, duplicate rates) and performance baselines so abnormal Spark behavior is detected automatically instead of after users complain.
>
> **Quality and reliability.** A config-driven DQ framework with severity levels and quarantine, automated count reconciliation between layers, and automated schema-drift detection at Landing.
>
> **Performance and cost.** Scheduled Hudi table services (clustering and cleaning), right-sizing Glue per table class with auto scaling, and tracking cost per run against volume and SLA.
>
> **Engineering practice.** Reusable Terraform modules, unit and data-contract tests in CI, deferrable operators in Airflow, and clearer runbooks for recurring failures. Where sources allow, evaluate zero-ETL for low-latency consumers.
>
> If I had to choose one first, I'd choose data-level observability and reconciliation, because silent data issues cost the business most.”

---

## Q94. What level of architecture decisions did you make?

### Model answer
> “I was not the person defining the entire enterprise architecture. The high-level architecture and standards were defined by senior architects and leads.
>
> My responsibility was to take those designs and implement them correctly, make technical decisions within my pipeline scope and troubleshoot issues in production.
>
> I also contributed recommendations around Spark optimization, partitioning, error handling and pipeline reliability based on the workloads I owned.”

---

# 14. Project Explanation (Short & Deep)

## Q95. Explain your project in 60 seconds.

### Model answer
> “My project is in the retail and e-commerce domain. We build data pipelines that prepare data for downstream teams such as Data Science, Operations, Marketing and Purchasing.
>
> Upstream systems deliver files and CDC data into S3 through an integration layer. Our Landing layer validates the incoming data and registers metadata in Glue Catalog.
>
> We then move the data through Raw and Standardized layers. In Standardized, we use AWS Glue, Spark and Hudi to handle incremental processing, deduplication, inserts, updates and deletes using CDC.
>
> In the Curated layer, we use Redshift Spectrum and Materialized Views to provide business-ready data to downstream consumers.
>
> Airflow orchestrates the end-to-end pipeline, and I have been involved in building, optimizing, troubleshooting and deploying these pipelines.”

---

## Q96. Explain your project as if I want technical depth.

### Model answer
> “The project is a retail and e-commerce data platform.
>
> Multiple upstream systems provide files and CDC feeds. The integration team delivers those datasets into our S3 Landing area.
>
> At Landing, we perform file-arrival, filename/batch and structural validation, and register metadata through the Glue Data Catalog.
>
> In Raw, we preserve the ingested information and add technical metadata such as ingestion timestamps and primary-key information.
>
> The Standardized layer is where the main data-engineering logic happens. We use Glue with Spark and Apache Hudi. The CDC payload contains the operation type and update timestamp. Inserts and updates are handled through upsert logic, while deletes are handled separately. The record key identifies the target record, and the update-ordering field helps determine the latest valid event.
>
> Once standardized, the data is exposed through Redshift Spectrum in the Curated layer. We apply the required business transformations and expose consumer-friendly datasets through Materialized Views.
>
> Airflow orchestrates the dependencies, scheduling and retries.
>
> From a performance perspective, we monitor partitioning, shuffle, data skew, join strategy, small files and Glue resource usage. From a reliability perspective, we focus on validation, idempotent processing, retries and safe recovery.”

---

# Appendix A — Duplicates Removed (where the original questions went)

- Original Q13, Q14, Q79 (Fail at midnight / debug failed Glue job / general pipeline approach) → now **Q16**
- Original Q15 (Partial output rerun) → now **Q18**
- Original Q16 (Retries) → now **Q17**
- Original Q17, Q65, Q71, Q73, Q80 (Slow Spark/Glue job, optimization techniques, Glue 5x slower, join slow after growth, general Spark approach) → now **Q24**
- Original Q18, Q64, Q83, Q94, Q98 (Data skew, real skew example, repartition vs skew, 60% customer, one 40-minute task) → now **Q28**
- Original Q19, Q23, Q90 (Broadcast join, 2 TB + 50 MB join, how small) → now **Q29**
- Original Q82, Q84, Q85 (Repartition cost, too many partitions, too few partitions) → now **Q30**
- Original Q24, Q25 (Partition strategy followed, choosing partition columns) → now **Q35**
- Original Q26, Q27 (Small files, coalesce(1)) → now **Q36**
- Original Q28 (Late-arriving data (rewritten in place)) → now **Q54**
- Original Q29, Q61, Q62 (CDC design, CDC/Hudi flow, most important CDC point) → now **Q39**
- Original Q32, Q88 (Hudi upserts, Hudi duplicates trap) → now **Q44**
- Original Q33, Q34 (Duplicate CDC event, out-of-order events) → now **Q43**
- Original Q78 (Two pipelines on same Hudi table) → now **Q51**
- Original Q35, Q36, Q37, Q89, Q96 (Schema evolution, new column, int to string, schema merge trap, column missing downstream) → now **Q57**
- Original Q38, Q39 (Incremental loads, unreliable timestamp) → now **Q56**
- Original Q40 (Data quality (rewritten in place)) → now **Q59**
- Original Q41, Q74 (Duplicate records, duplicates in output) → now **Q60**
- Original Q42 (Huge CSV (rewritten in place)) → now **Q38**
- Original Q43 (Billions of records (rewritten in place)) → now **Q15**
- Original Q44, Q45 (5x and 10x volume) → now **Q14**
- Original Q46, Q81 (SQL performance, general SQL approach) → now **Q68**
- Original Q50 (Redshift Spectrum (rewritten in place)) → now **Q76**
- Original Q53 (Lambda (rewritten in place)) → now **Q80**
- Original Q55 (Autoscaling) → now **Q32**
- Original Q56 (Cost savings (rewritten in place)) → now **Q89**
- Original Q57, Q58, Q75 (Monitoring, monitoring-only-failures, no data but job succeeds) → now **Q23**
- Original Q59 (S3 accidental deletion) → now **Q22**
- Original Q63 (Major project challenge) → now **Q91**
- Original Q76 (Malformed file) → now **Q20**
- Original Q77, Q95 (Delayed source, daily file six hours late) → now **Q55**
- Original Q92 (What would you improve) → now **Q93**
- Original Q93, Q100 (Glue under one hour, only one optimization under SLA) → now **Q33**
- Original Q10, Q11, Q12 (Success rate, cluster configuration, execution time (rewritten in place)) → now Q10, Q11, Q12
- Original Q66 (Which integrations have you worked with?) → removed as a duplicate of the services and deployment answers (Q2, and the Terraform, CI/CD and event-flow answers)

Original questions kept word-for-word (renumbered): Q1→Q1, Q2→Q2, Q3→Q3, Q4→Q4, Q5→Q5, Q6→Q6, Q7→Q7, Q8→Q8, Q9→Q9, Q20→Q25, Q21→Q26, Q22→Q27, Q30→Q40, Q31→Q42, Q47→Q69, Q48→Q71, Q49→Q72, Q51→Q77, Q52→Q78, Q54→Q83, Q60→Q61, Q67→Q87, Q68→Q84, Q69→Q45, Q70→Q46, Q72→Q37, Q86→Q95, Q87→Q96, Q91→Q94, Q97→Q70, Q99→Q19

---

# Appendix B — Pre-Interview Verification Checklist

These answers use plausible scenarios or ballpark numbers. Check each against your real project before you rely on it.

- **Q10. What was the pipeline success rate?**
  - Replace "97–98%" with a real figure from Airflow (DAG runs filtered by state over the last 30–90 days).
- **Q11. What was your cluster size/configuration?**
  - Check the real worker type and count in the Glue job definitions (or Terraform variables) for 2–3 of your tables and replace the ranges.
- **Q12. What was the average execution time?**
  - Pull the actual min/typical/max task durations from the Airflow Gantt or task duration view.
- **Q28. How do you handle data skew?**
  - If you hit skew in a real table, replace the inventory example with it.
- **Q35. How do you choose partition columns (and what did you follow)?**
  - Confirm your real partition columns for 2–3 tables.
- **Q43. What if the same CDC event arrives twice, or events arrive out of order?**
  - Check your Hudi version and configured payload class or merge mode.
- **Q47. Copy-on-Write vs Merge-on-Read: which did you use and why?**
  - Confirm your actual table type. If you used MoR, swap the conclusion and mention compaction scheduling.
- **Q53. How are Hudi tables queried through Athena and Redshift Spectrum?**
  - Confirm how your Curated layer reads the Standardized Hudi tables and what restrictions apply for your version.
- **Q62. How did you design your Airflow DAGs for 17 interfaces and 57 tables?**
  - Confirm whether your DAGs are per interface or per table and whether they are generated from config.
- **Q67. What should I know about running on managed Airflow (MWAA)?**
  - Confirm your environment class, worker limits and how DAGs are deployed.
- **Q75. How do materialized view refreshes work in your Curated layer?**
  - Check how your MVs refresh (auto or scheduled by Airflow) and whether they're incremental.
- **Q79. Walk me through an event-driven file-arrival flow with S3, SNS, SQS, Lambda and Airflow.**
  - Confirm which of these services your project actually uses for file arrival and where each sits. Trim the answer to match reality.
- **Q85. How did Terraform help in your project?**
  - Replace the resource list with what your Terraform actually manages.
- **Q86. Walk me through your CI/CD with GitHub Actions.**
  - Match the stages to what your repository really does.
- **Q89. Did you achieve cost savings? How would you measure them?**
  - Replace the example with a real change from your project, with a before and after number if you can find one.
- **Q90. What production issues have you handled, and how?**
  - These are built from your architecture. Keep only those that match real incidents and swap in real details, and never claim one you can't discuss in depth.
- **Q91. What was the biggest issue you faced and what did you learn?**
  - This is a plausible scenario, not a record of events. Replace it with your real biggest incident and keep the same structure.

---

# Final Interview Cheat Sheet

## The answer pattern to use in almost every technical question

### Step 1 — Answer directly

> “Yes, I have handled this. My approach is…”

### Step 2 — Explain your investigation

> “First I would check…”

### Step 3 — Connect it to your project

> “In our project, we handled this through…”

### Step 4 — Explain the technical decision

> “The reason we used this approach was…”

### Step 5 — Mention validation

> “After the change, I would validate…”

### Step 6 — Mention trade-offs

> “I would not blindly do X because…”

---

# Golden Rules for the Interview

## 1. Never start with “increase workers”

For Spark/Glue performance questions:

**Diagnose → Optimize data → Optimize Spark → Scale resources → Validate cost/SLA**

## 2. Never say “repartition fixes skew”

Instead:

> “I first confirm skew. Repartitioning by the same skewed key may not solve it. Depending on the workload, I would consider broadcast, salting or AQE.”

## 3. Never say “Hudi automatically prevents duplicates”

Instead:

> “Hudi supports key-based upserts, but correct idempotency depends on the record key, ordering/precombine field and overall write design.”

## 4. Never say “Glue Catalog schema update means the pipeline automatically supports new columns”

Instead:

> “Catalog metadata, transformation schema and target schema are separate concerns.”

## 5. Never claim an exact metric you cannot defend

If asked for worker configuration, execution time or success rate and you do not remember:

> “I don't want to quote an inaccurate number. What I can explain is how we configured and monitored it…”

This sounds much better than inventing a number.

---

## 6. Always give the real default when one exists

For broadcast joins: the default `spark.sql.autoBroadcastJoinThreshold` is 10 MB. Then explain why you still verify the plan.

## 7. Never say "Zero-ETL means no transformation"

Instead:

> "Zero-ETL removes the extraction pipeline for supported sources. Transformation, quality checks and modeling still happen downstream."

## 8. Never say "retries will fix it"

Instead:

> "Retry transient failures, fix data failures, and make sure the processing is idempotent so a retry is safe."

## 9. Never present a plausible scenario as fact

Anything marked **⚠ Verify** in this bank must be replaced with what actually happened in your project before the interview.

---

# One-Line Mental Models

| Topic | Remember |
|---|---|
| Pipeline troubleshooting | **Detect → Locate → Diagnose → Recover → Validate → Prevent** |
| Spark optimization | **Read less → Shuffle less → Skew less → Compute efficiently** |
| SQL optimization | **Plan → Scan → Filter → Join → Aggregate → Index/Partition → Benchmark** |
| CDC | **Key + Operation + Ordering + Idempotency** |
| SCD Type 1 | **Keep the latest state; don't preserve old versions** |
| S3 optimization | **Partition + Parquet + Compression + Right file sizes** |
| Data skew | **Find the hot key first** |
| Retry | **Retry transient failures; fix data failures** |
| Schema evolution | **Detect → Classify → Validate → Evolve → Test** |
| Scaling | **Optimize first, then scale** |
| Production debugging | **Follow evidence, don't guess** |
| 3–5 YOE positioning | **Own implementation, explain decisions, don't pretend to be the architect** |

---

# Final Interview Mindset

You do **not** need to sound like a Senior Architect.

At 3–5 years, a strong answer should show that you can:

- Understand the end-to-end pipeline
- Explain the AWS services you actually used
- Debug production failures systematically
- Understand Spark execution and optimization
- Handle CDC and incremental processing
- Think about data correctness, not just job success
- Explain partitioning and file layout
- Understand joins, shuffle and skew
- Make sensible scaling decisions
- Explain trade-offs
- Own the pipelines you worked on
- Clearly distinguish what you personally implemented from what the architecture team designed

The strongest pattern is:

> **“Here is what I would check first, here is what I found, here is why I chose this solution, and here is how I validated it.”**

That is the tone to maintain throughout the interview.
