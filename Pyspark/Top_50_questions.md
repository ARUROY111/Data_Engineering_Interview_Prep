# Top 50 PySpark Coding Questions (with Solutions)

Frequently asked in Data Engineering interviews. Grouped by topic. Each solution is runnable with the shared setup below.

## Common Setup

```python
from pyspark.sql import SparkSession, Window
from pyspark.sql import functions as F
from pyspark.sql.types import *

spark = SparkSession.builder.appName("interview").getOrCreate()

emp = spark.createDataFrame([
    (1, "Amit",  "IT",  90000, "2020-01-15", None),
    (2, "Neha",  "IT",  75000, "2019-03-10", 1),
    (3, "Raj",   "HR",  60000, "2021-07-01", 1),
    (4, "Sara",  "HR",  60000, "2018-11-20", 3),
    (5, "John",  "FIN", 85000, "2022-02-05", 1),
    (6, "Priya", "FIN", 70000, "2017-09-09", 5),
], ["emp_id", "name", "dept", "salary", "join_date", "manager_id"])

orders = spark.createDataFrame([
    (101, 1, "2024-01-05", 250.0),
    (102, 2, "2024-01-06", 100.0),
    (103, 1, "2024-02-10", 400.0),
    (104, 4, "2024-02-11", 150.0),
], ["order_id", "cust_id", "order_date", "amount"])
```

## Table of Contents

1. [DataFrame Basics (Q1-Q10)](#a-dataframe-basics)
2. [Aggregations (Q11-Q18)](#b-aggregations--grouping)
3. [Joins (Q19-Q25)](#c-joins)
4. [Window Functions (Q26-Q32)](#d-window-functions)
5. [Complex Types and Reshaping (Q33-Q38)](#e-complex-types--reshaping)
6. [Strings, Dates, Words (Q39-Q42)](#f-strings-dates--text)
7. [Performance, IO and Pipelines (Q43-Q50)](#g-performance-io--pipeline-patterns)

---

## A. DataFrame Basics

### Q1. Create a DataFrame with an explicit schema

```python
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
])
df = spark.createDataFrame([(1, "A"), (2, "B")], schema)
df.printSchema()
```

### Q2. Read CSV / JSON / Parquet with an explicit schema

```python
df_csv = (spark.read.option("header", True).schema(schema)
          .option("mode", "DROPMALFORMED").csv("/path/file.csv"))
df_json = spark.read.schema(schema).json("/path/file.json")
df_pq = spark.read.parquet("/path/data/")
```
Passing a schema avoids the extra pass `inferSchema` needs and gives stable types.

### Q3. Select, filter, add, rename and drop columns

```python
res = (emp.select("emp_id", "name", "salary")
          .filter((F.col("salary") > 65000) & (F.col("name") != "Raj"))
          .withColumn("bonus", F.col("salary") * 0.1)
          .withColumnRenamed("name", "emp_name")
          .drop("bonus"))
```

### Q4. Handle null values

```python
emp.fillna({"manager_id": 0})                       # constant fill
emp.dropna(subset=["manager_id"])                   # drop rows
emp.withColumn("mgr", F.coalesce("manager_id", F.lit(-1)))
emp.filter(F.col("manager_id").isNull())            # find nulls
```

### Q5. Remove duplicate rows

```python
emp.distinct()
emp.dropDuplicates(["dept", "salary"])   # on subset (keeps an arbitrary row)
```
To keep a specific row (e.g. latest), use `row_number()` (see Q31).

### Q6. Find duplicate records

```python
dups = (emp.groupBy("dept", "salary").count().filter("count > 1"))
# Return full duplicate rows:
w = Window.partitionBy("dept", "salary")
emp.withColumn("cnt", F.count("*").over(w)).filter("cnt > 1")
```

### Q7. Count nulls in every column

```python
emp.select([F.sum(F.col(c).isNull().cast("int")).alias(c) for c in emp.columns]).show()
```

### Q8. Cast types and parse dates

```python
df = (emp.withColumn("salary", F.col("salary").cast("double"))
         .withColumn("join_date", F.to_date("join_date", "yyyy-MM-dd")))
```

### Q9. Split one column into multiple columns

```python
df = spark.createDataFrame([("John Smith",), ("Jane Doe",)], ["full_name"])
parts = F.split("full_name", " ")
df.withColumn("first", parts[0]).withColumn("last", parts[1]).show()
```

### Q10. Concatenate columns

```python
emp.withColumn("label", F.concat_ws(" - ", "name", "dept"))
# concat() returns NULL if any input is NULL; concat_ws skips NULLs
```

---

## B. Aggregations & Grouping

### Q11. Multiple aggregations per group

```python
emp.groupBy("dept").agg(
    F.count("*").alias("cnt"),
    F.avg("salary").alias("avg_sal"),
    F.max("salary").alias("max_sal"),
    F.countDistinct("salary").alias("distinct_salaries"),
)
```

### Q12. Top-N salaries per department

```python
w = Window.partitionBy("dept").orderBy(F.desc("salary"))
emp.withColumn("rn", F.row_number().over(w)).filter("rn <= 2").drop("rn")
```

### Q13. Second highest salary (overall)

```python
w = Window.orderBy(F.desc("salary"))
emp.withColumn("r", F.dense_rank().over(w)).filter("r = 2").select("salary").distinct()
```

### Q14. Nth highest salary per department

```python
n = 2
w = Window.partitionBy("dept").orderBy(F.desc("salary"))
emp.withColumn("r", F.dense_rank().over(w)).filter(F.col("r") == n)
```

### Q15. Employees earning more than their department average

```python
w = Window.partitionBy("dept")
(emp.withColumn("dept_avg", F.avg("salary").over(w))
    .filter("salary > dept_avg"))
```

### Q16. Running (cumulative) total

```python
w = (Window.partitionBy("cust_id").orderBy("order_date")
     .rowsBetween(Window.unboundedPreceding, Window.currentRow))
orders.withColumn("running_total", F.sum("amount").over(w))
```

### Q17. 3-row moving average

```python
w = Window.partitionBy("cust_id").orderBy("order_date").rowsBetween(-2, 0)
orders.withColumn("mov_avg", F.avg("amount").over(w))
```

### Q18. Percentage contribution to the group total

```python
w = Window.partitionBy("dept")
emp.withColumn("pct", F.round(F.col("salary") / F.sum("salary").over(w) * 100, 2))
```

---

## C. Joins

### Q19. All join types

```python
emp.join(orders, emp.emp_id == orders.cust_id, "inner")
emp.join(orders, emp.emp_id == orders.cust_id, "left")        # also right, full/outer
emp.join(orders, emp.emp_id == orders.cust_id, "left_semi")   # rows in emp with a match, emp columns only
emp.join(orders, emp.emp_id == orders.cust_id, "left_anti")   # rows in emp with NO match
```

### Q20. Customers who never placed an order (anti join)

```python
emp.join(orders, emp.emp_id == orders.cust_id, "left_anti").select("emp_id", "name")
```

### Q21. Employee and manager name (self join)

```python
e, m = emp.alias("e"), emp.alias("m")
(e.join(m, F.col("e.manager_id") == F.col("m.emp_id"), "left")
  .select(F.col("e.name").alias("employee"), F.col("m.name").alias("manager")))
```

### Q22. Broadcast join for a small dimension table

```python
from pyspark.sql.functions import broadcast
big.join(broadcast(small), "key")
# Threshold: spark.sql.autoBroadcastJoinThreshold (default 10 MB)
```

### Q23. Fix data skew with salting

```python
SALT = 8
big_s = big.withColumn("salt", (F.rand() * SALT).cast("int"))
small_s = small.withColumn("salt", F.explode(F.array([F.lit(i) for i in range(SALT)])))
big_s.join(small_s, ["key", "salt"]).drop("salt")
```
On Spark 3+, also enable AQE: `spark.sql.adaptive.enabled=true` and `spark.sql.adaptive.skewJoin.enabled=true`.

### Q24. Avoid duplicate column names after a join

```python
emp.join(orders, emp.emp_id == orders.cust_id).drop(orders.cust_id)
# Or join on the column name when both sides share it:
a.join(b, "id")     # keeps a single 'id'
```

### Q25. Generate a date spine and fill missing dates

```python
dates = (spark.sql("""
    SELECT explode(sequence(to_date('2024-01-01'), to_date('2024-01-10'), interval 1 day)) AS d
"""))
daily = orders.groupBy("order_date").agg(F.sum("amount").alias("amt"))
(dates.join(daily, dates.d == F.to_date(daily.order_date), "left")
      .fillna({"amt": 0}))
```

---

## D. Window Functions

### Q26. Month-over-month change with lag

```python
m = (orders.withColumn("month", F.date_format("order_date", "yyyy-MM"))
           .groupBy("month").agg(F.sum("amount").alias("amt")))
w = Window.orderBy("month")
(m.withColumn("prev", F.lag("amt").over(w))
  .withColumn("mom_pct", F.round((F.col("amt") - F.col("prev")) / F.col("prev") * 100, 2)))
```

### Q27. rank vs dense_rank vs row_number

```python
w = Window.partitionBy("dept").orderBy(F.desc("salary"))
emp.select("dept", "name", "salary",
    F.row_number().over(w).alias("row_number"),  # 1,2,3 unique
    F.rank().over(w).alias("rank"),              # 1,1,3 gaps after ties
    F.dense_rank().over(w).alias("dense_rank"))  # 1,1,2 no gaps
```

### Q28. First and last value per group

```python
w = (Window.partitionBy("cust_id").orderBy("order_date")
     .rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing))
orders.select("*",
    F.first("amount").over(w).alias("first_amt"),
    F.last("amount").over(w).alias("last_amt"))
```
The explicit frame matters: the default frame ends at the current row, so `last` would just return the current row.

### Q29. Longest consecutive-day login streak (gaps and islands)

```python
logins = spark.createDataFrame(
    [(1, "2024-01-01"), (1, "2024-01-02"), (1, "2024-01-04"), (1, "2024-01-05"), (1, "2024-01-06")],
    ["user", "d"]).withColumn("d", F.to_date("d"))

w = Window.partitionBy("user").orderBy("d")
streaks = (logins.withColumn("rn", F.row_number().over(w))
    .withColumn("grp", F.date_sub("d", F.col("rn")))      # constant within a streak
    .groupBy("user", "grp")
    .agg(F.count("*").alias("streak_len"), F.min("d").alias("start"), F.max("d").alias("end")))
streaks.groupBy("user").agg(F.max("streak_len").alias("longest"))
```

### Q30. Sessionization (new session after 30 min inactivity)

```python
ev = spark.createDataFrame(
    [("u1", "2024-01-01 10:00:00"), ("u1", "2024-01-01 10:10:00"), ("u1", "2024-01-01 11:00:00")],
    ["user", "ts"]).withColumn("ts", F.to_timestamp("ts"))

w = Window.partitionBy("user").orderBy("ts")
(ev.withColumn("gap_min", (F.col("ts").cast("long") - F.lag("ts").over(w).cast("long")) / 60)
   .withColumn("new_sess", F.when(F.col("gap_min").isNull() | (F.col("gap_min") > 30), 1).otherwise(0))
   .withColumn("session_id", F.sum("new_sess").over(w)))
```

### Q31. Keep only the latest record per key (dedupe)

```python
rec = spark.createDataFrame([(1, "a", "2024-01-01"), (1, "b", "2024-02-01"), (2, "c", "2024-01-15")],
                            ["id", "val", "updated_at"])
w = Window.partitionBy("id").orderBy(F.desc("updated_at"))
rec.withColumn("rn", F.row_number().over(w)).filter("rn = 1").drop("rn")
```

### Q32. Quartiles with ntile and percentiles

```python
emp.withColumn("quartile", F.ntile(4).over(Window.orderBy("salary")))
emp.agg(F.percentile_approx("salary", [0.25, 0.5, 0.75]).alias("pcts"))
emp.agg(F.expr("percentile(salary, 0.5)").alias("exact_median"))
```

---

## E. Complex Types & Reshaping

### Q33. Explode an array column

```python
df = spark.createDataFrame([(1, ["a", "b"]), (2, ["c"]), (3, None)], ["id", "tags"])
df.select("id", F.explode("tags").alias("tag"))             # drops id=3
df.select("id", F.explode_outer("tags").alias("tag"))       # keeps id=3 with null
df.select("id", F.posexplode("tags").alias("pos", "tag"))   # with index
```

### Q34. Flatten a nested struct

```python
df = spark.createDataFrame([(1, ("Pune", "MH"))],
    "id int, addr struct<city:string,state:string>")
df.select("id", "addr.*")

# Generic flatten of all struct columns
def flatten(df):
    cols = []
    for f in df.schema.fields:
        if isinstance(f.dataType, StructType):
            cols += [F.col(f"{f.name}.{c.name}").alias(f"{f.name}_{c.name}") for c in f.dataType.fields]
        else:
            cols.append(F.col(f.name))
    return df.select(cols)
```

### Q35. Parse a JSON string column

```python
raw = spark.createDataFrame([('{"id":1,"city":"Pune"}',)], ["js"])
sch = StructType([StructField("id", IntegerType()), StructField("city", StringType())])
raw.withColumn("p", F.from_json("js", sch)).select("p.*")
# Quick path: F.get_json_object("js", "$.city")
```

### Q36. Pivot rows into columns

```python
sales = spark.createDataFrame(
    [("A", "Q1", 100), ("A", "Q2", 150), ("B", "Q1", 80)], ["prod", "qtr", "amt"])
sales.groupBy("prod").pivot("qtr", ["Q1", "Q2"]).sum("amt")
# Passing the value list avoids an extra job to discover distinct values
```

### Q37. Unpivot columns into rows

```python
wide = spark.createDataFrame([("A", 100, 150)], ["prod", "Q1", "Q2"])
wide.select("prod", F.expr("stack(2, 'Q1', Q1, 'Q2', Q2) as (qtr, amt)"))
# Spark 3.4+: wide.unpivot("prod", ["Q1", "Q2"], "qtr", "amt")
```

### Q38. Aggregate rows into a list or set

```python
emp.groupBy("dept").agg(
    F.collect_list("name").alias("names"),     # keeps duplicates
    F.collect_set("salary").alias("salaries")) # unique values
```

---

## F. Strings, Dates & Text

### Q39. Word count

```python
lines = spark.createDataFrame([("spark is fast",), ("spark is fun",)], ["line"])
(lines.select(F.explode(F.split(F.lower("line"), r"\s+")).alias("word"))
      .groupBy("word").count().orderBy(F.desc("count")))
```

### Q40. Regex extract and replace

```python
df = spark.createDataFrame([("order-1234-IN",)], ["s"])
df.select(F.regexp_extract("s", r"order-(\d+)-", 1).alias("num"),
          F.regexp_replace("s", r"\d+", "#").alias("masked"))
```

### Q41. Date arithmetic

```python
d = emp.withColumn("jd", F.to_date("join_date"))
d.select("name",
    F.datediff(F.current_date(), "jd").alias("days_since_join"),
    F.months_between(F.current_date(), "jd").alias("months"),
    F.add_months("jd", 6).alias("plus_6m"),
    F.date_trunc("month", "jd").alias("month_start"),
    F.year("jd").alias("yr"), F.dayofweek("jd").alias("dow"))
```

### Q42. Employees with more than 3 years of tenure

```python
(emp.withColumn("tenure_yrs", F.months_between(F.current_date(), F.to_date("join_date")) / 12)
    .filter("tenure_yrs > 3"))
```

---

## G. Performance, IO & Pipeline Patterns

### Q43. Built-in functions vs UDF vs Pandas UDF

```python
# Prefer built-ins: they are optimized by Catalyst and avoid Python serialization
emp.withColumn("up", F.upper("name"))

# Plain Python UDF (slow: row-at-a-time, opaque to the optimizer)
@F.udf(StringType())
def tag(s): return f"emp_{s}"

# Pandas UDF (vectorized via Arrow, much faster)
import pandas as pd
from pyspark.sql.functions import pandas_udf
@pandas_udf("string")
def tag_v(s: pd.Series) -> pd.Series:
    return "emp_" + s
emp.withColumn("t", tag_v("name"))
```

### Q44. repartition vs coalesce

```python
df.repartition(200)               # full shuffle, can increase or decrease, balanced sizes
df.repartition(50, "dept")        # hash partition by column, useful before joins/writes
df.coalesce(10)                   # no full shuffle, only decreases, may leave uneven partitions
df.rdd.getNumPartitions()
```
Rule of thumb: target roughly 128 MB per partition. Use `coalesce` to cut output files, `repartition` to fix skew or raise parallelism.

### Q45. cache vs persist

```python
from pyspark import StorageLevel
df.cache()                                   # MEMORY_AND_DISK for DataFrames
df.persist(StorageLevel.MEMORY_AND_DISK_SER)
df.count()                                   # an action materializes the cache
df.unpersist()
```
Cache only DataFrames that are reused across multiple actions.

### Q46. Write partitioned Parquet and choose a save mode

```python
(emp.write.mode("overwrite")        # append | overwrite | ignore | errorifexists
     .partitionBy("dept")
     .option("compression", "snappy")
     .parquet("/out/emp"))

# Overwrite only the partitions present in the data:
spark.conf.set("spark.sql.sources.partitionOverwriteMode", "dynamic")
```

### Q47. Incremental upsert (SCD Type 1) without a table format

```python
target = spark.read.parquet("/data/target")
delta  = spark.read.parquet("/data/delta")

merged = (target.withColumn("src", F.lit(0))
          .unionByName(delta.withColumn("src", F.lit(1))))
w = Window.partitionBy("id").orderBy(F.desc("src"), F.desc("updated_at"))
result = merged.withColumn("rn", F.row_number().over(w)).filter("rn = 1").drop("rn", "src")
```
With Delta/Hudi/Iceberg use `MERGE INTO` instead:
```python
from delta.tables import DeltaTable
(DeltaTable.forPath(spark, "/data/target").alias("t")
   .merge(delta.alias("s"), "t.id = s.id")
   .whenMatchedUpdateAll().whenNotMatchedInsertAll().execute())
```

### Q48. SCD Type 2 (history tracking)

```python
# dim: id, name, eff_start, eff_end, is_current   |  src: id, name
cur = dim.filter("is_current = true")
chg = (src.alias("s").join(cur.alias("d"), "id")
          .filter(F.col("s.name") != F.col("d.name")).select("s.*"))

# 1) close old records
closed = (cur.join(chg.select("id"), "id", "left_semi")
             .withColumn("eff_end", F.current_date()).withColumn("is_current", F.lit(False)))
# 2) open new versions (changed and brand-new keys)
new_keys = src.join(cur, "id", "left_anti")
opened = (chg.unionByName(new_keys)
             .withColumn("eff_start", F.current_date())
             .withColumn("eff_end", F.lit(None).cast("date"))
             .withColumn("is_current", F.lit(True)))
# 3) untouched rows: old history + current rows that did not change
history   = dim.filter("is_current = false")
unchanged = cur.join(chg.select("id"), "id", "left_anti")

final = history.unionByName(unchanged).unionByName(closed).unionByName(opened)
```

### Q49. Compare two DataFrames (find differences)

```python
a.exceptAll(b)        # rows in a not in b (keeps duplicates)
b.exceptAll(a)
a.subtract(b)         # distinct version

# Column-level diff on a key
(a.alias("a").join(b.alias("b"), "id", "full")
  .filter("a.val IS DISTINCT FROM b.val"))     # null-safe, use F.expr in older versions
```

### Q50. Basic data quality checks and reading the plan

```python
total = df.count()
checks = df.agg(
    F.count(F.when(F.col("id").isNull(), 1)).alias("null_ids"),
    (F.count("*") - F.countDistinct("id")).alias("dup_ids"),
    F.count(F.when(F.col("salary") < 0, 1)).alias("neg_salary"),
).first()
assert checks["null_ids"] == 0 and checks["dup_ids"] == 0, "DQ failed"

# Inspect optimizer decisions (join strategy, pushdown, partition pruning)
df.explain("formatted")
```

---

## Quick Revision Cheatsheet

| Topic | Remember |
|---|---|
| Transformations vs actions | Lazy: `select/filter/join/groupBy`. Actions: `count/show/collect/write` trigger execution |
| Narrow vs wide | Narrow: no shuffle (`filter`, `map`). Wide: shuffle (`groupBy`, `join`, `distinct`) |
| Shuffle partitions | `spark.sql.shuffle.partitions` (default 200), tune or use AQE |
| Joins | Broadcast small side, salt skewed keys, filter early, select needed columns only |
| Window default frame | With `orderBy`: unbounded preceding to current row (RANGE), set `rowsBetween` explicitly for `last` |
| Avoid | `collect()` on big data, Python UDFs when built-ins exist, too many small files |
| File format | Parquet/ORC for analytics, Delta/Hudi/Iceberg for upserts and ACID |

---

*Tip: for each question, practice explaining the time/shuffle cost and one alternative approach. Interviewers often ask "how would you optimize this?" as a follow-up.*
