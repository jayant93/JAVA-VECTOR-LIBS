# JTsfresh — Migration Plan: Python tsfresh → Java 26

## Overview
tsfresh automatically extracts 100+ features from time series data including statistical moments, autocorrelation, Fourier coefficients, and complexity measures for ML classification.

## Java 26 Equivalent
**Smile** for time series feature extraction.

## Why Java 26
- **Virtual Threads:** Concurrent feature computation across entity groups.
- **Vector API:** SIMD-accelerate statistical aggregation primitives.
- **Structured Concurrency:** Parallel feature extraction across time series.
- **Pattern Matching:** Feature type dispatch (numerical, categorical, temporal).

## GPU / TPU Support
GPU-accelerated feature computation via RAPIDS cuML in Spark. ND4J CUDA for batch statistical operations. GPU embedding features via DJL CUDA for text/image feature generation.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace DFS auto-generation with Spark MLlib `Pipeline` stages.
- Implement custom `Transformer` for domain-specific feature derivation.
- Use virtual threads for concurrent per-entity feature computation.
- Cache computed features in Redis/PostgreSQL for online serving.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (tsfresh):**
```python
from tsfresh import extract_features
from tsfresh.utilities.dataframe_functions import impute
features = extract_features(timeseries_df, column_id='id', column_sort='time')
impute(features)
print(features.shape)
```

**Java 26 (JTsfresh):**
```java
import smile.math.*;
import smile.stat.*;
// Manual tsfresh-equivalent feature extraction
double[] ts = ...; // time series
double mean = MathEx.mean(ts);
double std = MathEx.sd(ts);
double[] autocorr = new double[20];
for (int lag = 1; lag <= 20; lag++)
    autocorr[lag-1] = Statistics.autocor(ts, lag);
double skewness = Statistics.skewness(ts);
double kurtosis = Statistics.kurtosis(ts);
double[] features = ArrayUtils.concat(new double[]{mean,std,skewness,kurtosis}, autocorr);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>com.github.haifengl</groupId>
    <artifactId>smile-core</artifactId>
    <version>3.1.1</version>
</dependency>
```
