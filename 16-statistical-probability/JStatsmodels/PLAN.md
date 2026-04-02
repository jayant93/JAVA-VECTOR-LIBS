# JStatsmodels — Migration Plan: Python Statsmodels → Java 26

## Overview
Statsmodels provides statistical modeling for Python — OLS, GLM, ARIMA, VAR, survival analysis, and hypothesis testing with results tables similar to R.

## Java 26 Equivalent
**Smile Statistics** and **Apache Commons Math**.

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

**Python (Statsmodels):**
```python
import statsmodels.api as sm
X = sm.add_constant(X)
model = sm.OLS(y, X).fit()
print(model.summary())
```

**Java 26 (JStatsmodels):**
```java
import smile.regression.*;
import smile.data.*;
OLS model = OLS.fit(formula("y ~ x1 + x2"), dataFrame);
System.out.println(model);
// Commons Math alternative
OLSMultipleLinearRegression ols = new OLSMultipleLinearRegression();
ols.newSampleData(y, X);
System.out.println(Arrays.toString(ols.estimateRegressionParameters()));
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
