# JPandas — Migration Plan: Python Pandas → Java 26

## Overview
Pandas is the go-to Python library for tabular data manipulation: DataFrames, Series,
groupby aggregations, time-series resampling, and rich I/O (CSV, Parquet, SQL, JSON).
It is used heavily in data wrangling, EDA, and feature engineering pipelines.

## Java 26 Equivalent
**Tablesaw** — a high-performance Java dataframe and visualization library that provides
a DataFrame API similar to Pandas, with column-oriented storage, filtering, sorting,
groupby, and CSV/JSON/SQL I/O built in.

## Why Java 26
- **Vector API (JEP 489):** Column scans and aggregations (sum, mean, variance) can be
  SIMD-accelerated over `DoubleColumn` backing arrays using `DoubleVector` lanes.
- **Virtual Threads (Project Loom):** Concurrent I/O — loading multiple Parquet/CSV files
  simultaneously without blocking, each on a cheap virtual thread.
- **Pattern Matching (JEP 441):** Expressive column-type dispatch when applying
  transformations across mixed-type frames.
- **Foreign Function & Memory API:** Map large Parquet files directly as off-heap
  `MemorySegment` buffers for zero-copy reads.

## GPU / TPU Support
- Tablesaw itself is CPU-only; for GPU-accelerated aggregations, bridge data to ND4J
  arrays and use the CUDA backend for batch numeric transforms before returning.
- Apache Arrow Java (`arrow-vector`) can share buffers with GPU via RAPIDS cuDF JNI.
- For TPU, export aggregated frames as TFRecords via TensorFlow Java API.

## Migration Phases

### Phase 1 – Setup
- Add Tablesaw `tablesaw-core` and `tablesaw-parquet` to Maven.
- Configure Apache Arrow dependency for Parquet I/O if needed.
- Map Pandas dtypes to Tablesaw column types (`DoubleColumn`, `StringColumn`, etc.).

### Phase 2 – Core Migration
- Replace `pd.read_csv(...)` with `Table.read().csv(...)`.
- Replace `df['col']` with `table.column("col")`.
- Port `groupby().agg()` to `table.summarize(col, AggregateFunctions.mean).by("group")`.
- Replace `df.merge(...)` with `table.joinOn("key").inner(other)`.

### Phase 3 – Optimization
- Use `Table.sortOn(...)` with parallel sort for large frames.
- Cache repeated groupby keys using `table.categorize()` for string columns.
- Batch CSV loads with virtual threads, then `Table.append()` for merge.

### Phase 4 – GPU/Vector Acceleration
- Extract `DoubleColumn` backing arrays and wrap with Java Vector API for SIMD aggregations.
- Bridge to ND4J CUDA for matrix operations on numeric subframes.
- Use off-heap `MemorySegment` for zero-copy CSV memory-mapped reads.

## API Comparison

**Python (Pandas):**
```python
import pandas as pd
df = pd.read_csv("sales.csv")
summary = df.groupby("region")["revenue"].mean()
print(summary)
```

**Java 26 (Tablesaw):**
```java
import tech.tablesaw.api.*;
import tech.tablesaw.aggregate.AggregateFunctions;

Table df = Table.read().csv("sales.csv");
Table summary = df.summarize("revenue", AggregateFunctions.mean).by("region");
System.out.println(summary.print());
```

## Performance Considerations
- Tablesaw stores columns in primitive arrays — no boxing overhead unlike row-oriented stores.
- Java Vector API allows 4–8x speedup on double column reductions (AVX-256/512).
- Virtual threads enable concurrent file I/O for multi-file loads without thread pool tuning.
- Apache Arrow interop allows zero-copy handoff to Spark or RAPIDS for large-scale work.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>tech.tablesaw</groupId>
    <artifactId>tablesaw-core</artifactId>
    <version>0.43.1</version>
</dependency>
<dependency>
    <groupId>tech.tablesaw</groupId>
    <artifactId>tablesaw-parquet</artifactId>
    <version>0.43.1</version>
</dependency>
```
