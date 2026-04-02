# JPolars — Migration Plan: Python Polars → Java 26

## Overview
Polars is a blazing-fast DataFrame library written in Rust, designed for multi-threaded,
lazy-evaluated, columnar data processing. It outperforms Pandas significantly on large
datasets thanks to its query optimizer and Apache Arrow memory model.

## Java 26 Equivalent
**Tablesaw** for eager DataFrame operations; **Smile** for analytics and statistical
transforms. For Polars-style lazy evaluation and query planning, **Apache Spark Java API**
in local mode provides similar lazy plan optimization.

## Why Java 26
- **Vector API (JEP 489):** Replicate Polars' SIMD columnar kernels for filter, project,
  and aggregation operations on primitive arrays.
- **Virtual Threads (Project Loom):** Polars uses Rayon (Rust thread pool) for data
  parallelism; virtual threads provide analogous concurrent task execution in Java.
- **Structured Concurrency (JEP 480):** Coordinate parallel column scans and combine
  results with clear ownership semantics.
- **Pattern Matching + Sealed Classes:** Model lazy expression trees (Expr AST) that
  mirror Polars' `pl.Expr` for query plan construction.

## GPU / TPU Support
- Convert Arrow-backed frames to ND4J CUDA arrays for GPU aggregations.
- RAPIDS cuDF (via JNI) can receive Arrow buffers from Tablesaw for GPU-accelerated joins.
- Spark local mode on a GPU node uses Spark RAPIDS plugin for GPU query execution.

## Migration Phases

### Phase 1 – Setup
- Introduce Tablesaw for single-node DataFrame work and Spark Java for lazy/distributed.
- Set up Apache Arrow Java for interop between Tablesaw and Spark layers.
- Map Polars expression API concepts to Spark Column expressions.

### Phase 2 – Core Migration
- Replace `pl.read_csv(...)` with Tablesaw `Table.read().csv()` or Spark `spark.read().csv()`.
- Replace `df.lazy().filter(pl.col("x") > 5).collect()` with Spark `df.filter(col("x").gt(5))`.
- Port `groupby().agg(pl.mean("val"))` to Tablesaw summarize or Spark `groupBy().agg(mean(...))`.
- Replace `df.with_columns(...)` with `df.withColumn(...)` in Spark.

### Phase 3 – Optimization
- Use Spark's Catalyst optimizer for lazy plan optimization analogous to Polars' optimizer.
- Enable Tablesaw parallel sort with `table.sortOn(...)` backed by virtual thread executor.
- Use columnar batch reads with Arrow IPC format for cross-process data sharing.

### Phase 4 – GPU/Vector Acceleration
- Enable Spark RAPIDS GPU plugin for Catalyst plan execution on CUDA device.
- Use Java Vector API for custom Tablesaw column transforms on CPU.
- Bridge Arrow buffers to ND4J for GPU matrix ops on numeric subframes.

## API Comparison

**Python (Polars):**
```python
import polars as pl
df = pl.read_csv("data.csv")
result = df.lazy().filter(pl.col("amount") > 100).groupby("category").agg(pl.mean("amount")).collect()
print(result)
```

**Java 26 (Spark local mode):**
```java
import org.apache.spark.sql.*;
import static org.apache.spark.sql.functions.*;

SparkSession spark = SparkSession.builder().master("local[*]").appName("JPolars").getOrCreate();
Dataset<Row> df = spark.read().option("header","true").csv("data.csv");
df.filter(col("amount").gt(100)).groupBy("category").agg(mean("amount")).show();
```

## Performance Considerations
- Polars uses Apache Arrow natively; Spark also uses Arrow for shuffle optimization.
- Java Vector API closes the gap for single-node column scans vs. Polars SIMD kernels.
- Virtual threads prevent thread pool exhaustion during concurrent partition reads.
- Structured concurrency ensures clean cancellation of partial plan execution.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>tech.tablesaw</groupId>
    <artifactId>tablesaw-core</artifactId>
    <version>0.43.1</version>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.13</artifactId>
    <version>3.5.1</version>
</dependency>
```
