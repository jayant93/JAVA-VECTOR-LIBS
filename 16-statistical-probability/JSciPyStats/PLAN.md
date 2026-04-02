# JSciPyStats — Migration Plan: Python SciPy.stats → Java 26

## Overview
SciPy.stats provides 100+ statistical distributions, hypothesis tests (t-test, chi-square, ANOVA), correlation measures, and goodness-of-fit tests.

## Java 26 Equivalent
**Apache Commons Math Statistics** module.

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

**Python (SciPy.stats):**
```python
from scipy import stats
t_stat, p_value = stats.ttest_ind(group_a, group_b)
print(f't={t_stat:.3f}, p={p_value:.3f}')
ks_stat, ks_p = stats.kstest(data, 'norm')
print(f'KS test: {ks_stat:.3f}, p={ks_p:.3f}')
```

**Java 26 (JSciPyStats):**
```java
import org.apache.commons.math3.stat.inference.*;
import org.apache.commons.math3.stat.*;
// t-test
double[] groupA = ..., groupB = ...;
TTest tTest = new TTest();
System.out.printf("p-value: %.3f%n", tTest.tTest(groupA, groupB));
// Kolmogorov-Smirnov
KolmogorovSmirnovTest ks = new KolmogorovSmirnovTest();
System.out.printf("KS p-value: %.3f%n", ks.kolmogorovSmirnovTest(new NormalDistribution(), data));
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
