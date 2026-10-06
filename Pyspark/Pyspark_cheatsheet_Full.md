# PySpark Cheat Sheet (Data Engineering Edition)

> Sections **1-11** follow the original Draphony cheat sheet structure, each expanded with the missing pieces.
> Sections **12-22** are new and cover what real data engineering pipelines need. Section **23** is the original "Ending Spark".
> Section **24** has real-world practice scenarios with sample data and collapsible solutions.
> Targets Spark 3.x. Items needing a newer version are noted inline.

**Contents**

| # | Section | # | Section |
|---|---|---|---|
| 1 | Basic Operations | 13 | Null Handling & Data Quality |
| 2 | Selecting and Filtering Data | 14 | Spark SQL |
| 3 | Joining DataFrames | 15 | UDFs |
| 4 | Aggregations | 16 | Partitioning & Performance Tuning |
| 5 | Window Functions | 17 | Table Formats & Upserts (Delta / Hudi / Iceberg) |
| 6 | Conditional Statements | 18 | CDC & SCD Patterns |
| 7 | String Functions | 19 | Structured Streaming |
| 8 | Number Functions | 20 | JDBC, Redshift & S3 |
| 9 | Date & Time Functions | 21 | AWS Glue Specifics |
| 10 | Column Operations | 22 | Testing & Production Patterns |
| 11 | Write Data | 23 | Ending Spark |
| 12 | Schemas & Complex Types | 24 | Real-World Practice Scenarios |

---

## 1. Basic Operations

```python
# Create a SparkSession
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('your_app_name').getOrCreate()

# Common builder options
spark = (SparkSession.builder
         .appName('your_app_name')
         .master('local[*]')                                   # local testing only
         .config('spark.sql.shuffle.partitions', '200')
         .config('spark.sql.adaptive.enabled', 'true')
         .enableHiveSupport()                                  # for metastore / Glue Catalog tables
         .getOrCreate())

# Load Data
df = spark.read.csv('/path/to/input.csv', header=True)         # CSV
df = spark.read.json('/path/to/input.json')                    # JSON
df = spark.read.parquet('/path/to/input.parquet')              # Parquet
df = spark.read.orc('/path/to/input.orc')                      # ORC
df = spark.read.table('db.table_name')                         # Catalog table
df = spark.table('db.table_name')                              # Same as above

# Read with options
df = (spark.read
      .option('header', True)
      .option('inferSchema', True)          # extra pass over data; avoid in production
      .option('delimiter', '|')
      .option('multiLine', True)            # multi-line JSON / CSV records
      .option('mode', 'PERMISSIVE')         # PERMISSIVE | DROPMALFORMED | FAILFAST
      .option('recursiveFileLookup', True)
      .option('pathGlobFilter', '*.csv')
      .csv('/path/to/dir/'))

# Read multiple paths / wildcards
df = spark.read.parquet('/data/year=2026/month=*/')
df = spark.read.parquet('/path/a', '/path/b')

# Show data
df.show(5)                    # Show the first 5 rows
df.show(5, truncate=False)    # Don't truncate wide columns
df.show(5, vertical=True)     # One column per line
df.collect()                  # Collect all rows (use with caution)
df.take(5)                    # First 5 rows as a list of Row
df.head()                     # First Row
df.toPandas()                 # To pandas (small data only, pulls everything to driver)

# Inspect
df.printSchema()
df.columns
df.dtypes
df.schema
df.count()
df.describe().show()          # count, mean, stddev, min, max
df.summary().show()           # adds percentiles
df.explain()                  # Physical plan
df.explain(True)              # Parsed, analyzed, optimized, physical plans
df.rdd.getNumPartitions()
```

---

## 2. Selecting and Filtering Data

```python
from pyspark.sql import functions as F

# Select columns
df = df.select('column1', 'column2')
df = df.select(F.col('column1'), F.col('column2').alias('c2'))
df = df.select('*', F.lit(1).alias('flag'))
df = df.selectExpr('column1', 'column2 * 2 AS double_col', 'upper(name) AS name_uc')

# Filter rows
df = df.filter(df.column > 10)
df = df.filter(df.column.isNull())
df = df.filter(df.column.isNotNull())
df = df.where(F.col('column') > 10)                            # where == filter
df = df.filter("column > 10 AND status = 'A'")                 # SQL string

# Multiple conditions (use parentheses, & | ~)
df = df.filter((F.col('a') > 10) & (F.col('b') == 'x'))
df = df.filter((F.col('a') > 10) | (F.col('b') == 'x'))
df = df.filter(~F.col('a').isin(1, 2, 3))

# Predicates
df = df.filter(F.col('status').isin('A', 'B'))
df = df.filter(F.col('amount').between(10, 100))
df = df.filter(F.col('name').like('Ab%'))
df = df.filter(F.col('name').rlike('^[A-Z]{3}\\d+$'))
df = df.filter(F.col('name').contains('abc'))
df = df.filter(F.col('name').startswith('abc'))
df = df.filter(F.col('name').endswith('xyz'))

# Drop duplicates
df = df.dropDuplicates()
df = df.dropDuplicates(['id', 'event_date'])                   # by subset
df = df.distinct()

# Drop a column
df = df.drop('column_name')
df = df.drop('col1', 'col2')

# Sort by a column
df = df.orderBy(df.column.asc())
df = df.orderBy(df.column.desc())
df = df.orderBy(F.col('a').asc(), F.col('b').desc())
df = df.orderBy(F.col('a').desc_nulls_last())
df = df.sort('a')                                              # sort == orderBy
df = df.sortWithinPartitions('a')                              # cheaper, no global sort

# Limit / sample
df = df.limit(100)
df = df.sample(fraction=0.1, seed=42)
df = df.sample(withReplacement=False, fraction=0.1, seed=42)
```

---

## 3. Joining DataFrames

```python
# Inner join
df = df1.join(df2, df1.id == df2.id, 'inner')

# Left join
df = df1.join(df2, df1.id == df2.id, 'left')

# Full outer join
df = df1.join(df2, df1.id == df2.id, 'outer')

# Right join
df = df1.join(df2, df1.id == df2.id, 'right')

# Left semi join (rows in df1 that HAVE a match in df2; only df1 columns)
df = df1.join(df2, 'id', 'left_semi')

# Left anti join (rows in df1 with NO match in df2; great for "new records only")
df = df1.join(df2, 'id', 'left_anti')

# Cross join (cartesian product, use deliberately)
df = df1.crossJoin(df2)

# Join on same-named column (avoids duplicate column in output)
df = df1.join(df2, 'id', 'inner')
df = df1.join(df2, ['id', 'region'], 'inner')                  # multiple keys

# Multiple conditions
df = df1.join(df2, (df1.id == df2.id) & (df1.dt >= df2.start_dt), 'left')

# Null-safe equality (NULL == NULL treated as true)
df = df1.join(df2, df1.id.eqNullSafe(df2.id), 'inner')

# Alias to disambiguate columns
a, b = df1.alias('a'), df2.alias('b')
df = a.join(b, F.col('a.id') == F.col('b.id')).select('a.*', F.col('b.amount').alias('b_amount'))

# Broadcast join (small table sent to every executor, avoids shuffle)
from pyspark.sql.functions import broadcast
df = df1.join(broadcast(df2), 'id')
df = df1.join(df2.hint('broadcast'), 'id')                     # same, via hint
# Other hints: 'shuffle_hash', 'merge', 'shuffle_replicate_nl'

# Set operations
df = df1.union(df2)                                            # by position
df = df1.unionByName(df2)                                      # by column name
df = df1.unionByName(df2, allowMissingColumns=True)            # Spark 3.1+
df = df1.intersect(df2)
df = df1.subtract(df2)                                         # like EXCEPT DISTINCT
df = df1.exceptAll(df2)                                        # keeps duplicates
```

---

## 4. Aggregations

```python
from pyspark.sql import functions as F

# Aggregations
df = df.groupBy('column').agg(F.count('*').alias('count'))
df = df.groupBy('column').agg(F.sum('amount').alias('totalAmount'))

# Multiple aggregations in one pass
df = df.groupBy('column1', 'column2').agg(
    F.count('*').alias('cnt'),
    F.countDistinct('user_id').alias('uniq_users'),
    F.sum('amount').alias('total'),
    F.avg('amount').alias('avg_amt'),
    F.min('dt').alias('first_dt'),
    F.max('dt').alias('last_dt'),
    F.approx_count_distinct('user_id', 0.05).alias('approx_users'),   # cheaper on big data
    F.collect_list('item').alias('items'),                            # keeps duplicates
    F.collect_set('item').alias('uniq_items'),
    F.first('name', ignorenulls=True).alias('any_name'),
    F.stddev('amount').alias('sd'),
)

# Basic Aggregations (whole DataFrame)
df = df.agg(F.max('column').alias('max'))
df = df.agg(F.min('column').alias('min'))
df = df.agg({'amount': 'sum', 'id': 'count'})                  # dict form

# Conditional aggregation
df = df.groupBy('k').agg(F.sum(F.when(F.col('status') == 'A', 1).otherwise(0)).alias('a_cnt'))
df = df.groupBy('k').agg(F.count(F.when(F.col('status') == 'A', True)).alias('a_cnt'))

# HAVING equivalent
df = df.groupBy('k').agg(F.sum('amount').alias('t')).filter(F.col('t') > 1000)

# Pivot (pass the values list to skip an extra pass)
df = df.groupBy('year').pivot('quarter', ['Q1', 'Q2', 'Q3', 'Q4']).agg(F.sum('sales'))

# Rollup / Cube (subtotals / grand totals)
df = df.rollup('region', 'country').agg(F.sum('sales'))
df = df.cube('region', 'country').agg(F.sum('sales'))

# Count rows / distinct / nulls per column
df.select([F.count(F.when(F.col(c).isNull(), c)).alias(c) for c in df.columns]).show()
```

---

## 5. Window Functions

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F

# Define a window
windowSpec = Window.partitionBy('column').orderBy('column2')

# Row number within window
df = df.withColumn('row_number', F.row_number().over(windowSpec))

# Running total within window
df = df.withColumn('running_total', F.sum('amount').over(windowSpec))

# Ranking
df = df.withColumn('rank', F.rank().over(windowSpec))              # gaps after ties
df = df.withColumn('dense_rank', F.dense_rank().over(windowSpec))  # no gaps
df = df.withColumn('ntile', F.ntile(4).over(windowSpec))           # quartiles
df = df.withColumn('pct_rank', F.percent_rank().over(windowSpec))
df = df.withColumn('cume_dist', F.cume_dist().over(windowSpec))

# Previous / next row
df = df.withColumn('prev_amt', F.lag('amount', 1).over(windowSpec))
df = df.withColumn('next_amt', F.lead('amount', 1, 0).over(windowSpec))   # 0 = default

# First / last in window
w_full = (Window.partitionBy('k').orderBy('dt')
          .rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing))
df = df.withColumn('first_val', F.first('amount').over(w_full))
df = df.withColumn('last_val', F.last('amount').over(w_full))

# Frame specs
w_rows  = Window.partitionBy('k').orderBy('dt').rowsBetween(-6, 0)          # last 7 rows
w_range = Window.partitionBy('k').orderBy('ts').rangeBetween(-3600, 0)      # last 1h (numeric order col)
df = df.withColumn('moving_avg_7', F.avg('amount').over(w_rows))
df = df.withColumn('cum_sum', F.sum('amount').over(
    Window.partitionBy('k').orderBy('dt').rowsBetween(Window.unboundedPreceding, Window.currentRow)))

# Aggregate over partition without ordering (no collapse of rows)
df = df.withColumn('k_total', F.sum('amount').over(Window.partitionBy('k')))

# PATTERN: keep latest record per key (dedupe)
w = Window.partitionBy('id').orderBy(F.col('updated_at').desc())
df = (df.withColumn('rn', F.row_number().over(w))
        .filter('rn = 1')
        .drop('rn'))

# PATTERN: sessionization / gap detection
w = Window.partitionBy('user').orderBy('ts')
df = df.withColumn('gap_sec', F.col('ts').cast('long') - F.lag('ts').over(w).cast('long'))
df = df.withColumn('new_session', (F.col('gap_sec') > 1800).cast('int'))
df = df.withColumn('session_id', F.sum(F.coalesce('new_session', F.lit(0))).over(w))
```

---

## 6. Conditional Statements

```python
df = df.withColumn('status', F.when(df['column'] > 50, 'High')
                              .when(df['column'] > 20, 'Medium')
                              .otherwise('Low'))

# SQL CASE via expr
df = df.withColumn('status', F.expr("CASE WHEN col > 50 THEN 'High' WHEN col > 20 THEN 'Medium' ELSE 'Low' END"))

# First non-null value
df = df.withColumn('val', F.coalesce('col_a', 'col_b', F.lit('default')))

# Null-if-equal / if-null helpers
df = df.withColumn('val', F.expr("nullif(col, '')"))           # '' -> NULL
df = df.withColumn('val', F.expr("nvl(col, 'x')"))             # same idea as coalesce(col, 'x')
df = df.withColumn('val', F.expr("if(col > 0, 'pos', 'non-pos')"))

# Combine conditions
cond = (F.col('a') > 1) & (F.col('b').isNotNull()) | (F.col('c') == 'x')
df = df.withColumn('flag', F.when(cond, 1).otherwise(0))

# Greatest / least across columns
df = df.withColumn('mx', F.greatest('a', 'b', 'c')).withColumn('mn', F.least('a', 'b', 'c'))
```

---

## 7. String Functions

```python
df = df.withColumn('upper_col', F.upper('column'))
df = df.withColumn('lower_col', F.lower('column'))
df = df.withColumn('substr_col', F.substring('column', 1, 3))      # 1-indexed: (col, start, length)
df = df.withColumn('trim_col', F.trim('column'))
df = df.withColumn('concat_col', F.concat(F.col('column1'), F.col('column2')))

# More string functions
df = df.withColumn('ltrim_col', F.ltrim('column'))
df = df.withColumn('rtrim_col', F.rtrim('column'))
df = df.withColumn('initcap_col', F.initcap('column'))
df = df.withColumn('len', F.length('column'))
df = df.withColumn('concat_ws', F.concat_ws('-', 'a', 'b', 'c'))   # skips NULLs
df = df.withColumn('lpad_col', F.lpad('column', 10, '0'))          # zero-pad
df = df.withColumn('rpad_col', F.rpad('column', 10, ' '))
df = df.withColumn('rev', F.reverse('column'))
df = df.withColumn('rep', F.repeat('column', 2))
df = df.withColumn('pos', F.instr('column', 'abc'))                # 1-indexed, 0 if not found
df = df.withColumn('tr', F.translate('column', 'abc', 'xyz'))      # char mapping

# Regex
df = df.withColumn('digits', F.regexp_extract('column', r'(\d+)', 1))
df = df.withColumn('clean', F.regexp_replace('column', r'[^a-zA-Z0-9]', ''))
df = df.withColumn('parts', F.split('column', ','))                # -> array
df = df.withColumn('first_part', F.split('column', ',').getItem(0))
df = df.filter(F.col('column').rlike(r'^\d{5}$'))

# Hashing / encoding
df = df.withColumn('md5', F.md5('column'))
df = df.withColumn('sha', F.sha2('column', 256))
df = df.withColumn('row_hash', F.sha2(F.concat_ws('||', *df.columns), 256))   # change detection (CDC)
df = df.withColumn('b64', F.base64(F.col('column').cast('binary')))
df = df.withColumn('fmt', F.format_string('%s-%05d', F.col('a'), F.col('b')))
```

---

## 8. Number Functions

```python
df = df.withColumn('round_col', F.round('column', 0))          # HALF_UP
df = df.withColumn('floor_col', F.floor('column'))
df = df.withColumn('ceil_col', F.ceil('column'))
df = df.withColumn('abs_col', F.abs('column'))
df = df.withColumn('sqrt_col', F.sqrt('column'))

# More numeric functions
df = df.withColumn('bround_col', F.bround('column', 2))        # HALF_EVEN (banker's rounding)
df = df.withColumn('pow_col', F.pow('column', 2))
df = df.withColumn('exp_col', F.exp('column'))
df = df.withColumn('ln_col', F.log('column'))                  # natural log
df = df.withColumn('log10_col', F.log10('column'))
df = df.withColumn('mod_col', F.col('column') % 3)
df = df.withColumn('sign_col', F.signum('column'))
df = df.withColumn('rand_col', F.rand(seed=42))                # uniform [0, 1)
df = df.withColumn('randn_col', F.randn(seed=42))              # standard normal

# Casting
df = df.withColumn('int_col', F.col('column').cast('int'))
df = df.withColumn('dec_col', F.col('column').cast('decimal(18,2)'))
df = df.withColumn('dbl_col', F.col('column').cast('double'))
df = df.withColumn('str_col', F.col('column').cast('string'))
from pyspark.sql.types import IntegerType
df = df.withColumn('int_col', F.col('column').cast(IntegerType()))
# NOTE: a failed cast returns NULL (unless spark.sql.ansi.enabled=true, which raises an error)
```

---

## 9. Date & Time Functions

```python
# Add a column with the current date and time
df = df.withColumn('current_date', F.current_date())
df = df.withColumn('current_timestamp', F.current_timestamp())

# Convert a string to a date
df = df.withColumn('date_col', F.to_date('string_col', 'yyyy-MM-dd'))

# Convert a string to a timestamp
df = df.withColumn('ts_col', F.to_timestamp('string_col', 'yyyy-MM-dd HH:mm:ss'))

# Get number of days between two dates
df = df.withColumn('date_diff', F.datediff('end_date', 'start_date'))

# Get number of months between two dates
df = df.withColumn('months_between', F.months_between('date1', 'date2'))

# Add / subtract
df = df.withColumn('plus_7d', F.date_add('date_col', 7))
df = df.withColumn('minus_7d', F.date_sub('date_col', 7))
df = df.withColumn('plus_2m', F.add_months('date_col', 2))
df = df.withColumn('plus_1h', F.col('ts_col') + F.expr('INTERVAL 1 HOUR'))

# Extract parts
df = df.withColumn('yr', F.year('ts_col'))
df = df.withColumn('mon', F.month('ts_col'))
df = df.withColumn('day', F.dayofmonth('ts_col'))
df = df.withColumn('dow', F.dayofweek('ts_col'))               # 1 = Sunday
df = df.withColumn('woy', F.weekofyear('ts_col'))
df = df.withColumn('qtr', F.quarter('ts_col'))
df = df.withColumn('hr', F.hour('ts_col'))

# Format / truncate
df = df.withColumn('fmt', F.date_format('ts_col', 'yyyyMMdd'))
df = df.withColumn('month_start', F.trunc('date_col', 'month'))        # date -> first of month
df = df.withColumn('hour_bucket', F.date_trunc('hour', 'ts_col'))      # timestamp truncation
df = df.withColumn('month_end', F.last_day('date_col'))
df = df.withColumn('next_mon', F.next_day('date_col', 'Mon'))

# Epoch conversions
df = df.withColumn('epoch', F.unix_timestamp('ts_col'))                # ts -> seconds
df = df.withColumn('ts_from_epoch', F.from_unixtime('epoch'))          # seconds -> string
df = df.withColumn('ts_from_epoch', F.timestamp_seconds('epoch'))      # seconds -> timestamp (3.1+)

# Time zones
df = df.withColumn('utc_ts', F.to_utc_timestamp('ts_col', 'Asia/Kolkata'))
df = df.withColumn('ist_ts', F.from_utc_timestamp('utc_ts', 'Asia/Kolkata'))
spark.conf.set('spark.sql.session.timeZone', 'UTC')                    # set session TZ explicitly

# Time-bucket aggregation
df = df.groupBy(F.window('ts_col', '15 minutes')).count()
```

---

## 10. Column Operations

```python
# Add or rename a column
df = df.withColumn('new_column', F.lit('value'))
df = df.withColumnRenamed('old_name', 'new_name')

# Mathematical operations
df = df.withColumn('rounded', F.round('column', 2))
df = df.withColumn('absolute', F.abs('column'))

# Replace values in a column
df = df.replace(float('nan'), None)

# Add / rename many columns at once (avoids long withColumn chains)
df = df.withColumns({'a': F.lit(1), 'b': F.col('x') * 2})      # Spark 3.3+
df = df.withColumnsRenamed({'old1': 'new1', 'old2': 'new2'})   # Spark 3.4+
df = df.toDF(*[c.lower().replace(' ', '_') for c in df.columns])   # rename ALL columns

# Rename in bulk (works on all versions)
for c in df.columns:
    df = df.withColumnRenamed(c, c.strip().lower())

# Replace specific values
df = df.replace('N/A', None)
df = df.replace({'M': 'Male', 'F': 'Female'}, subset=['gender'])

# Reorder columns
df = df.select('id', 'name', *[c for c in df.columns if c not in ('id', 'name')])

# Apply the same function to many columns
from functools import reduce
str_cols = [c for c, t in df.dtypes if t == 'string']
df = reduce(lambda d, c: d.withColumn(c, F.trim(F.col(c))), str_cols, df)

# Chain custom transformations
def add_audit(df):
    return df.withColumn('load_ts', F.current_timestamp()).withColumn('src_file', F.input_file_name())
df = df.transform(add_audit)

# Useful metadata columns
df = df.withColumn('src_file', F.input_file_name())
df = df.withColumn('part_id', F.spark_partition_id())
df = df.withColumn('row_id', F.monotonically_increasing_id())  # unique, NOT consecutive

# Select by type
num_cols = [c for c, t in df.dtypes if t in ('int', 'bigint', 'double') or t.startswith('decimal')]
```

---

## 11. Write Data

```python
# Write DataFrame to different formats
df.write.csv('/path/to/output.csv', header=True)               # CSV
df.write.json('/path/to/output.json')                          # JSON
df.write.parquet('/path/to/output.parquet')                    # Parquet
df.write.orc('/path/to/output.orc')                            # ORC

# Save modes
df.write.mode('overwrite').parquet(path)                       # replace
df.write.mode('append').parquet(path)                          # add
df.write.mode('ignore').parquet(path)                          # skip if exists
df.write.mode('error').parquet(path)                           # default: fail if exists

# Partitioned write (folder per partition value)
df.write.mode('overwrite').partitionBy('year', 'month').parquet(path)

# Dynamic partition overwrite (only overwrite the partitions present in df)
spark.conf.set('spark.sql.sources.partitionOverwriteMode', 'dynamic')
df.write.mode('overwrite').partitionBy('dt').parquet(path)

# Compression and file size control
df.write.option('compression', 'snappy').parquet(path)         # snappy | gzip | zstd | none
df.write.option('maxRecordsPerFile', 1_000_000).parquet(path)

# Control number of output files
df.coalesce(1).write.parquet(path)                             # 1 file (use only for small data)
df.repartition(20).write.parquet(path)
df.repartition('dt').write.partitionBy('dt').parquet(path)     # ~1 file per partition value

# Write to catalog tables
df.write.mode('overwrite').saveAsTable('db.table')
df.write.mode('append').insertInto('db.table')                 # POSITION-based, not name-based!
df.write.format('parquet').partitionBy('dt').saveAsTable('db.table')

# Bucketing (pre-shuffled for repeated joins; only with saveAsTable)
df.write.bucketBy(16, 'id').sortBy('id').saveAsTable('db.bucketed_tbl')

# CSV write options
df.write.option('header', True).option('delimiter', '|').option('quote', '"').csv(path)
```

---

## 12. Schemas & Complex Types

```python
from pyspark.sql.types import (StructType, StructField, StringType, IntegerType, LongType,
                               DoubleType, DecimalType, DateType, TimestampType, BooleanType,
                               ArrayType, MapType)

# Explicit schema (faster and safer than inferSchema)
schema = StructType([
    StructField('id',         LongType(),          nullable=False),
    StructField('name',       StringType(),        True),
    StructField('amount',     DecimalType(18, 2),  True),
    StructField('created_at', TimestampType(),     True),
    StructField('tags',       ArrayType(StringType()), True),
    StructField('attrs',      MapType(StringType(), StringType()), True),
    StructField('address',    StructType([
        StructField('city', StringType(), True),
        StructField('zip',  StringType(), True),
    ]), True),
])
df = spark.read.schema(schema).json(path)

# DDL-string schema (shorter)
df = spark.read.schema('id BIGINT, name STRING, amount DECIMAL(18,2), created_at TIMESTAMP').csv(path, header=True)

# Capture bad records
schema_with_corrupt = schema.add('_corrupt_record', StringType())
df = (spark.read.schema(schema_with_corrupt)
      .option('mode', 'PERMISSIVE')
      .option('columnNameOfCorruptRecord', '_corrupt_record')
      .json(path))

# Create DataFrame manually
df = spark.createDataFrame([(1, 'a'), (2, 'b')], ['id', 'val'])
df = spark.createDataFrame([(1, 'a')], schema='id INT, val STRING')

# Schema evolution on read / write
df = spark.read.option('mergeSchema', 'true').parquet(path)    # merge differing parquet schemas
df.write.option('mergeSchema', 'true').mode('append').format('delta').save(path)   # Delta

# ---- Structs ----
df = df.withColumn('city', F.col('address.city'))              # access nested field
df = df.withColumn('address', F.struct('city', 'zip'))         # build struct
df = df.select('id', 'address.*')                              # flatten a struct

# ---- Arrays ----
df = df.withColumn('tag', F.explode('tags'))                   # 1 row per element (drops null/empty arrays)
df = df.withColumn('tag', F.explode_outer('tags'))             # keeps rows with null/empty arrays
df = df.select('*', F.posexplode('tags').alias('pos', 'tag'))  # with position
df = df.withColumn('n', F.size('tags'))
df = df.withColumn('has_x', F.array_contains('tags', 'x'))
df = df.withColumn('first_tag', F.col('tags')[0])
df = df.withColumn('first_tag', F.element_at('tags', 1))       # 1-indexed; -1 = last
df = df.withColumn('uniq', F.array_distinct('tags'))
df = df.withColumn('sorted', F.array_sort('tags'))
df = df.withColumn('joined', F.array_join('tags', ','))
df = df.withColumn('arr', F.array('a', 'b', 'c'))
df = df.withColumn('flat', F.flatten('array_of_arrays'))
df = df.withColumn('merged', F.array_union('a1', 'a2'))        # also array_intersect, array_except

# Higher-order functions (no UDF needed)
df = df.withColumn('upper_tags', F.transform('tags', lambda x: F.upper(x)))
df = df.withColumn('big_nums',   F.filter('nums', lambda x: x > 10))
df = df.withColumn('total',      F.aggregate('nums', F.lit(0), lambda acc, x: acc + x))
df = df.withColumn('any_neg',    F.exists('nums', lambda x: x < 0))

# ---- Maps ----
df = df.withColumn('m', F.create_map(F.lit('k1'), F.col('v1'), F.lit('k2'), F.col('v2')))
df = df.withColumn('v', F.col('attrs')['key'])
df = df.withColumn('keys', F.map_keys('attrs')).withColumn('vals', F.map_values('attrs'))
df = df.select('id', F.explode('attrs').alias('k', 'v'))

# ---- JSON strings ----
json_schema = 'id INT, name STRING, items ARRAY<STRUCT<sku: STRING, qty: INT>>'
df = df.withColumn('parsed', F.from_json('json_str', json_schema))   # string -> struct
df = df.withColumn('json_str', F.to_json(F.struct('id', 'name')))    # struct -> string
df = df.withColumn('name', F.get_json_object('json_str', '$.name'))  # quick path extract
df = df.select('parsed.*')

# Generic recursive flatten of structs (one level at a time)
def flatten_structs(df):
    cols = []
    for f in df.schema.fields:
        if isinstance(f.dataType, StructType):
            cols += [F.col(f'{f.name}.{c.name}').alias(f'{f.name}_{c.name}') for c in f.dataType.fields]
        else:
            cols.append(F.col(f.name))
    return df.select(cols)
```

---

## 13. Null Handling & Data Quality

```python
# Fill nulls
df = df.fillna(0)                                              # all numeric columns
df = df.fillna({'amount': 0, 'name': 'unknown'})               # per column
df = df.na.fill('NA', subset=['city'])

# Drop rows with nulls
df = df.dropna()                                               # any null in any column
df = df.dropna(how='all')                                      # all columns null
df = df.dropna(subset=['id', 'dt'])                            # nulls in key columns
df = df.dropna(thresh=3)                                       # keep rows with >= 3 non-null values

# Null / NaN checks
df = df.filter(F.col('a').isNull() | F.isnan('a'))             # isnan is for float/double NaN
df = df.withColumn('val', F.coalesce('a', 'b', F.lit(0)))

# Null count per column
df.select([F.sum(F.col(c).isNull().cast('int')).alias(c) for c in df.columns]).show()

# Distinct count / profile
df.select([F.countDistinct(c).alias(c) for c in df.columns]).show()
df.groupBy('col').count().orderBy(F.desc('count')).show()      # frequency / top values

# Duplicate key check
df.groupBy('id').count().filter('count > 1').show()

# Row count reconciliation
src_cnt, tgt_cnt = df_src.count(), df_tgt.count()
assert src_cnt == tgt_cnt, f'Count mismatch: {src_cnt} vs {tgt_cnt}'

# Diff two DataFrames
df_src.exceptAll(df_tgt).show()                                # in source, not in target
df_tgt.exceptAll(df_src).show()

# Simple validation pattern: split good vs bad
rules = (F.col('id').isNotNull()) & (F.col('amount') >= 0) & (F.col('dt').isNotNull())
good = df.filter(rules)
bad  = df.filter(~rules | rules.isNull())                      # include rows where rule evaluates to NULL
bad.withColumn('reject_reason', F.lit('failed_basic_rules')).write.mode('append').parquet(reject_path)

# Schema contract check
expected = {'id', 'name', 'amount'}
missing = expected - set(df.columns)
if missing:
    raise ValueError(f'Missing columns: {missing}')
```

---

## 14. Spark SQL

```python
# Register temp views
df.createOrReplaceTempView('orders')                           # session-scoped
df.createOrReplaceGlobalTempView('orders')                     # query as global_temp.orders

# Run SQL (returns DataFrame)
result = spark.sql("""
    WITH recent AS (
        SELECT * FROM orders WHERE order_dt >= date_sub(current_date(), 30)
    ),
    ranked AS (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_dt DESC) AS rn
        FROM recent
    )
    SELECT customer_id, order_id, amount FROM ranked WHERE rn = 1
""")

# Parameterised SQL (Spark 3.4+; avoids string formatting / injection)
spark.sql('SELECT * FROM orders WHERE amount > :min_amt', args={'min_amt': 100})

# Catalog / metadata
spark.sql('SHOW DATABASES').show()
spark.sql('SHOW TABLES IN my_db').show()
spark.sql('DESCRIBE EXTENDED my_db.my_table').show(100, False)
spark.sql('SHOW PARTITIONS my_db.my_table').show()
spark.catalog.listTables('my_db')
spark.catalog.tableExists('my_db.my_table')                    # Spark 3.3+

# Partition management (external tables)
spark.sql('MSCK REPAIR TABLE my_db.my_table')                  # discover partitions on disk
spark.sql("ALTER TABLE my_db.my_table ADD IF NOT EXISTS PARTITION (dt='2026-10-01')")

# DDL / DML
spark.sql('CREATE DATABASE IF NOT EXISTS my_db')
spark.sql('DROP TABLE IF EXISTS my_db.tmp')
spark.sql('INSERT OVERWRITE TABLE my_db.tgt PARTITION (dt) SELECT * FROM staging')

# Mix SQL expressions into DataFrame code
df = df.selectExpr('id', "CASE WHEN amount > 100 THEN 'big' ELSE 'small' END AS size")
df = df.filter(F.expr('amount > 100 AND status IN ("A","B")'))
```

---

## 15. UDFs

```python
from pyspark.sql.functions import udf, pandas_udf
from pyspark.sql.types import StringType, DoubleType
import pandas as pd

# Preference order: built-in functions > higher-order functions > pandas UDF > Python UDF

# Python UDF (row-at-a-time, slow: serialization overhead, opaque to Catalyst optimizer)
@udf(returnType=StringType())
def clean_name(s):
    return s.strip().title() if s else None
df = df.withColumn('name_clean', clean_name('name'))

# Register for use in SQL
spark.udf.register('clean_name', lambda s: s.strip().title() if s else None, StringType())

# Pandas UDF (vectorised via Arrow, much faster)
@pandas_udf('double')
def pct(s: pd.Series) -> pd.Series:
    return s / s.sum()
df = df.withColumn('pct', pct('amount'))

# Grouped map with pandas (per-group custom logic)
def normalise(pdf: pd.DataFrame) -> pd.DataFrame:
    pdf['z'] = (pdf['x'] - pdf['x'].mean()) / pdf['x'].std()
    return pdf
df = df.groupBy('k').applyInPandas(normalise, schema='k STRING, x DOUBLE, z DOUBLE')

# Enable Arrow for toPandas / createDataFrame(pandas_df)
spark.conf.set('spark.sql.execution.arrow.pyspark.enabled', 'true')

# UDF gotchas: handle None inside the function, declare the right return type,
# and keep UDFs deterministic (the optimizer may call them more than once)
```

---

## 16. Partitioning & Performance Tuning

```python
# ---- Partitions ----
df.rdd.getNumPartitions()
df = df.repartition(200)                                       # full shuffle, increase or rebalance
df = df.repartition(200, 'customer_id')                        # hash partition by key
df = df.repartitionByRange(200, 'dt')                          # range partition (sorted layout)
df = df.coalesce(10)                                           # reduce partitions WITHOUT full shuffle

# ---- Caching ----
from pyspark import StorageLevel
df.cache()                                                     # MEMORY_AND_DISK for DataFrames
df.persist(StorageLevel.MEMORY_AND_DISK)
df.persist(StorageLevel.DISK_ONLY)
df.unpersist()
df.count()                                                     # cache is lazy; an action materializes it
df = df.checkpoint()                                           # truncates lineage (needs spark.sparkContext.setCheckpointDir)
# Cache only DataFrames that are reused multiple times

# ---- Read the plan ----
df.explain('formatted')                                        # look for: Exchange (shuffle), BroadcastHashJoin,
                                                               # SortMergeJoin, PushedFilters, PartitionFilters

# ---- Key configs ----
spark.conf.set('spark.sql.shuffle.partitions', 400)            # default 200; size to data (aim ~128-200 MB/partition)
spark.conf.set('spark.sql.autoBroadcastJoinThreshold', 50 * 1024 * 1024)   # -1 disables auto broadcast
spark.conf.set('spark.sql.files.maxPartitionBytes', 128 * 1024 * 1024)     # read split size
spark.conf.set('spark.sql.parquet.compression.codec', 'snappy')

# ---- Adaptive Query Execution (Spark 3.x; on by default from 3.2) ----
spark.conf.set('spark.sql.adaptive.enabled', 'true')
spark.conf.set('spark.sql.adaptive.coalescePartitions.enabled', 'true')    # merges small shuffle partitions
spark.conf.set('spark.sql.adaptive.skewJoin.enabled', 'true')              # splits skewed partitions
spark.conf.set('spark.sql.adaptive.localShuffleReader.enabled', 'true')

# ---- Data skew ----
# Detect:
df.groupBy('join_key').count().orderBy(F.desc('count')).show(10)
df.groupBy(F.spark_partition_id()).count().show()

# Fix 1: AQE skew join (above)
# Fix 2: broadcast the small side
# Fix 3: filter / handle hot keys (e.g. NULL keys) separately, then union
# Fix 4: salting
N = 10
big   = big.withColumn('salt', (F.rand() * N).cast('int'))
small = small.withColumn('salt', F.explode(F.array([F.lit(i) for i in range(N)])))
joined = big.join(small, ['key', 'salt']).drop('salt')

# ---- Storage / layout ----
# Prefer columnar formats (Parquet/ORC) with snappy or zstd compression
# Target file size: ~128 MB - 1 GB; avoid many tiny files (small-file problem)
# Partition by low-cardinality columns used in filters (date, region); never by high-cardinality IDs
# Filter early so predicate pushdown and partition pruning kick in
# Select only needed columns (column pruning); avoid SELECT * on wide tables
# Avoid: collect() on big data, UDFs where built-ins work, repeated count() calls,
#        wide withColumn chains (use select/withColumns), unnecessary orderBy/distinct

# Compact small files
(spark.read.parquet(src)
      .repartition(50)
      .write.mode('overwrite').parquet(dst))

# ---- spark-submit / cluster sizing knobs ----
# --num-executors / --executor-cores 4-5 / --executor-memory / --driver-memory
# spark.executor.memoryOverhead (raise for PySpark / pandas UDF OOMs, "container killed by YARN")
# spark.dynamicAllocation.enabled=true
# spark.serializer=org.apache.spark.serializer.KryoSerializer
# spark.sql.broadcastTimeout, spark.network.timeout (for slow stages)
```

---

## 17. Table Formats & Upserts (Delta / Hudi / Iceberg)

Parquet files can't be updated in place. Table formats add ACID transactions, upsert / merge, deletes, time travel and schema evolution.

### Delta Lake

```python
# Session config (open-source Delta; Databricks has this built in)
# .config('spark.sql.extensions', 'io.delta.sql.DeltaSparkSessionExtension')
# .config('spark.sql.catalog.spark_catalog', 'org.apache.spark.sql.delta.catalog.DeltaCatalog')

# Write / read
df.write.format('delta').mode('overwrite').save('/path/delta_tbl')
df.write.format('delta').mode('append').partitionBy('dt').save('/path/delta_tbl')
df = spark.read.format('delta').load('/path/delta_tbl')

# MERGE (upsert)
from delta.tables import DeltaTable
tgt = DeltaTable.forPath(spark, '/path/delta_tbl')
(tgt.alias('t')
    .merge(updates.alias('s'), 't.id = s.id')
    .whenMatchedUpdateAll()                                    # or whenMatchedUpdate(set={...})
    .whenNotMatchedInsertAll()
    .execute())

# Conditional update / delete
(tgt.alias('t').merge(updates.alias('s'), 't.id = s.id')
    .whenMatchedDelete(condition="s.op = 'D'")
    .whenMatchedUpdateAll(condition="s.updated_at > t.updated_at")
    .whenNotMatchedInsertAll(condition="s.op != 'D'")
    .execute())

# Update / delete
tgt.update('status = "old"', {'status': F.lit('archived')})
tgt.delete("dt < '2025-01-01'")

# Time travel, history, maintenance
spark.read.format('delta').option('versionAsOf', 5).load(path)
spark.read.format('delta').option('timestampAsOf', '2026-10-01').load(path)
tgt.history().show()
tgt.optimize().executeCompaction()                             # compact small files
tgt.vacuum(168)                                                # remove old files (hours)
```

### Apache Hudi

```python
# Session config
# .config('spark.serializer', 'org.apache.spark.serializer.KryoSerializer')
# .config('spark.sql.extensions', 'org.apache.spark.sql.hudi.HoodieSparkSessionExtension')

hudi_options = {
    'hoodie.table.name': 'my_table',
    'hoodie.datasource.write.recordkey.field': 'id',                   # primary key
    'hoodie.datasource.write.partitionpath.field': 'dt',               # partition column ('' for none)
    'hoodie.datasource.write.precombine.field': 'updated_at',          # on duplicate keys, the largest value wins
    'hoodie.datasource.write.operation': 'upsert',                     # insert | upsert | bulk_insert | delete
    'hoodie.datasource.write.table.type': 'COPY_ON_WRITE',             # or MERGE_ON_READ
}
df.write.format('hudi').options(**hudi_options).mode('append').save('s3://bucket/hudi/my_table')

# Read snapshot
df = spark.read.format('hudi').load('s3://bucket/hudi/my_table')

# Incremental read (changes since an instant, handy for downstream layers)
inc = (spark.read.format('hudi')
       .option('hoodie.datasource.query.type', 'incremental')
       .option('hoodie.datasource.read.begin.instanttime', '20261001000000')
       .load('s3://bucket/hudi/my_table'))

# Hard-delete records
df_del.write.format('hudi').options(**{**hudi_options, 'hoodie.datasource.write.operation': 'delete'}) \
      .mode('append').save(path)
# Use COW for read-heavy tables; MOR for write-heavy / low-latency ingestion (needs compaction)
# Register in Glue / Hive with the hudi hive-sync options if you need Athena / Spectrum access
```

### Apache Iceberg

```python
# Catalog config (example: Glue catalog)
# .config('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
# .config('spark.sql.catalog.glue', 'org.apache.iceberg.spark.SparkCatalog')
# .config('spark.sql.catalog.glue.catalog-impl', 'org.apache.iceberg.aws.glue.GlueCatalog')
# .config('spark.sql.catalog.glue.warehouse', 's3://bucket/warehouse')

df.writeTo('glue.db.tbl').using('iceberg').partitionedBy(F.days('ts')).createOrReplace()
df.writeTo('glue.db.tbl').append()
df.writeTo('glue.db.tbl').overwritePartitions()

spark.sql("""
    MERGE INTO glue.db.tbl t
    USING updates s ON t.id = s.id
    WHEN MATCHED THEN UPDATE SET *
    WHEN NOT MATCHED THEN INSERT *
""")

spark.sql('SELECT * FROM glue.db.tbl.snapshots')               # metadata tables
spark.sql("SELECT * FROM glue.db.tbl TIMESTAMP AS OF '2026-10-01 00:00:00'")
spark.sql("CALL glue.system.rewrite_data_files(table => 'db.tbl')")      # compaction
spark.sql("CALL glue.system.expire_snapshots(table => 'db.tbl', older_than => TIMESTAMP '2026-09-01 00:00:00')")
```

### Quick comparison

| Feature | Delta | Hudi | Iceberg |
|---|---|---|---|
| Upsert / merge | `MERGE` | `upsert` operation (key + precombine) | `MERGE INTO` |
| Time travel | Yes | Yes (incremental / point-in-time) | Yes |
| Incremental reads | Change data feed | Native incremental query | Incremental snapshots |
| Strength | Spark / Databricks ecosystem | Record-level upserts, streaming ingestion | Engine-neutral, hidden partitioning |

---

## 18. CDC & SCD Patterns

```python
# ---- Pattern 1: Latest-record dedupe (pre-step before any upsert) ----
w = Window.partitionBy('id').orderBy(F.col('updated_at').desc(), F.col('op_seq').desc())
latest = df_changes.withColumn('rn', F.row_number().over(w)).filter('rn = 1').drop('rn')

# ---- Pattern 2: SCD Type 1 (overwrite in place, keep no history) ----
# Delta / Iceberg: MERGE ... WHEN MATCHED THEN UPDATE ... WHEN NOT MATCHED THEN INSERT
# Hudi: write with operation = 'upsert' + recordkey + precombine (see section 17)
# Plain parquet (no table format): anti-join + union, then overwrite to a NEW path
unchanged = df_target.join(latest.select('id'), 'id', 'left_anti')
new_full  = unchanged.unionByName(latest)
new_full.write.mode('overwrite').parquet(new_path)             # never overwrite the path you read from in the same job

# ---- Pattern 3: SCD Type 2 (keep history with validity windows) ----
# Target columns: id, attrs..., row_hash, eff_start, eff_end, is_current
src = (latest
       .withColumn('row_hash', F.sha2(F.concat_ws('||', 'name', 'city', 'status'), 256))
       .withColumn('eff_start', F.current_timestamp()))

# Step A: stage rows that are new OR changed vs. the current version
cur = spark.table('dim_customer').filter('is_current = true')
changed = (src.alias('s').join(cur.alias('t'), 'id', 'left')
              .filter('t.id IS NULL OR s.row_hash <> t.row_hash')
              .select('s.*'))
changed.createOrReplaceTempView('changed')

# Step B (Delta MERGE): close old rows, then insert new versions
spark.sql("""
    MERGE INTO dim_customer t
    USING changed s
    ON t.id = s.id AND t.is_current = true
    WHEN MATCHED THEN UPDATE SET t.is_current = false, t.eff_end = s.eff_start
""")
spark.sql("""
    INSERT INTO dim_customer
    SELECT id, name, city, status, row_hash, eff_start,
           CAST(NULL AS TIMESTAMP) AS eff_end, true AS is_current
    FROM changed
""")

# ---- Pattern 4: Applying CDC events (op = I/U/D) ----
# Keep latest event per key -> deletes become tombstones -> MERGE:
#   WHEN MATCHED AND s.op = 'D' THEN DELETE
#   WHEN MATCHED THEN UPDATE SET *
#   WHEN NOT MATCHED AND s.op != 'D' THEN INSERT *

# ---- Pattern 5: Incremental load with a high-water mark ----
last_wm = spark.read.parquet(wm_path).agg(F.max('wm')).first()[0]
incr = spark.read.jdbc(url, f"(SELECT * FROM src WHERE updated_at > '{last_wm}') q", properties=props)
# After a successful write, persist the new max(updated_at) as the next watermark

# ---- Medallion layers (typical naming) ----
# landing/raw  -> as-received, immutable, add audit columns (load_ts, src_file, batch_id)
# std/silver   -> typed, deduped, validated, conformed schema
# curated/gold -> business-ready aggregates and dimensional models
```

---

## 19. Structured Streaming

```python
# Read streams (file sources need an explicit schema)
raw = (spark.readStream
       .schema(schema)
       .option('maxFilesPerTrigger', 100)
       .json('s3://bucket/landing/'))

# Kafka source
raw = (spark.readStream.format('kafka')
       .option('kafka.bootstrap.servers', 'broker:9092')
       .option('subscribe', 'topic')
       .option('startingOffsets', 'latest')                    # earliest | latest | JSON per partition
       .load())
events = (raw.select(F.col('value').cast('string').alias('json'))
             .select(F.from_json('json', schema).alias('e')).select('e.*'))

# Transformations are the same as batch
agg = (events
       .withWatermark('event_time', '10 minutes')              # late-data tolerance + state cleanup
       .groupBy(F.window('event_time', '5 minutes'), 'user_id')
       .agg(F.count('*').alias('cnt')))

# Write stream
query = (agg.writeStream
         .format('parquet')                                    # delta | parquet | console | kafka
         .outputMode('append')                                 # append | update | complete
         .option('checkpointLocation', 's3://bucket/chk/job1') # REQUIRED for recovery / exactly-once
         .option('path', 's3://bucket/silver/')
         .trigger(processingTime='1 minute')
         .start())

# Triggers
# .trigger(processingTime='30 seconds')   micro-batch every 30s
# .trigger(availableNow=True)             process everything available, then stop (batch-style incremental, Spark 3.3+)
# .trigger(once=True)                     one micro-batch then stop (older)
# .trigger(continuous='1 second')         experimental low latency

# foreachBatch: reuse batch writers (MERGE, JDBC, multiple sinks)
def upsert_to_delta(batch_df, batch_id):
    batch_df.createOrReplaceTempView('updates')
    batch_df.sparkSession.sql('MERGE INTO tgt t USING updates s ON t.id = s.id '
                              'WHEN MATCHED THEN UPDATE SET * WHEN NOT MATCHED THEN INSERT *')
(events.writeStream.foreachBatch(upsert_to_delta)
       .option('checkpointLocation', chk).start())

# Stream-static join (static side is re-read per batch for some sources)
enriched = events.join(dim_df, 'product_id', 'left')

# Manage queries
query.status
query.lastProgress
query.stop()
query.awaitTermination()
spark.streams.active
# Rules: never share a checkpoint between jobs; changing aggregations / schema may require a new checkpoint
```

---

## 20. JDBC, Redshift & S3

```python
# ---- JDBC read (parallel) ----
df = (spark.read.format('jdbc')
      .option('url', 'jdbc:postgresql://host:5432/db')
      .option('dbtable', 'public.orders')                      # or a subquery: '(SELECT ... ) AS q'
      .option('user', user).option('password', pwd)            # fetch from Secrets Manager, never hard-code
      .option('driver', 'org.postgresql.Driver')
      .option('partitionColumn', 'id')                         # numeric / date column
      .option('lowerBound', 1).option('upperBound', 10_000_000)
      .option('numPartitions', 8)                              # parallel connections (be kind to the DB)
      .option('fetchsize', 10000)
      .load())

# Pushdown query
df = spark.read.jdbc(url, '(SELECT id, amt FROM orders WHERE dt = current_date) AS q', properties=props)

# ---- JDBC write ----
(df.write.format('jdbc')
   .option('url', url).option('dbtable', 'schema.tgt')
   .option('user', user).option('password', pwd)
   .option('batchsize', 10000)
   .option('numPartitions', 4)                                 # caps concurrent connections
   .option('truncate', 'true')                                 # with mode overwrite: TRUNCATE instead of DROP
   .mode('append').save())

# ---- Redshift ----
# Option A: plain JDBC (fine for small / medium data)
rs_url = 'jdbc:redshift://cluster.xxxx.region.redshift.amazonaws.com:5439/dev'
# driver: com.amazon.redshift.jdbc42.Driver

# Option B (recommended for big loads): write Parquet to S3, then COPY into Redshift
df.write.mode('overwrite').parquet('s3://bucket/stage/tbl/')
# then run: COPY schema.tbl FROM 's3://bucket/stage/tbl/' IAM_ROLE 'arn:aws:iam::...' FORMAT AS PARQUET;
# (via redshift_connector / Redshift Data API / Airflow operator)

# Option C: Redshift Spectrum / Athena read the S3 data in place (external tables over Parquet / Hudi)
# For upserts: load into a staging table, then MERGE (or DELETE + INSERT) in a single transaction

# ---- S3 ----
# s3://  (EMR / Glue), s3a:// (open-source Hadoop S3A connector), s3n:// is legacy
df = spark.read.parquet('s3://bucket/prefix/')
spark.conf.set('spark.hadoop.fs.s3a.connector.name', 's3a')    # if needed in OSS Spark
# Tips:
# - Avoid overwriting the same S3 prefix you are reading from in the same job
# - Use partition folders (dt=YYYY-MM-DD) so Athena / Spectrum can prune
# - S3 listing is slow with huge numbers of small files; compact them
# - Use IAM roles, not access keys in code
# - Commit protocols: EMRFS S3-optimized committer (EMR); S3A magic committer (OSS)
```

---

## 21. AWS Glue Specifics

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from awsglue.context import GlueContext
from awsglue.job import Job
from pyspark.context import SparkContext

# ---- Job boilerplate ----
args = getResolvedOptions(sys.argv, ['JOB_NAME', 'source_path', 'run_date'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)
# ... your logic ...
job.commit()                                                   # needed for job bookmarks to persist state

# ---- DynamicFrame <-> DataFrame ----
dyf = glueContext.create_dynamic_frame.from_catalog(
    database='my_db', table_name='my_table',
    transformation_ctx='src_ctx',                              # REQUIRED for job bookmarks
    push_down_predicate="dt >= '2026-10-01'")                  # partition pruning at read time
df  = dyf.toDF()
dyf = DynamicFrame.fromDF(df, glueContext, 'dyf_name')         # from awsglue.dynamicframe import DynamicFrame

# Read directly from S3
dyf = glueContext.create_dynamic_frame.from_options(
    connection_type='s3',
    connection_options={'paths': ['s3://bucket/in/'], 'recurse': True},
    format='json', transformation_ctx='s3_src')

# ---- Glue transforms ----
dyf = ApplyMapping.apply(frame=dyf, mappings=[('id', 'string', 'id', 'long'), ('nm', 'string', 'name', 'string')])
dyf = ResolveChoice.apply(frame=dyf, specs=[('amount', 'cast:double')])   # fix mixed-type (choice) columns
dyf = DropNullFields.apply(frame=dyf)
dyf = Filter.apply(frame=dyf, f=lambda r: r['status'] == 'A')
dyf = Relationalize.apply(frame=dyf, staging_path='s3://bucket/tmp/', name='root')   # flatten nested

# ---- Write ----
glueContext.write_dynamic_frame.from_options(
    frame=dyf, connection_type='s3',
    connection_options={'path': 's3://bucket/out/', 'partitionKeys': ['dt']},
    format='glueparquet', format_options={'compression': 'snappy'},
    transformation_ctx='sink_ctx')

# Write and update the Glue Data Catalog in one go
sink = glueContext.getSink(connection_type='s3', path='s3://bucket/out/', enableUpdateCatalog=True,
                           updateBehavior='UPDATE_IN_DATABASE', partitionKeys=['dt'])
sink.setCatalogInfo(catalogDatabase='my_db', catalogTableName='my_out')
sink.setFormat('glueparquet')
sink.writeFrame(dyf)

# ---- Glue tips ----
# Job bookmarks: need transformation_ctx on every source / sink + job.init() / job.commit()
# Plain DataFrame reads / writes work too (bookmarks only track DynamicFrame sources)
# Worker types: G.1X (4 vCPU, 16 GB), G.2X (8 vCPU, 32 GB); scale workers before rewriting code
# Use --additional-python-modules for pip packages; --datalake-formats hudi,delta,iceberg to enable table formats
# Pass job params with --key value, read them via getResolvedOptions (e.g. Airflow passing run_date)
# Glue Catalog = Hive metastore for Athena, Redshift Spectrum, EMR
```

---

## 22. Testing & Production Patterns

```python
# ---- Local test session (pytest fixture) ----
import pytest
from pyspark.sql import SparkSession

@pytest.fixture(scope='session')
def spark():
    s = (SparkSession.builder.master('local[2]').appName('tests')
         .config('spark.sql.shuffle.partitions', '2')          # small = fast tests
         .config('spark.ui.enabled', 'false')
         .getOrCreate())
    yield s
    s.stop()

# ---- Keep transforms pure: DataFrame in, DataFrame out ----
def dedupe_latest(df, key, order_col):
    w = Window.partitionBy(key).orderBy(F.col(order_col).desc())
    return df.withColumn('_rn', F.row_number().over(w)).filter('_rn = 1').drop('_rn')

def test_dedupe_latest(spark):
    src = spark.createDataFrame([(1, 'a', 1), (1, 'b', 2)], ['id', 'v', 'ts'])
    out = dedupe_latest(src, 'id', 'ts')
    assert out.collect()[0]['v'] == 'b'

# ---- DataFrame equality ----
from pyspark.testing import assertDataFrameEqual               # PySpark 3.5+
assertDataFrameEqual(actual_df, expected_df)
# Older versions: compare sorted collect() results, or use the chispa library

# ---- Idempotency (re-running a job must not duplicate data) ----
# - Overwrite a deterministic partition (dynamic partition overwrite), or
# - MERGE / upsert on a business key, or
# - Write to a run-scoped path, then atomically swap / register the partition

# ---- Config management ----
# Pass env / run params via job args (Glue getResolvedOptions, spark-submit --conf, Airflow templated args)
# Keep paths, table names, thresholds in config, not in code
# Secrets: AWS Secrets Manager / SSM Parameter Store, never in code or logs

# ---- Logging ----
import logging
logger = logging.getLogger(__name__)
logger.info('rows_in=%s rows_out=%s', rows_in, rows_out)       # avoid extra count() calls just for logs
log4j = spark._jvm.org.apache.log4j.LogManager.getLogger('my_job')   # JVM-side logger
spark.sparkContext.setLogLevel('WARN')

# ---- Error handling ----
try:
    run_pipeline(spark, args)
except Exception:
    logger.exception('Pipeline failed')
    raise                                                      # re-raise so Airflow / Glue marks the task failed

# ---- Audit columns on every load ----
df = (df.withColumn('load_ts', F.current_timestamp())
        .withColumn('batch_id', F.lit(args['run_date']))
        .withColumn('src_file', F.input_file_name()))

# ---- Airflow orchestration hints ----
# Parameterise by logical date (ds / data_interval_start) so backfills just work
# One task = one idempotent unit; retry-safe
# Use Glue / EMR / Spark submit operators with job args; prefer sensors over sleep loops
# Emit row counts / quality metrics per run to a metrics table or CloudWatch
```

### Common pitfalls

| Pitfall | Fix |
|---|---|
| Calling `collect()` / `toPandas()` on large data | Aggregate or filter first; write to storage instead |
| Too many tiny output files | `coalesce` / `repartition` before write; `maxRecordsPerFile`; compaction jobs |
| One task far slower than others | Skew: AQE skew join, salting, broadcast, isolate hot keys |
| `inferSchema` on every run | Define an explicit schema |
| Python UDF everywhere | Built-ins, higher-order functions, or pandas UDF |
| Reading and overwriting the same path | Write to a new path, or use a table format |
| `insertInto` writing to wrong columns | It's positional; align column order or use `saveAsTable` / `writeTo` |
| Lazy evaluation surprises (cache not applied, repeated recompute) | Remember transformations are lazy; an action triggers work |
| Joins on columns with NULL keys | `eqNullSafe`, or handle NULLs separately |
| Timezone drift | Set `spark.sql.session.timeZone` explicitly; store UTC |
| `withColumn` in a long loop | Use `select` / `withColumns` (huge plans slow analysis) |
| `count()` used just to test emptiness | Use `df.isEmpty()` (3.3+) or `len(df.take(1)) == 0` |

---

## 23. Ending Spark

```python
# Cache DataFrame in memory
df.cache()

# Remove DataFrame from memory
df.unpersist()

# Clear every cached table / DataFrame in the session
spark.catalog.clearCache()

# Stop SparkSession
spark.stop()
# In Glue: call job.commit() BEFORE the script ends; avoid spark.stop() before commit
```

---

## 24. Real-World Practice Scenarios

**How to use:** read the problem, try it yourself in a notebook or local Spark session, then expand the solution to compare.
Each scenario lists the cheat-sheet sections it draws on. Difficulty: 🟢 easy, 🟡 medium, 🔴 hard.

| # | Scenario | Level | Skills practiced |
|---|---|---|---|
| 1 | Collapse a CDC feed to current state | 🟢 | Windows, dedupe, delete handling (5, 18) |
| 2 | Sessionize clickstream events | 🟡 | `lag`, running sum, aggregation (5, 9) |
| 3 | Flatten nested JSON orders | 🟡 | `from_json`, `explode_outer`, structs (12) |
| 4 | Upsert customers into a Hudi table (SCD1) | 🟡 | Hudi options, precombine, idempotency (17, 18) |
| 5 | Build an SCD2 customer dimension | 🔴 | Hash diff, MERGE, staged updates (17, 18) |
| 6 | Fix a skewed join | 🔴 | Skew detection, AQE, broadcast, salting (3, 16) |
| 7 | Compact the small-files problem | 🟡 | `repartition`, `maxRecordsPerFile`, safe overwrite (11, 16) |
| 8 | Data quality gate with quarantine | 🟡 | Rule flags, reject table, thresholds (13, 22) |
| 9 | Incremental load that is safe to re-run | 🔴 | Watermarks, idempotent writes, partition design (11, 18, 22) |
| 10 | Longest login streak per user | 🟡 | Gaps-and-islands technique (5, 9) |

---

### Scenario 1: Collapse a CDC feed to current state 🟢

**Context:** A DMS task lands change records for a `customers` table. Each row has an operation (`I`/`U`/`D`), a change timestamp and a sequence number. Several records can exist per customer, and timestamps can tie.

**Sample data**

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

data = [
    (1, 'Asha',  'Pune',    'I', '2026-10-01 09:00:00', 1),
    (1, 'Asha',  'Mumbai',  'U', '2026-10-02 10:00:00', 2),
    (2, 'Ravi',  'Delhi',   'I', '2026-10-01 09:30:00', 3),
    (2, 'Ravi',  'Delhi',   'D', '2026-10-03 08:00:00', 4),
    (3, 'Meera', 'Kolkata', 'I', '2026-10-01 11:00:00', 5),
    (3, 'Meera', 'Howrah',  'U', '2026-10-01 11:00:00', 6),   # same timestamp as the insert
]
cdc = (spark.createDataFrame(data, ['id', 'name', 'city', 'op', 'updated_at', 'seq'])
            .withColumn('updated_at', F.to_timestamp('updated_at')))
```

**Task**
1. Produce the current state: one row per `id`, using the latest change.
2. Customers whose latest change is `D` must not appear in the current state, but list their ids separately so they can be deleted downstream.
3. Handle the tie for customer 3 deterministically.

**Expected:** customer 1 in Mumbai, customer 3 in Howrah, customer 2 absent from current state and present in the deleted list.

<details>
<summary>Show solution</summary>

```python
w = Window.partitionBy('id').orderBy(F.col('updated_at').desc(), F.col('seq').desc())   # seq breaks ties

latest = (cdc.withColumn('rn', F.row_number().over(w))
             .filter('rn = 1')
             .drop('rn'))

current_state = latest.filter(F.col('op') != 'D').drop('op', 'seq')
deleted_ids   = latest.filter(F.col('op') == 'D').select('id')

current_state.show()
deleted_ids.show()
```

**Why it works:** `row_number` over a descending order picks exactly one row per key even with ties, as long as the order has a final tiebreaker. Never use `rank` for dedupe, because ties would keep several rows.

</details>

---

### Scenario 2: Sessionize clickstream events 🟡

**Context:** Web events have a user and a timestamp. A new session starts when a user is idle for more than 30 minutes.

**Sample data**

```python
events = spark.createDataFrame([
    ('u1', '2026-10-01 10:00:00', 'home'),
    ('u1', '2026-10-01 10:10:00', 'search'),
    ('u1', '2026-10-01 11:00:00', 'home'),       # gap of 50 min -> new session
    ('u1', '2026-10-01 11:05:00', 'checkout'),
    ('u2', '2026-10-01 09:00:00', 'home'),
], ['user_id', 'event_ts', 'page']).withColumn('event_ts', F.to_timestamp('event_ts'))
```

**Task:** Assign a `session_id` to every event, then produce one row per session with start, end, event count and duration in minutes.

**Expected:** u1 has 2 sessions (2 events each), u2 has 1 session (1 event).

<details>
<summary>Show solution</summary>

```python
w = Window.partitionBy('user_id').orderBy('event_ts')
w_cum = w.rowsBetween(Window.unboundedPreceding, Window.currentRow)

sessions = (events
    .withColumn('prev_ts', F.lag('event_ts').over(w))
    .withColumn('new_session',
                F.when(F.col('prev_ts').isNull() |
                       ((F.col('event_ts').cast('long') - F.col('prev_ts').cast('long')) > 30 * 60), 1)
                 .otherwise(0))
    .withColumn('session_seq', F.sum('new_session').over(w_cum))          # running count of session starts
    .withColumn('session_id', F.concat_ws('-', 'user_id', 'session_seq')))

summary = (sessions.groupBy('session_id', 'user_id')
    .agg(F.min('event_ts').alias('session_start'),
         F.max('event_ts').alias('session_end'),
         F.count('*').alias('events'))
    .withColumn('duration_min',
                (F.col('session_end').cast('long') - F.col('session_start').cast('long')) / 60))
summary.orderBy('user_id', 'session_start').show(truncate=False)
```

**Why it works:** flag each row that starts a session (first event or a big gap), then a running sum of those flags gives an incrementing session number per user.

</details>

---

### Scenario 3: Flatten nested JSON orders 🟡

**Context:** An orders API dumps one JSON document per line into a single string column. Each order has a customer struct and an array of items. Some orders have no items.

**Sample data**

```python
raw = spark.createDataFrame([
    ('{"order_id":"O1","customer":{"id":"C1","country":"IN"},"items":[{"sku":"A","qty":2,"price":10.0},{"sku":"B","qty":1,"price":5.5}]}',),
    ('{"order_id":"O2","customer":{"id":"C2","country":"US"},"items":[]}',),
], ['json_str'])
```

**Task**
1. Parse with an explicit schema (no inference).
2. Produce an `order_items` table: one row per item with `order_id`, `customer_id`, `country`, `sku`, `qty`, `price`, `line_total`. Orders with no items must still appear, with null item fields.
3. Produce an `order_totals` table: one row per order with `order_total` (0 when there are no items).

<details>
<summary>Show solution</summary>

```python
schema = ('order_id STRING, customer STRUCT<id: STRING, country: STRING>, '
          'items ARRAY<STRUCT<sku: STRING, qty: INT, price: DOUBLE>>')

orders = raw.select(F.from_json('json_str', schema).alias('o')).select('o.*')

order_items = (orders
    .select('order_id',
            F.col('customer.id').alias('customer_id'),
            F.col('customer.country').alias('country'),
            F.explode_outer('items').alias('item'))               # outer keeps O2
    .select('order_id', 'customer_id', 'country',
            'item.sku', 'item.qty', 'item.price',
            (F.col('item.qty') * F.col('item.price')).alias('line_total')))

order_totals = (order_items.groupBy('order_id', 'customer_id')
    .agg(F.coalesce(F.sum('line_total'), F.lit(0.0)).alias('order_total')))

order_items.show()
order_totals.show()
```

**Why it works:** plain `explode` drops rows with empty or null arrays, so `explode_outer` is the safe default when you can't lose parents. Providing the schema avoids an extra pass over the data and makes types predictable.

</details>

---

### Scenario 4: Upsert customers into a Hudi table (SCD1) 🟡

**Context:** The raw layer contains duplicate and out-of-order updates for customers. The standardized layer should be a Hudi copy-on-write table holding the latest version per `customer_id`, with no history (SCD Type 1). The job runs daily and may be retried.

**Task**
1. Write the first batch to a Hudi table.
2. Run a second batch containing an update for an existing customer, a new customer, and an *older* duplicate of an existing customer that must not win.
3. Verify the table has exactly one row per customer with the newest data.

<details>
<summary>Show solution</summary>

```python
path = 's3://my-bucket/std/customers_hudi'

hudi_options = {
    'hoodie.table.name': 'customers',
    'hoodie.datasource.write.recordkey.field': 'customer_id',
    'hoodie.datasource.write.partitionpath.field': 'country',
    'hoodie.datasource.write.precombine.field': 'updated_at',      # largest value wins on duplicate keys
    'hoodie.datasource.write.operation': 'upsert',
    'hoodie.datasource.write.table.type': 'COPY_ON_WRITE',
}

batch1 = spark.createDataFrame([
    (1, 'Asha', 'IN', '2026-10-01 09:00:00'),
    (2, 'Ravi', 'IN', '2026-10-01 09:30:00'),
], ['customer_id', 'name', 'country', 'updated_at'])
batch1.write.format('hudi').options(**hudi_options).mode('append').save(path)

batch2 = spark.createDataFrame([
    (1, 'Asha K', 'IN', '2026-10-02 10:00:00'),   # newer update
    (1, 'Asha',   'IN', '2026-09-30 08:00:00'),   # older duplicate in the same batch, must lose
    (3, 'Meera',  'IN', '2026-10-02 11:00:00'),   # new customer
], ['customer_id', 'name', 'country', 'updated_at'])
batch2.write.format('hudi').options(**hudi_options).mode('append').save(path)

result = spark.read.format('hudi').load(path)
result.select('customer_id', 'name', 'country', 'updated_at').orderBy('customer_id').show()
assert result.count() == result.select('customer_id').distinct().count()    # one row per key
```

**Why it works:** Hudi resolves duplicates by record key, and `precombine.field` decides which version wins, both within a batch and against existing data. Re-running the same batch leaves the table unchanged, so retries are safe. Use `mode('append')` with `upsert`, since `overwrite` would replace the whole table.

**Watch out:** if the partition column value can change for a key (for example a customer changes country), the record lands in a different partition. Choose a stable partition column or use a global index.

</details>

---

### Scenario 5: Build an SCD2 customer dimension 🔴

**Context:** Analysts need history: when a customer's `city` or `name` changes, close the old row and insert a new current row. The dimension has columns `id, name, city, row_hash, eff_start, eff_end, is_current`. Today's incoming snapshot contains new, changed and unchanged customers.

**Task:** Write the logic so that, in a single MERGE:
1. Changed customers: old row gets `is_current = false` and `eff_end` set; a new current row is inserted.
2. New customers: inserted as current.
3. Unchanged customers: untouched.

<details>
<summary>Show solution (Delta Lake)</summary>

```python
from delta.tables import DeltaTable

incoming = (snapshot
    .withColumn('row_hash', F.sha2(F.concat_ws('||', 'name', 'city'), 256))
    .withColumn('eff_start', F.current_timestamp()))

dim     = DeltaTable.forName(spark, 'dim_customer')
current = spark.table('dim_customer').filter('is_current = true')

# Rows that need a NEW version: key exists but the hash differs.
# A NULL merge_key guarantees they fall into "not matched" and get inserted.
changed = (incoming.alias('s')
    .join(current.alias('t'), F.col('s.id') == F.col('t.id'))
    .filter(F.col('s.row_hash') != F.col('t.row_hash'))
    .select('s.*')
    .withColumn('merge_key', F.lit(None).cast('long')))

# Every incoming row also joins on its real key, so changed rows get expired.
staged = incoming.withColumn('merge_key', F.col('id')).unionByName(changed)

(dim.alias('t')
    .merge(staged.alias('s'), 't.id = s.merge_key AND t.is_current = true')
    .whenMatchedUpdate(condition='t.row_hash <> s.row_hash',
                       set={'is_current': 'false', 'eff_end': 's.eff_start'})
    .whenNotMatchedInsert(values={
        'id': 's.id', 'name': 's.name', 'city': 's.city', 'row_hash': 's.row_hash',
        'eff_start': 's.eff_start', 'eff_end': 'null', 'is_current': 'true'})
    .execute())

# Sanity checks
spark.sql("SELECT id, COUNT(*) FROM dim_customer WHERE is_current GROUP BY id HAVING COUNT(*) > 1").show()   # must be empty
```

**Why it works:** MERGE can only act once per source row. The staged trick duplicates changed rows, one copy with the real key (matches, so it expires the old version) and one with a NULL key (never matches, so it inserts the new version).

**Extend it:** handle deletes with a soft-delete flag, and use `eff_start` from the source change timestamp instead of `current_timestamp()` so reruns and backfills stay deterministic.

</details>

---

### Scenario 6: Fix a skewed join 🔴

**Context:** A job joining `orders` (2 billion rows) with `customers` (5 million rows) on `customer_id` runs 10x longer than expected. In the Spark UI, 199 tasks finish in a minute and one runs for an hour. About 15% of orders have `customer_id = NULL` (guest checkouts), and one reseller account owns another 10%.

**Task:** Diagnose and fix it. List at least three techniques, ordered from least to most effort.

<details>
<summary>Show solution</summary>

```python
# 1) Diagnose: confirm skew and find the hot keys
orders.groupBy('customer_id').count().orderBy(F.desc('count')).show(10)
orders.groupBy(F.spark_partition_id().alias('pid')).count().orderBy(F.desc('count')).show(5)

# 2) Cheapest: let AQE split skewed partitions (Spark 3.x)
spark.conf.set('spark.sql.adaptive.enabled', 'true')
spark.conf.set('spark.sql.adaptive.skewJoin.enabled', 'true')

# 3) NULL keys never match in an equi-join, so don't shuffle them at all
with_key = orders.filter(F.col('customer_id').isNotNull())
no_key   = orders.filter(F.col('customer_id').isNull())
joined   = with_key.join(customers, 'customer_id', 'left')
no_key_out = no_key.join(customers.limit(0), 'customer_id', 'left')    # same output schema, null customer columns
result = joined.unionByName(no_key_out)

# 4) Broadcast if the small side fits in memory (5M narrow rows often does)
from pyspark.sql.functions import broadcast
result = with_key.join(broadcast(customers.select('customer_id', 'segment', 'country')), 'customer_id', 'left')

# 5) Salting for one extreme hot key when the other side is too big to broadcast:
#    split the hot key out, salt only that part, join, then union back
N = 16
hot_id = 'RESELLER_001'

orders_hot  = with_key.filter(F.col('customer_id') == hot_id)
orders_rest = with_key.filter(F.col('customer_id') != hot_id)

cust_hot  = customers.filter(F.col('customer_id') == hot_id)
cust_rest = customers.filter(F.col('customer_id') != hot_id)

# Hot part: spread orders over N salts, replicate the single customer row N times
orders_hot_s = orders_hot.withColumn('salt', (F.rand() * N).cast('int'))
cust_hot_s   = cust_hot.withColumn('salt', F.explode(F.array([F.lit(i) for i in range(N)])))
hot_joined   = orders_hot_s.join(cust_hot_s, ['customer_id', 'salt'], 'left').drop('salt')

# Normal part: regular join
rest_joined  = orders_rest.join(cust_rest, 'customer_id', 'left')

result = hot_joined.unionByName(rest_joined)      # plus no_key_out from step 3 if you need the NULL-key rows
```

**Order to try:** AQE skew join, isolate NULLs, broadcast, then salting the hot keys. Re-check the Spark UI after each change, since the goal is even task durations, not just a faster total.

</details>

---

### Scenario 7: Compact the small-files problem 🟡

**Context:** A streaming job writes every minute, leaving 40,000 tiny Parquet files (about 2 MB each) across `dt=` partitions. Athena queries and downstream Spark jobs are slow, mostly spending time listing and opening files. The goal is files of roughly 256 MB.

**Task:** Write a compaction job that is safe (never reads and overwrites the same path in one job) and that you could run daily on the previous day's partition.

<details>
<summary>Show solution</summary>

```python
src = 's3://my-bucket/raw/events/'
dst = 's3://my-bucket/compacted/events/'
target_mb = 256

dt = '2026-10-05'
day = spark.read.parquet(f'{src}dt={dt}/')

# Estimate how many output files we need from the on-disk size of the partition
jvm = spark._jvm
fs  = jvm.org.apache.hadoop.fs.Path(f'{src}dt={dt}/').getFileSystem(spark._jsc.hadoopConfiguration())
size_bytes = fs.getContentSummary(jvm.org.apache.hadoop.fs.Path(f'{src}dt={dt}/')).getLength()
num_files = max(1, int(size_bytes / (target_mb * 1024 * 1024)))

# Write to a DIFFERENT location; overwrite only this partition
spark.conf.set('spark.sql.sources.partitionOverwriteMode', 'dynamic')
(day.withColumn('dt', F.lit(dt))
    .repartition(num_files)
    .write.mode('overwrite')
    .partitionBy('dt')
    .parquet(dst))

# Alternative when you can't compute size: cap rows per file instead
# day.write.option('maxRecordsPerFile', 2_000_000).partitionBy('dt').parquet(dst)
```

After validating row counts match between source and destination for that `dt`, repoint the table (or the partition location) to the compacted path and delete the old files.

**Why it works:** `repartition(n)` controls how many tasks, and therefore files, are written. `coalesce` is cheaper but can produce unbalanced files. With Delta, Hudi or Iceberg, prefer built-in compaction (`OPTIMIZE`, Hudi clustering/compaction, `rewrite_data_files`) over a hand-rolled job.

**Prevent it:** write less often (trigger `processingTime='10 minutes'`), or land streaming output in a table format and compact on a schedule.

</details>

---

### Scenario 8: Data quality gate with quarantine 🟡

**Context:** A daily orders file arrives from a partner and is sometimes malformed. Bad rows must never reach the curated layer, but they must not be silently dropped either. If more than 2% of rows are bad, the job should fail so someone investigates.

**Sample data**

```python
orders = spark.createDataFrame([
    ('O1', 100.0, '2026-10-01', 'PAID'),
    ('O2', -5.0,  '2026-10-01', 'PAID'),        # negative amount
    (None, 20.0,  '2026-10-01', 'NEW'),         # missing id
    ('O4', 50.0,  'not-a-date', 'NEW'),         # bad date
    ('O5', 75.0,  '2026-10-02', 'UNKNOWN'),     # bad status
    ('O6', 10.0,  '2026-10-02', 'CANCELLED'),
], ['order_id', 'amount', 'order_dt', 'status'])
```

**Task:** Apply four rules (id present, amount >= 0, parseable date, status in a known list). Write valid rows to `good`, invalid rows to `quarantine` with a column listing *which* rules failed, and raise an error if the reject rate exceeds the threshold.

<details>
<summary>Show solution</summary>

```python
from functools import reduce

rules = {
    'id_present':   F.col('order_id').isNotNull(),
    'amount_ok':    F.col('amount') >= 0,
    'date_valid':   F.to_date('order_dt', 'yyyy-MM-dd').isNotNull(),
    'status_known': F.col('status').isin('NEW', 'PAID', 'CANCELLED'),
}

checked = orders
for name, cond in rules.items():
    checked = checked.withColumn(f'ok_{name}', F.coalesce(cond, F.lit(False)))   # NULL result counts as failure

ok_cols = [F.col(f'ok_{n}') for n in rules]
checked = checked.withColumn('is_valid', reduce(lambda a, b: a & b, ok_cols)).cache()

failed_rules = F.filter(
    F.array(*[F.when(~F.col(f'ok_{n}'), F.lit(n)) for n in rules]),
    lambda x: x.isNotNull())

helper_cols = [f'ok_{n}' for n in rules] + ['is_valid']
good       = checked.filter('is_valid').drop(*helper_cols)
quarantine = (checked.filter(~F.col('is_valid'))
                     .withColumn('failed_rules', failed_rules)
                     .withColumn('rejected_at', F.current_timestamp())
                     .drop(*helper_cols))

total, bad = checked.count(), quarantine.count()
reject_rate = bad / total if total else 0.0
print(f'total={total} bad={bad} reject_rate={reject_rate:.2%}')

quarantine.write.mode('append').parquet('s3://my-bucket/quarantine/orders/')
if reject_rate > 0.02:
    raise ValueError(f'Reject rate {reject_rate:.2%} exceeds 2% threshold, failing the run')
good.write.mode('overwrite').parquet('s3://my-bucket/curated/orders/')
```

**Why it works:** wrapping each rule in `coalesce(..., False)` stops NULL comparisons from slipping through as "not false". Caching `checked` avoids recomputing the source for the counts and both writes. Writing quarantine before raising keeps evidence for debugging.

</details>

---

### Scenario 9: Incremental load that is safe to re-run 🔴

**Context:** An Airflow DAG runs daily and pulls rows from a source database where `updated_at` is later than the last successful run. Requirements: backfills and retries must not duplicate data, and a failed run must not advance the watermark.

**Task:** Design the load and write the key code. Think about: where the watermark lives, when it is updated, and how the write stays idempotent.

<details>
<summary>Show solution</summary>

```python
run_date = args['run_date']                         # e.g. '2026-10-05', passed by Airflow (logical date)
run_end  = f'{run_date} 23:59:59'

# 1) Read the last committed watermark (small table / file / DynamoDB item / Airflow Variable)
wm_df = spark.read.parquet('s3://my-bucket/control/orders_watermark/')
last_wm = wm_df.agg(F.max('wm')).first()[0] or '1970-01-01 00:00:00'

# 2) Extract a BOUNDED window (upper bound makes reruns deterministic)
query = f"""(SELECT * FROM public.orders
             WHERE updated_at > '{last_wm}' AND updated_at <= '{run_end}') AS q"""
delta = spark.read.jdbc(url, query, properties=props)

# 3) Land it in a raw layer partitioned by LOAD date, not business date
spark.conf.set('spark.sql.sources.partitionOverwriteMode', 'dynamic')   # overwrite only the partitions being written
(delta.withColumn('load_dt', F.lit(run_date))
      .write.mode('overwrite')                      # replaces only load_dt=<run_date>, other runs stay untouched
      .partitionBy('load_dt')
      .parquet('s3://my-bucket/raw/orders/'))

# 4) Only after a successful write, advance the watermark
new_wm = delta.agg(F.max('updated_at')).first()[0]
if new_wm is not None:
    (spark.createDataFrame([(str(new_wm), run_date)], ['wm', 'run_date'])
          .write.mode('append').parquet('s3://my-bucket/control/orders_watermark/'))
```

**Why it works**
- **Idempotent:** rerunning a date overwrites only `load_dt=<run_date>`, so no duplicates appear.
- **Safe on failure:** the watermark moves after the data is written. A crash in between means the next run re-extracts the same window, which step 3 absorbs.
- **Deterministic:** the upper bound prevents a retry from picking up rows that arrived after the first attempt.

**Common mistake:** partitioning the *raw* layer by business date and overwriting those partitions with only the delta. Updated rows for old dates would wipe out the other rows in those partitions. To maintain a current-state table by business key, upsert into Hudi/Delta/Iceberg instead (see scenarios 4 and 5), using the raw `load_dt` partitions as the source.

**Edge cases to consider:** rows sharing the exact watermark timestamp (use `>=` with dedupe, or a sequence column), source clock skew (subtract a small safety lag), and hard deletes (need CDC or a periodic full reconcile).

</details>

---

### Scenario 10: Longest login streak per user 🟡

**Context:** Growth wants the longest run of consecutive daily logins per user (a classic gaps-and-islands problem).

**Sample data**

```python
logins = spark.createDataFrame([
    ('u1', '2026-10-01'), ('u1', '2026-10-02'), ('u1', '2026-10-03'),
    ('u1', '2026-10-05'), ('u1', '2026-10-06'),
    ('u1', '2026-10-06'),                                   # duplicate login same day
    ('u2', '2026-10-01'), ('u2', '2026-10-03'),
], ['user_id', 'login_date']).withColumn('login_date', F.to_date('login_date'))
```

**Task:** For each user return the longest streak length plus its start and end dates.

**Expected:** u1 has a 3-day streak (10-01 to 10-03); u2 has a 1-day streak.

<details>
<summary>Show solution</summary>

```python
daily = logins.select('user_id', 'login_date').distinct()            # one row per user per day

w = Window.partitionBy('user_id').orderBy('login_date')

# Subtracting the row number from the date gives a constant for each consecutive run
islands = (daily
    .withColumn('rn', F.row_number().over(w))
    .withColumn('grp', F.expr('date_sub(login_date, rn)')))

streaks = (islands.groupBy('user_id', 'grp')
    .agg(F.min('login_date').alias('streak_start'),
         F.max('login_date').alias('streak_end'),
         F.count('*').alias('streak_days')))

w_best = Window.partitionBy('user_id').orderBy(F.desc('streak_days'), F.desc('streak_end'))
best = (streaks.withColumn('r', F.row_number().over(w_best))
               .filter('r = 1')
               .drop('r', 'grp'))
best.show()
```

**Why it works:** within a consecutive run, the date increases by 1 and `row_number` increases by 1, so `date - row_number` stays the same. A gap breaks that pattern and starts a new group. Always dedupe to one row per day first, otherwise duplicates shift the row numbers.

</details>

---

### Extra practice ideas

| Idea | What to build | Sections |
|---|---|---|
| Source-to-target reconciliation | Compare counts, sums and a row hash between two tables with a full outer join; report missing, extra and changed keys | 3, 7, 13 |
| Late-arriving streaming data | Kafka events into 5-minute windows with a 10-minute watermark; upsert results with `foreachBatch` | 19 |
| Rolling 7-day revenue per store | `rangeBetween` on a date cast to epoch seconds so gaps in days are handled correctly | 5, 9 |
| Top 3 products per category per month | `row_number` over `partitionBy(category, month)` ordered by revenue | 4, 5 |
| Redshift staging upsert | Write Parquet to S3, `COPY` into staging, then delete + insert in one transaction | 20 |
| Glue job with bookmarks | Incremental S3 ingestion using `transformation_ctx`, then verify a rerun processes nothing new | 21 |
| Unit-test a transform | Pytest fixture with a local `SparkSession` and `assertDataFrameEqual` for the dedupe function | 22 |

---

*Based on the structure of the Draphony PySpark Cheat Sheet (https://draphony.de/), extended for data engineering workloads.*
