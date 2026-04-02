# JProphet — Migration Plan: Python Prophet → Java 26

## Overview
Prophet is Facebook's time series forecasting library that handles seasonality, trends, and holidays. It decomposes time series into trend, seasonality, and holiday components using additive models.

## Java 26 Equivalent
**Smile Time Series** for forecasting in Java.

## Why Java 26
- **Virtual Threads:** Parallel MCMC chain execution.
- **Vector API:** SIMD-accelerate distribution PDF/CDF computation.
- **Structured Concurrency:** Fan-out parallel statistical test execution.
- **Pattern Matching:** Distribution type dispatch.

## GPU / TPU Support
Statistical computation is CPU-bound. GPU acceleration possible via ND4J CUDA for large matrix operations in Bayesian computation. MCMC chains parallelizable via virtual threads.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace Python stats functions with Commons Math equivalents.
- Replace pandas DataFrames with Tablesaw or double[][] arrays.
- Run multiple MCMC chains in parallel with structured concurrency.
- Export results as JSON for visualization.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Prophet):**
```python
from prophet import Prophet
df = pd.DataFrame({'ds': dates, 'y': values})
model = Prophet(yearly_seasonality=True)
model.fit(df)
future = model.make_future_dataframe(periods=365)
forecast = model.predict(future)
```

**Java 26 (JProphet):**
```java
import smile.timeseries.*;
import smile.data.*;
// ARIMA-based forecasting in Smile
double[] ts = ...; // time series values
ARIMA model = ARIMA.fit(ts, 1, 1, 1); // p=1, d=1, q=1
double[] forecast = model.forecast(365);
System.out.println(Arrays.toString(forecast));
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
