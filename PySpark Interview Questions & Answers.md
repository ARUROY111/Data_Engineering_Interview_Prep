# PySpark Interview Questions & Answers

## Data Engineering — 3–5 Years Experience

This document covers the most important **PySpark interview questions**, from basic DataFrame operations to joins, window functions, Spark architecture, optimization, data skew, incremental processing, and real-world Data Engineering scenarios.

---

# 1. How do you read a CSV file in PySpark?

```python
df = spark.read.csv(
    "employees.csv",
    header=True,
    inferSchema=True
)

df.show()
```

### Alternative

```python
df = (
    spark.read
    .format("csv")
    .options(
        header="true",
        inferSchema="true"
    )
    .load("employees.csv")
)
```

### Interview point

`inferSchema=True` automatically determines column data types.

For production pipelines, explicitly defining the schema is generally better because schema inference adds overhead and can produce unexpected types.

---

# 2. How do you create a SparkSession?

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("PySparkPractice")
    .getOrCreate()
)
```

### What is SparkSession?

`SparkSession` is the entry point for working with Spark DataFrames, SQL, and Spark functionality.

---

# 3. How do you check the Spark version?

```python
print(spark.version)
```

---

# 4. How do you create a DataFrame manually?

```python
data = [
    (1, "Arun", 50000),
    (2, "Rahul", 65000),
    (3, "Priya", 45000)
]

columns = ["id", "name", "salary"]

df = spark.createDataFrame(data, columns)

df.show()
```

---

# 5. How do you inspect a DataFrame?

### Display data

```python
df.show()
```

### Display schema

```python
df.printSchema()
```

### Get column names

```python
print(df.columns)
```

### Get number of rows

```python
df.count()
```

### Get first row

```python
df.first()
```

### Get first few rows

```python
df.head(5)
```

---

# 6. How do you select columns?

```python
df.select(
    "name",
    "salary"
).show()
```

Using `col()`:

```python
from pyspark.sql.functions import col

df.select(
    col("name"),
    col("salary")
).show()
```

---

# 7. How do you filter rows?

```python
df.filter(
    col("salary") > 50000
).show()
```

You can also use:

```python
df.where(
    col("salary") > 50000
).show()
```

`filter()` and `where()` are essentially equivalent.

---

# 8. How do you filter and select together?

```python
df.filter(
    col("salary") > 50000
).select(
    "name",
    "department"
).show()
```

---

# 9. How do you rename a column?

### Using `alias()`

```python
df.select(
    col("salary").alias("monthly_salary")
).show()
```

### Using `withColumnRenamed()`

```python
df = df.withColumnRenamed(
    "salary",
    "monthly_salary"
)
```

### Difference

`alias()` is generally used while selecting/expressing a column.

`withColumnRenamed()` changes the DataFrame column name.

---

# 10. How do you create a new column using `withColumn()`?

Suppose:

```text
salary = 50000
```

We want:

```text
annual_salary = salary * 12
```

```python
df = df.withColumn(
    "annual_salary",
    col("salary") * 12
)
```

---

# 11. How do you use `when()` and `otherwise()`?

Example:

```python
from pyspark.sql.functions import when

df = df.withColumn(
    "salary_category",
    when(col("salary") >= 60000, "High")
    .when(col("salary") >= 50000, "Medium")
    .otherwise("Low")
)
```

Result:

```text
salary    salary_category
65000     High
50000     Medium
45000     Low
```

---

# 12. How do you perform GROUP BY and COUNT?

```python
df.groupBy(
    "department"
).count().show()
```

Equivalent SQL:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

---

# 13. How do you perform multiple aggregations?

```python
from pyspark.sql.functions import (
    avg,
    max,
    min,
    sum,
    count
)

df.groupBy("department").agg(
    count("*").alias("employee_count"),
    avg("salary").alias("avg_salary"),
    max("salary").alias("max_salary"),
    min("salary").alias("min_salary"),
    sum("salary").alias("total_salary")
).show()
```

---

# 14. How do you filter aggregated results?

This is similar to SQL `HAVING`.

```python
df.groupBy("department").agg(
    avg("salary").alias("avg_salary")
).filter(
    col("avg_salary") > 50000
).show()
```

Equivalent SQL:

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

---

# 15. How do you check NULL values?

```python
df.filter(
    col("salary").isNull()
).show()
```

For NOT NULL:

```python
df.filter(
    col("salary").isNotNull()
).show()
```

---

# 16. How do you replace NULL values?

```python
df.fillna({
    "salary": 0,
    "department": "Unknown"
})
```

You can also replace one column:

```python
df = df.fillna(
    0,
    subset=["salary"]
)
```

---

# 17. How do you remove rows containing NULL values?

```python
df.dropna().show()
```

For specific columns:

```python
df.dropna(
    subset=["name", "salary"]
).show()
```

---

# 18. How do you get distinct records?

```python
df.distinct().show()
```

This removes completely duplicate rows.

---

# 19. How do you remove duplicates based on specific columns?

```python
df.dropDuplicates(
    ["employee_id"]
).show()
```

### Important interview point

If you simply do:

```python
df.dropDuplicates(["employee_id"])
```

Spark does **not** guarantee that the record you want is retained.

If you need to keep the **latest record**, use a window function.

---

# 20. How do you perform an INNER JOIN?

Suppose:

```text
employees
employee_id | name | department_id

departments
department_id | department
```

```python
result = employees.join(
    departments,
    employees.department_id == departments.department_id,
    "inner"
)
```

---

# 21. How do you perform a LEFT JOIN?

```python
result = employees.join(
    departments,
    employees.department_id == departments.department_id,
    "left"
)
```

A left join keeps **all records from the left DataFrame**.

---

# 22. How do you find records that don't exist in another DataFrame?

Use a **LEFT ANTI JOIN**.

```python
result = employees.join(
    departments,
    employees.department_id == departments.department_id,
    "left_anti"
)
```

This returns employees whose department does not exist in the departments DataFrame.

### Very important interview topic

Know:

```text
inner
left
right
full
left_semi
left_anti
```

---

# 23. What is a LEFT SEMI JOIN?

A left semi join returns rows from the left DataFrame where a matching record exists in the right DataFrame.

```python
result = employees.join(
    departments,
    employees.department_id == departments.department_id,
    "left_semi"
)
```

### Difference

`left_semi`:

> Give me left-side records that have a match.

`left_anti`:

> Give me left-side records that do NOT have a match.

---

# 24. How do you join using multiple columns?

```python
result = df1.join(
    df2,
    (df1.customer_id == df2.customer_id) &
    (df1.country == df2.country),
    "inner"
)
```

Remember:

```text
&  → AND
|  → OR
~  → NOT
```

Always put individual conditions inside parentheses.

---

# 25. How do you find the highest-paid employee in each department?

Use a window function.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number

window = Window.partitionBy(
    "department"
).orderBy(
    col("salary").desc()
)

result = df.withColumn(
    "rn",
    row_number().over(window)
).filter(
    col("rn") == 1
)

result.show()
```

---

# 26. How do you find the top 3 salaries in each department?

```python
window = Window.partitionBy(
    "department"
).orderBy(
    col("salary").desc()
)

result = df.withColumn(
    "rn",
    row_number().over(window)
).filter(
    col("rn") <= 3
)
```

---

# 27. Difference between ROW_NUMBER, RANK and DENSE_RANK

Suppose salaries are:

```text
100000
90000
90000
80000
```

### ROW_NUMBER

```text
100000 → 1
90000  → 2
90000  → 3
80000  → 4
```

Every row gets a unique number.

### RANK

```text
100000 → 1
90000  → 2
90000  → 2
80000  → 4
```

There is a gap after the tie.

### DENSE_RANK

```text
100000 → 1
90000  → 2
90000  → 2
80000  → 3
```

No gap after the tie.

---

# 28. How do you find the third-highest salary per department?

Use `dense_rank()`.

```python
from pyspark.sql.functions import dense_rank

window = Window.partitionBy(
    "department"
).orderBy(
    col("salary").desc()
)

result = df.withColumn(
    "salary_rank",
    dense_rank().over(window)
).filter(
    col("salary_rank") == 3
)
```

### Why DENSE_RANK?

Because if two employees have the same salary, they should have the same rank.

---

# 29. How do you find the latest record for each customer?

Suppose:

```text
customer_id
transaction_id
transaction_date
```

Use:

```python
window = Window.partitionBy(
    "customer_id"
).orderBy(
    col("transaction_date").desc()
)

result = df.withColumn(
    "rn",
    row_number().over(window)
).filter(
    col("rn") == 1
)
```

This is one of the **most common PySpark interview questions**.

---

# 30. How do you remove duplicates while keeping the latest record?

```python
window = Window.partitionBy(
    "customer_id"
).orderBy(
    col("updated_at").desc()
)

result = df.withColumn(
    "rn",
    row_number().over(window)
).filter(
    col("rn") == 1
).drop(
    "rn"
)
```

### Why not simply use `dropDuplicates()`?

Because `dropDuplicates()` does not allow you to specify:

> Keep the latest record.

Window functions give you deterministic control over which record is retained.

---

# 31. How do you get the previous value using LAG?

```python
from pyspark.sql.functions import lag

window = Window.partitionBy(
    "customer_id"
).orderBy(
    "transaction_date"
)

result = df.withColumn(
    "previous_amount",
    lag("amount").over(window)
)
```

Example:

```text
date        amount    previous_amount

Jan 1       100       NULL
Jan 2       200       100
Jan 3       300       200
```

---

# 32. How do you get the next value using LEAD?

```python
from pyspark.sql.functions import lead

result = df.withColumn(
    "next_amount",
    lead("amount").over(window)
)
```

Example:

```text
date        amount    next_amount

Jan 1       100       200
Jan 2       200       300
Jan 3       300       NULL
```

---

# 33. How do you calculate a running total?

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import sum

window = Window.orderBy(
    "transaction_date"
).rowsBetween(
    Window.unboundedPreceding,
    Window.currentRow
)

result = df.withColumn(
    "running_total",
    sum("amount").over(window)
)
```

---

# 34. How do you calculate a running total for each customer?

```python
window = (
    Window
    .partitionBy("customer_id")
    .orderBy("transaction_date")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

result = df.withColumn(
    "running_total",
    sum("amount").over(window)
)
```

---

# 35. What is the difference between repartition() and coalesce()?

This is a **very important Spark interview question**.

## repartition()

```python
df = df.repartition(10)
```

or:

```python
df = df.repartition(
    10,
    "department"
)
```

`repartition()` performs a **shuffle**.

It can increase or decrease the number of partitions.

---

## coalesce()

```python
df = df.coalesce(5)
```

`coalesce()` is primarily used to **reduce** the number of partitions without a full shuffle.

### Simple interview answer

> `repartition()` causes a shuffle and can increase or decrease partitions, while `coalesce()` is generally used to reduce partitions with less data movement.

---

# 36. What is data skew?

Data skew occurs when data is distributed unevenly across partitions.

Example:

```text
Partition 1 → 1 million records
Partition 2 → 10 records
Partition 3 → 15 records
Partition 4 → 20 records
```

One task takes much longer than the others.

This creates a **straggler task**.

---

# 37. How do you handle data skew?

Common approaches:

### 1. Broadcast join

If one table is small:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "department_id"
)
```

### 2. Salting

Add a random or calculated salt to distribute heavily repeated keys.

### 3. Adaptive Query Execution

Enable AQE:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

### 4. Repartitioning

Partition data appropriately based on the join/grouping key.

---

# 38. What is a broadcast join?

A broadcast join sends a small DataFrame to all worker nodes.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "id"
)
```

Instead of shuffling the large DataFrame across the cluster, Spark can use the small table locally on each executor.

### Good use case

```text
Large employee transactions
+
Small department lookup table
```

---

# 39. What is lazy evaluation in Spark?

Spark transformations are **lazy**.

For example:

```python
df2 = df.filter(
    col("salary") > 50000
)

df3 = df2.select(
    "name",
    "salary"
)
```

Nothing is actually executed yet.

Spark waits until an **action** occurs:

```python
df3.show()
```

or:

```python
df3.count()
```

### Why?

Spark can optimize the entire execution plan before running it.

---

# 40. What are transformations and actions?

## Transformations

Transformations create a new DataFrame/RDD.

Examples:

```text
select()
filter()
where()
withColumn()
join()
groupBy()
drop()
```

They are lazy.

## Actions

Actions trigger execution.

Examples:

```text
show()
count()
collect()
first()
take()
write
```

---

# 41. What is the difference between narrow and wide transformations?

## Narrow transformation

Data does not need to move between partitions.

Examples:

```text
filter()
select()
withColumn()
```

## Wide transformation

Data needs to move between partitions.

Examples:

```text
groupBy()
join()
distinct()
orderBy()
```

Wide transformations usually involve a **shuffle**.

---

# 42. What is a shuffle?

Shuffle is the process of redistributing data across partitions.

For example:

```python
df.groupBy("department").count()
```

Spark needs all records belonging to the same department to be brought together.

That requires data movement between executors.

This is expensive because of:

- Network I/O
- Disk I/O
- Serialization
- CPU overhead

### Interview answer

> Shuffle is the redistribution of data across partitions, usually caused by operations such as joins, groupBy, distinct and orderBy.

---

# 43. What is cache()?

```python
df.cache()
```

Caching stores the DataFrame so subsequent operations can reuse the computed data.

Example:

```python
df.cache()

df.count()
df.filter(col("salary") > 50000).show()
```

The first action materializes the cache.

---

# 44. What is the difference between cache() and persist()?

`cache()` is essentially shorthand for a default persistence level.

```python
df.cache()
```

`persist()` lets you specify the storage level.

```python
from pyspark import StorageLevel

df.persist(
    StorageLevel.MEMORY_AND_DISK
)
```

### Important

Do not cache everything.

Cache when:

- Data is reused multiple times.
- Recomputing it is expensive.
- It fits reasonably well in available resources.

---

# 45. Explain Spark architecture.

A simplified Spark architecture is:

```text
                Driver
                  |
           Cluster Manager
                  |
       ---------------------
       |         |         |
   Executor   Executor   Executor
       |         |         |
     Tasks     Tasks     Tasks
```

## Driver

The driver:

- Creates SparkSession
- Builds the execution plan
- Creates jobs
- Coordinates execution
- Schedules tasks

## Cluster Manager

Allocates resources.

Examples:

```text
YARN
Kubernetes
Standalone
```

## Executors

Executors run tasks and store cached data.

---

# 46. What are Jobs, Stages and Tasks?

Spark execution hierarchy:

```text
Application
    ↓
   Job
    ↓
  Stages
    ↓
  Tasks
```

### Job

Usually created when an action is called.

Example:

```python
df.count()
```

### Stage

A job is divided into stages around shuffle boundaries.

### Task

A task is the smallest unit of execution sent to an executor.

Typically:

> One task processes one partition for a stage.

---

# 47. What is the Catalyst Optimizer?

Catalyst is Spark SQL's query optimization framework.

It analyzes and optimizes the logical and physical execution plan.

Examples of optimizations include:

- Predicate pushdown
- Column pruning
- Constant folding
- Join optimization
- Expression simplification

---

# 48. What is predicate pushdown?

Suppose we have:

```python
df = spark.read.parquet("employees")
```

Then:

```python
df.filter(
    col("salary") > 50000
)
```

Spark can push the filter closer to the data source.

Instead of reading all data and then filtering, the storage layer may read only relevant data.

### Benefit

Less:

- Disk I/O
- Network I/O
- Memory usage

---

# 49. What is column pruning?

Suppose a Parquet file contains:

```text
id
name
salary
department
address
phone
email
```

But we only need:

```python
df.select(
    "name",
    "salary"
)
```

Spark can read only the required columns instead of all columns.

This is called **column pruning**.

---

# 50. Why is Parquet commonly used in Data Engineering?

Parquet is a **columnar storage format**.

Advantages:

- Column pruning
- Predicate pushdown
- Compression
- Efficient analytical queries
- Good integration with Spark
- Supports schema information

Example:

```python
df.write.parquet(
    "s3://bucket/employees/"
)
```

For analytical workloads, Parquet is usually much better than CSV.

---

# 51. What is the small-file problem?

Suppose your S3 location contains:

```text
file1.parquet
file2.parquet
file3.parquet
...
file1,000,000.parquet
```

Millions of tiny files create overhead.

Problems include:

- File listing overhead
- Too many tasks
- Metadata overhead
- Poor performance
- Excessive S3/API calls

### Solution

Control output partitioning:

```python
df.coalesce(10).write.parquet(
    "s3://bucket/output/"
)
```

or:

```python
df.repartition(10).write.parquet(
    "s3://bucket/output/"
)
```

Choose based on whether a shuffle is appropriate.

---

# 52. Why would you repartition before writing data?

Suppose the DataFrame has thousands of partitions:

```python
df.write.parquet("output/")
```

This may create too many output files.

You can control the number of output partitions:

```python
df.coalesce(20).write.parquet(
    "output/"
)
```

If you need redistribution based on a key:

```python
df.repartition(
    "customer_id"
).write.parquet(
    "output/"
)
```

---

# 53. What is `explain()` in PySpark?

Use:

```python
df.explain()
```

or:

```python
df.explain(True)
```

It shows Spark's execution plan.

This is extremely useful for debugging performance.

Look for things such as:

```text
Exchange
BroadcastHashJoin
SortMergeJoin
Scan
Filter
```

`Exchange` is often an indication of a shuffle.

---

# 54. What is a good PySpark performance optimization checklist?

When a Spark job is slow, investigate:

### 1. Data volume

Are we processing more data than necessary?

### 2. Filtering

Filter early where possible.

```python
df.filter(...)
```

### 3. Column pruning

Select only required columns.

```python
df.select(...)
```

### 4. File format

Prefer Parquet/ORC over CSV for analytical processing.

### 5. Partitioning

Check whether data is appropriately partitioned.

### 6. Shuffle

Look for unnecessary:

```text
groupBy
distinct
orderBy
joins
```

### 7. Broadcast

Broadcast small lookup tables.

### 8. Data skew

Check whether one partition is much larger than others.

### 9. Small files

Avoid generating huge numbers of tiny files.

### 10. Cache

Cache only when reused.

### 11. UDFs

Avoid unnecessary Python UDFs.

### 12. Explain plan

Use:

```python
df.explain(True)
```

---

# 55. Why is `collect()` dangerous?

```python
df.collect()
```

brings the entire DataFrame to the **driver**.

If the DataFrame is very large, this can cause:

```text
Driver OutOfMemoryError
```

Avoid:

```python
df.collect()
```

for large datasets.

Use:

```python
df.show(10)
```

or:

```python
df.take(10)
```

when only a small sample is required.

---

# 56. What are UDFs and why can they be slow?

A UDF is a User Defined Function.

Example:

```python
from pyspark.sql.functions import udf

def upper_name(name):
    return name.upper()

upper_udf = udf(upper_name)

df.withColumn(
    "new_name",
    upper_udf("name")
)
```

Python UDFs can be slower because data may need to move between the JVM and Python process.

### Prefer built-in Spark functions

Instead of:

```python
udf(...)
```

prefer:

```python
from pyspark.sql.functions import upper

df.withColumn(
    "new_name",
    upper("name")
)
```

Built-in functions are generally better optimized by Spark.

---

# 57. How do you explode an array column?

Suppose:

```text
id | skills
1  | [Python, SQL, Spark]
```

Use:

```python
from pyspark.sql.functions import explode

result = df.withColumn(
    "skill",
    explode("skills")
)
```

Result:

```text
id | skill
1  | Python
1  | SQL
1  | Spark
```

---

# 58. How do you process nested JSON?

Suppose JSON looks like:

```json
{
    "id": 1,
    "name": "Arun",
    "address": {
        "city": "Kolkata",
        "country": "India"
    }
}
```

Read it:

```python
df = spark.read.json(
    "employees.json"
)
```

Access nested columns:

```python
df.select(
    "id",
    "name",
    "address.city",
    "address.country"
).show()
```

For arrays, use functions such as:

```python
explode()
```

---

# 59. How would you implement incremental processing?

Suppose the source contains:

```text
id
name
updated_at
```

Instead of processing the entire dataset every time, process only records after the last successful timestamp.

Conceptually:

```python
new_data = df.filter(
    col("updated_at") > last_processed_timestamp
)
```

In AWS Glue, this can be combined with mechanisms such as:

- Job bookmarks
- S3 partitioning
- Timestamp-based filtering
- CDC data

### Interview answer

> Incremental processing means processing only newly arrived or changed records instead of reprocessing the entire dataset.

---

# 60. What is SCD Type 1?

SCD means **Slowly Changing Dimension**.

### SCD Type 1

Type 1 does **not maintain historical values**.

Suppose:

```text
Customer
Name: Arun
City: Kolkata
```

Later:

```text
City: Bangalore
```

Type 1 simply updates:

```text
Name: Arun
City: Bangalore
```

The old value is lost.

### Typical implementation

CDC data may contain operations:

```text
i → insert
u → update
d → delete
```

The target table is updated to represent the latest state.

---

# 61. Find employees whose salary is greater than their department's average salary.

This is a very common interview problem.

First calculate department average:

```python
from pyspark.sql.functions import avg

window = Window.partitionBy(
    "department"
)

result = df.withColumn(
    "dept_avg_salary",
    avg("salary").over(window)
).filter(
    col("salary") > col("dept_avg_salary")
)
```

---

# 62. Find customers who have never placed an order.

Use a left anti join.

```python
result = customers.join(
    orders,
    customers.customer_id == orders.customer_id,
    "left_anti"
)
```

This is usually cleaner than:

```python
left join + where order_id is null
```

---

# 63. Find the latest transaction for each customer.

```python
window = (
    Window
    .partitionBy("customer_id")
    .orderBy(col("transaction_date").desc())
)

result = transactions.withColumn(
    "rn",
    row_number().over(window)
).filter(
    col("rn") == 1
).drop("rn")
```

### Interview pattern to remember

```text
partitionBy(grouping key)
        +
orderBy(latest timestamp DESC)
        +
row_number()
        +
filter rn = 1
```

This pattern solves many real-world problems.

---

# 64. How would you optimize a slow Spark join?

First investigate the execution plan:

```python
df.explain(True)
```

Then check:

### 1. Is one DataFrame small?

Use broadcast:

```python
large.join(
    broadcast(small),
    "id"
)
```

### 2. Is there data skew?

Check key distribution.

### 3. Are unnecessary columns being joined?

Select only required columns:

```python
small = small.select(
    "id",
    "department"
)
```

### 4. Are unnecessary rows being joined?

Filter before joining:

```python
large = large.filter(...)
```

### 5. What join strategy is Spark using?

Look for:

```text
BroadcastHashJoin
SortMergeJoin
Exchange
```

### 6. Is partitioning appropriate?

Consider repartitioning when justified.

---

# 65. What is Adaptive Query Execution (AQE)?

AQE allows Spark to modify the execution plan during runtime based on actual statistics.

It can help with:

- Skewed joins
- Coalescing shuffle partitions
- Switching join strategies
- Optimizing partition sizes

Example:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

AQE is especially important in modern Spark versions.

---

# 66. What is `spark.sql.shuffle.partitions`?

This controls the default number of partitions used for many shuffle operations.

Example:

```python
spark.conf.set(
    "spark.sql.shuffle.partitions",
    200
)
```

If the value is too high:

```text
Too many small tasks
```

If too low:

```text
Large partitions
Long-running tasks
Possible memory pressure
```

The optimal value depends on:

- Data volume
- Cluster resources
- Workload
- Partition size

---

# 67. What is the difference between partitioning and repartitioning?

### Partitioning

Partitioning describes how data is physically divided into partitions.

Example:

```text
Partition 1
Partition 2
Partition 3
```

### Repartitioning

Repartitioning actively redistributes the data.

```python
df.repartition(
    10,
    "customer_id"
)
```

This generally causes a shuffle.

---

# 68. What is partition pruning?

Suppose data is stored as:

```text
year=2025/
year=2026/
```

Query:

```python
df.filter(
    col("year") == 2026
)
```

Spark can avoid reading:

```text
year=2025
```

and read only:

```text
year=2026
```

This is partition pruning.

### Benefit

Less data scanned → better performance.

---

# 69. What is the difference between partitioning and bucketing?

### Partitioning

Creates directory-level separation.

Example:

```text
employees/
    department=IT/
    department=HR/
    department=Finance/
```

### Bucketing

Distributes data into a fixed number of buckets based on a hash of a column.

Useful for certain join and aggregation workloads.

---

# 70. What is schema evolution?

Schema evolution means the schema changes over time.

For example:

Initial:

```text
id
name
salary
```

Later:

```text
id
name
salary
department
```

A robust pipeline needs to handle such changes.

Depending on the storage system and architecture, strategies include:

- Explicit schema management
- Schema merging
- Controlled migrations
- Data validation
- Schema registry/governance

---

# 71. What is schema drift?

Schema drift means the incoming data structure unexpectedly changes.

Examples:

```text
salary → string
```

or:

```text
employee_name
```

becomes:

```text
name
```

or:

```text
new column appears
```

### Difference

**Schema evolution** can be an expected/managed change.

**Schema drift** is often an unexpected change that can break pipelines.

---

# 72. How would you handle schema drift in a production pipeline?

A good approach is:

```text
Source
   ↓
Schema validation
   ↓
Detect changes
   ↓
Accept / reject / quarantine
   ↓
Transformation
   ↓
Target
```

You can:

- Maintain an expected schema
- Validate incoming files
- Detect added/removed columns
- Validate data types
- Send bad data to a quarantine location
- Alert the team
- Version schemas

---

# 73. How would you design a PySpark ETL pipeline?

A typical Data Engineering pipeline could look like:

```text
Source Systems
      ↓
Landing
      ↓
Raw
      ↓
Standardized
      ↓
Curated
      ↓
Analytics / BI
```

For an AWS-based architecture:

```text
Source
  ↓
S3 Landing
  ↓
AWS Glue
  ↓
S3 Raw
  ↓
Glue / Spark transformations
  ↓
S3 Standardized
  ↓
Curated
  ↓
Athena / Redshift / BI
```

Important production considerations:

- Incremental processing
- Schema validation
- Partitioning
- Idempotency
- Error handling
- Monitoring
- Data quality
- Logging
- Retry mechanisms
- Cost optimization

---

# 74. What is idempotency in Data Engineering?

An operation is idempotent if running it multiple times produces the same final result as running it once.

Example:

```text
Process file A
Process file A again
```

The target should not contain duplicate records.

Techniques include:

- Job bookmarks
- Unique keys
- MERGE/upsert logic
- Processed-file tracking
- CDC operations
- Checkpoints

### Interview answer

> Idempotency ensures that retries or repeated executions do not create duplicate or inconsistent results.

---

# 75. How would you debug a failed PySpark job?

I would follow a systematic process:

### Step 1 — Check logs

Look for:

```text
Exception
OutOfMemory
Task failure
Fetch failure
Serialization error
```

### Step 2 — Identify the failed stage

Look at the Spark UI.

### Step 3 — Check data volume

Did the input suddenly increase?

### Step 4 — Check data skew

Is one task processing significantly more data?

### Step 5 — Check shuffle

Look for:

```text
Exchange
Shuffle Read
Shuffle Write
```

### Step 6 — Check memory

Look for executor/driver OOM.

### Step 7 — Check schema

Did the source schema change?

### Step 8 — Reproduce with a smaller dataset

This helps isolate the problem.

---

# 76. What are common causes of Spark OutOfMemory errors?

Possible causes include:

### Driver OOM

Often caused by:

```python
df.collect()
```

or:

```python
large_data.toPandas()
```

### Executor OOM

Possible causes:

- Very large partitions
- Data skew
- Large joins
- Excessive caching
- Broadcast table too large
- Large aggregations

### Solutions

- Reduce partition size
- Handle skew
- Avoid collect
- Avoid unnecessary caching
- Optimize joins
- Filter earlier
- Select fewer columns

---

# 77. What happens when you call `df.show()`?

`show()` is an **action**.

It causes Spark to execute the required transformations needed to produce the requested rows.

Example:

```python
df.filter(
    col("salary") > 50000
).select(
    "name",
    "salary"
).show()
```

Spark builds an execution plan and executes it.

---

# 78. What is the difference between DataFrame and RDD?

### RDD

Lower-level distributed data structure.

```python
rdd = spark.sparkContext.parallelize(
    [1, 2, 3]
)
```

### DataFrame

Structured distributed data with schema.

```python
df = spark.createDataFrame(
    [(1, "Arun")],
    ["id", "name"]
)
```

### DataFrame advantages

- Optimized execution
- Catalyst optimizer
- Easier APIs
- SQL support
- Better performance for structured workloads

For modern Data Engineering work, DataFrames are generally preferred.

---

# 79. What is Spark SQL?

Spark SQL allows you to query DataFrames using SQL.

Example:

```python
df.createOrReplaceTempView(
    "employees"
)
```

Then:

```python
result = spark.sql("""
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
""")
```

---

# 80. SQL vs DataFrame API — which one should you use?

Both ultimately use Spark's execution engine.

DataFrame API:

```python
df.groupBy(
    "department"
).agg(
    avg("salary")
)
```

SQL:

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

Choose based on:

- Team preference
- Complexity
- Maintainability
- Existing codebase
- Developer familiarity

---

# 81. How would you find the top 3 employees by salary overall?

```python
result = df.orderBy(
    col("salary").desc()
).limit(3)
```

If ties must be handled differently, use an appropriate ranking function.

---

# 82. How would you find the second-highest salary?

Using `dense_rank()`:

```python
window = Window.orderBy(
    col("salary").desc()
)

result = df.withColumn(
    "rank",
    dense_rank().over(window)
).filter(
    col("rank") == 2
)
```

---

# 83. How would you find duplicate customer IDs?

```python
df.groupBy(
    "customer_id"
).count().filter(
    col("count") > 1
).show()
```

---

# 84. How would you find the number of employees in each department?

```python
df.groupBy(
    "department"
).count().show()
```

---

# 85. How would you find the average salary per department?

```python
df.groupBy(
    "department"
).agg(
    avg("salary").alias("avg_salary")
).show()
```

---

# 86. How would you find departments with more than 10 employees?

```python
df.groupBy(
    "department"
).count().filter(
    col("count") > 10
).show()
```

A cleaner approach is:

```python
df.groupBy(
    "department"
).count().filter(
    col("count") > 10
)
```

---

# 87. How would you find employees earning more than 50,000 in the IT department?

```python
df.filter(
    (col("department") == "IT") &
    (col("salary") > 50000)
).select(
    "name",
    "salary",
    "department"
).show()
```

Remember:

```python
(condition1) & (condition2)
```

not:

```python
condition1 and condition2
```

---

# 88. What are the most important PySpark topics for interviews?

For a **3–5 year Data Engineer**, prioritize these topics:

## Tier 1 — Must Know

```text
SparkSession
DataFrame creation
CSV / JSON / Parquet
show()
printSchema()
select()
filter()
where()
withColumn()
withColumnRenamed()
drop()
distinct()
dropDuplicates()
groupBy()
agg()
NULL handling
JOINs
Window functions
row_number()
rank()
dense_rank()
lag()
lead()
```

---

# 89. Tier 2 — Very Important

```text
repartition()
coalesce()
partitioning
shuffle
data skew
broadcast joins
cache()
persist()
lazy evaluation
transformations
actions
narrow transformations
wide transformations
Spark architecture
jobs
stages
tasks
```

---

# 90. Tier 3 — Performance

```text
Catalyst Optimizer
AQE
predicate pushdown
column pruning
partition pruning
Parquet
small-file problem
spark.sql.shuffle.partitions
explain()
join strategies
UDF performance
driver vs executor memory
```

---

# 91. Tier 4 — Real-World Data Engineering

```text
Incremental processing
CDC
SCD Type 1
SCD Type 2
Idempotency
Schema evolution
Schema drift
Data quality
Data validation
Error handling
Retry mechanisms
Monitoring
AWS Glue
S3
Athena
Redshift
```

---

# 92. Most Important Coding Patterns to Memorize

## Pattern 1 — Filter

```python
df.filter(
    col("salary") > 50000
)
```

---

## Pattern 2 — Multiple conditions

```python
df.filter(
    (col("department") == "IT") &
    (col("salary") > 50000)
)
```

---

## Pattern 3 — Aggregation

```python
df.groupBy(
    "department"
).agg(
    avg("salary")
)
```

---

## Pattern 4 — Window ranking

```python
window = Window.partitionBy(
    "department"
).orderBy(
    col("salary").desc()
)

df.withColumn(
    "rank",
    dense_rank().over(window)
)
```

---

## Pattern 5 — Latest record

```python
window = Window.partitionBy(
    "customer_id"
).orderBy(
    col("updated_at").desc()
)

df.withColumn(
    "rn",
    row_number().over(window)
).filter(
    col("rn") == 1
)
```

---

## Pattern 6 — Previous record

```python
df.withColumn(
    "previous_value",
    lag("amount").over(window)
)
```

---

## Pattern 7 — Next record

```python
df.withColumn(
    "next_value",
    lead("amount").over(window)
)
```

---

## Pattern 8 — Running total

```python
window = (
    Window
    .partitionBy("customer_id")
    .orderBy("transaction_date")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

df.withColumn(
    "running_total",
    sum("amount").over(window)
)
```

---

## Pattern 9 — Records missing from another table

```python
df1.join(
    df2,
    "customer_id",
    "left_anti"
)
```

---

## Pattern 10 — Broadcast join

```python
df1.join(
    broadcast(df2),
    "customer_id"
)
```

---

# 93. PySpark Interview Cheat Sheet

| Topic | Remember |
|---|---|
| Read CSV | `spark.read.csv()` |
| Read Parquet | `spark.read.parquet()` |
| Inspect | `show()`, `printSchema()` |
| Select | `select()` |
| Filter | `filter()` / `where()` |
| New column | `withColumn()` |
| Rename | `withColumnRenamed()` |
| Remove column | `drop()` |
| Unique rows | `distinct()` |
| Remove duplicates | `dropDuplicates()` |
| Aggregate | `groupBy().agg()` |
| Join | `join()` |
| Missing matches | `left_anti` |
| Existing matches | `left_semi` |
| Ranking | `row_number`, `rank`, `dense_rank` |
| Previous row | `lag()` |
| Next row | `lead()` |
| Running total | `sum().over(window)` |
| Increase partitions | `repartition()` |
| Reduce partitions | `coalesce()` |
| Small table join | `broadcast()` |
| Reuse DataFrame | `cache()` / `persist()` |
| Execution | Lazy |
| Trigger execution | Action |
| Data movement | Shuffle |
| Uneven data | Data skew |
| Query optimization | Catalyst |
| Runtime optimization | AQE |
| File format | Parquet |
| Read fewer columns | Column pruning |
| Read fewer partitions | Partition pruning |
| Inspect plan | `explain()` |

---

# 94. Recommended Interview Practice Order

For your current preparation, I would practice PySpark in this order:

### Stage 1 — Basics

```text
1. SparkSession
2. DataFrame creation
3. CSV loading
4. JSON loading
5. Parquet loading
6. show()
7. printSchema()
8. select()
9. filter()
10. where()
11. withColumn()
12. withColumnRenamed()
13. drop()
```

### Stage 2 — SQL-like operations

```text
14. groupBy()
15. agg()
16. count()
17. sum()
18. avg()
19. min()
20. max()
21. NULL handling
22. distinct()
23. dropDuplicates()
```

### Stage 3 — Joins

```text
24. Inner join
25. Left join
26. Right join
27. Full join
28. Left semi
29. Left anti
30. Multiple-column joins
31. Broadcast joins
```

### Stage 4 — Windows

```text
32. Window()
33. partitionBy()
34. orderBy()
35. row_number()
36. rank()
37. dense_rank()
38. lag()
39. lead()
40. Running totals
41. Top N per group
42. Latest record per key
43. Deduplication
```

### Stage 5 — Spark Internals

```text
44. Lazy evaluation
45. Transformations
46. Actions
47. Narrow transformations
48. Wide transformations
49. Shuffle
50. Jobs
51. Stages
52. Tasks
53. Executors
54. Driver
55. Cluster manager
```

### Stage 6 — Performance

```text
56. repartition()
57. coalesce()
58. Data skew
59. Broadcast
60. Cache
61. Persist
62. Catalyst
63. AQE
64. Predicate pushdown
65. Column pruning
66. Partition pruning
67. Small files
68. explain()
```

### Stage 7 — Real-world Data Engineering

```text
69. Incremental processing
70. CDC
71. SCD Type 1
72. SCD Type 2
73. Idempotency
74. Schema drift
75. Schema evolution
76. Data quality
77. Error handling
78. Monitoring
79. Retry mechanisms
80. Production pipeline design
```

---

# Final Interview Priority

If you have limited time before the interview, focus heavily on these:

## 🔥 Top 15

1. **DataFrame operations**
2. **Filtering**
3. **withColumn**
4. **Aggregations**
5. **Joins**
6. **Left semi / left anti joins**
7. **Window functions**
8. **row_number vs rank vs dense_rank**
9. **Latest record per ID**
10. **Deduplication**
11. **repartition vs coalesce**
12. **Data skew**
13. **Broadcast joins**
14. **Lazy evaluation + shuffle**
15. **Spark architecture**

Then move to:

```text
Catalyst
AQE
Predicate Pushdown
Partition Pruning
Column Pruning
Parquet
Small Files
Incremental Processing
CDC
SCD
Idempotency
```

These topics together cover a large portion of the **practical PySpark questions expected from a mid-level Data Engineer**.

---

# One-Line Interview Definitions

### SparkSession
> Entry point for working with Spark DataFrames and Spark SQL.

### DataFrame
> Distributed collection of structured data organized into named columns.

### Transformation
> Lazy operation that creates a new DataFrame.

### Action
> Operation that triggers Spark execution.

### Lazy Evaluation
> Spark delays execution until an action is called.

### Shuffle
> Redistribution of data across partitions.

### Narrow Transformation
> Transformation where data does not need to move across partitions.

### Wide Transformation
> Transformation that generally requires data movement across partitions.

### Partition
> A logical chunk of distributed data processed by a Spark task.

### Repartition
> Redistributes data across partitions and generally causes a shuffle.

### Coalesce
> Reduces the number of partitions with less data movement than repartition.

### Data Skew
> Uneven distribution of data across partitions.

### Broadcast Join
> Sends a small dataset to executors to avoid shuffling the large dataset.

### Catalyst
> Spark SQL's query optimization framework.

### AQE
> Runtime optimization of Spark execution plans using actual execution statistics.

### Parquet
> Columnar storage format optimized for analytical workloads.

### Predicate Pushdown
> Pushing filters closer to the data source to reduce data scanned.

### Column Pruning
> Reading only the columns required by the query.

### Partition Pruning
> Reading only relevant physical partitions.

### Idempotency
> Repeated execution produces the same final result without unwanted duplicates.

### SCD Type 1
> Updates dimension data without retaining historical values.

---

# Final Note

For a **3–5 year Data Engineering interview**, don't focus only on memorizing syntax.

For every PySpark concept, be able to explain:

```text
1. What it is
2. Why we use it
3. How it works
4. When it becomes expensive
5. How to optimize it
6. A real-world example
```

For example, don't just know:

```python
df.repartition(10)
```

Be able to explain:

> `repartition()` redistributes data across partitions and causes a shuffle. I would use it when I need better distribution or a specific partition count, but I would avoid unnecessary repartitioning because shuffle is expensive.

That distinction is what moves an answer from **basic PySpark knowledge** toward a **mid-level Data Engineer interview answer**.