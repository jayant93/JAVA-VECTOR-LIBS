# JEvidently — Migration Plan: Python Evidently → Java 26

## Overview
Evidently provides ML monitoring and drift detection — generating HTML reports and dashboards for data drift, model performance, and data quality metrics.

## Java 26 Equivalent
**Apache Flink** monitoring and custom drift detection.

## Why Java 26
- **Virtual Threads:** Concurrent column validation across large DataFrames.
- **Vector API:** SIMD-accelerate statistical checks (mean, std, quantiles).
- **Structured Concurrency:** Fan-out validation rules with early termination.
- **Pattern Matching:** Validation rule type dispatch.

## GPU / TPU Support
Data quality validation is CPU-bound. GPU acceleration via ND4J CUDA for statistical distribution computation on large arrays. Drift detection can leverage GPU for KS test on large datasets.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Define validation schema using Bean Validation annotations.
- Run column-level checks with virtual threads for parallel validation.
- Export validation reports to HTML or JSON.
- Integrate into Spark pipeline as custom validator stages.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Evidently):**
```python
from evidently.report import Report
from evidently.metrics import DataDriftTable
report = Report(metrics=[DataDriftTable()])
report.run(reference_data=ref_df, current_data=cur_df)
report.save_html('drift_report.html')
```

**Java 26 (JEvidently):**
```java
import org.apache.commons.math3.stat.*;
import org.apache.flink.streaming.api.*;
// Kolmogorov-Smirnov drift detection
KolmogorovSmirnovTest ks = new KolmogorovSmirnovTest();
for (String col : columns) {
    double[] reference = getReferenceData(col);
    double[] current = getCurrentData(col);
    double pValue = ks.kolmogorovSmirnovTest(reference, current);
    if (pValue < 0.05) {
        System.out.printf("DRIFT DETECTED in column %s (p=%.4f)%n", col, pValue);
    }
}
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```
